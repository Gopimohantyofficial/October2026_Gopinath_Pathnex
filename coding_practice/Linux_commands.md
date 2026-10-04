

# **Linux Commands for DevOps — Master Table**










Production Alert
     ↓
Is server alive?
     ↓
CPU / Memory / Disk?
     ↓
Process running?
     ↓
Port listening?
     ↓
Network/DNS working?
     ↓
Application logs?
     ↓
Permissions/files?
     ↓
Service/config issue?




              PRODUCTION ISSUE
                     │
                     ▼
              Is server alive?
                  uptime
                     │
        ┌────────────┼─────────────┐
        ▼            ▼             ▼
       CPU         Memory         Disk
       top         free -h        df -h
        │            │             │
        └────────────┼─────────────┘
                     ▼
                  Process
                  ps aux
                     │
                     ▼
                  Service
               systemctl
                     │
                     ▼
                   Logs
                journalctl
                tail / grep
                     │
                     ▼
                   Port
                 ss -lntp
                     │
                     ▼
                Local request
                    curl
                     │
                     ▼
                DNS / Network
              dig / nc / ping
                     │
                     ▼
                Dependency
            DB / Redis / API etc.




  ==========================================================



Server
  ↓
hostname / uptime
  ↓
CPU
top
  ↓
Memory
free -h
  ↓
Disk
df -h
  ↓
Process
ps aux
  ↓
Service
systemctl
  ↓
Logs
journalctl / tail / grep
  ↓
Port
ss -lntp
  ↓
Application
curl
  ↓
DNS
dig / nslookup
  ↓
Dependency
nc -zv


-----------------------------------------------------------------

hostname
uptime

top
free -h
df -h
du -sh

ps aux
pgrep -af
kill

systemctl status
journalctl -u

tail -f
grep
find

ip addr
ip route
ss -lntp

curl -v
dig
nslookup
nc -zv

ls -lah
chmod
chown

ssh
scp

docker ps
docker logs
docker inspect
docker exec

kubectl get
kubectl describe
kubectl logs
kubectl exec
# Linux Commands for DevOps — Master Table

| Category | Main Purpose | Typical Production Problem |
|---|---|---|
| 1. System Information | Identify server and environment | Wrong server, unknown OS, high load |
| 2. CPU | Find CPU bottlenecks | CPU alert > 90% |
| 3. Memory | Check RAM/swap pressure | App killed, OOM, slowness |
| 4. Disk | Find disk usage issues | `No space left on device` |
| 5. Files & Directories | Manage files safely | Config backup, file location |
| 6. Logs & File Reading | Inspect application/system logs | 500/502 errors, failures |
| 7. Processes | Inspect and control processes | Hung or high-resource app |
| 8. Services | Manage systemd services | nginx/backend stopped |
| 9. Networking | Test ports, IPs and routes | Connection refused, timeout |
| 10. DNS | Troubleshoot name resolution | Hostname not resolving |
| 11. Port Connectivity | Test specific service ports | DB/Redis unreachable |
| 12. Permissions | Fix access/ownership issues | Permission denied |
| 13. SSH | Access remote servers | Production VM troubleshooting |
| 14. Text Processing | Analyze logs/data | Traffic spike analysis |
| 15. Archive | Compress and extract files | Collect incident logs |
| 16. Environment Variables | Inspect runtime config | Wrong DB/API endpoint |
| 17. Docker | Troubleshoot containers | Container unhealthy/crashed |
| 18. Kubernetes | Troubleshoot pods/services | CrashLoopBackOff, DNS, service issue |

---

# 1. System Information

| Command | Usage | Example | Real Production Example |
|---|---|---|---|
| `hostname` | Show server hostname | `hostname` → `prod-api-01` | Confirm you are troubleshooting the correct production server |
| `hostname -I` | Show server IP addresses | `hostname -I` | Confirm server/private IP |
| `uname -a` | Kernel/system information | `uname -a` | Check kernel compatibility issue |
| `cat /etc/os-release` | Linux OS/version | `cat /etc/os-release` | Determine Ubuntu vs AlmaLinux before package install |
| `uptime` | Uptime + load average | `uptime` | Server is slow; check load |
| `whoami` | Current user | `whoami` | Confirm whether you are root or normal user |
| `id` | User/group information | `id appuser` | Troubleshoot permission problems |
| `date` | Current server date/time | `date` | Compare server time with application logs |
| `timedatectl` | Timezone/NTP info | `timedatectl` | Logs across servers have different timestamps |

