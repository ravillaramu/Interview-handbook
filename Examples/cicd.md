# End-to-End CI/CD Pipeline Examples

These examples test an application, build a container image, publish it from the protected `main` branch, and deploy it to Kubernetes. They assume:

- The repository has a Node.js application with `package-lock.json`, `npm test`, and a Dockerfile at its root.
- Kubernetes has a Deployment named `sample-app` with a container named `app` in namespace `production`.
- Save the Kubernetes manifest from [kubernetes.md](./kubernetes.md) in the application repository as `k8s/deployment.yaml`, change its namespace to `production`, and bootstrap the resources before using Jenkins or GitHub Actions. The Azure DevOps example applies the manifest as part of its deployment.
- The target cluster can pull images from the selected registry.
- Production credentials are stored in the CI system's protected credential store, not in the repository.

Replace the sample names, registry, cluster connection, and test commands to match your project. In production, pin third-party actions and plugins to reviewed versions and use short-lived identity federation where supported.

## Jenkins — `Jenkinsfile`

Requires a Jenkins agent with Node.js, Docker CLI/daemon, `kubectl`, and the Docker Pipeline and Credentials Binding plugins. Create Jenkins credentials named `registry-creds` (username/password) and `kubeconfig-prod` (secret file). Restrict production credentials to the protected job/folder.

```groovy
pipeline {
  agent any

  options {
    timestamps()
    disableConcurrentBuilds()
    buildDiscarder(logRotator(numToKeepStr: '20'))
  }

  environment {
    REGISTRY = 'registry.example.com'
    IMAGE_NAME = 'team/sample-app'
    IMAGE = ''
    K8S_NAMESPACE = 'production'
    DEPLOYMENT_NAME = 'sample-app'
    CONTAINER_NAME = 'app'
  }

  stages {
    stage('Test') {
      steps {
        sh 'npm ci'
        sh 'npm test'
      }
    }

    stage('Build and publish') {
      when {
        branch 'main'
      }
      steps {
        script {
          def commit = sh(
            script: 'git rev-parse --short=12 HEAD',
            returnStdout: true
          ).trim()
          env.IMAGE = "${env.REGISTRY}/${env.IMAGE_NAME}:${commit}"

          def image = docker.build(env.IMAGE)
          docker.withRegistry("https://${env.REGISTRY}", 'registry-creds') {
            image.push()
          }
        }
      }
    }

    stage('Deploy production') {
      when {
        branch 'main'
      }
      steps {
        input message: 'Approve deployment to production?'
        withCredentials([
          file(credentialsId: 'kubeconfig-prod', variable: 'KUBECONFIG')
        ]) {
          sh '''
            set -eu
            kubectl set image \
              "deployment/${DEPLOYMENT_NAME}" \
              "${CONTAINER_NAME}=${IMAGE}" \
              --namespace "${K8S_NAMESPACE}"
            kubectl rollout status \
              "deployment/${DEPLOYMENT_NAME}" \
              --namespace "${K8S_NAMESPACE}" \
              --timeout=180s
          '''
        }
      }
    }
  }

  post {
    always {
      echo 'Pipeline finished. Review stage logs and deployment health.'
    }
  }
}
```

Use a Multibranch Pipeline so `when { branch 'main' }` is evaluated from the branch name. The `input` step is a basic approval example; configure Jenkins authorization and an auditable production approval policy. A restricted, short-lived Kubernetes identity is preferable to a long-lived kubeconfig.

## GitHub Actions — `.github/workflows/ci-cd.yml`

Create a protected GitHub environment named `production`, configure required reviewers if needed, and store a restricted kubeconfig as the `KUBE_CONFIG` environment secret. Grant that identity only the permissions needed to update and observe this Deployment. For cloud-managed Kubernetes, prefer OIDC-based short-lived credentials rather than a long-lived kubeconfig.

