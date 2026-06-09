\# Lab 1: SRE Philosophy — Deploy, Break, Understand



\## Task 1 — Deploy \& Break QuickTicket (6 pts)



\### 1.1 docker compose ps output



\~\~\~

NAME             IMAGE                COMMAND                  SERVICE    CREATED              STATUS                        PORTS

app-events-1     app-events           "uvicorn main:app --…"   events     About a minute ago   Up 54 seconds                 0.0.0.0:8081->8081/tcp, \[::]:8081->8081/tcp

app-gateway-1    app-gateway          "uvicorn main:app --…"   gateway    About a minute ago   Up 54 seconds                 0.0.0.0:3080->8080/tcp, \[::]:3080->8080/tcp

app-payments-1   app-payments         "uvicorn main:app --…"   payments   About a minute ago   Up About a minute             0.0.0.0:8082->8082/tcp, \[::]:8082->8082/tcp

app-postgres-1   postgres:17-alpine   "docker-entrypoint.s…"   postgres   About a minute ago   Up About a minute (healthy)   0.0.0.0:5432->5432/tcp, \[::]:5432->5432/tcp

app-redis-1      redis:7-alpine       "docker-entrypoint.s…"   redis      About a minute ago   Up About a minute (healthy)   0.0.0.0:6379->6379/tcp, \[::]:6379->6379/tcp

\~\~\~



\### 1.2 Critical Path Output



\*\*List events:\*\*

\~\~\~

&#x20; {

&#x20;     "id": 1,

&#x20;     "name": "Go Conference 2026",

&#x20;     "venue": "Main Hall A",

&#x20;     "date": "2026-09-15T09:00:00+00:00",

&#x20;     "total\_tickets": 100,

&#x20;     "price\_cents": 5000,

&#x20;     "available": 99

&#x20; },

&#x20; {

&#x20;     "id": 4,

&#x20;     "name": "Python Workshop",

&#x20;     "venue": "Lab 301",

&#x20;     "date": "2026-09-22T14:00:00+00:00",

&#x20;     "total\_tickets": 25,

&#x20;     "price\_cents": 2000,

&#x20;     "available": 25

&#x20; },

&#x20; {

&#x20;     "id": 2,

&#x20;     "name": "SRE Meetup",

&#x20;     "venue": "Room 204",

&#x20;     "date": "2026-10-01T18:00:00+00:00",

&#x20;     "total\_tickets": 30,

&#x20;     "price\_cents": 0,

&#x20;     "available": 30

&#x20; },

&#x20; {

&#x20;     "id": 5,

&#x20;     "name": "Kubernetes Deep Dive",

&#x20;     "venue": "Auditorium B",

&#x20;     "date": "2026-10-10T10:00:00+00:00",

&#x20;     "total\_tickets": 80,

&#x20;     "price\_cents": 8000,

&#x20;     "available": 80

&#x20; },

&#x20; {

&#x20;     "id": 3,

&#x20;     "name": "Cloud Native Summit",

&#x20;     "venue": "Expo Center",

&#x20;     "date": "2026-11-20T10:00:00+00:00",

&#x20;     "total\_tickets": 500,

&#x20;     "price\_cents": 15000,

&#x20;     "available": 500

&#x20; }

\~\~\~



\*\*Reserve ticket:\*\*

\~\~\~

{

&#x20;   "reservation\_id": "cb810559-370a-4d5a-a00c-ba88b52402f4",

&#x20;   "event\_id": 1,

&#x20;   "quantity": 1,

&#x20;   "total\_cents": 5000,

&#x20;   "expires\_in\_seconds": 300

}

\~\~\~



\*\*Pay for reservation:\*\*

\~\~\~

{

&#x20;   "order\_id": "cb810559-370a-4d5a-a00c-ba88b52402f4",

&#x20;   "event\_id": 1,

&#x20;   "quantity": 1,

&#x20;   "total\_cents": 5000,

&#x20;   "status": "confirmed"

}

\~\~\~

\### 1.3 Output of curl -s http://localhost:3080/health when everything is healthy

\~\~\~

{

&#x20;   "status": "healthy",

&#x20;   "checks": {

&#x20;       "events": "ok",

&#x20;       "payments": "ok",

&#x20;       "circuit\_payments": "CLOSED"

&#x20;   }

}

\~\~\~

\### 1.4 A dependency map

\~\~\~

Gateway (port 3080)

&#x20;   ├── Events Service (port 8081)

&#x20;   │       ├── PostgreSQL (port 5432)

&#x20;   │       └── Redis (port 6379)

&#x20;   └── Payments Service (port 8082)

\~\~\~

\### 1.5 A failure table:

\~\~\~

| Component Killed | Events List       | Reserve         | Pay                 | Health Check | User Impact     |

|------------------|-------------------|-----------------|---------------------|--------------|-----------------|

