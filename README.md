# Custom Monitoring Stack for OpenShift

Create config maps

# Create the prom secret
$ oc create cm prom --from-file=prometheus.yml

# Create the prom-alerts secret
$ oc create cm prom-alerts --from-file=alertmanager.yml

# Create the prometheus instance
$ oc process -f prometheus-standalone.yaml | oc apply -f -

# Create Grafana ConfigMap

$ oc create configmap grafana-config --from-file=defaults.ini -n custom-monitoring

# Apply Datasource Secret

oc apply -f custom-monitoring-grafana-datasource.yaml

# Deploy Grafana

$ oc process -f grafana.yaml | oc create -f -n custom-monitoring

# Connect Grafana to Prometheus

* Secret was already created when deploying Grafana Template

oc create clusterrolebinding grafana-cluster-monitoring-view \
  --clusterrole=cluster-monitoring-view \
  --serviceaccount=custom-monitoring:grafana

# Get SA Token

oc sa get-token grafana -n custom-monitoring

# Mount Secret to Grafana POD

oc set volume deployment grafana \
  --add \
  --name=customer-monitoring-grafana-datasource-volume \
  --type=secret \
  --secret-name=custom-monitoring-grafana-datasource \
  --mount-path=/usr/share/grafana/conf/provisioning/datasources \
  --namespace=custom-monitoring
