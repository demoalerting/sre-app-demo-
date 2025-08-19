# sre-app-demo-
Informacije vezane za deployment demo aplikacije i scrape metrike
Sav deployment je uradzen rootless na RHEL distribucij na sljedeci nacin 
VM1
podman pod create --name app -p 8080:80 -p 9113:9113 -p 9100:9100
podman run -d --name demo-app --pod app --replace \ quay.io/openshifttest/prometheus-example-app:multiarch
podman run -d --name nginx --pod app --replace \
  -v /home/azureuser/appvm/nginx/nginx.conf:/etc/nginx/nginx.conf:ro,Z \
  docker.io/library/nginx:latest
  podman run -d --name nginx-exporter --pod app --replace \
  docker.io/nginx/nginx-prometheus-exporter:v1.8.1 \
  -nginx.scrape-uri http://127.0.0.1/nginx_status
podman run -d --name minio --pod app --replace \
  -e MINIO_ROOT_USER=minioadmin -e MINIO_ROOT_PASSWORD=minioadmin \
  -e MINIO_PROMETHEUS_AUTH_TYPE=public \
  -e MINIO_BROWSER_REDIRECT_URL=http://10.0.0.4:8080/minio/console \
  -e MINIO_SERVER_URL=http://10.0.0.4:8080 \
  -v /home/azureuser/minio-data:/data:Z \
  docker.io/minio/minio:latest server /data --console-address ":9001"

sudo systemctl enable --now firewalld
sudo firewall-cmd --zone=public --permanent --add-port=8080/tcp
sudo firewall-cmd --zone=public --permanent --add-port=9113/tcp
sudo firewall-cmd --zone=public --permanent --add-port=9100/tcp
sudo firewall-cmd --reload

VM2

podman pod create --name monitoring -p 9090:9090 -p 9093:9093 -p 3000:3000
podman run -d --name prometheus --pod monitoring --replace \
  -v /home/azureuser/monitoring/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro,Z \
  -v /home/azureuser/monitoring/prometheus/rules.yml:/etc/prometheus/rules.yml:ro,Z \
  docker.io/prom/prometheus:latest \
  --config.file=/etc/prometheus/prometheus.yml
podman run -d --name prometheus --pod monitoring --replace \
  -v /home/azureuser/monitoring/prometheus/prometheus.yml:/etc/prometheus/prometheus.yml:ro,Z \
  -v /home/azureuser/monitoring/prometheus/rules.yml:/etc/prometheus/rules.yml:ro,Z \
  docker.io/prom/prometheus:latest \
  --config.file=/etc/prometheus/prometheus.yml
podman run -d --name grafana --pod monitoring --replace \
  -e GF_SECURITY_ADMIN_PASSWORD=admin \
  docker.io/grafana/grafana:latest


sudo systemctl enable --now firewalld
sudo firewall-cmd --zone=public --permanent --add-port=3000/tcp
sudo firewall-cmd --zone=public --permanent --add-port=9090/tcp
sudo firewall-cmd --zone=public --permanent --add-port=9093/tcp
sudo firewall-cmd --reload

Grafana DashBoards

Node Exporter Full: 1860

NGINX Exporter: 12708

MinIO Server Metrics: 13502

<!-- !!!!JASNO MI JE BILO DA ZADATAK TRAZI DA SE URADI DEPLOY I SCRAPE 3 CUSTOM METRIKE MEDZUTIM TO NIJE URADZENO!!!!
Ima jedan dashboard koji prikazuje kao 3 custom metrike ali isti je bukvalno samo copy paste -->


