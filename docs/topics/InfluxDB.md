tags: #kubernetes #influxDB

# InfluxDB

links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]

---
## InfluxDB v1

```bash
influx v1 shell --host http://127.0.0.1:8086
> show DATABASES
> use <db-name>
> show MEASUREMENTS
```

```yaml
version: '3.8'
services:
  influxdb:
    image: influxdb:1.8
    container_name: influxdb
    restart: unless-stopped
    volumes:
      - ./config.yaml:/etc/influxdb/influxdb.conf
      - ./influxdb_data:/var/lib/influxdb
    networks:
      - influxdb
    ports:
      - 8086:8086
      - 8089:8089
      - 8089:8089/udp
networks:
  influxdb:
    driver: bridge
```

```toml
[meta]
  dir = "/var/lib/influxdb/meta"

[data]
  dir = "/var/lib/influxdb/data"
  engine = "tsm1"
  wal-dir = "/var/lib/influxdb/wal"

[http]
	enabled = true
	bind-address = ":8086"
	auth-enabled = false

[[udp]]
  enabled = true
  bind-address = ":8089"
  database = "udp"
  batch-size = 1000
  batch-timeout = "1s"

[[graphite]]
  enabled = false
  database = "graphite"
  # retention-policy = ""
  bind-address = ":2003"
  protocol = "tcp"
  consistency-level = "one"

  # These next lines control how batching works. You should have this enabled
  # otherwise you could get dropped metrics or poor performance. Batching
  # will buffer points in memory if you have many coming in.

  # Flush if this many points get buffered
  batch-size = 5000

  # number of batches that may be pending in memory
  batch-pending = 10

  # Flush at least this often even if we haven't hit buffer limit
  batch-timeout = "1s"

  ### Each template line requires a template pattern.  It can have an optional
  ### filter before the template and separated by spaces.  It can also have optional extra
  ### tags following the template.  Multiple tags should be separated by commas and no spaces
  ### similar to the line protocol format.  There can be only one default template.
  templates = [
    # ...
  ]
```

### InfluxDB v2

```bash
# ping
influx ping --host http://<IP>:<PORT>
```

### Config with docker

```yaml
version: '3.8'
services:
  influxdb:
    image: influxdb:2.7-alpine
    container_name: influxdb2
    restart: unless-stopped
    volumes:
      - ./influxdb2:/var/lib/influxdb2:rw
    networks:
      - influxdb
    ports:
      - 8086:8086
    environment:
      - DOCKER_INFLUXDB_INIT_MODE=setup
      - DOCKER_INFLUXDB_INIT_USERNAME=username
      - DOCKER_INFLUXDB_INIT_PASSWORD=passwordpasswordpassword
      - DOCKER_INFLUXDB_INIT_ORG=myorg
      - DOCKER_INFLUXDB_INIT_BUCKET=bucket
      - DOCKER_INFLUXDB_INIT_ADMIN_TOKEN=mytoken
networks:
  influxdb:
    driver: bridge
```

---
links: [[300 Kubernetes MOC|Kubernetes MOC]] - [[000 Index|Index]]