## Introduction

If You like to start developing sites using Vue Storefront, probably You need to start with the [Installation guide](https://github.com/DivanteLtd/vue-storefront/blob/master/doc/Installing%20on%20Linux%20and%20MacOS.md). For the development purposes You'll probably be using `yarn install` / `npm run installer` sequence which will setup Vue Storefront locally using the automated installer and prepared Docker images for having Elastic Search and Redis support.

Development mode means You're using node.js based server as HTTP service and running the app on the `3000` TCP port. As it's great for local testing it's **not recommended** to use installer and direct user access to node.js in production configurations.

## Production setup - bare VPS

To run Vue Storefront in the production mode without Docker/Kubernetes You'll need the Virtual Private Server with `root` access (for the setup purposes). We'll assume that You're using `Debian GNU Linux` in the following steps.

Assumptions for the rest of this tutorial:
- You're having root access to Debian Linux machine 
- That's all ;)

### Prerequisites

### Nginx Setup

Here is the complete `/etc/nginx/sites-enabled/prod.vuestorefront.io` file - just divided into sections to comment each specific directive separately.

```
server {
	listen 80;
	server_name prod.vuestorefront.io; 
	return 301 https://prod.vuestorefront.io$request_uri;
}
```

This section runs the standard http://prod.vuestorefront.io and creates a wildcard redirect from http://prod.vuestorefront.io/* -> https://prod.vuestorefront.io/. SSL secured connection is a must for run PWA and use service-workers.

```
server {
	listen 443 ssl;
	server_name prod.vuestorefront.io http2;

	ssl on;
```

We're using `http2` but it's not required. This section is for setting up the SSL secured virtual host of Vue Storefront frontend.

```

	ssl_certificate /etc/nginx/ssl/prod.vuestorefront.io.chained.crt;
	ssl_certificate_key /etc/nginx/ssl/prod.vuestorefront.io.key;

```

We assume that the certificate related files are stored in the `/etc/nginx/ssl/`. Please point it to Your certificate files.

```

	ssl_protocols TLSv1 TLSv1.1 TLSv1.2 TLSv1.3;
	ssl_prefer_server_ciphers on;	
	ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES256-SHA:DHE-RSA-AES256-SHA;
	ssl_ecdh_curve secp384r1;
	ssl_session_timeout 10m;
	ssl_session_cache shared:SSL:10m;
	ssl_session_tickets off;
	ssl_stapling on;
	ssl_stapling_verify on;
	resolver 8.8.8.8 8.8.4.4 valid=300s;
	resolver_timeout 5s; 

	ssl_dhparam /etc/nginx/ssl/dhparam.pem;

	add_header Strict-Transport-Security "max-age=31536000" always;
	add_header X-Frame-Options DENY;
	add_header X-Content-Type-Options nosniff;
	add_header X-XSS-Protection "1; mode=block";
	add_header X-Robots-Tag none; 
```

Here we go with the SSL settings - based on our best experiences from the past ;) Please read details in the [nginx documentation if You like](http://nginx.org/en/docs/http/configuring_https_servers.html) ;)

``` 
	gzip on;
	gzip_proxied any;
	gzip_types
		text/css
		text/javascript
		text/xml
		application/javascript
		application/json
		text/json
		text/html;
	}
```

Vue Storefront SSR responses contain the full markup + JSON objects included for speed-up the first page view. Unfortunatelly - among with the significant JS bundle sizes - it can generate significat network load. We're optimizing it with using gzip compression server side.

```

	location / {
		proxy_pass http://localhost:3000/;
	}
```

We're using [`proxy_pass`](http://nginx.org/en/docs/http/ngx_http_proxy_module.html) from the `ngx_http_proxy_module` to pull the content from the Vue Storefront nodejs server. Site will be available under https://prod.vuestorefront.io/
 
```
	location /assets/ {
		proxy_pass http://localhost:3000/assets/;
	}
```
The same module is used for providing user with the static assets. Assets will be available under: https://prod.vuestorefront.io/assets

```
	location /api/ {
		proxy_pass http://localhost:8080/api/;
	}
```

The next proxy section is used for serving the API. It's a proxy to [`vue-storefront-api`](https://github.com/DivanteLtd/vue-storefront-api) app. running on `8080` port (default config). API will be available under: https://prod.vuestorefront.io/api

```
	location /img/ {
		proxy_pass http://localhost:8080/img/;
	}
}

```
The last proxy  is used for serving the product images. It's a proxy to [`vue-storefront-api`](https://github.com/DivanteLtd/vue-storefront-api) app. running on `8080` port (default config). The `/img` endpoint. Images will be available under: https://prod.vuestorefront.io/img


### Vue Storefront Setup

### Usefull database commands





## Production setup - using Docker / Kubernetes

To be prepared.