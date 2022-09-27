# Secure the Node



```
sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install certbot
sudo certbot certonly --standalone -d mynodesite.com

```



```
server {

listen 443 ssl;
server_name your.domain.com;


ssl_certificate /usr/local/nginx/conf/server.pem;
ssl_certificate_key /usr/local/nginx/conf/server.key;
ssl_session_cache shared:SSL:10m;

location / {

    proxy_pass http://localhost:8545; # my existing apache instance
    proxy_set_header Host $host;

    # re-write redirects to http as to https, example: /home
    proxy_redirect http:// https://;
}}
```
