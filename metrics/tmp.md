## docker-flow-monitor-full.yml

TODO

### Requirements

* Network called `proxy` is created
* *Docker Flow Proxy* is running
* Environment variable `DOMAIN` is created
* Docker secret `alert_manager_config` with AlertManager configuration is created

```bash
docker network create --driver overlay proxy

docker stack deploy -c ../proxy/docker-flow-proxy.yml proxy

# Replace [...] with the domain or IP where the stack will be deployed.
export DOMAIN=[...]

# Replace [...] with the SLACK API URL https://hooks.slack.com/services/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
export SLACK_API_URL=[...]

echo "route:
  group_by: [service,scale]
  repeat_interval: 5m
  group_interval: 5m
  receiver: 'slack'

receivers:
  - name: 'slack'
    slack_configs:
      - send_resolved: true
        title: '[{{ .Status | toUpper }}] {{ .GroupLabels.service }} service is in danger!'
        title_link: 'http://$DOMAIN/monitor/alerts'
        text: '{{ .CommonAnnotations.summary}}'
        api_url: '$SLACK_API_URL'
" | docker secret create alert_manager_config -
```

### Deployment

Before deploying the stack, please make sure to adjust resource `reservations` and `limits`.

```bash
docker network create --driver overlay monitor

# TODO: Remove GRAFANA_TAG

GRAFANA_TAG=4.5.0-beta1 \
    docker stack deploy -c docker-flow-monitor-full.yml monitor

open "http://$DOMAIN/monitor"

open "http://$DOMAIN/grafana/"
```

* Click *Add data source*
* Type *prometheus* as *Name*
* Select *Prometheus* as *Type*
* Type *http://monitor:9090/monitor* as *Url*

## exporters.yml

This stack deploys *HA Proxy Exporter*, *cAdvisor*, and *Node Exporter*.

### Requirements

* Network called `proxy` is created
* Network called `monitor` is created
* *Docker Flow Proxy* is running
* *Docker Flow Monitor* is running

```bash
docker network create --driver overlay proxy

docker stack deploy -c ../proxy/docker-flow-proxy.yml proxy

docker network create --driver overlay monitor

docker stack deploy -c docker-flow-monitor-proxy.yml monitor
```

### Deployment

```bash
docker stack deploy -c exporters.yml exporter

open "http://localhost/monitor"
```

