\# Runbook: QuickTicket High Error Rate



\## Alert

\- \*\*Fires when:\*\* Gateway 5xx error rate > 5% for 2 minutes

\- \*\*Dashboard:\*\* QuickTicket — Golden Signals



\## Diagnosis

1\. Check which service is failing:

&#x20;  - `curl -s http://localhost:3080/health | python3 -m json.tool`

2\. Check payments service directly:

&#x20;  - `curl -s http://localhost:8082/health`

3\. Check events service:

&#x20;  - `curl -s http://localhost:8081/health`

4\. Check logs for errors:

&#x20;  - `docker compose logs gateway --tail=20 --since=5m`

&#x20;  - `docker compose logs payments --tail=20 --since=5m`



\## Common Causes

| Cause | How to identify | Fix |

|-------|----------------|-----|

| Payments service down | health shows payments: down | Restart: `docker compose start payments` |

| Payments high failure rate | health OK but errors in logs | Check PAYMENT\_FAILURE\_RATE env var |

| Events service down | health shows events: down | Restart: `docker compose start events` |

| Database connection exhausted | events logs show pool errors | Restart events, check DB\_MAX\_CONNS |



\## Escalation

\- If not resolved in 10 minutes, escalate to: \[instructor/TA]

