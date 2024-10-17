```bash
mkdir monitoring
cd monitoring
```

monitoring-stack.yml
```yaml
version: '3.8'
networks:
  monitoring:
    driver: overlay

services:
  prometheus:
    image: prom/prometheus:latest
    command:
      - '--config.file=/etc/prometheus/prometheus.yml'
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    networks:
      - monitoring
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager

  node-exporter:
    image: prom/node-exporter:latest
    deploy:
      mode: global
    ports:
      - "9100:9100"
    networks:
      - monitoring

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    networks:
      - monitoring
    volumes:
      - grafana-storage:/var/lib/grafana
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.role == manager

volumes:
  grafana-storage:
```

prometheus.yml
```yaml
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']

  - job_name: 'node_exporter'
    static_configs:
      - targets:
          - '192.168.11.11:9100'
          - '192.168.11.12:9100'
          - '192.168.11.13:9100'

```

```bash
docker stack deploy -c monitoring-stack.yml monitoring
```