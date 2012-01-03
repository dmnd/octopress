---
layout: post
title: "Using nginx as a reverse proxy for speedy App Engine development"
date: 2011-12-05 18:24
comments: true
categories: appengine
---

*Context:* I recently posted about [using Apache as a reverse-proxy for Google App Engine development](/blog/2011/11/15/speed-up-your-app-engine-dev-server-with-an-apache-reverse-proxy). See that post for the motivation for monkeying with Real Web Servers when what you really want to be doing is developing your GAE app. The quick version is that you can reduce your local pageload time by a factor of 20 or more, depending how many static assets (images, javascript, stylesheets) your app has.

Proxying with Apache was a big improvement using the single-threaded, I/O blocking `dev_appserver.py` directly, but it was by no means perfect. I noticed that if I hadn't hit Apache for a while, it'd take roughly 10 seconds before my next request would be served. If you know what you're doing, it's probably possible to configure Apache not to do this. But I don't know what I'm doing, and Apache configuration isn't exactly friendly.

{% img right /images/nginx-logo.png Now that is a badass logo %}

Instead, I switched to using [nginx](http://wiki.nginx.org/Main), which is sometimes described as a reverse proxy first and webserver second. It's insanely fast at serving static files, uses little memory, and configuring it turned out to be easy. Also, who can resist that logo?

### How to set up nginx as a reverse proxy

First, install nginx. With homebrew on OS X this is just `brew install nginx`.

Next, we need to configure nginx to work as a reverse proxy. The following configuration did the trick for me. I put this file at `/usr/local/etc/nginx/kadev.conf`, as the `include` refers to a file `mime.types` in that directory.

Notice how [DRY](http://c2.com/cgi/wiki?DontRepeatYourself) this config is compared to the Apache equivalent. It's really easy to add extra `server` sections if you have multiple development servers.

{% codeblock /usr/local/etc/nginx/kadev.conf lang:nginx %}
worker_processes        1;

events {
    worker_connections  1024;
}

http {
    include             mime.types;
    default_type        application/octet-stream;
    sendfile            on;
    keepalive_timeout   65;

    server {
        listen          khanacademy.local:80;
        server_name     khanacademy.local;
        root            /Users/dmnd/Projects/khan/src/stable;

        location / {
            try_files   $uri   @proxy;
        }

        location @proxy {
            proxy_pass   http://127.0.0.1:8080;
        }
    }
}

{% endcodeblock %}

Line 19 is where the magic happens. It tells nginx to check first for a file matching the path for a request. If a file exists, nginx serves it directly. Otherwise, nginx forwards the request to the proxied server, in this case creaky old `dev_appserver.py`.

Finally, run nginx with `sudo nginx -c ~/path/to/config/file.conf`. `sudo` is needed to listen on port 80. If you're running on another port, you don't need it.

Now you should be able use your proxy to serve your static files quickly so you can get on with development.

### Debugging tips

If something doesn't work, here are some things to try.

* Check that your configuration is valid with `nginx -c ~/path/to/config/file.conf -t`.

* The above configuration tells nginx to listen on port 80, so if you have Apache running you will need to disable it first. On OS X, open System Preferences, select Sharing, then uncheck "Web Sharing". Or you can tell it to exit with `sudo apachectl -k stop`.

* If you installed with `brew install nginx`, logs will be stored at `/usr/local/Cellar/nginx/1.0.7/logs` by default. If something isn't working, `tail -f` the files in that directory.

If you needed to do anything else to get it running, please leave a comment to help the next person.
