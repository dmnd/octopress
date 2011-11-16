---
layout: post
title: "Speed up your App Engine dev server with an Apache reverse proxy"
date: 2011-11-15 21:59
comments: true
categories: appengine
---

When using the Google App Engine SDK in development mode, you have probably noticed that `dev_appserver.py` is incredibly slow. This is because all requests -- even requests for static files like javascript, stylesheets or images -- are handled sequentially by a single thread. Take a look at the timeline below. Does the left side look familiar?

At [Khan Academy](http://www.khanacademy.org), a single pageload in development mode might end up requesting as many as 100 different resources. This takes `dev_appserver.py` more than 35 seconds to serve, which sucks. Coupled with [Webkit's annoyingly aggressive cache](https://bugs.webkit.org/show_bug.cgi?id=30862), this can really kill your [productivity](http://xkcd.com/303/).

{% img center /images/gae-proxy-before-after.png Chrome timelines before and after serving static assets from Apache %}

It's possible to get a big speed boost by setting up Apache as a [reverse proxy](http://en.wikipedia.org/wiki/Reverse_proxy) in front of your dev server. All requests for static assets will be handled by Apache, which is blazing fast compared to `dev_appserver.py`. The 35 second request above is fulfilled in only 2 seconds, with most of the static files loading in parallel (see the right side).

### How to set this up

First, enable Virtual Hosts in Apache. Edit `/etc/apache2/httpd.conf` and go to line 623. Uncomment the line for vhosts, so it looks like the following.

{% codeblock /etc/apache2/httpd.conf lang:apache line:622 %}
# Virtual hosts
Include /private/etc/apache2/extra/httpd-vhosts.conf
{% endcodeblock %}

Next, open `/etc/apache2/extra/httpd-vhosts.conf` and insert something like the following. Fellow KA devs shouldn't have to edit much, but if you're working on a different app you will obviously have to change the static directories. Look in `app.yaml` to see the full list of statically served paths.

{% codeblock /etc/apache2/extra/httpd-vhosts.conf lang:apache %}
<VirtualHost *:80>
    ServerName khanacademy.local

    # don't proxy these paths. Instead, serve them directly from apache
    ProxyPass /javascript !
    Alias /javascript "/Users/dmnd/Projects/khan/src/stable/javascript"

    ProxyPass /stylesheets !
    Alias /stylesheets "/Users/dmnd/Projects/khan/src/stable/stylesheets"

    ProxyPass /images !
    Alias /images "/Users/dmnd/Projects/khan/src/stable/images"

    ProxyPass /gae_bingo/static !
    Alias /gae_bingo/static "/Users/dmnd/Projects/khan/src/stable/gae_bingo/static"

    ProxyPass /gae_mini_profiler/static !
    Alias /gae_mini_profiler/static "/Users/dmnd/Projects/khan/src/stable/gae_mini_profiler/static"

    ProxyPass /khan-exercises !
    Alias /khan-exercises "/Users/dmnd/Projects/khan/src/stable/khan-exercises"

    # everything else gets proxied through to the dev server
    ProxyPass / http://localhost:8080/

    # let apache rewrite URLs in response headers
    ProxyPassReverse / http://localhost:8080/

    # give apache some permissions to the src directory so it can serve static files
    <Directory "/Users/dmnd/Projects/khan/src">
        Options Indexes FollowSymLinks Includes ExecCGI
        AllowOverride All
        Order allow,deny
        Allow from all
        AddDefaultCharset utf-8
    </Directory>
</VirtualHost>
{% endcodeblock %}

Finally, map the `ServerName` you picked to localhost by editing your `/etc/hosts` file. See line 12 below.

{% codeblock /etc/hosts %}
##
# Host Database
#
# localhost is used to configure the loopback interface
# when the system is booting.  Do not change this entry.
##
127.0.0.1   localhost
255.255.255.255 broadcasthost
::1             localhost
fe80::1%lo0 localhost
# Easy access to app engine dev server
127.0.0.1   khanacademy.local
{% endcodeblock %}

This allows you to access your dev server via something other than localhost, which is needed for the virtual host to work. If you don't already have `--address=0.0.0.0` as a parameter to `dev_appserver.py` [you will need to add this](http://code.google.com/appengine/docs/python/tools/devserver.html#Command_Line_Arguments).

Also, Apache needs to be enabled - the easiest way to do this is to go to Sharing under System Preferences and check the "Web Sharing" item. If you already have it enabled, you may need to clear and check it again to force a restart. If it doesn't start, check your config syntax with `apachectl -St`.

This setup should work on OS X Lion. Small changes might be needed for other OSes. If you had to tweak anything, mention it in the comments.

