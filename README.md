# Kurrent-Grafana
Grafana Dashboards and Alert Rules for KurrentDB

## Dashboards

### Summary

- https://grafana.com/grafana/dashboards/22824

### Panels

- https://grafana.com/grafana/dashboards/22823

## Quick-start

### 1. Run KurrentDB

Run one or more nodes securely as normal with `NodePort` set to 2111, 2112, 2113, 2114

Does not have to be in containers.

### 2. Run Prometheus/Grafana

```
docker-compose up
```

Grafana: http://localhost:3000/dashboards admin/admin

Prometheus: http://localhost:9090/

### 3. Edit the dashboards & save

http://localhost:3000/dashboards

When you save the dashboard it will save it to the grafana database but not to the dashboards in the git repository. For that we need to export.

### 4. Export a dashboard for committing

_todo: automate!_

1. open the dashboard in Grafana
1. expand/collapse the sections as appropriate
    - for the summary dashboard, expand them all
    - for the panels dashboard, collapse them all
1. set the 'Instance' filter at the top to 'All'
1. in the top-right click the `Export` button
1. click 'Export as JSON'
1. set `Export the dashboard to use in another instance` **false**
1. Adjust the appropriate dashboard in `./dashboards/` by either clicking
    - `Download file` and replace the file with the new one or
    - `Copy to clipboard` and paste over the content of the file
1. consider taking some screen shots for `./img`

Alternatively, the json in `./dashboards/` can be edited manually.

Note that editing the file in `./dashboards` will cause it to be reimported automatically after a few seconds.

### 6. PR

- commit, open a PR, get it merged

### 7. Upload to grafana.com

_todo: automate!_

1. on the `main` branch
2. add a git tag (todo: elaborate)
3. export for sharing. similar to the internal export except:
    - set `Export the dashboard to use in another instance` **true**
    - `Save to file` and use this file for uploading
4. upload (todo: elaborate)

### 8. Create github release with link to grafana.com

(this announces the release in #release-announcements)

### Clean up (optional)

- this is not usually necessary, updating to a commit with a different version of the dashboard will cause it to be reimported into the existing database
- docker-compose down -v

## Alternative to Edit/Export: directly edit the json

The json files in `./dashboards` can be edited directly. After a few seconds and a browser refresh they will automatically appear in grafana. PR can then be opened.

## Alerting

`./alerts/kurrentdb-alert-rules.yaml` contains the alert rules from the whitepaper _Monitoring and Alerting for KurrentDB 26.1 with Grafana_: 19 rules covering resources (disk, memory, CPU, GC pauses), throughput (reader queue, stream-info cache), projections (progress, status, state size), and subscriptions/cluster (persistent subscription lag, parked messages, gRPC failures, node availability, elections, replication lag). Comments in the file map each rule to its whitepaper rule number. The thresholds are starting points — review them against your own workload.

### Import into Grafana (UI)

Requires Grafana 12 or later (with the same Prometheus data source the dashboards use):

1. download [`alerts/kurrentdb-alert-rules.yaml`](alerts/kurrentdb-alert-rules.yaml)
1. in Grafana open **Alerting → Alert rules**
1. click **More → Import alert rules**
1. upload the file, select your Prometheus data source and a target folder
1. review the preview and confirm

The imported rules are created as Grafana-managed rules with "no data" handling set to Normal, preserving Prometheus semantics (most rules return no series while the system is healthy). The rules only evaluate — see [Contact points and notification policies](#contact-points-and-notification-policies) below, or nothing is delivered when they fire.

On Grafana 10/11 the import UI is not available — use the provisioning file below instead.

### Import via provisioning (any Grafana 10.x+)

`./config/grafana/etc-grafana-provisioning/alerting/kurrentdb-alert-rules.yaml` is the same rule set in Grafana's alerting provisioning format. Copy it into `/etc/grafana/provisioning/alerting/` on your Grafana instance, set `datasourceUid` to your Prometheus data source UID (the file ships with `PBFA97CFB590B2093`, the UID this repo's stack uses), and restart Grafana. The rules appear in a "KurrentDB Alerts" folder and start evaluating immediately, but notify no one until the contact points and policies below exist.

### Contact points and notification policies

Neither import path creates contact points or notification policies — without them the rules evaluate but deliver nothing. The rules couple to your notification setup through **labels only**: every rule attaches `severity` (`warning` or `critical`), and the two parked-message rules add `team=application`. Grafana matches notification policies to alerts by labels, so you are free to define whatever contact points and policy tree suit your organization — the one exact-match requirement is that your policy matchers use these label keys and values as the rules define them. Following Steps 3–4 of the whitepaper:

1. **Contact points** — create one per destination, with any names you like, for example `kurrentdb-slack` (Slack bot token) and `kurrentdb-email` (SMTP).
1. **Notification policies** — add routes whose matchers use the rules' labels exactly, pointing at whichever contact points you created. The whitepaper's reference policy: set the default route's contact point to `kurrentdb-slack`, then add a child policy with matcher `severity = critical` pointing at `kurrentdb-email` with **Continue matching subsequent sibling nodes** enabled (so critical alerts also reach Slack), and optionally a child policy matching `team = application` for the channel your application team owns.
1. **Test before you trust it** — use each contact point's **Test** button, then force one rule to fire (the whitepaper's Step 6 walks through temporarily lowering the disk rule's threshold) and confirm delivery end to end.

### Trying it in this repo's stack

`docker-compose up` provisions the alert rules automatically alongside the dashboards. Note that the bundled Prometheus scrape config lists four node targets (ports 2111–2114), so `KurrentDBNodeDown` fires for any node you are not running — useful for seeing the pipeline work, but expected noise on a single-node setup.

### Notes

- `KurrentDBNodeDown` matches scrape jobs with `job=~".*kurrentdb.*"`, which covers both this repo's `scrape-kurrentdb` job and the whitepaper's `kurrentdb` job. Align it with your own scrape job name if yours differs.
- Conditions that compare against 0 or a lower bound (`== bool 0`, `< bool 0.99`) use the PromQL `bool` modifier so the rule fires correctly; without it, a matched value of 0 is read as "not firing".
- Two optional variants are included as comments in the rules file: the index-disk utilization rule (enable it if your index is on its own volume) and the Windows CPU gauge (`kurrentdb_sys_cpu`).
- After tuning rules in Grafana, export them (**Alerting → Alert rules → Export rules**) and update the provisioning YAML here, mirroring the dashboard edit/export workflow.