| payments         | Works             | Works           | Fails (504 Timeout) | degraded     | Cannot pay      |

| events           | Fails(Timeout)    | Fails (Timeout) | Fails               | degraded     | Complete outage |

| redis            | Works             | Fails (Timeout) | Fails               | degraded     | Cannot reserve  |

| postgres         | Fails(Unavailable)| Fails           | Fails               | degraded     | Complete outage |

\~\~\~



\### 1.6 Generator output

\~\~\~

QuickTicket Load Generator

Target: http://localhost:3080 | RPS: 5 | Duration: 30s

\---

✓ Request 1: HTTP 200

✓ Request 2: HTTP 200

✓ Request 3: HTTP 200

✓ Request 4: HTTP 200

✓ Request 5: HTTP 200

✓ Request 6: HTTP 200

✓ Request 7: HTTP 200

✓ Request 8: HTTP 200

✓ Request 9: HTTP 200

✓ Request 10: HTTP 200

✓ Request 11: HTTP 200

✓ Request 12: HTTP 200

✓ Request 13: HTTP 200

✓ Request 14: HTTP 200

✓ Request 15: HTTP 200

✓ Request 16: HTTP 200

✓ Request 17: HTTP 200

✓ Request 18: HTTP 200

✓ Request 19: HTTP 200

✓ Request 20: HTTP 200

✓ Request 21: HTTP 200

✓ Request 22: HTTP 200

✓ Request 23: HTTP 200

✓ Request 24: HTTP 200

✓ Request 25: HTTP 200

✓ Request 26: HTTP 200

✓ Request 27: HTTP 200

✓ Request 28: HTTP 200

✓ Request 29: HTTP 200

✓ Request 30: HTTP 200

✓ Request 31: HTTP 200

✓ Request 32: HTTP 200

✓ Request 33: HTTP 200

✓ Request 34: HTTP 200

✓ Request 35: HTTP 200

✓ Request 36: HTTP 200

✓ Request 37: HTTP 200

✓ Request 38: HTTP 200

✓ Request 39: HTTP 200

✓ Request 40: HTTP 200

✓ Request 41: HTTP 200

✓ Request 42: HTTP 200

✓ Request 43: HTTP 200

✓ Request 44: HTTP 200

✓ Request 45: HTTP 200

✓ Request 46: HTTP 200

✗ Request 47: HTTP 504

✓ Request 48: HTTP 200

✗ Request 49: HTTP 504

✓ Request 50: HTTP 200

✗ Request 51: HTTP 504

\---

Done. total=51 success=48 fail=3 error\_rate=5.9%

\~\~\~



\## Task 2 — Graceful Degradation (3 pts)



\### Diff of gateway change

\~\~\~



diff --git a/app/gateway/main.py b/app/gateway/main.py

index abc123def..456ghi789 100644

\--- a/app/gateway/main.py

+++ b/app/gateway/main.py

@@ -250,7 +250,15 @@ async def pay\_reservation(reservation\_id: str):

&#x20;    except CircuitOpenError:

&#x20;        log.error("circuit open, skipping payments call")

&#x20;        raise HTTPException(503, "Payment service temporarily unavailable (circuit open)")

\-    except httpx.TimeoutException:

\-        raise HTTPException(504, "Payment service timeout")

\+    except httpx.TimeoutException as e:

\+        log.error(f"payment timeout: {e}")

\+        raise HTTPException(

\+            status\_code=503,

\+            detail={

\+                "error": "payments\_unavailable",

\+                "message": "Payment service is temporarily down. Your reservation is held — try again in a few minutes.",

\+                "reservation\_id": reservation\_id

\+            }

\+        )

&#x20;    except httpx.HTTPStatusError as e:

&#x20;        raise HTTPException(e.response.status\_code, "Payment failed")

&#x20;    except Exception as e:

\~\~\~



\### Output when payments is down

\#### Reserve (works)

\~\~\~

{

&#x20;   "reservation\_id": "425c8744-f2ee-4a52-9bd5-2d43064d80c8",

&#x20;   "event\_id": 1,

&#x20;   "quantity": 1,

&#x20;   "total\_cents": 5000,

&#x20;   "expires\_in\_seconds": 300

}

\~\~\~



\#### Pay (clear 503 error)

\~\~\~

{

&#x20;   "detail": {

&#x20;       "error": "payments\_unavailable",

&#x20;       "message": "Payment service is temporarily down. Your reservation is held — try again in a few minutes.",

&#x20;       "reservation\_id": "425c8744-f2ee-4a52-9bd5-2d43064d80c8"

&#x20;   }

}

\~\~\~

\## GitHub Community

Starring helps discover popular projects, shows support to maintainers, and makes it easy to find repositories later. Following developers helps build professional network, learn from their work, and stay updated on interesting projects.





