---
title: "Choose a Grafana dashboard (optional)"
sidebar_position: 8
sidebar_label: Dashboards
---

## Choose local or cloud Grafana

You have a choice of running Grafana locally, by including `grafana.yml`, or in the cloud, with `grafana-cloud.yml`.
If you choose Grafana Cloud, you **must** edit `prometheus/custom-prom.yml` and add your cloud remote write
credentials. Please see `prometheus/custom-prom.yml.sample` for the syntax of that addition.

`grafana-cloud.yml` runs a local Prometheus but no local Grafana, and enables adding custom Prometheus config items.
Its contents are merged with the pre-provisioned configuration.

If you want to add additional scrape targets, place these into `prometheus/conf.d`.

## Local Grafana dashboards

A baseline set of dashboards has been included.  
- [Metanull's Prysm Dashboard JSON](https://raw.githubusercontent.com/metanull-operator/eth2-grafana/master/eth2-grafana-dashboard-single-source-beacon_node.json)
- [Prysm Dashboard JSON](https://raw.githubusercontent.com/GuillaumeMiralles/prysm-grafana-dashboard/master/less_10_validators.json)
- [Prysm Dashboard JSON for more than 10 validators](https://raw.githubusercontent.com/GuillaumeMiralles/prysm-grafana-dashboard/master/more_10_validators.json)
- [Lighthouse Beacon Dashboard JSON](https://raw.githubusercontent.com/sigp/lighthouse-metrics/master/dashboards/Summary.json)
- [Lighthouse Validator Client Dashboard JSON](https://raw.githubusercontent.com/sigp/lighthouse-metrics/master/dashboards/ValidatorClient.json)
- [Lighthouse Yoldark34 Dashboard JSON](https://raw.githubusercontent.com/Yoldark34/lighthouse-staking-dashboard/main/Yoldark_ETH_staking_dashboard.json)
- [Nimbus Dashboard JSON](https://raw.githubusercontent.com/status-im/nimbus-eth2/master/grafana/beacon_nodes_Grafana_dashboard.json)
- [Teku Dashboard JSON](https://grafana.com/api/dashboards/12199/revisions/1/download)
- [Geth Dashboard JSON](https://gist.githubusercontent.com/karalabe/e7ca79abdec54755ceae09c08bd090cd/raw/3a400ab90f9402f2233280afd086cb9d6aac2111/dashboard.json)
- [Lodestar Dashboard JSON](https://raw.githubusercontent.com/ChainSafe/lodestar/stable/dashboards/lodestar_summary.json)
- [Reth Dashboard JSON](https://raw.githubusercontent.com/paradigmxyz/reth/main/etc/grafana/dashboards/overview.json)

You can additional dashboards from the Grafana repository, for example `11133` as a node exporter dashboard. You can
also import dashboards by their JSON, as you see fit.

## Connecting to local Grafana  

Connect to https://grafana.yourdomain.com/ (or http://YOURSERVERIP:3000/ if not using the reverse proxy), log in as
admin/admin, and set a new password.

> If you run Grafana over http without encryption, do not expose the Grafana port to the Internet. You can
> use [SSH tunneling](https://www.howtogeek.com/168145/how-to-use-ssh-tunneling/) to reach Grafana securely over the Internet.

In order to load other Dashboards, follow these instructions.

- Click on the + icon on the left, choose "Import".
- Copy/paste JSON code from the Raw github page of the Dashboard you chose - click anywhere inside the page, use Ctrl-A
to select all and Ctrl-C to copy
- Click "Load"
- If prompted for a data source choose the "prometheus" data source
- Click "Import".

## Alerting with Grafana

Grafana supports setting up alerts and sending notifications to email, Slack, Discord, PagerDuty, etc. Some alerts
are pre-previsioned.

To receive these alerts, use the alert bell icon on the left-hand side and set up contact points and notification
policies. The pre-provisioned alerts use a `severity` label. You could for example send `medium` severity alerts to
email, Discord or Telegram, and `critical` severity alerts to PagerDuty or OpsGenie.
