---
layout: post
title: "Caffeinated keeps your PC awake"
date: 2011-09-19 23:21
comments: true
categories: caffeinated
---
{% img right /images/caffeinated/menu.png %}
I use an excellent if simple program called [Caffeine](http://lightheadsw.com/caffeine/) on OS X. Its only purpose is to temporarily prevent your computer from automically sleeping, or displaying the screensaver. A similar program called [Insomnia][insomnia] is available for Windows, but I dislike its UI.

So, I built [Caffeinated](/caffeinated). It's a port of Caffeine that runs on Windows. The UI is straight-up lifted from Caffeine, and the entire program is pretty much just a usable wrapper around the [SetThreadExecutionState][msdn] function from the Windows API.

{% img center /images/caffeinated/settings.png %}

Despite its simplicity, I find it useful, and maybe you will too. [Read more](/caffeinated) about it, or [download it](https://github.com/downloads/dmnd/Caffeinated/Caffeinated-1.0.zip) now.

<div class="download-link"><a href="https://github.com/downloads/dmnd/Caffeinated/Caffeinated-1.0.zip"><img src="/images/download_64.png"><span>Download</span></a>
<div>
Version 1.0
<span style="margin-left: 2em">125 kB</span>
</div>
</div>

[insomnia]: http://blogs.msdn.com/b/delay/archive/2009/09/30/give-your-computer-insomnia-free-tool-and-source-code-to-temporarily-prevent-a-machine-from-going-to-sleep.aspx
[msdn]: http://msdn.microsoft.com/en-us/library/aa373208(VS.85).aspx
