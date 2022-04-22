# Install Prometheus and Grafana in EKS and Add Exporter

# Steps

1. Create cluster
   ```
   eksctl create cluster -f cluster1.yaml
   ```
2. Install Prometheus

   ```
   kubectl create namespace prometheus

   helm repo add prometheus https://prometheus-community.github.io/helm-charts

   helm install prometheus prometheus-community/prometheus \
    --namespace prometheus
   ```

   Akses Prometheus

   ```
   kubectl port-forward -n prometheus deploy/prometheus-server 9090
   ```

   Buka `localhost:9090`

3. Install Grafana

   ```
   kubectl create namespace grafana

   helm repo add grafana https://grafana.github.io/helm-charts

   helm install grafana grafana/grafana \
    --namespace grafana \
    --set persistence.enabled=true \
    --set adminPassword='password' \
    --values grafana.yaml \
    --set service.type=LoadBalancer
   ```

4. Add Dashboard \
   Akses Grafana dan login dengan username `admin` password `password`

   Cluster Monitoring Dashboard:

   - Klik tombol **+** di samping kiri dan pilih **import**
   - Masukkan dashboard id **3119**
   - Klik **Load**
   - Pilih **Prometheus** sebagai data source
   - Klik **Import**

   Pods Monitoring Dashboard:

   - Klik tombol **+** di samping kiri dan pilih **import**
   - Masukkan dashboard id **6417**
   - Klik **Load**
   - Masukkan **Kubernetes Pods Monitoring** sebagai nama dashboard
   - Pilih **Prometheus** sebagai data source
   - Klik **Import**

5. Create MongoDB pod and service

   Buat pod dan service mongodb

   ```
   kubectl apply -f mongo.yaml
   ```

6. Add MongoDB exporter
   ```
   helm install mongodb-exporter prometheus-community/prometheus-mongodb-exporter -f mongodb-exporter.yaml
   ```
7. Testing

   Menggunakan **kubectl port-forward**

   ```
   kubectl port-forward service/mongodb-exporter-prometheus-mongodb-exporter 9216
   ```

   Buka `localhost:9216`

   Buka `/targets` pada Prometheus dan akan telrihat podnya

   Buka dashboard Kubernetes pod pada Grafana dan akan terlihat podnya

8. Cleanup

   ```
   helm uninstall mongodb-exporter

   kubectl delete -f mongodb.yaml

   helm uninstall grafana -n grafana

   helm uninstall prometheus -n prometheus

   eksctl delete cluster -f cluster1.yaml
   ```

# Refrences

1. https://www.eksworkshop.com/intermediate/240_monitoring/
2. https://www.youtube.com/watch?v=mLPg49b33sA
3. https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus
4. https://github.com/grafana/helm-charts/tree/main/charts/grafana
5. https://github.com/prometheus-community/helm-charts/tree/main/charts/prometheus-mongodb-exporter
