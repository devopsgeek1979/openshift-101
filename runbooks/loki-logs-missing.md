# Loki Logs Missing Troubleshooting

## Symptoms

- Logs are not appearing in Loki queries
- Promtail is running but no data ingested

## Possible Causes

1. Promtail configuration issues
2. Network connectivity between Promtail and Loki
3. Log file permissions
4. Loki storage issues

## Steps to Troubleshoot

1. Check Promtail logs: `kubectl logs <promtail-pod>`
2. Verify Loki endpoint: `curl http://loki:3100/ready`
3. Ensure log paths are correct in promtail-config.yaml
4. Check firewall rules

## Resolution

- Update configurations as needed
- Restart services
