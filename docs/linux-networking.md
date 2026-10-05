# Linux & Networking Cheat Sheet

## System basics

```bash
uname -a
cat /etc/os-release
uptime
whoami
id
hostname
```

## CPU / memory

```bash
top
htop
free -h
vmstat 1
```

Processes:

```bash
ps aux
ps aux --sort=-%cpu | head
ps aux --sort=-%mem | head
pgrep -af <name>
```

## Disk

```bash
df -h
df -i
du -sh *
du -sh /var/* 2>/dev/null | sort -h
lsblk
```

## Files

```bash
ls -lah
find . -type f -name '*.log'
find /var/log -type f -mtime -1
```

Large files:

```bash
find / -xdev -type f -size +1G -printf '%s %p\n' 2>/dev/null | sort -n
```

## Text/logs

```bash
tail -f app.log
tail -n 200 app.log
grep -R "ERROR" .
grep -Rni "connection refused" .
```

With context:

```bash
grep -n -C 3 "ERROR" app.log
```

`jq`:

```bash
cat data.json | jq .
jq '.items[] | {name,status}' data.json
```

## Permissions

```bash
ls -l
chmod +x script.sh
chmod 640 file
chown user:group file
```

Avoid `chmod 777` as a reflex; determine which identity actually needs which permission.

## Network interfaces/routes

```bash
ip addr
ip route
ip neigh
```

## Listening sockets

```bash
ss -lntp
ss -lnup
```

Alternative:

```bash
lsof -i
lsof -i :8080
```

## Connectivity

HTTP:

```bash
curl -v https://example.com
curl -I https://example.com
curl -sS http://localhost:8080/health
```

TCP:

```bash
nc -vz <host> 443
nc -vz <host> 5432
```

DNS:

```bash
dig <hostname>
dig +short <hostname>
nslookup <hostname>
```

Route/path:

```bash
traceroute <host>
```

## TLS

Inspect remote certificate:

```bash
openssl s_client -connect <host>:443 -servername <host>
```

Certificate details:

```bash
openssl s_client -connect <host>:443 -servername <host> </dev/null 2>/dev/null \
  | openssl x509 -noout -subject -issuer -dates
```

## systemd

```bash
systemctl status <service>
systemctl start <service>
systemctl stop <service>
systemctl restart <service>
systemctl enable <service>
```

Logs:

```bash
journalctl -u <service>
journalctl -u <service> -f
journalctl -u <service> --since "10 minutes ago"
journalctl -p err -b
```

## Archives

```bash
tar -czf archive.tar.gz directory/
tar -xzf archive.tar.gz
tar -tf archive.tar.gz
```

## Environment

```bash
env
printenv
echo "$PATH"
export KEY=value
```

## Shell history

```bash
history
```

Avoid typing secrets on command lines. Many shells persist them.
