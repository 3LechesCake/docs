# Secure the Node



<pre><code>sudo add-apt-repository ppa:certbot/certbot
sudo apt-get update
sudo apt-get install certbot
<strong>sudo certbot certonly --standalone -d rpc-test.tresleches.finance
</strong></code></pre>

Install ngix [https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04](https://www.digitalocean.com/community/tutorials/how-to-install-nginx-on-ubuntu-20-04)



```
server {

listen 443 ssl;



ssl_certificate /etc/letsencrypt/live/rpc-test.tresleches.finance/fullchain.pem;
ssl_certificate_key /etc/letsencrypt/live/rpc-test.tresleches.finance/privkey.pem;
ssl_session_cache shared:SSL:10m;

location / {

    proxy_pass http://rpc-test.tresleches.finance:8545; # my existing apache instance
    proxy_set_header Host $host;

    # re-write redirects to http as to https, example: /home
    proxy_redirect http:// https://;
}}
```
