---
layout: post
title: "Your own personal hgignore file"
date: 2011-10-06 12:28
comments: true
categories:
---
Sometimes people don't agree on the contents of the tracked .hgignore file in the repository root. For example, I don't like having *orig in .hgignore as having backup files show up when I grep is annoying. I solved that problem by removing the *orig pattern and telling other repository users about hg purge.

But today I found another way to deal with different opinions for ignore files. [Hidden away on the Mercurial wiki][1] is a nice tip about per-user hgignore files. In a repository's hgrc you can reference an arbitrary file to be used in addition to the tracked .hgignore file. No more .hgignore wars!

[1]: http://mercurial.selenic.com/wiki/TipsAndTricks#Ignore_files_in_local_working_copy_only
