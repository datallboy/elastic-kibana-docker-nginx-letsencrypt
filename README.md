# Elastic - Kibana - Docker - Nginx - Letsencrypt

---

### Introduction

Setup a Elastic + Kibana stack in seconds! Ready for public use with TLS enabled between nodes, and automatic SSL/TLS certificates + renewal with certbot and Nginx.

Docker-compose follows Elastic's official documentation for creating a Elastic Stack on Docker. More information can be found on their official site.
https://www.elastic.co/guide/en/elastic-stack-get-started/current/get-started-docker.html
https://www.elastic.co/guide/en/elasticsearch/reference/current/configuring-tls-docker.html

### Instructions

1. Create TLS certificates for encrypted communications between nodes
   `docker-compose -f create-certs.yml run --rm create_certs`

2. Edit nginx/config.conf and init-letsencrypt.sh and replace **DOMAINNAME.com** with your actual domain.

3. Execute the init-letsencrypt.sh script to generate LetsEncrypt certificates for nginx.
   ```
   chmod +x init-letsencrypt.sh
   sudo ./init-letsencrypt.sh
   ```
4. (Optional) In case there is an error starting the stack:
   a. Run :

   ```
   sysctl -w vm.max_map_count=262144
   ```

   b. To make the changes permanent insert the new entry into the /etc/sysctl.conf file with the required parameter:

   ```
   vm.max_map_count = 262144

   ```

   c. To take effect restart docker :

   ```
   sudo systemctl restart docker
   ```

5. Run the _elasticsearch-generate-passwords_ tool on es01 to generate passwords for all built-in users and kibana_system. Make note of these passwords.
   ```
   docker exec es01 /bin/bash -c "ln -s /usr/share/elasticsearch/config/certificates/ca/ca.crt /etc/pki/ca-trust/source/anchors/es-cluster-ca.crt"
   docker exec es01 /bin/bash -c "update-ca-trust"
   docker exec es01 /bin/bash -c "bin/elasticsearch-setup-passwords auto --batch --url https://es01:9200"
   ```
6. Edit .env file : Replace **ELASTIC_PASSWORD** with the randomly generated password for _kibana_system_. You'll also want to replace **KIBANA_ENCRYPTION_KEY** with a randomly generated (use your own), 32 character alphanumeric value. This is used for encrypting API keys for Elastic Agent fleets.

7. Restart your stack, and you should have a fully working elastic stack with HTTPS enabled!
   ```
   docker-compose stop
   docker-compose up -d
   ```
8. To login to Kibana the username is elastic and your password is the value of elastic (the one generated in step 6)
