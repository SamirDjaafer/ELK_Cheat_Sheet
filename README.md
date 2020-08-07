# ELK Stack Cheat Sheet

### Install Java and NGINX on the instance 

1. Create ELK EC2 Instance with: 4GB RAM and 2 vCPU's (t2.medium), 20GB storage.
2. Create provision script with: 
3. `sudo apt-get update`, `sudo apt-get install -y openjdk-8-jdk`
4. `sudo apt-get -y install nginx`, `sudo systemctl enable nginx`

### Install Elasticsearch

1. `wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -`
2. `echo "deb https://artifacts.elastic.co/packages/7.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-7.x.list`
3. `sudo apt-get update`
4. `sudo apt-get install elasticsearch`

### Install Kibana

- Because we already added the Elastic package source, we can just install the remaining packages for the ELK stack

1. `sudo apt-get install kibana`
2. `sudo apt-get install -y apt-transport-https`

### Install Logstash

1. `sudo apt-get install logstash`

### Install Filebeat

1. `sudo apt-get install filebeat`

## Our Installations are complete! Lets configure the ELK stack

### Configure Elasticsearch

1. `sudo nano /etc/elasticsearch/elasticsearch.yml`
2. uncomment `cluster.name`, `node.name`, `http.port: 9200`, `network.host` -> make network.host as `localhost`, 
3. Save and Exit
4. `sudo systemctl start elasticsearch`
5. `sudo systemctl enable elasticsearch`
6. `sudo systemctl status elasticsearch`
7. We should see that it is active and running!
8. We can test the service by sending a HTTP request `curl -X GET "localhost:9200"`

### Configure Kibana

1. `sudo nano /etc/kibana/kibana.yml`
2. uncomment `server.port: 5601`, `server.host: "localhost"`
3. Save and Exit
4. `sudo systemctl enable kibana`, `sudo systemctl start kibana`
5. `sudo systemctl status kibana`, we should see it is active and running!

### Give Kibana a username and password

1. `sudo apt-get install -y apache2-utils`
2. `sudo htpasswd -c /etc/nginx/htpasswd.users kibanaadmin`
3. Give a password `kibanapassword`
4. `sudo nano /etc/nginx/htpasswd.users` We should see our hashed password

#### Connect NGINX to Kibana

1. `sudo nano /etc/nginx/sites-available/default`
2. Create the reverse proxy 
```
server {
    listen 80;

    server_name {Public IP of ELK stack machine};

    auth_basic "Restricted Access";
    auth_basic_user_file /etc/nginx/htpasswd.users;

    location / {
        proxy_pass http://localhost:5601;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```
3. Save and Exit!
4. `sudo systemctl start nginx`
5. `sudo systemctl status nginx`
6. Active and running!
7. Go to your IP adress for the ELK machine
8. Enter Username and Password and now we should have our Kibana UI.

#### Install Elasticsearch plugins

1. `sudo /usr/share/elasticsearch/bin/elasticsearch-plugin install ingest-geoip`
2. `sudo /usr/share/elasticsearch/bin/elasticsearch-plugin install ingest-user-agent`
3. `sudo systemctl restart elasticsearch`

## Next: On to the Application Machine

### Install Filebeats on Application Machine

1. `wget -qO - https://artifacts.elastic.co/GPG-KEY-elasticsearch | sudo apt-key add -`
2. `sudo apt-get install apt-transport-https`
3. `echo "deb https://artifacts.elastic.co/packages/6.x/apt stable main" | sudo tee -a /etc/apt/sources.list.d/elastic-6.x.list`
4. `sudo apt-get update && sudo apt-get install heartbeat`
5. `sudo update-rc.d filebeat defaults 95 10`

### Configure Filebeats to find ELK machine

1. `sudo nano /etc/filebeat/filebeat.yml`
2. Uncomment `host` in the Kibana section. Give the IP of the server running ELK.
3. Go to the outputs section, change the Elastic search hosts: url to the IP of the server running ELK.

- We can test if this was successful with `sudo filebeat test output`

### Enable and configure filebeat modules

1. `sudo filebeat modules list`
2. `sudo filebeat modules enable system`
3. Re-run the first command and check that the system module has been enabled!

### Enable filebeat

1. `sudo systemctl status filebeat`
2. `sudo systemctl enable filebeat`
3. `sudo filebeat setup`
4. `sudo systemctl start filebeat`
5. `sudo systemctl status filebeat`