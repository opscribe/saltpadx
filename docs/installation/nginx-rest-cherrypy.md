# Proxy saltpad with nginx

## Install nginx

nginx is packaged in most OSes, install it on your salt-master with your favorite package manager:

```bash
apt-get install nginx
```

## Configure nginx

Once you have a configurated salt-api and saltpad to be served by rest_cherrypy, we can now configure nginx to proxy requests to salt-api to make it available from the internet.

If you have configured salt-api to listen on `http://localhost:8000`, a very basic nginx configuration that will serve saltpad is:

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

You can put this configuration and replace the content of the file "/etc/nginx/sites-enabled/default" or ask your system administrator to configure Nginx.

Now reload the webserver:

```bash
sudo /etc/init.d/nginx reload
```

__Warning, the previous example configurations ARE NOT SUITABLE for production, you will need to configure ssl for production environment so don't open the webserver to the whole web yet, we will configure ssl later.__

You will need to change the root path if you deployed saltpad elsewhere and the server_name to point to the DNS name of your salt-master.

## Check configuration

We can now check that everything should works correctly.

Check that nginx proxy saltpad correctly:

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

Now it's unlickely that you have a browser on the salt-master so congratulations you now have a saltpad installation only accessible from your salt-master machine.

We will change that and allow you to access saltpad from your browser in the [next part](nginx-rest-cherrypy-ssl.md).
