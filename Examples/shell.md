# Shell Script Examples

## HTTP health-check script

This Bash script checks a service endpoint with a bounded number of retries. It exits non-zero if the endpoint remains unhealthy, making it suitable for a simple CI/CD verification step.

Save as `check-health.sh`:

#!/bin/bash

set -Eeuo pipefail

# Validate URL
: "${URL:?Set URL, e.g. URL=https://service.example.com/health}"

# Configuration with default values
MAX_ATTEMPTS="${MAX_ATTEMPTS:-10}"
DELAY_SECONDS="${DELAY_SECONDS:-3}"
TIMEOUT="${TIMEOUT:-5}"

# Validate input values
if [[ ! "$MAX_ATTEMPTS" =~ ^[1-9][0-9]*$ ]]; then
    echo "ERROR: MAX_ATTEMPTS must be a positive integer"
    exit 2
fi

if [[ ! "$DELAY_SECONDS" =~ ^[0-9]+$ ]]; then
    echo "ERROR: DELAY_SECONDS must be a non-negative integer"
    exit 2
fi

if [[ ! "$TIMEOUT" =~ ^[1-9][0-9]*$ ]]; then
    echo "ERROR: TIMEOUT must be a positive integer"
    exit 2
fi

echo "Starting application health check..."
echo "URL: $URL"
echo "Maximum attempts: $MAX_ATTEMPTS"

# Health check with retry mechanism
for ((attempt = 1; attempt <= MAX_ATTEMPTS; attempt++)); do

    status_code=$(curl \
        --silent \
        --show-error \
        --output /dev/null \
        --write-out '%{http_code}' \
        --max-time "$TIMEOUT" \
        "$URL") || status_code="000"

    if [[ "$status_code" == "200" ]]; then
        echo "SUCCESS: Application is healthy."
        echo "Health check passed on attempt $attempt"
        exit 0
    fi

    echo "WARNING: Attempt $attempt/$MAX_ATTEMPTS failed. HTTP status: $status_code" >&2

    if (( attempt < MAX_ATTEMPTS )); then
        echo "Retrying in $DELAY_SECONDS seconds..."
        sleep "$DELAY_SECONDS"
    fi

done

echo "ERROR: Health check failed after $MAX_ATTEMPTS attempts." >&2
echo "Application is unhealthy: $URL" >&2

exit 1

- Use a health endpoint intended for readiness checks, avoid printing credentials or sensitive response bodies, and set retry limits appropriate to your deployment timeout.

## Backup Files
#!/bin/bash 
tar -czf backup_$(date +%F).tar.gz /path/to/directory

Purpose: 
- Compresses a directory into a .tar.gz backup file with the current date.

## Jenkins Job Trigger
#!/bin/bash
curl -X POST http://jenkins.local/job/your-job-name/build \
--user your-user:your-api-token

Purpose: 
- Triggers a Jenkins CI job remotely using a POST request and authentication

## System Health Check
#!/bin/bash
echo "CPU Load:"; uptime
echo -e "\nMemory Usage:"; free -m
echo -e "\nDisk Usage:"; df -h
echo -e "\nTop 5 Memory Consuming Processes:"; ps aux --sort=-%mem | head -n 6

Purpose: 
- Shows system metrics like CPU load, memory, disk, and top memory-consuming processes.

## Service Restart on Failure
#!/bin/bash
SERVICE="nginx"
if ! systemctl is-active --quiet $SERVICE; then
echo "$SERVICE is down. Restarting..."
systemctl start $SERVICE
else
echo "$SERVICE is running"
fi

Purpose: 
- Checks if nginx service is down and restarts it automatically.