---

# 2. CPU Troubleshooting

| Command | Usage | Example | Real Production Example |
|---|---|---|---|
| `top` | Live CPU/process usage | `top` | Monitoring says CPU is 95%; find offending process |
| `htop` | Interactive CPU/process viewer | `htop` | Easier real-time investigation |
| `nproc` | Number of CPU cores | `nproc` → `4` | Check whether all cores are saturated |
| `lscpu` | Detailed CPU info | `lscpu` | Check architecture/core layout |
| `ps aux --sort=-%cpu \| head` | Highest CPU processes | `ps aux --sort=-%cpu \| head` | Find Java/Python process causing CPU spike |
| `ps -fp PID` | Inspect specific PID | `ps -fp 8912` | Identify which service owns high-CPU PID |

Example:

```bash
top
```

You see:

```text
PID     %CPU   COMMAND
8912    92.4   java
```

Then:

```bash
ps -fp 8912
```

Result:

```text
java -jar payment-service.jar
```

Meaning:

```text
High CPU
   ↓
PID 8912
   ↓
payment-service
```

---

# 3. Memory Troubleshooting

| Command | Usage | Example | Real Production Example |
|---|---|---|---|
| `free -h` | RAM and swap usage | `free -h` | Server/application becoming slow |
| `ps aux --sort=-%mem \| head` | Highest memory consumers | `ps aux --sort=-%mem \| head` | Find memory leak candidate |
| `vmstat 1` | Live CPU/memory stats | `vmstat 1` | Check swapping or memory pressure |
| `dmesg \| grep -i oom` | OOM killer logs | `dmesg \| grep -i oom` | Process disappeared unexpectedly |
| `journalctl -k \| grep -i oom` | Kernel OOM events | `journalctl -k \| grep -i oom` | Confirm Linux killed an app |

Example:

```bash
free -h
```

Output:

```text
Mem:  16G  15G  300M
Swap: 2G   2G     0
```

Then:

```bash
dmesg | grep -i oom
```

Output:

```text
Out of memory: Killed process 9123 (java)
```

---

# 4. Disk & Storage

| Command | Usage | Example | Real Production Example |
|---|---|---|---|
| `df -h` | Filesystem usage | `df -h` | Check disk-full alert |
| `du -sh /var/*` | Directory sizes | `du -sh /var/*` | Find what is consuming `/var` |
| `du -sh * \| sort -h` | Sort folder sizes | `du -sh * \| sort -h` | Locate largest directories |
| `find /var -type f -size +1G` | Find large files | `find /var -type f -size +1G` | Find huge log files |
| `df -i` | Inode usage | `df -i` | Disk has free GB but can't create files |
| `lsblk` | Disk/partition layout | `lsblk` | Verify attached disks |

Example:

```bash
df -h
```

Output:

```text
/dev/sda1   100G   99G   1G   99%
```

Then:

```bash
du -sh /var/*
```

Output:

```text
92G /var/log
```

Then:

```bash
du -sh /var/log/* | sort -h
```

Output:

```text
85G /var/log/payment.log
```

---

# 5. Files & Directories

| Command | Usage | Example | Real Production Example |
|---|---|---|---|
| `pwd` | Current directory | `pwd` | Confirm location before deleting/editing |
| `ls -lah` | Detailed file listing | `ls -lah` | Check size, owner, permissions |
| `cd` | Change directory | `cd /var/log` | Move to logs directory |
| `mkdir -p` | Create nested directories | `mkdir -p /opt/app/logs` | Create app folders |
| `touch` | Create empty file | `touch test.log` | Test write permissions |
| `cp` | Copy file | `cp nginx.conf nginx.conf.bak` | Back up config before editing |
| `cp -r` | Copy directory | `cp -r app app-backup` | Back up deployment directory |
| `mv` | Rename/move file | `mv app.log app.log.old` | Rotate/archive file |
| `rm` | Delete file | `rm temp.log` | Remove verified obsolete file |
| `find` | Find file | `find / -name app.log` | Locate unknown log/config |

