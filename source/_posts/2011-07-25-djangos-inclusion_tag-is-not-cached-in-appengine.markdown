---
layout: post
title: "Django's inclusion_tag is not cached in AppEngine"
date: 2011-07-25 23:33
comments: true
categories: appengine
---
Since we starting using the massively convenient [GAE Mini Profiler](http://bjk5.com/post/6944602865/google-app-engine-mini-profiler), we were surprised to discover that we spend a significant amount of time reading files from disk. Here's a particulary extreme example:

<a href="/images/gae_file_read_is_slow.png">{% img /images/gae_file_read_is_slow.png This is just stupidly slow %}</a>

This was contrary to my understanding that App Engine tries to cache almost any code-related file read. After investigating, I found that App Engine does try to cache templates -- see the [source of template.py](http://code.google.com/p/googleappengine/source/browse/trunk/python/google/appengine/ext/webapp/template.py#76). But it turns out this only works when you render a template directly with <code>webapp.template.render</code>, and _not_ when you use Django's <code>inclusion_tag</code>.

To verify this, I put together a basic page with some repeated template use and used [opensnoop](http://www.brendangregg.com/DTrace/opensnoop_example.txt) (and after discovering that tool, I need to learn more about [DTrace](http://en.wikipedia.org/wiki/DTrace)) to observe changes to the filesystem. Here's the result when using <code>inclusion_tag</code>. You can see the template <code>simple_student_info.html</code> was loaded over and over again:
<pre>
$ sudo opensnoop -n Python
  UID    PID COMM          FD PATH
  501  27864 Python         7 .
  501  27864 Python         6 /Applications/GoogleAppEngineLauncher.app/Contents/Resources/GoogleAppEngine-default.bundle/Contents/Resources/google_appengine/google/appengine/../../VERSION
  501  27864 Python         6 /var/folders/TX/TXTcFXvEFKKsTqfua-9AGE+++TI/-Tmp-/request.7SyMKG.tmp
  501  27864 Python         6 /var/folders/TX/TXTcFXvEFKKsTqfua-9AGE+++TI/-Tmp-/request.7SyMKG.tmp
  501  27864 Python         7 /Users/dmnd/projects/khan/src/desmond/templatetest.html
  501  27864 Python         7 /Users/dmnd/projects/khan/src/desmond/simple_student_info.html
  501  27864 Python         7 /Users/dmnd/projects/khan/src/desmond/simple_student_info.html
  501  27864 Python         7 /Users/dmnd/projects/khan/src/desmond/simple_student_info.html
  501  27864 Python         7 /Users/dmnd/projects/khan/src/desmond/simple_student_info.html
  501  27864 Python         7 /Users/dmnd/projects/khan/src/desmond/simple_student_info.html
</pre>

When calling <code>webapp.template.render</code> directly, the template is only read once:
<pre>
$ sudo opensnoop -n Python
  UID    PID COMM          FD PATH
  501  27864 Python         6 /Applications/GoogleAppEngineLauncher.app/Contents/Resources/GoogleAppEngine-default.bundle/Contents/Resources/google_appengine/google/appengine/../../VERSION
  501  27864 Python         6 /var/folders/TX/TXTcFXvEFKKsTqfua-9AGE+++TI/-Tmp-/request.j-MxJs.tmp
  501  27864 Python         6 /var/folders/TX/TXTcFXvEFKKsTqfua-9AGE+++TI/-Tmp-/request.j-MxJs.tmp
  501  27864 Python         7 .
  501  27864 Python         7 /Users/dmnd/projects/khan/src/desmond/templatetest.html
  501  27864 Python         7 .
  501  27864 Python         7 /Users/dmnd/projects/khan/src/desmond/simple_student_info.html
  501  27864 Python         7 .
  501  27864 Python         7 .
  501  27864 Python         7 .
  501  27864 Python         7 .
</pre>

As we're already using <code>inclusion_tag</code> all over the place, [I added support for AppEngine's template caching to it](https://khanacademy.kilnhg.com/Repo/Website/Group/stable/File/template_cached.py?rev=d22e97b482ad) replacing all usages of <code>django.template.Library</code> with my own subclass. You can use it by including [this file](https://khanacademy.kilnhg.com/Repo/Website/Group/stable/File/template_cached.py?rev=d22e97b482ad) in your project, and changing the top of your <code>templatetags.py</code> files like so:

{% codeblock Without caching.py %}
from google.appengine.ext import webapp
register = webapp.template.create_template_register()
{% endcodeblock %}

{% codeblock With caching.py %}
import template_cached
register = template_cached.create_template_register()
{% endcodeblock %}

Caveats: we're still using Django 0.96, so there's a chance this only applies to that version of Django on AppEngine.
