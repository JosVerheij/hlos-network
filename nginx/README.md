# Nginx Reverse Proxy

For Qlik Sense.

## Usage

1. `cd` to network folder
2. To start/setup: `docker-compose up -d`
3. To stop: `docker-compose down`
4. To restart: `docker-compose restart`

## Adding Qlik (Sense) sites

> Currently incomplete. Certificate generation needs to be added. 2 iterations are required to 1) first enable certificate generation, and 2) enable the Qlik Sense site.

1. Copy `conf.d/sense.example` to `conf.d/my.domain.com.conf`
2. Replace all instances of `[DOMAIN_NAME_HERE]` to desired domain (eg. `my.domain.com`)
3. Replace all instances of IP ADRESS... TODO...
3. Restart Ningx: `docker-compose restart`
4. 

## Resources

[Certbot with Nginx](https://miki725.github.io/docker/crypto/2017/01/29/docker+nginx+letsencrypt.html)

[Nginx and Qlik Sense](https://github.com/braathen/qlik-misc-stuff/blob/master/nginx/nginx.conf)

[Alpine Linux and Nginx](https://wiki.alpinelinux.org/wiki/Nginx_as_reverse_proxy_with_acme_(letsencrypt\)#Automatic_generation_of_certificates)