---

# 6. Logs & File Reading

| Command | Usage | Example | Real Production Example |
|---|---|---|---|
| `cat` | Read small file | `cat config.yml` | Inspect simple config |
| `less` | Read large file | `less app.log` | Inspect large production logs safely |
| `head -20` | First lines | `head -20 app.log` | Check header/startup messages |
| `tail -100` | Last lines | `tail -100 app.log` | Check latest errors |
| `tail -f` | Follow logs live | `tail -f app.log` | Watch logs while reproducing issue |
| `grep` | Search text | `grep ERROR app.log` | Find errors |
| `grep -i` | Case-insensitive search | `grep -i error app.log` | Match Error/error/ERROR |
| `grep -n` | Show line numbers | `grep -n ERROR app.log` | Locate exact error |
| `grep -E` | Multiple patterns | `grep -E "ERROR\|WARN" app.log` | Search multiple error types |
| `journalctl -u service` | systemd service logs | `journalctl -u nginx` | Find why nginx failed |

Example:

```bash
grep "14:30" checkout.log | grep -i error
```

Output:

```text
14:30:44 ERROR database connection timeout
```

---

# 7. Processes

| Command | Usage | Example | Real Production Example |
|---|---|---|---|
| `ps aux` | List processes | `ps aux` | Check whether application is running |
| `ps -ef` | Process tree style list | `ps -ef` | Inspect parent/child relationships |
| `pgrep -af java` | Find process by name | `pgrep -af java` | Locate Java applications |
| `ps -fp PID` | Inspect PID | `ps -fp 4512` | Identify suspicious process |
| `kill PID` | Graceful termination | `kill 4512` | Stop stuck app cleanly |
| `kill -9 PID` | Force kill | `kill -9 4512` | Last resort for frozen process |
| `pkill name` | Kill by name | `pkill myapp` | Stop matching process carefully |

---

# 8. Service Management

| Command | Usage | Example | Real Production Example |
|---|---|---|---|
| `systemctl status` | Check service status | `systemctl status nginx` | Website down |
| `systemctl start` | Start service | `systemctl start nginx` | Service stopped |
| `systemctl stop` | Stop service | `systemctl stop nginx` | Planned maintenance |
| `systemctl restart` | Restart service | `systemctl restart nginx` | Recover/apply config |
| `systemctl reload` | Reload config | `systemctl reload nginx` | Apply config with less disruption |
| `systemctl enable` | Start after reboot | `systemctl enable nginx` | Ensure service survives restart |
| `journalctl -u service -n 100` | Recent logs | `journalctl -u nginx -n 100` | Find startup/config error |

Example:

```bash
systemctl status nginx
```

Output:

```text
failed
```

Then:

```bash
journalctl -u nginx -n 100
```

Output:

```text
bind() to 0.0.0.0:80 failed
Address already in use
```

Then:

```bash
ss -lntp | grep :80
```

---

# 9. Networking

| Command | Usage | Example | Real Production Example |
|---|---|---|---|
| `ip addr` | Show IP/interfaces | `ip addr` | Verify expected server IP |
| `ip route` | Routing table | `ip route` | Server can't reach private subnet |
| `ping` | Basic connectivity | `ping 10.0.0.20` | Test whether host is reachable |
| `ss -lntp` | Listening TCP ports | `ss -lntp` | Find application port |
| `ss -lntp \| grep 8080` | Check specific port | `ss -lntp \| grep 8080` | Backend expected on 8080 |
| `curl URL` | HTTP request | `curl localhost:8080` | Test app locally |
| `curl -I URL` | HTTP headers | `curl -I https://example.com` | Check status code/redirect |
| `curl -v URL` | Verbose HTTP test | `curl -v localhost:8080` | Debug connection/TLS |
| `traceroute host` | Network path | `traceroute db.internal` | Find where connectivity fails |

