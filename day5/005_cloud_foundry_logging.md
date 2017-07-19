# Logging in Cloud Foundry

## Have ELK running

### 1 - Install Docker

```
sudo apt-get remove -y docker docker-engine docker.io
sudo apt-get install -y linux-image-extra-$(uname -r) linux-image-extra-virtual
sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install -y docker-ce
sudo docker run hello-world
```

### 2 - Install Docker Compose

```
sudo su -
curl -L https://github.com/docker/compose/releases/download/1.14.0/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose
exit
```

### 3 - Download and start ELK

```
git clone https://github.com/deviantony/docker-elk
cd docker-elk
sudo docker-compose up
```

### Allow external comunication

Add two rules to your Jumpbox security group to allow inboud traffic on port 5000 and 5601, CIDR `0.0.0.0/0`

Wait for Kibana to start and open your browser, and navigate to `http://YOUR_JUMPBOX_IP:5601`.

### Use the Firehose to Syslog application

```
cd ~/deployment
git clone https://github.com/cloudfoundry-community/firehose-to-syslog
cd firehose-to-syslog
git checkout master
cd firehose-to-syslog
rm manifest.yml
vim manifest.yml
```

Add the following text to the `manifest.yml` file, and edit the placeholders with the corresponding values:

```
applications: 
- name: firehose-to-syslog
  health-check-type: process
env:
  API_ENDPOINT: https://api.YOUR_CF_DOMAIN
  DEBUG: false
  DOPPLER_ENDPOINT: wss://doppler.YOUR_CF_DOMAIN:443
  EVENTS: LogMessage
  FIREHOSE_CLIENT_ID: influxdb-firehose-nozzle
  FIREHOSE_CLIENT_SECRET: password
  FIREHOSE_SUBSCRIPTION_ID: firehose-a
  SKIP_SSL_VALIDATION: true
  SYSLOG_ENDPOINT: YOUR_JUMPBOX_IP:5000
  LOG_FORMATTER_TYPE: json
```

Deploy the application:

```
cf push
```

Edit the `docker-elk/logstash/pipeline/logstash.conf`

Add the following filter:

```
filter {
        grok {
                match => { "message" => ".*(?<data>{.*})" }
        }
        json {
                source => "data"
        }
}
```


