# Introduction

`init-letsencrypt.sh` fetches and ensures the renewal of a Let’s
Encrypt certificate for one or multiple domains in a docker-compose
setup with nginx.
This is useful when you need to set up nginx as a reverse proxy for an
application.

_Asteroid-belt, July,2021 | written by Mohsen K. Amini_

## Installation
1. [Install docker-compose](https://docs.docker.com/compose/install/#install-compose).

2. Clone this repository and `cd nginx/ssl/nginx-certbot/`

3. Modify configuration:
- Add domains and email addresses to init-letsencrypt.sh (Replace `www.ssl.mohsenkamini.ir` with your own domain)
- Replace all occurrences of www.ssl.mohsenkamini.ir with primary domain (the first one you added to init-letsencrypt.sh) in data/nginx/app.conf
~~~
server_name www.ssl.mohsenkamini.ir;
.
.
.
server_name www.ssl.mohsenkamini.ir;
    ssl_certificate /etc/letsencrypt/live/www.ssl.mohsenkamini.ir/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/www.ssl.mohsenkamini.ir/privkey.pem;
~~~
4. or you could Add Your own configuration data to ./data/nginx/app.conf file

5. Add your website data under ./data/nginx/web/

6. Run the init script:

        ./init-letsencrypt.sh
        press `y` when you're prompted if you wanna replace the existing data.

7. Run the server:

        docker-compose up


