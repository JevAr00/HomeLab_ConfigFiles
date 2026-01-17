# Important
1. If complete installation will be performed in the same node, use de all in one docker file
2. If panel and wings will be installed in different nodes, traefik configuration file won't work. Wings node will required to create its own SSL certificate to connect properly to the panel. Certbot documentation with cloudflare and autorenewal should be followed

## Config file
config.yml file should be created/modified in the following route /etc/pterodactyl/ (wings node), then, node required to be restarted. If installed in a separate node do not forget to change the SSL route, system default won't work due to folder restrictions

## Certbot autorenewal
1. Install the required dependencies
```bash
sudo apt update && sudo apt install certbot python3-certbot-dns-cloudflare -y
```

2. Create the file for certbot dns challenge 
```bash
mkdir -p ~/.secrets
cat <<EOF > ~/.secrets/cloudflare.ini
dns_cloudflare_api_token = TOKEN_HERE
EOF
chmod 600 ~/.secrets/cloudflare.ini
```

__NOTE__: For cloudflare token use DNS zone template ```Zone:DNS:Edit``` with your specific domain

3. Create the certs folder
```bash
sudo mkdir -p /etc/pterodactyl/certs
sudo chown -R YOURUSER:YOURUSER /etc/pterodactyl/certs
```

3. Create the script ```sync-certs.sh```
```bash
#!/bin/bash
SOURCE="/etc/letsencrypt/live/YOURDOMAIN.com"
DEST="/etc/pterodactyl/certs"

# copy the files following the symbolic links (-L)
cp -L ${SOURCE}/fullchain.pem ${DEST}/fullchain.pem
cp -L ${SOURCE}/privkey.pem ${DEST}/privkey.pem

# assign necessary file permissions
chmod 644 ${DEST}/*.pem

# Restart wings service
docker restart wings
```

4. Run the script
```bash
sudo certbot certonly --dns-cloudflare \
--dns-cloudflare-credentials ~/.secrets/cloudflare.ini \
-d "YOURDOMAIN.com" -d "*.YOURDOMAIN.com" \
--preferred-challenges dns \
--deploy-hook "/home/YOURUSER/scripts/sync-certs.sh"
```

__NOTE__: If wings is not running you will see an error in the console but certificate will be created and autorenewal will be set correctly

5. change certs path in config file
```yml
ssl:
    enabled: true
    cert: /etc/pterodactyl/certs/fullchain.pem
    key: /etc/pterodactyl/certs/privkey.pem
```

## Panel - DB Error - Missing Dependency
It is possible that the panel container wont work correctly

if the error is
```bash
Migrating and Seeding D.B
   INFO  Preparing database.  
   Creating migration table ...................................... 30.11ms DONE
   INFO  Loading stored database schemas.  
   database/schema/mysql-schema.sql .............................. 19.76ms FAIL
   
In Process.php line 275:

  The command "mysql  --user="${:LARAVEL_LOAD_USER}"
      --password="${:LARAVEL_LOAD_PASSWORD}" --host="${:LARAVEL_LOAD_HOST}"
      --port="${:LARAVEL_LOAD_PORT}" --database="${:LARAVEL_LOAD_DATABASE}" < "${:LARAVEL_LOAD_PATH}"" failed.  

  Exit Code: 127(Command not found)                                            

  Working directory: /app                                                      

  Output:                                                                      

  ================                                                             

  Error Output:                                                                

  ================                                                             

  sh: mysql: not found                                                         

Starting cron jobs.
Starting supervisord.
2026-01-11 02:14:00,839 CRIT Server 'unix_http_server' running without any HTTP authentication checking
```


It is required to execute the following command
```bash
docker exec -it pterodactyl_panel apk add --no-cache mariadb-client
```
Root Cause: Use of alpine container on the panel with missing dependencies

Command to create log in user 
```bash
docker compose run --rm panel php artisan p:user:make --email=admin@domain.com --username={yourusername} --name-first=admin --name-last=user --password={yourpassword} --admin=1 --no-password
```


