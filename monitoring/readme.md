helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm dependency update metrics/
helm upgrade --install -n monitoring --create-namespace monitoring metrics/