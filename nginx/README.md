# Nginx Reverse Proxy

For Qlik Sense.

## Usage

1. `cd` to network folder
2. To start/setup: `docker-compose up -d`
3. To stop: `docker-compose down`
4. To restart: `docker-compose restart`

## Adding Qlik (Sense) sites

1. Copy `conf.d/sense.example.com.conf` to `conf.d/desired.domain.com.conf`
2. Edit all instances of `example.com` to desired domain
3. Restart Ningx: `docker-compose restart`

## Resources

[Certbot with Nginx](https://miki725.github.io/docker/crypto/2017/01/29/docker+nginx+letsencrypt.html)

[Nginx and Qlik Sense](https://github.com/braathen/qlik-misc-stuff/blob/master/nginx/nginx.conf)