---

# 10. DNS

| Command | Usage | Example | Real Production Example |
|---|---|---|---|
| `nslookup` | DNS lookup | `nslookup db.internal` | Hostname not resolving |
| `dig` | Detailed DNS query | `dig api.company.com` | Inspect DNS records |
| `dig +short` | Show resolved IP only | `dig +short api.company.com` | Quick resolution check |
| `getent hosts` | OS-level resolver lookup | `getent hosts db.internal` | Verify how app sees hostname |

Example:

```bash
getent hosts redis.internal
```

No output.

Then:

```bash
dig redis.internal
```

DNS problem confirmed.

---

# 11. Port Connectivity

| Command | Usage | Example | Real Production Example |
|---|---|---|---|
| `nc -zv host port` | Test TCP port | `nc -zv db.internal 5432` | Test PostgreSQL access |
| `nc -zv host 6379` | Test Redis | `nc -zv redis.internal 6379` | Redis timeout |
| `curl host:port` | HTTP port test | `curl localhost:5000/health` | Test backend health endpoint |

Example:

```bash
nc -zv db.internal 5432
```

If:

```text
Connection timed out
```

Possible areas:

```text
DB down
Firewall
Security group
Network route
DB not listening
```

---

# 12. Permissions

| Command | Usage | Example | Real Production Example |
|---|---|---|---|
| `ls -l` | View permissions/owner | `ls -l app.log` | App can't write logs |
| `chmod` | Change permissions | `chmod 755 deploy.sh` | Script not executable |
| `chown` | Change ownership | `chown app:app /opt/app` | App user lacks access |
| `id user` | User groups | `id appuser` | Confirm group membership |

Example:

```text
Permission denied: /var/log/myapp/app.log
```

Check:

```bash
ls -ld /var/log/myapp
```

If owned by root:

```bash
chown -R appuser:appuser /var/log/myapp
```

only after confirming that's the intended ownership.

---

# 13. SSH / Remote Access

| Command | Usage | Example | Real Production Example |
|---|---|---|---|
| `ssh user@host` | Remote login | `ssh ubuntu@10.0.0.10` | Access production server |
| `ssh -i key.pem user@host` | SSH with key | `ssh -i prod.pem ubuntu@10.0.0.10` | Cloud VM access |
| `scp file user@host:/tmp` | Upload file | `scp config.yml user@host:/tmp` | Transfer config |
| `scp user@host:/file .` | Download file | `scp user@host:/var/log/app.log .` | Collect logs |

---

# 14. Text Processing

| Command | Usage | Example | Real Production Example |
|---|---|---|---|
| `awk` | Extract/process columns | `awk '{print $1}' access.log` | Extract client IPs |
| `sort` | Sort data | `sort ips.txt` | Prepare data for analysis |
| `uniq -c` | Count duplicates | `sort ips.txt \| uniq -c` | Count requests per IP |
| `cut` | Extract fields | `cut -d: -f1 /etc/passwd` | Extract usernames |
| `sed` | Replace/edit text | `sed 's/dev/prod/g' file` | Automated config transformation |

Example:

```bash
awk '{print $1}' access.log | sort | uniq -c | sort -nr
```

Output:

```text
8450 10.10.1.20
220  10.10.1.15
95   10.10.1.10
```

This can help identify a source generating unusually high traffic.

---

# 15. Archive & Compression

| Command | Usage | Example | Real Production Example |
|---|---|---|---|
| `tar -czvf` | Create compressed archive | `tar -czvf logs.tar.gz logs/` | Bundle incident logs |
| `tar -xzvf` | Extract archive | `tar -xzvf app.tar.gz` | Extract application package |
| `gzip` | Compress file | `gzip app.log` | Compress old logs |
| `gunzip` | Decompress file | `gunzip app.log.gz` | Inspect archived logs |

---

# 16. Environment Variables

