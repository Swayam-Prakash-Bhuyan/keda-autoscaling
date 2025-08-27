# Metrics Server
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/

helm upgrade --install metrics-server metrics-server/metrics-server -n kube-system

# Prometheus
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts

helm upgrade --install prometheus prometheus-community/kube-prometheus-stack -n monitoring --create-namespace

# KEDA
helm repo add kedacore https://kedacore.github.io/charts

helm upgrade --install keda kedacore/keda -n keda --create-namespace
