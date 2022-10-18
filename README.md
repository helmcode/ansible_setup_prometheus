# Setup Prometheus and Grafana

Este role instala, configura y crea servicios de:

- Prometheus
- Node Exporter
- Alertmanager
- Grafana

Requisitos necesarios para usar este Role:

- Descargar los paquetes para Linux de prometheus, node exporter y alertmanager del [repositorio oficial](https://prometheus.io/download/).
- Subir los paquetes a un bucket de S3. Del cual indicaremos su URI como en este ejemplo:

```yaml
s3_uri: "s3://example/path/"
```

- Una vez subidos los paquetes indicaremos la versión a instalar según el paquete que hayamos descargado de la siguiente forma:

```yaml
list_prometheus_packages:
  prometheus: "prometheus-2.37.1.linux-amd64"
  node_exporter: "node_exporter-1.4.0.linux-amd64"
  alertmanager: "alertmanager-0.24.0.linux-amd64"
```

> Importante indicar el nombre de los paquetes SIN la extensión .tar.gz

- Para Grafana buscaremos en su [repositorio oficial](https://grafana.com/grafana/download?edition=oss) la versión deseada y copiaremos solamente el nombre del paquete en el map anterior de la siguiente forma:

```yaml
list_prometheus_packages:
  prometheus: "prometheus-2.37.1.linux-amd64"
  node_exporter: "node_exporter-1.4.0.linux-amd64"
  alertmanager: "alertmanager-0.24.0.linux-amd64"
  grafana: "grafana_9.2.1_amd64.deb"
```


### Prometheus Config:

La configuración principal gira entorno al scrapeo de prometheus sobre node exporter. Para indicar qué node exports debe scrapear prometheus se ha de usar este map de la siguiente forma:

```yaml
list_prometheus_node_jobs:
  example: 10.1.1.2
```

Para poder monitorizar grupos de autoscaling, es necesario que prometheus pueda escanear nuestra infra en AWS. Así que deberemos indicar la región y credenciales correspondientes en las variables:

```yaml
AWS_REGION: ""

AWS_ACCESS_KEY_ID: ""

AWS_SECRET_ACCESS_KEY: ""
```

> Se recomienda encarecidamente encriptar estas variables con Ansible Vault.

Tras esto ya podremos indicar la KEY de los *TAGS* que identifiquen a los nodos de nuestros grupos de autoscaling para poder ser monitorizados de la siguiente forma:

```yaml
list_autoscaling_node_jobs:
  - example_key_tag
```


*RULES:*

Se han creado rules específicas para monitorizar servicios básicos como:

- Consumo de CPU
- Consumo de RAM
- Llenado del Filesystem raíz
- Monitorización de procesos y demonios del sistema

Para ello por defecto se copian las siguientes rules pre-configuradas:

```yaml
list_rules_files:
  - node_exporter_recording_rules.yml
  - cpu_alert.yml
  - ram_alert.yml
  - filesystems_alert.yml
  - process_alert.yml
```

Para poder utilizar estas rules, deberemos indicarles qué hosts monitorizar. Siguiendo con el ejemplo utilizado en `list_prometheus_node_jobs`:

```yaml
list_monitored_hosts:
  - example
```

En caso de querer utilizar las rules con nodos de grupos de autoscaling, podemos utilizar el VALUE de la KEY que hayamos indicado en `list_autoscaling_node_jobs`:

```yaml
list_monitored_hosts:
  - example_value_tag
```

Para poder monitorizar procesos sobre todos estos servidores individuales o de grupos de autoscaling. Haremos el siguiente listado de los procesos o demonios a monitorizar. Por ejemplo:

```yaml
list_process_alerts:
  - prometheus
  - node_exporter
  - alertmanager
  - grafana-server
```

### Alertmanager Config:

Por defecto, se ha configurado Alertmanager para que envíe alertas a Slack. Para ello solo deberemos indicar el [webhook que generemos en Slack](https://api.slack.com/messaging/webhooks). Por ejemplo:

```yaml
slack_hook: 'https://hooks.slack.com/services/FFFFF/AAAAA/1234qwerty'
```

Además de indicarle el canal a donde queremos que nos lleguen las alertas. Por ejemplo:

```yaml
slack_channel: '#alarms'
```
