# EventStore-Grafana
Grafana Dashboards for EventStoreDB

## Dashboards

### Summary

- https://grafana.com/grafana/dashboards/19455-eventstore-cluster-summary/

### Panels

- https://grafana.com/grafana/dashboards/19461-eventstore-panels/

## Quick-start

### 1. Run EventStore

Run securely as normal with http exposed on any of the following ports: 2111, 2112, 2113, 2114

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
1. in the top-right click the blue `Share` button
1. in the pop up click the `Export` tab
1. set `Export for sharing externally` **false**
1. Adjust the appropriate dashboard in `./dashboards/` by either
    - `Save to file` and replace the file with the new one or
    - `View JSON` and copy/paste over the content of the file
1. consider taking some screen shots for `./img`
1. note that editing the file in `./dashboards` will cause it to be reimported automatically after a few seconds

### 6. PR

- commit, open a PR, get it merged

### 7. Upload to grafana.com

_todo: automate!_

1. on the `main` branch
2. add a git tag (todo: elaborate)
3. export for sharing. similar to the internal export except:
    - set `Export for sharing externally` **true**
    - `Save to file` and use this file for uploading
4. upload (todo: elaborate)

### Clean up (optional)

- this is not usually necessary, updating to a commit with a different version of the dashboard will cause it to be reimported into the existin database
- docker-compose rm
- delete the data directory (nothing in there is committed)

## Alternative to Edit/Export: directly edit the json

The json files in `./dashboards` can be edited directly. After a few seconds and a browser refresh they will automatically appear in grafana. PR can then be opened.
