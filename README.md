# Kurrent-Grafana
Grafana Dashboards for KurrentDB

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
