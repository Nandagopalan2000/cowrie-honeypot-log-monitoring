# Cowrie SSH/Telnet Honeypot with Centralized Log Monitoring

## Overview
A personal lab project using Cowrie to capture SSH/Telnet interaction events and a lightweight log pipeline to centralize and visualize those events with Promtail, Loki, and Grafana.

> **Portfolio note:** This repository contains a sanitized report of the project. The original lab virtual machines are not currently available, so this is documentation of a previously completed lab, not a claim that the stack is running today.

## Objectives
- Deploy Cowrie as a low-interaction SSH/Telnet honeypot on Debian.
- Enable Cowrie's JSON event logging.
- Forward logs to a separate log server with Promtail.
- Store and query logs in Loki.
- Build Grafana views for failed login attempts and connection activity.
- Verify the log flow from honeypot to dashboard.

## Architecture
```text
+--------------------------+        HTTP push        +---------------------------+
| Debian honeypot          | ----------------------> | Separate log server       |
|                          |                         |                           |
| Cowrie                   |                         | Loki                      |
|   writes cowrie.json     |                         |   stores/queries logs     |
| Promtail                 |                         | Grafana                   |
|   tails and ships logs   |                         |   dashboards/Explore      |
+--------------------------+                         +---------------------------+
```

## Components
- **Cowrie:** records connection attempts, login attempts, and simulated shell activity.
- **Promtail:** tails the Cowrie JSON log and ships new entries to Loki.
- **Loki:** stores log streams and supports label-based querying.
- **Grafana:** provides Explore and dashboard panels for reviewing activity.
- **Docker Compose:** used to run Loki and Grafana together on the log server.

## Data Flow
1. A connection or test interaction reaches the honeypot.
2. Cowrie records the event in JSON format.
3. Promtail tails the log and pushes entries to Loki.
4. Loki stores the log stream and makes it queryable.
5. Grafana queries Loki and displays the results.

## Example Log Queries
In Grafana Explore, select the Loki data source and use queries matching the labels configured in the lab. Examples from the report:

```logql
{job="cowrie-honeypot"}
```

```logql
{job="cowrie-honeypot"} |= "login.failed"
```

Example dashboard expressions documented in the report:

```logql
count_over_time({job="cowrie-honeypot"} |= "login.failed" [5m])
```

```logql
{job="cowrie-honeypot"} |= "session.connect"
```

The exact labels and paths must match the configuration used in the deployment.

## What I Practiced
- Deploying and checking a honeypot service.
- Separating log generation from centralized storage and analysis.
- Forwarding structured JSON logs.
- Configuring a log shipper and verifying end-to-end delivery.
- Querying event types and building basic monitoring panels.
- Checking service status and recovering the stack after a shutdown.

## Report
See [`Cowrie_Honeypot_Log_Monitoring_Project.pdf`](https://github.com/Nandagopalan2000/cowrie-honeypot-log-monitoring/blob/main/Cowrie_Honeypot_Log_Monitoring_Project.pdf) for the detailed deployment notes and verification steps.

## Security Notes
- Never commit real passwords, API keys, tokens, private IP addresses, or sensitive hostnames.
- The published PDF has been sanitized to remove the example Grafana password and selected identifying/path/network details.
- Use a unique strong password or a secret-management mechanism in any rebuilt environment; do not put secrets directly in Compose files.
- Expose honeypot services only in a controlled, authorized environment and isolate them from personal or production networks.

## Future Improvements
- Rebuild and test the stack on the current machine.
- Add sanitized screenshots of the dashboard and sample events.
- Add deployment configuration files with secrets externalized and versions reviewed.
- Add alerting for repeated failed logins and suspicious session activity.
