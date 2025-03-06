Provide your solution here:

# Storage Issue Troubleshooting Guide for NGINX Load Balancer VM

## Initial Assessment

1. **Quick System Overview**
```bash
df -h                  # Check disk usage by filesystem
du -h --max-depth=1 / # Identify large directories
lsof | grep deleted   # Check for held deleted files
ps aux --sort=-%mem   # Check running processes
```

## Common Root Causes and Solutions

### 1. Log File Accumulation

**Cause:**
- NGINX access and error logs growing without rotation
- Failed log rotation
- Aggressive logging settings

**Impact:**
- Disk space exhaustion
- Potential service disruption
- Reduced performance due to large log files

**Investigation Steps:**
```bash
# Check log sizes
du -h /var/log/nginx/
ls -lh /var/log/nginx/

# Check log rotation config
cat /etc/logrotate.d/nginx

# Check current NGINX logging settings
cat /etc/nginx/nginx.conf | grep log
```

**Recovery Steps:**
1. Immediate relief:
   ```bash
   # Safely truncate large log files
   truncate -s 0 /var/log/nginx/access.log
   truncate -s 0 /var/log/nginx/error.log
   
   # Rotate logs manually
   logrotate -f /etc/logrotate.d/nginx
   ```

2. Long-term fix:
   - Configure proper log rotation:
   ```nginx
   /var/log/nginx/*.log {
       daily
       rotate 7
       compress
       delaycompress
       missingok
       notifempty
       create 0640 nginx nginx
       sharedscripts
       postrotate
           nginx -s reload
       endscript
   }
   ```
   - Implement log shipping to centralized logging system

### 2. Core Dumps

**Cause:**
- System or application crashes generating core dumps
- Unconfigured core dump cleanup

**Impact:**
- Large core dump files consuming disk space
- No impact on service unless disk fills up

**Investigation Steps:**
```bash
# Find core dumps
find / -name "core.*" -type f

# Check core dump configuration
cat /proc/sys/kernel/core_pattern
ulimit -c
```

**Recovery Steps:**
1. Immediate relief:
   ```bash
   # Remove old core dumps
   find / -name "core.*" -type f -delete
   ```

2. Long-term fix:
   - Configure core dump limits:
   ```bash
   # Add to /etc/security/limits.conf
   * soft core 0
   * hard core 0
   ```

### 3. Temporary Files Accumulation

**Cause:**
- Uncleared temporary files in /tmp
- Failed cleanup processes
- Application leaving temporary files

**Impact:**
- Gradual disk space consumption
- Potential performance impact

**Investigation Steps:**
```bash
# Check tmp directory usage
du -sh /tmp/*
find /tmp -type f -atime +7
```

**Recovery Steps:**
1. Immediate relief:
   ```bash
   # Remove old temp files
   find /tmp -type f -atime +7 -delete
   ```

2. Long-term fix:
   - Configure automatic tmp cleanup:
   ```bash
   # Add to /etc/cron.daily/tmp-cleanup
   #!/bin/bash
   find /tmp -type f -atime +7 -delete
   ```

### 4. NGINX Caching Issues

**Cause:**
- Excessive proxy cache size
- Improper cache cleanup
- Cache configuration issues

**Impact:**
- High disk usage from cached responses
- Potential service slowdown

**Investigation Steps:**
```bash
# Check NGINX cache directory size
du -sh /var/cache/nginx/

# Review cache configuration
cat /etc/nginx/nginx.conf | grep proxy_cache
```

**Recovery Steps:**
1. Immediate relief:
   ```bash
   # Clear NGINX cache
   rm -rf /var/cache/nginx/*
   nginx -s reload
   ```

2. Long-term fix:
   - Optimize cache configuration:
   ```nginx
   proxy_cache_path /var/cache/nginx levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m use_temp_path=off;
   ```

## Preventive Measures

1. **Monitoring Setup**
   ```bash
   # Add to /etc/cron.hourly/disk-check
   #!/bin/bash
   THRESHOLD=90
   CURRENT=$(df / | grep / | awk '{ print $5}' | sed 's/%//g')
   if [ $CURRENT -gt $THRESHOLD ]; then
       echo "Disk usage above ${THRESHOLD}%" | mail -s "High Disk Usage Alert" admin@example.com
   fi
   ```

2. **Regular Maintenance**
   - Schedule weekly cleanup jobs
   - Implement automated log shipping
   - Configure alerts at 80% disk usage

3. **Documentation**
   - Maintain runbook for storage issues
   - Document log rotation policies
   - Keep recovery procedures updated

## Long-term Recommendations

1. **Infrastructure Improvements**
   - Consider separate partition for logs
   - Implement centralized logging (ELK Stack)
   - Set up automated monitoring and alerting

2. **Policy Changes**
   - Implement log retention policies
   - Regular system maintenance schedule
   - Disaster recovery procedures

3. **Capacity Planning**
   - Regular storage usage review
   - Growth trend analysis
   - Upgrade planning based on trends
