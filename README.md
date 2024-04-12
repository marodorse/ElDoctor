# El Doctor - A monitoring tool to rule them all!

- Module: **Scripting** 
- Competence: `is able to write a custom script to monitor machines`
- Type of Challenge: `Consolidation`
- Duration: `3 day`
- Deadline: `12/04/2024`
- Participants: : `solo`

## The mission

There exist a plethora of amazing monitoring tools out there, some of which go as far as offering a full blown graphical dashboard collecting metrics on your entire system in a single unified interface, isn't it great?! Well, this challenge will have you throw all those pre-made solution out the window to **create your own** monitoring script!

You will have multiple days to make it as **useful** (_collect the data you want or need_) and **fancy** (_interactive interface_, _features_, _..._) as possible, the goal is for you to **be creative** and **make it your own**! As such, we won't give you clear instructions to follow nor specific features to implement. Still, we are no monster so here are some idea to inspire you:

- Make an interactive [curses interface](https://en.wikipedia.org/wiki/Curses_(programming_library)) (_or similar_) for your script.
- Deploy your script on a machine you manage and use something like [cron](https://en.wikipedia.org/wiki/Cron) to execute once an hour.
- Collect metrics every hour and store them in an [CSV file](https://en.wikipedia.org/wiki/Comma-separated_values).
- If the state of the machine is critical (_not enough ram_, _..._), notify yourself by mail.
- Send yourself a system report once a week.

In order to validate this challenge, you must have a repository containing your script and its documentation along with your script. 

----------------------------------------------------------------
## raw script without interactivity

### For this script, the services monitored are the services installed on the ubuntu server 22.04 of localhost darkosrvr.

```
#!/bin/bash

# Log file path
LOG_FILE="/var/log/monitoringProject.log"


# Function to log messages to the log file
log_message() {
    local log_level="$1"
    local message="$2"
    echo "$(date +"%Y-%m-%d %H:%M:%S") [$log_level] - $message" >> "$LOG_FILE"
}

# Function to check service status
check_service() {
    local service="$1"
    if sudo systemctl is-active --quiet "$service"; then
        echo "$service: Running"
        log_message "INFO" "$service is running"
    else
        echo "$service: Not Running"
        log_message "ERROR" "$service is not running"
    fi
}

# Function to check internet connectivity
check_internet() {
    ping -c 1 google.com &> /dev/null && echo "Status: Connected" || echo "Status: Disconnected"
}

# Function to check disk usage
check_disk_usage() {
    df -h
}

# Function to check memory usage
check_memory_usage() {
    free -m
}

# Function to check CPU usage
check_cpu_usage() {
    top -bn1 | grep "Cpu(s)"
}

# Function to check load average
check_load_average() {
    uptime
}

# Function to check failed login attempts
check_failed_logins() {
    # Check if the file is empty
    if [ ! -s /var/log/btmp ]; then
        echo "File empty so far"
        return
    fi

    # Process the file if it's not empty
    grep "Failed password" /var/log/btmp | awk '{print $9}' | sort | uniq -c | sort -nr
}

# Main function
main() {
    # Create log file if it doesn't exist
    touch "$LOG_FILE" || { echo "Error: Unable to create log file $LOG_FILE"; exit 1; }

    # System Information
    echo "## System Information"
    echo "Hostname: $(hostname)"
    echo "Operating System: $(lsb_release -d | cut -f 2-)"
    echo "Kernel Version: $(uname -r)"
    echo ""

    # Service Status
    echo "## Service Status"
    services=("apache2" "ufw" "bind9" "mariadb" "ssh")
    for service in "${services[@]}"; do
        check_service "$service"
    done
    echo ""

    # Network Connectivity
    echo "## Network Connectivity"
    check_internet
    echo ""

    # Resource Utilization
    echo "## Resource Utilization"
    echo "Disk Usage:"
    check_disk_usage
    echo ""
    echo "Memory Usage:"
    check_memory_usage
    echo ""
    echo "CPU Usage:"
    check_cpu_usage
    echo ""

    # Performance Metrics
    echo "## Performance Metrics"
    echo "Load Average:"
    check_load_average
    echo ""

    # Security
    echo "## Security"
    echo "Failed Login Attempts:"
    check_failed_logins
    echo ""

    # Log script execution completion
    log_message "INFO" "Script execution completed successfully"
}

# Run main function
main

# If script execution fails, log error message and exit with non-zero status
if [ $? -ne 0 ]; then
    log_message "ERROR" "Script execution failed"
    exit 1
fi
```	