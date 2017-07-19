# Monitoring your CF deployment

In this exercise we are going to deploy a Grafana dashboard that is going to be displaying the CF metrics. These metrics will be sent from CF to the InfluxDB with a Nozzle application deployed into Cloud Foundry itself.

## Change the size of your Jumpbox instance

1. Go to your EC2 console
1. Stop your Jumpbox instance (**WARNING**: do NOT click on Terminate. Just Stop.)
1. When the instance has stopped, change the size to `m4.large`
1. Start the instance again

## Install and configure Grafana access

### Install Grafana

```
wget https://s3-us-west-2.amazonaws.com/grafana-releases/release/grafana_4.2.0_amd64.deb
sudo apt-get install -y adduser libfontconfig
sudo dpkg -i grafana_4.2.0_amd64.deb
sudo service grafana-server start
```

### Allow communication from the outside world

Open your Jumpbox security group and add an inbound rule on port 3000 from CIDR `0.0.0.0/0`.

### Access Grafana

Open your browser, navigate to `http://YOUR_JUMPBOX_IP:3000`

Credentials are `admin/admin`

## Install and configure InfluxDB

### Install InfluxDB

```
curl -sL https://repos.influxdata.com/influxdb.key | sudo apt-key add -
source /etc/lsb-release
echo "deb https://repos.influxdata.com/${DISTRIB_ID,,} ${DISTRIB_CODENAME} stable" | sudo tee /etc/apt/sources.list.d/influxdb.list

sudo apt-get update && sudo apt-get install influxdb
```

### Allow communication from the outside world

Open your Jumpbox security group and add an inbound rule on port 8086 from CIDR `0.0.0.0/0`.

### Configure InfluxDB

```
influx

# You are going to be directed into the Influx CLI

create database cf;
exit
```

## Configure your Nozzle

### Open your Cloud Foundry deployment manifest

Add the following lines to the manifest, right into the UAA Clients section (`properties/uaa/clients`):

```
influxdb-firehose-nozzle:
  value:
    access-token-validity: 1209600
    authorized-grant-types: authorization_code,client_credentials,refresh_token
    override: true
    secret: password
    scope: openid,oauth.approvals,doppler.firehose
    authorities: oauth.login,doppler.firehose
    redirect-uri: “localhost”
```

Save and exit

### Redeploy Cloud Foundry

`bosh -n deploy`

## Install the Nozzle application

### Clone the application

```
git clone https://github.com/s-matyukevich/influxdb-firehose-nozzle
cd influxdb-firehose-nozzle
rm manifest.yml
touch manifest.yml
rm config//influxdb-firehose-nozzle.json
echo "{}" > config//influxdb-firehose-nozzle.json
```

Edit the `manifest.yml` file and paste this contect. Replace the placeholders with the appropiate values:

```
---
applications:
- name: nozzle
  buildpack: https://github.com/cloudfoundry/go-buildpack#v1.7.16
  env:
    NOZZLE_UAAURL: https://uaa.YOUR_CF_DOMAIN
    NOZZLE_CLIENT: influxdb-firehose-nozzle
    NOZZLE_CLIENT_SECRET: password
    NOZZLE_TRAFFICCONTROLLERURL: wss://doppler.YOUR_CF_DOMAIN:443
    NOZZLE_INFLUXDB_URL: http://YOUR_JUMPBOX_PUBLIC_IP:8086
    NOZZLE_INFLUXDB_DATABASE: cf 
    NOZZLE_DEPLOYMENT: cf
    NOZZLE_SSL_SKIPVERIFY: true
    NOZZLE_FLUSHDURATIONSECONDS: 15
    NOZZLE_FIREHOSESUBSCRIPTIONID: influx
```

Now, deploy the application:

```
cf push
```

### Check the metrics are flowing into InfluxDB

```
influx
show databases;
use cf;
show measurements;
select * from "gorouter.latency";
```

## Configure your dashboard

1. Login to Grafana and click on “Add data source”
2. Fill the datasource properties with the following settings:
 ```
        Name: metrics
        Type: InfluxDb
        Url: http://localhost:8086
        Database: cf
```
3. Click on "New dashboard"
4. In the upper panel click on settings icon and select “Templating”
5. Create 2 templates with the following settings:
```
        # Dashboard 1
        Name: metrics
        Type: Query
        Query: SHOW MEASUREMENTS
        Datasource: test

        # Dashboard 2
        Name: group_by
        Type: Custom
        Value: deployment,job,index,ip
```        
6. Click on “Graph”
7. Click on Panel Title and select “Edit”. Set the following parameters:
```        
        Title (General tab): $metrics
        Query (Metrics tab): SELECT mean("value") FROM /$metrics/ WHERE $timeFilter GROUP BY time($interval), $group_by 
```

> NOTE: You can click “Toggle Edit mode” to be able to paste the query instead of using query builder

> NOTE: On the `Display tab` you can play around with different settings. You can select `Draw Model = Bars`, or `Draw Mode = Lines + Null Value = connected` for better visibility.


