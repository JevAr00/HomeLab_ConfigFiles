# Important
1. If complete installation will be performed in the same node, use de all in one docker file
2. If panel and wings intallation will be installed in different nodes, traefik configuration file won't work. Wings node will required to create its own SSL certificate to connect properly to the panel. Certbot documentation with cloudflare and autorenewal should be followed

## Config file
config.yml file should be created/modified in the following route /etc/pterodactyl/ (wings node), then, node required to be restarted. If installed in a separate node do not forget to change the SSL route, system default won't work due to folder restrictions

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


