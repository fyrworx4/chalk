---
layout: post
title: 	"BOLdozer"
description: "A quick python project."
tags: [python]
---
After reviewing a bit of Python the past couple weeks, as well as helping out my dad at his work, I was reminded of some of his manual, data-entry tasks that he would offload to my siblings.

Back in October of 2019, I attempted to streamline the manual process by using Microsoft Word’s Mail Merge feature with Excel, but then I would have to manually copy and paste data into an Excel sheet. Although I was able to cut a 2 hour process into about 15 minutes, there was still a lot of clicking and copy-and-pasting that my dad had to do.

So I decided to take on a new project involving Python – to automate these tasks with just a few clicks.

The goal was to extract certain contents of an HTML file and then place them into specific places in a document. 

This was done by using the Python module BeautifulSoup, which then returned the whole HTML into one huge list. I then used lots of indexing to extract the important information into a few dictionaries, and then used a Python module called docx-mailmerge to finally insert them into specific spots on a Word document “template”.

As someone who hasn’t had much experience with Python, this was a really fun way to use my knowledge to a whole different level and even make something that is useful. There was a lot of trial error, many hours trying to debug processes, but overall I can say that I am more comfortable with Python.

Here is a link to the [project][link-to-project].

[link-to-project]: https://github.com/fyrworx4/BOLdozer