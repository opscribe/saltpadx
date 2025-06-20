# Make nginx accessible from internet

Now that your nginx configuration works locally, let's make it accessible from the internet.

## Configure ssl

If you want to access saltpad and your salt-api over the internet, you're likely to want to configure SSL on nginx.

Your configuration should currently looks like:

```Nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server ipv6only=on;

    # Saltpad specific

    server_name SALTPAD.YOURDNS;

    location / {
    	proxy_pass http://localhost:8000;
    }
}
```

Mozilla has a very nice [ssl generator website](https://mozilla.github.io/server-side-tls/ssl-config-generator/) that could helps tweak your nginx configuration, but here is a most secure nginx configuration for reference:

```Nginx
server {
    listen 80 default_server;
    listen [::]:80 default_server;

    # Redirect all HTTP requests to HTTPS with a 301 Moved Permanently response.
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;

    # certs sent to the client in SERVER HELLO are concatenated in ssl_certificate
    ssl_certificate /path/to/signed_cert_plus_intermediates;
    ssl_certificate_key /path/to/private_key;
    ssl_session_timeout 1d;
    ssl_session_cache shared:SSL:50m;
    ssl_session_tickets off;

    # Diffie-Hellman parameter for DHE ciphersuites, recommended 2048 bits
    ssl_dhparam /path/to/dhparam.pem;

    # modern configuration. tweak to your needs.
    ssl_protocols TLSv1.2;
    ssl_ciphers 'ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-GCM-SHA384:ECDHE-ECDSA-CHACHA20-POLY1305:ECDHE-RSA-CHACHA20-POLY1305:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-RSA-AES128-GCM-SHA256:ECDHE-ECDSA-AES256-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-ECDSA-AES128-SHA256:ECDHE-RSA-AES128-SHA256';
    ssl_prefer_server_ciphers on;

    # HSTS (ngx_http_headers_module is required) (15768000 seconds = 6 months)
    add_header Strict-Transport-Security max-age=15768000;

    # OCSP Stapling ---
    # fetch OCSP records from URL in ssl_certificate and cache them
    ssl_stapling on;
    ssl_stapling_verify on;

    ## verify chain of trust of OCSP response using Root CA and Intermediate certs
    ssl_trusted_certificate /path/to/root_CA_cert_plus_intermediates;

    resolver <IP DNS resolver>;

    # Saltpad specific
    
    server_name SALTPAD.YOURDNS;

    location / {
    	proxy_pass http://localhost:8000;
    }
}
```

You will need a ssl certificate for activating ssl, you can either buy one, generate one or use [let's encrypt](https://letsencrypt.org/) for generating one.

From your salt-master, let's check that the configuration works correctly.

If you have enabled ssl, replace `http://localhost/` by `https://SALTPAD.YOURDNS/` in below commands.

Check that nginx proxy saltpad correclty:

```
curl http://localhost/saltpad/
```

Expected output:

```
<!doctype html>
<html lang="en" data-framework="react">
    <head>
        <meta charset="utf-8">
        <title>SaltPad</title>
    <link href="/static/styles.css" rel="stylesheet"></head>
    <body>
        <div id="app"></div>
    <script src="/static/vendors.js"></script><script src="/static/app.js"></script></body>
</html>
```

Check that nginx proxy saltpad config file correctly:

```
curl http://localhost/static/settings.json
```

The output should match the content of the `settings.json` file you deployed earlier.

## Saltpad configuration

Now that your salt-api instance is accessible from internet (don't forget to open the port and whitelist your IPs if necessary), we need to tweak the saltpad configuration a little.

You will need to change `API_URL` to match the DNS of your salt-api server (in our example it was `SALTPAD.YOURDNS`).

If you enabled ssl, you will also need to set `SECURE_HTTP` to `true` or saltpad will try to connect over a non encrypted connection.

You don't need to reload the webserver, just save the `settings.json` file and go at `http://SALTPAD.YOURDNS/` or `https://SALTPAD.YOURNDS/` to access Saltpad and start mastering your Saltstack environment.