| Command | Usage | Example | Real Production Example |
|---|---|---|---|
| `env` | Show all variables | `env` | Inspect container/server runtime config |
| `echo $VAR` | Show one variable | `echo $DATABASE_URL` | Check DB endpoint |
| `export` | Set variable | `export APP_ENV=production` | Configure application |
| `which` | Executable path | `which java` | Verify correct Java binary |
| `command -v` | Check command | `command -v docker` | Verify Docker installed |

---

# 17. Docker

| Command | Usage | Example | Real Production Example |
|---|---|---|---|
| `docker ps` | Running containers | `docker ps` | Check whether backend container runs |
| `docker ps -a` | All containers | `docker ps -a` | Find exited container |
| `docker logs` | Container logs | `docker logs backend` | Find app error |
| `docker logs --tail 100` | Last logs | `docker logs --tail 100 backend` | Fast incident check |
| `docker logs -f` | Live logs | `docker logs -f backend` | Watch issue in real time |
| `docker inspect` | Detailed container info | `docker inspect backend` | Check health/network/mounts |
| `docker stats` | CPU/RAM usage | `docker stats` | Find container resource problem |
| `docker exec -it` | Enter container | `docker exec -it backend sh` | Test app from inside |
| `docker top` | Processes inside container | `docker top backend` | Confirm app process |

Example:

```bash
docker ps
```

Container:

```text
backend    Up 20 minutes    unhealthy
```

Then:

```bash
docker inspect backend
docker logs --tail 100 backend
docker exec -it backend sh
curl localhost:5000/health
```

---

# 18. Kubernetes

| Command | Usage | Example | Real Production Example |
|---|---|---|---|
| `kubectl get pods` | List pods | `kubectl get pods` | Check app workload |
| `kubectl get pods -A` | All namespaces | `kubectl get pods -A` | Find failing components |
| `kubectl describe pod` | Pod events/details | `kubectl describe pod payment-abc` | Diagnose CrashLoopBackOff |
| `kubectl logs` | Pod logs | `kubectl logs payment-abc` | Application failures |
| `kubectl logs --previous` | Previous crashed logs | `kubectl logs payment-abc --previous` | Inspect crash before restart |
| `kubectl logs -f` | Live logs | `kubectl logs -f payment-abc` | Watch live error |
| `kubectl exec -it` | Enter pod | `kubectl exec -it payment-abc -- sh` | Test DNS/network |
| `kubectl get svc` | Services | `kubectl get svc` | Check ClusterIP |
| `kubectl get endpoints` | Backend endpoints | `kubectl get endpoints` | Service exists but no pods behind it |
| `kubectl get events` | Kubernetes events | `kubectl get events --sort-by=.metadata.creationTimestamp` | Find scheduling/image/mount errors |

---

# One Production Troubleshooting Flow

Suppose:

```text
ALERT:
Checkout API returning 502
```

| Step | Category | Command | Result | What it tells you |
|---|---|---|---|---|
| 1 | System | `hostname` | `prod-web-01` | Correct server |
| 2 | CPU | `top` | CPU 20% | CPU okay |
| 3 | Memory | `free -h` | 8 GB available | Memory okay |
| 4 | Disk | `df -h` | 55% | Disk okay |
| 5 | Service | `systemctl status nginx` | running | Nginx okay |
| 6 | Network | `ss -lntp \| grep 443` | listening | HTTPS okay |
| 7 | Network | `curl localhost:8080` | connection refused | Backend problem |
| 8 | Service | `systemctl status backend` | failed | Backend stopped |
| 9 | Logs | `journalctl -u backend -n 100` | DB timeout | Dependency issue |
| 10 | DNS | `getent hosts db.internal` | resolves | DNS okay |
| 11 | Port | `nc -zv db.internal 5432` | timeout | DB connectivity problem |

The incident becomes:

```text
User
 ↓
Nginx :443          ✅
 ↓
Backend :8080       ❌
 ↓
Backend service     ❌
 ↓
DB timeout
 ↓
DB :5432 unreachable
 ↓
Investigate DB / firewall / routing
```

