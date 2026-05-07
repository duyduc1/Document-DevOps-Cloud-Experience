# Monitor with Grafana

### Cài đặt Node Exporter trên các máy chủ

- Node Exporter thu thập metric hệ thống (CPU, RAM, disk, v.v.) từ các máy chủ. Cài đặt trên cả 3 máy chủ (10.1.0.175, 10.1.0.172, 10.1.0.176).

1. Cài đặt docker (đã có sẵn cách setup cài đặt docker tự động hoá)

2. Chạy Node Exporter bằng Docker

- Trên mỗi máy chủ, tạo một container Node Exporter:

``` bash
docker run -d \
  --name node-exporter \
  -p 9100:9100 \
  --restart always \
  prom/node-exporter:latest
```

3. Cấu hình Prometheus và Grafana trên máy chủ chính (10.1.0.175)

- Prometheus và Grafana sẽ được triển khai bằng Docker Compose trên 10.1.0.175.

``` bash
mkdir -p ~/monitoring
cd ~/monitoring
nano prometheus.yaml

# Nội dụng trong prometheus.yml	
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['10.1.0.175:9090']  # Prometheus tự giám sát

  - job_name: 'node-exporter'
    static_configs:
      - targets:
          - '10.1.0.175:9100'  # Node Exporter trên máy chủ chính
          - '10.1.0.172:9100'  # Node Exporter trên máy chủ 2
          - '10.1.0.176:9100'  # Node Exporter trên máy chủ 3
```

``` yaml
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
      - prometheus_data:/prometheus
    ports:
      - "9090:9090"
    restart: always
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    volumes:
      - grafana_data:/var/lib/grafana
    ports:
      - "3000:3000"
    restart: always
    networks:
      - monitoring

volumes:
  prometheus_data:
  grafana_data:

networks:
  monitoring:
    driver: bridge
```

- chạy file yaml

``` bash
docker-compose up -d --buid
docker ps
```

- Truy cập:
		
- Prometheus: http://10.1.0.175:9090
		
- Grafana: http://10.1.0.175:3000 (Tài khoản mặc định: admin/admin, bạn sẽ được yêu cầu đổi mật khẩu).

4. Cấu hình Grafana
			
* Đăng nhập vào Grafana tại http://10.1.0.175:3000.

* Thêm nguồn dữ liệu (Data Source):

* Vào menu Connections > Data Sources > Add data source.

* Chọn Prometheus.

* Đặt URL: http://10.1.0.175:9090 (dùng tên service trong Docker Compose).

* Nhấn Save & Test.

* Tạo Dashboard:

* Vào Create > Import.

* Sử dụng ID dashboard phổ biến cho Node Exporter, ví dụ: 1860 (Node Exporter Full).

* Chọn nguồn dữ liệu Prometheus vừa thêm.

* Lưu dashboard để xem metric từ 3 máy chủ.