```yaml
name: CI and deploy

on:
  pull_request:
    branches: ["main"]
  push:
    branches: ["main"]
  workflow_dispatch:

permissions:
  contents: read

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - name: Check out source
        uses: actions/checkout@v4
      - name: Set up Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "22"
          cache: npm
      - name: Install dependencies
        run: npm ci
      - name: Run tests
        run: npm test

  publish:
    needs: test
    if: github.ref == 'refs/heads/main' && (github.event_name == 'push' || github.event_name == 'workflow_dispatch')
    runs-on: ubuntu-latest
    permissions:
      contents: read
      packages: write
    outputs:
      digest: ${{ steps.build.outputs.digest }}
    env:
      IMAGE: ghcr.io/${{ github.repository }}
    steps:
      - name: Check out source
        uses: actions/checkout@v4
      - name: Log in to GitHub Container Registry
        uses: docker/login-action@v3
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3
      - name: Build and publish immutable commit tag
        id: build
        uses: docker/build-push-action@v6
        with:
          context: .
          push: true
          tags: ${{ env.IMAGE }}:sha-${{ github.sha }}

  deploy:
    needs: publish
    if: github.ref == 'refs/heads/main' && (github.event_name == 'push' || github.event_name == 'workflow_dispatch')
    runs-on: ubuntu-latest
    environment: production
    permissions:
      contents: read
    env:
      KUBE_CONFIG: ${{ secrets.KUBE_CONFIG }}
      IMAGE: ghcr.io/${{ github.repository }}@${{ needs.publish.outputs.digest }}
    steps:
      - name: Check out deployment manifests
        uses: actions/checkout@v4
      - name: Set up kubectl
        uses: azure/setup-kubectl@v4
      - name: Configure cluster access
        shell: bash
        run: |
          set -euo pipefail
          install -d -m 700 "$HOME/.kube"
          printf '%s' "$KUBE_CONFIG" > "$HOME/.kube/config"
          chmod 600 "$HOME/.kube/config"
      - name: Deploy and verify rollout
        run: |
          set -euo pipefail
          kubectl set image deployment/sample-app \
            "app=${IMAGE}" \
            --namespace production
          kubectl rollout status deployment/sample-app \
            --namespace production \
            --timeout=180s
```

The deployment uses the digest produced by the publish job so it deploys exactly the artifact that was built. Protect `main` and the `production` environment; do not expose production secrets to pull-request workflows. In high-assurance environments, pin actions to verified commit SHAs and authenticate to the cluster through your cloud provider's OIDC federation.

## Azure DevOps — `azure-pipelines.yml`

Create an Azure Container Registry service connection named `sample-acr` and a Kubernetes service connection named `production-k8s`. Authorize both connections only for this pipeline. Configure branch protection and environment approvals for production.

```yaml
trigger:
  branches:
    include:
      - main

pr:
  branches:
    include:
      - main

variables:
  imageRepository: sample-app
  registryName: sample.azurecr.io
  dockerfilePath: $(Build.SourcesDirectory)/Dockerfile
  imageTag: $(Build.BuildId)

pool:
  vmImage: ubuntu-latest

stages:
  - stage: Test
    displayName: Test
    jobs:
      - job: TestApplication
        steps:
          - task: UseNode@1
            inputs:
              version: "22.x"
          - script: npm ci
            displayName: Install dependencies
          - script: npm test
            displayName: Run tests

  - stage: BuildAndDeploy
    displayName: Build, publish, and deploy
    dependsOn: Test
    condition: and(succeeded(), eq(variables['Build.SourceBranch'], 'refs/heads/main'))
    jobs:
      - deployment: DeployProduction
        displayName: Deploy to production
        environment: production
        strategy:
          runOnce:
            deploy:
              steps:
                - checkout: self

                - task: Docker@2
                  displayName: Build and push image
                  inputs:
                    command: buildAndPush
                    containerRegistry: sample-acr
                    repository: $(imageRepository)
                    dockerfile: $(dockerfilePath)
                    buildContext: $(Build.SourcesDirectory)
                    tags: |
                      $(imageTag)

                - task: KubernetesManifest@1
                  displayName: Deploy and verify Kubernetes rollout
                  inputs:
                    action: deploy
                    kubernetesServiceConnection: production-k8s
                    namespace: production
                    manifests: |
                      $(Build.SourcesDirectory)/k8s/deployment.yaml
                    containers: |
                      $(registryName)/$(imageRepository):$(imageTag)
```

The deployment manifest should name its container `app` and can contain a placeholder image; `KubernetesManifest@1` substitutes the image supplied through `containers`. Ensure the cluster can pull from ACR (for example, through a managed identity or image-pull secret), and confirm the pipeline's service connections are authorized and protected.