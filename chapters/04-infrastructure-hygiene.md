# Chapter 4: Infrastructure Hygiene

## Disk Space Is a Business Risk

Our fleet runs on a VPS. When the disk fills up, everything stops. Not gracefully. Docker containers fail to write logs. Databases corrupt. SSL renewal fails silently. The first time it happened, we lost 6 hours of agent output because no one noticed until a client-facing delivery was late.

## Disk Management

### The 80% Rule
Alert at 80% disk usage. Act at 85%. Panic at 90%. At 95%, you are already in an incident.

```bash
# Add to cron, run every 6 hours
USAGE=$(df / | tail -1 | awk '{print $5}' | sed 's/%//')
if [ "$USAGE" -gt 80 ]; then
    curl -X POST "$WEBHOOK_URL" -d "{\"text\": \"DISK WARNING: ${USAGE}% used\"}"
fi
```

### Docker Is the Usual Suspect
Docker images, containers, and volumes accumulate silently. A fleet of 34 agents, each rebuilt weekly, generates gigabytes of orphaned layers.

```bash
# Weekly cleanup (add to maintenance cron)
docker system prune -af --volumes
docker builder prune -af
```

Run this on a schedule. Do not wait for disk alerts. The prune that runs on Sunday prevents the outage on Wednesday.

### Log Rotation
Agent logs grow without bound unless you cap them. Every Docker service should have log limits in the compose file:

```yaml
services:
  agent-email-triage:
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
```

## SSL and DNS

### SSL Renewal
Use Traefik or Caddy with automatic Let's Encrypt renewal. Never manage certificates manually. Manual cert renewal is a ticking time bomb that detonates on the day you are busiest.

Verify renewal is working:
```bash
# Check cert expiry for all domains
for domain in $(cat /etc/traefik/domains.txt); do
    echo | openssl s_client -servername "$domain" -connect "$domain":443 2>/dev/null \
        | openssl x509 -noout -dates
done
```

### DNS Propagation
After any DNS change, verify propagation before declaring success. Use multiple resolvers:
```bash
for ns in 8.8.8.8 1.1.1.1 9.9.9.9; do
    dig @$ns yourdomain.com A +short
done
```

## Docker Compose, Not Standalone

Run everything through Docker Compose with a reverse proxy (Traefik recommended). Never run standalone nginx containers managed by hand. The moment you have two services with conflicting port bindings and no orchestration layer, you have an incident waiting to happen.

## VPS Security Baseline

- fail2ban on SSH (ban after 3 attempts)
- UFW with explicit allow rules only
- Weekly Nmap self-scan to detect unexpected open ports
- No root SSH. Key-based auth only.

```bash
# Weekly security self-scan
nmap -sV -p 1-65535 localhost | diff - /opt/security/baseline_ports.txt
```

If the diff is non-empty, something changed. Investigate before it becomes an exploit.

## The Maintenance Window

Pick a time. Ours is Sunday 10 PM. Every week. No exceptions. Docker prune, log review, security scan, disk check, SSL verify. Thirty minutes of prevention saves days of incident response.