## Log Rotation Script
#!/bin/bash
LOG_DIR="/var/log/myapp"
ARCHIVE_DIR="/var/log/myapp/archive"
mkdir -p $ARCHIVE_DIR
find $LOG_DIR/*.log -mtime +7 -exec mv {} $ARCHIVE_DIR \;
gzip $ARCHIVE_DIR/*.log

Purpose: 
- Moves logs older than 7 days to an archive and compresses them.

## Clean Up Old Log Files
Script:
#!/bin/bash
# Define the log directory and number of days to retain logs
LOG_DIR="/var/log/myapp"
DAYS_TO_KEEP=30
# Find and delete log files older than the specified number of days
find $LOG_DIR -type f -name "*.log" -mtime +$DAYS_TO_KEEP -exec rm -f {} \;

# Find and delete log files older than the specified number of days + Specified size once matched
find $LOG_DIR -type f -name "*.log" -mtime +"$DAYS_TO_KEEP" -size +500M -exec rm -f {} \;

# Similer command to delete log files older than the specified number of days + Specified size once matched. It will print and delete the files.
find $LOG_DIR -type f -name "*.log" -mtime +"$DAYS_TO_KEEP" -size +500M -print -delete

# Print a success message
echo "Old log files deleted."

Explanation:
- find $LOG_DIR -type f -name "*.log" -mtime +$DAYS_TO_KEEP: Finds log files older than the specified number of days.
- -exec rm -f {}: Deletes each found file.
- -size +500M	Larger than 500 MB
- -print	Display files being deleted
- -delete	Permanently delete matching files
- New Learner Insight: Log files can accumulate and take up disk space. This script helps clean up old log files by deleting those that are older than a specified number of days.

## Check Port Availability
#!/bin/bash
PORT=8080
if lsof -i:$PORT > /dev/null; then
echo "Port $PORT is in use."
else
echo "Port $PORT is free."
fi

Purpose: 
- Checks if a specific port (like 8080) is being used by any process.

## Monitor CPU Usage and Send Alert
#!/bin/bash
THRESHOLD=80
CPU_LOAD=$(top -bn1 | grep "Cpu(s)" | awk '{print $2 + $4}')
if (( $(echo "$CPU_LOAD > $THRESHOLD" | bc -l) )); then
echo "High CPU Load: $CPU_LOAD%" | mail -s "Alert: CPU Load" admin@example.com
fi

Purpose: 
- Monitors the CPU usage and sends an email alert if it exceeds the threshold (80% in this case).
- Use: Ideal for alerting system administrators about high CPU usage.

## Monitor Memory Usage and Trigger Alert
Script:
#!/bin/bash
THRESHOLD=90
MEMORY_USAGE=$(free | grep Mem | awk '{print $3/$2 * 100.0}')
if (( $(echo "$MEMORY_USAGE > $THRESHOLD" | bc -l) )); then
echo "Memory usage is high: $MEMORY_USAGE%" | mail -s "Alert: High Memory Usage" admin@example.com
fi

Explanation:
- free: Displays memory usage statistics.
- awk '{print $3/$2 * 100.0}': Calculates the percentage of used memory.
- bc -l: This command-line calculator is used for floating-point comparison.
- New Learner Insight: This script monitors memory usage on a system and sends an email alert if usage exceeds a specified threshold.

## Check Disk Usage and Send Alert
Script:
#!/bin/bash
THRESHOLD=85
DISK_USAGE=$(df / | grep / | awk '{ print $5 }' | sed 's/%//g')
if [ $DISK_USAGE -gt $THRESHOLD ]; then
echo "Disk usage is over threshold: $DISK_USAGE%" | mail -s "Disk Usage Alert" admin@example.com
fi

Explanation:
- df /: Displays disk usage of the root file system.
- awk '{ print $5 }': Extracts the percentage of disk space used.
- sed 's/%//g': Removes the percentage sign.
- New Learner Insight: This script monitors disk usage and sends an email alert if usage exceeds a specified threshold.

## Restart All Failed Systemd Services
#!/bin/bash
for SERVICE in $(systemctl --failed --no-legend | awk '{print $1}'); do
systemctl restart $SERVICE
done

Purpose: 
- Restarts all failed systemd services.
- Use: Ensures that any failed services on a Linux system are automatically restarted.

## SSH Key Generator Script
Script:
#!/bin/bash
echo "Generating SSH key for GitHub..."
ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f ~/.ssh/id_rsa -N ""
# Show the public key
cat ~/.ssh/id_rsa.pub

Explanation:
- ssh-keygen: This generates a new SSH key pair (public and private keys).
- -t rsa -b 4096: This specifies the RSA algorithm with 4096 bits of encryption strength.
- -f ~/.ssh/id_rsa: This saves the private key in the specified file path.
- New Learner Insight: SSH keys are used for secure communication, such as connecting to servers or GitHub repositories without needing a password.

## Generate a Self-Signed SSL Certificate
Script:
#!/bin/bash
DOMAIN="example.com"
CERT_DIR="/etc/ssl/certs"
KEY_DIR="/etc/ssl/private"
openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout $KEY_DIR/$DOMAIN.key -out $CERT_DIR/$DOMAIN.crt

Explanation:
- openssl req -x509: This generates a self-signed SSL certificate.
- -newkey rsa:2048: This creates a new RSA key with 2048 bits.
- -keyout and -out: These specify the file paths to save the private key and certificate.
- New Learner Insight: SSL certificates are essential for securing communication over the internet. This script helps generate a self-signed certificate for testing purposes.

## Check SSL Certificate Expiry and Send Alert
Script:
#!/bin/bash
# Define the domain and alert email
DOMAIN="example.com"
ALERT_EMAIL="admin@example.com"
# Check SSL certificate expiration date
EXPIRY_DATE=$(echo | openssl s_client -connect $DOMAIN:443 -servername $DOMAIN 2>/dev/null | openssl x509 -noout -dates | grep 'notAfter' | sed 's/notAfter=//')
# Convert the expiry date to seconds since epoch
EXPIRY_EPOCH=$(date -d "$EXPIRY_DATE" +%s)
CURRENT_EPOCH=$(date +%s)
# Calculate the difference in days
DAYS_LEFT=$(( ($EXPIRY_EPOCH - $CURRENT_EPOCH) / 86400 ))
# Check if the certificate expires within 30 days
if [ $DAYS_LEFT -le 30 ]; then
echo "SSL certificate for $DOMAIN will expire in $DAYS_LEFT days. Please renew it." | mail -s "SSL Certificate Expiry Warning" $ALERT_EMAIL
echo "Alert sent about SSL certificate expiration."
else
echo "SSL certificate for $DOMAIN is valid for $DAYS_LEFT more days."
fi

Explanation:
- openssl s_client: Connects to the server and fetches the SSL certificate.
- openssl x509 -noout -dates: Extracts certificate expiration information.
- date -d "$EXPIRY_DATE" +%s: Converts the expiration date to seconds since epoch for comparison.
- New Learner Insight: This script automates SSL certificate expiry checks and alerts the administrator if renewal is needed, which is crucial for maintaining secure communication.
