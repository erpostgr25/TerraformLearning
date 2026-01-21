# ELK Stack with Filebeat for Server Logging

### Overview
The ELK stack (Elasticsearch, Logstash, Kibana) with Filebeat provides a powerful solution for collecting, processing, storing, and visualizing server logs.

**Architecture:**
```
Server Logs → Filebeat → Logstash → Elasticsearch → Kibana
```

### Prerequisites
- Ubuntu/Debian server (or RHEL/CentOS)
- Minimum 4GB RAM
- Java 11 or later
- Root or sudo access

---

### 1.1 Installation Steps

#### Step 1: Install Java
```bash
sudo apt update
sudo apt install -y openjdk-11-jdk

# Verify installation
java -version
```

#### Step 2: Add Elastic Repository
```bash
# Import GPG key
wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -

# Add repository
echo "deb https://artifacts.elastic.co/packages/8.x/apt stable main" | sudo tee /etc/apt/sources.list.d/elastic-8.x.list

sudo apt update
```

#### Step 3: Install Elasticsearch
```bash
sudo apt install -y elasticsearch

# Configure Elasticsearch using the elasticsearch.yml in the current folder
sudo vim /etc/elasticsearch/elasticsearch.yml
```

```bash
# Start and enable Elasticsearch
sudo systemctl daemon-reload
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch

# Check status
sudo systemctl status elasticsearch

# Set password for elastic user
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

#### Step 4: Install Kibana
```bash
sudo apt install -y kibana

# Configure Kibana using kibana.yaml in the current folder
sudo vim /etc/kibana/kibana.yml
```

```bash
# Reset kibana_system password
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system

# Start Kibana
sudo systemctl enable kibana
sudo systemctl start kibana
sudo systemctl status kibana
```

#### Step 5: Install Logstash
```bash
sudo apt install -y logstash

# Create Logstash configuration using the filebeat-logstash.conf in the current folder
sudo vim /etc/logstash/conf.d/filebeat-logstash.conf
```

```bash
# Start Logstash
sudo systemctl enable logstash
sudo systemctl start logstash
sudo systemctl status logstash
```

#### Step 6: Install Filebeat on the current server or any other server
```bash
sudo apt install -y filebeat

# Configure Filebeat using the filebeat.yml in the current folder
sudo vim /etc/filebeat/filebeat.yml
```

```bash
# Enable Filebeat modules (optional)
sudo filebeat modules enable system nginx

# Test configuration
sudo filebeat test config
sudo filebeat test output

# Start Filebeat
sudo systemctl enable filebeat
sudo systemctl start filebeat
sudo systemctl status filebeat
```

---

### 1.2 Usage and Verification

#### Access Kibana
1. Open browser: `http://your-server-ip:5601`
2. Login with username `elastic` and password you set earlier

#### Create Index Pattern
1. Go to **Stack Management** → **Index Patterns**
2. Click **Create index pattern**
3. Enter pattern: `filebeat-*`
4. Select timestamp field: `@timestamp`
5. Click **Create index pattern**

#### View Logs
1. Go to **Discover** in the left menu
2. Select your `filebeat-*` index pattern
3. You should see logs flowing in real-time

#### Create Sample Visualizations
1. **Logs over time:**
   - Go to **Visualize Library** → **Create visualization**
   - Select **Line** chart
   - Choose `filebeat-*` index
   - Y-axis: Count
   - X-axis: Date Histogram on `@timestamp`

2. **Top Log Sources:**
   - Create **Pie chart**
   - Split slices by `host.name.keyword`

3. **Geographic Distribution:**
   - Create **Maps** visualization
   - Use `geoip.location` field

#### Create a Dashboard
1. Go to **Dashboard** → **Create dashboard**
2. Add your visualizations
3. Save the dashboard

---

### 1.3 Testing the Setup

```bash
# Generate some test logs
logger "Test syslog message from ELK demo"
sudo tail -f /var/log/syslog

# For nginx logs (if nginx is installed)
curl http://localhost

# Check if logs are reaching Elasticsearch
curl -X GET "localhost:9200/filebeat-*/_search?pretty" -u elastic:YOUR_PASSWORD
```