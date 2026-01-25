
## ELK Installation Guide

### Network Ports

| Service | Port | Protocol | Purpose |
|---------|------|----------|---------|
| Elasticsearch | 9200 | HTTPS | API access |
| Elasticsearch | 9300 | TCP | Inter-node communication |
| Kibana | 5601 | HTTP | Web interface |
| Logstash | 5044 | TCP | Beats input |
| Nginx | 80 | HTTP | Web server (demo) |

---

## Installation Steps

### Step 1: Update System

```bash
sudo apt update
sudo apt upgrade -y
```

### Step 2: Import Elastic GPG Key

Elastic signs all packages with a GPG key for security verification.

```bash
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | \
  sudo gpg --dearmor -o /usr/share/keyrings/elasticsearch-keyring.gpg
```

### Step 3: Add Elastic 9.x Repository

```bash
# Install transport package if needed
sudo apt install apt-transport-https

# Add Elastic repository
echo "deb [signed-by=/usr/share/keyrings/elasticsearch-keyring.gpg] https://artifacts.elastic.co/packages/9.x/apt stable main" | \
  sudo tee /etc/apt/sources.list.d/elastic-9.x.list

# Update package list
sudo apt update
```

---

### Step 4: Install Elasticsearch

```bash
sudo apt install -y elasticsearch
```

#### Configure Elasticsearch

Edit `/etc/elasticsearch/elasticsearch.yml`:
Update the settings as per **elasticsearch.yml** file present in the current folder


Edit file `/etc/elasticsearch/jvm.options`:
Uncomment the lines:
-Xms4g
-Xmx4g

and change it to:
-Xms1g
-Xmx1g

#### Start Elasticsearch

```bash
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
```

#### Set Elastic User Password

Elasticsearch 9.x enables security by default. Set the password:

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

**Save this password!** You'll need it for Kibana and Logstash.

#### Verify Elasticsearch is Running

```bash
# Check service status
sudo systemctl status elasticsearch

# Test API access
curl --cacert /etc/elasticsearch/certs/http_ca.crt \
  -u elastic:$ELASTIC_PASSWORD \
  https://localhost:9200
```

---

### Step 5: Install Kibana

```bash
sudo apt install -y kibana
```

#### Reset kibana_system Password

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system
```

#### Configure Kibana

Edit `/etc/kibana/kibana.yml` as per the kibana.yml in the current folder

#### Start Kibana

```bash
sudo systemctl enable kibana
sudo systemctl start kibana
```

#### Monitor Kibana Startup

```bash
sudo journalctl -u kibana -f
```

Wait for:
```
[info][status] Kibana is now available
```

This can take 1-2 minutes.

#### Access Kibana

Open your browser:
```
http://YOUR_SERVER_IP:5601
```

Login:
- **Username:** `elastic`
- **Password:** (the elastic password you set earlier)

---

### Step 6: Install Logstash

```bash
sudo apt install -y logstash
```

#### Create Logstash Pipeline Configuration

Create the configuration file: /etc/logstash/conf.d/logstash-nginx-pipeline.conf
Update the settings as per **logstash-nginx-pipeline.conf** file present in the current folder


Update the heap memory in this file:
/etc/logstash/jvm.options

Uncomment the lines:
-Xms1g
-Xmx1g

to:
-Xms512m
-Xmx512m

#### Start Logstash

```bash
sudo systemctl enable logstash
sudo systemctl start logstash
```

#### Verify Logstash is Running

```bash
# Check status
sudo systemctl status logstash

# Monitor logs
sudo journalctl -u logstash -f
```

Wait for:
```
[INFO ][logstash.agent] Successfully started Logstash API endpoint {:port=>9600}
```

```bash
# Verify Logstash is listening on port 5044
sudo netstat -tulpn | grep 5044
```

---

### Step 7: Install and Configure Nginx (Demo Application)

```bash
# Install Nginx
sudo apt install -y nginx

# Start Nginx
sudo systemctl start nginx
sudo systemctl enable nginx

# Verify Nginx is running
curl http://localhost
```

You should see the Nginx welcome page HTML.

---

### Step 8: Install and Configure Filebeat

```bash
sudo apt install -y filebeat
```

#### Configure Filebeat

Edit `/etc/filebeat/filebeat.yml`:
Update the settings as per **filebeat.yml** file present in the current folder

#### Give Filebeat Permission to Read Nginx Logs

```bash
chmod -R 644 /var/log/nginx/
ls -la /var/log/nginx/
```

#### Test Filebeat Configuration

```bash
# Test config file
sudo filebeat test config

# Test output connection to Logstash
sudo filebeat test output
```

Expected output:
```
Config OK
logstash: localhost:5044...
  connection...
    parse host... OK
    dns lookup... OK
    addresses: 127.0.0.1
    dial up... OK
  talk to server... OK
```

#### Start Filebeat

```bash
sudo systemctl enable filebeat
sudo systemctl start filebeat
sudo systemctl status filebeat
```

#### Monitor Filebeat

```bash
sudo journalctl -u filebeat -f
```

---

## Testing and Verification

### Step 1: Generate Nginx Traffic

```bash
# Generate successful requests
for i in {1..50}; do 
  curl http://localhost
  sleep 0.1
done

# Generate 404 errors
for i in {1..20}; do 
  curl http://localhost/missing-page-$i
  sleep 0.1
done

# Different user agents
curl -A "Mozilla/5.0 (Windows NT 10.0; Win64; x64) Chrome/96.0" http://localhost
curl -A "Mozilla/5.0 (iPhone; CPU iPhone OS 15_0)" http://localhost
curl -A "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7)" http://localhost
```

### Step 2: Verify Data Flow

#### Check Filebeat is Reading Logs

```bash
sudo tail -f /var/log/filebeat/filebeat
```

Look for lines indicating log harvesting.

#### Check Logstash is Receiving Data

```bash
sudo journalctl -u logstash -f
```

If you enabled `stdout` output, you'll see parsed log entries.

#### Check Elasticsearch Has Indices

```bash
curl -u elastic:$ELASTIC_PASSWORD \
  --cacert /etc/elasticsearch/certs/http_ca.crt \
  "https://localhost:9200/_cat/indices/nginx*?v"
```

Expected output:
```
health status index                    pri rep docs.count docs.deleted store.size pri.store.size
yellow open   nginx-logs-2026.01.23     1   1         70            0     89.2kb         89.2kb
```

#### Query Some Logs

```bash
curl -u elastic:$ELASTIC_PASSWORD \
  --cacert /etc/elasticsearch/certs/http_ca.crt \
  -X GET "https://localhost:9200/nginx-logs-*/_search?pretty&size=2"
```

You should see JSON output with your parsed nginx logs.

---

## Visualization in Kibana

### Step 1: Create a Data View

1. Open Kibana: `http://YOUR_SERVER_IP:5601`
2. Login with `elastic` user
3. Click **☰ Menu** → **Management** → **Stack Management**
4. Under **Kibana**, click **Data Views**
5. Click **Create data view**
6. Configure:
   - **Name:** `Nginx Logs`
   - **Index pattern:** `nginx-logs-*`
   - **Timestamp field:** `@timestamp`
7. Click **Save data view to Kibana**

### Step 2: Explore Logs in Discover

1. Click **☰ Menu** → **Discover**
2. Select **Nginx Logs** from the dropdown
3. Set time range to **Last 1 hour**
4. You should see your nginx logs!

### Step 3: Create Visualizations

#### Visualization 1: HTTP Status Codes

1. Click **☰ Menu** → **Dashboard** → **Create dashboard**
2. Click **Create visualization**
3. Select **Pie** chart
4. Configure:
   - **Slice by:** `status` (or `status.keyword`)
5. Click **Save and return**
6. Name: "HTTP Status Codes"

#### Visualization 2: Requests Over Time

1. Click **Create visualization**
2. Select **Line** chart
3. Vertical axis: Count (automatic)
4. Horizontal axis: `@timestamp`
5. Click **Save and return**
6. Name: "Requests Over Time"

#### Visualization 3: Top URLs

1. Click **Create visualization**
2. Select **Table**
3. Configure:
   - **Rows:** `request.keyword`
   - **Metrics:** Count
   - Show top 10
4. Click **Save and return**
5. Name: "Top Requested URLs"

#### Visualization 4: Request Methods

1. Click **Create visualization**
2. Select **Bar vertical**
3. Configure:
   - **Vertical axis:** Count
   - **Horizontal axis:** `method.keyword`
4. Click **Save and return**
5. Name: "HTTP Methods"

### Step 4: Save Dashboard

1. Arrange visualizations by dragging and resizing
2. Click **Save** (top-right)
3. Name: "Nginx Monitoring Dashboard"
4. Click **Save**

### Step 5: Set Auto-Refresh

1. Click the time picker (top-right)
2. Click **Refresh every**
3. Select **10 seconds**
4. Your dashboard now updates automatically!

---