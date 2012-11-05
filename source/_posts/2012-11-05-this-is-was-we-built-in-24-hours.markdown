---
layout: post
title: "This is was we built in 24 hours"
date: 2012-11-05 11:12
comments: true
categories: [mobile, ios, android]
---

Last week our current client - [ninemsn](http://ninemsn.com.au/) - ran the 3rd edition of their HackDay.

It's akin to Atlassian's [ShipIt](http://www.atlassian.com/shipit-day) days so if you're not familiar with the concept, you should check their page but here's the gist:

* We're given 24 hours to work on whatever we like as long as it's related somehow to the business

* We're also free to use any technology we want

The team consisted of [@romainprieto](https://twitter.com/romainprieto), Pranav Raja, [@thepaddedcell](https://twitter.com/thepaddedcell), [@stewgleadow](https://twitter.com/stewgleadow) and myself ([@leonardo_borges](https://twitter.com/leonardo_borges)).


### The idea: Cool story, bro!

[@romainprieto](https://twitter.com/romainprieto) came up with it:

* A mobile app that could scan QR codes on a printed [JIRA](http://www.atlassian.com/software/jira/overview/) story card and provide features such as assigning a card to yourself, moving it between columns and adding comments to it.

This is a common need. Most people will manage their stories in some sort of software such as JIRA. But it's also common to have a physical story wall. 

The problem then is keeping them in sync. It's all too common for people to move the cards on the well and forget all about JIRA.

Well, not anymore. This is exactly what our HackDay project tackles.


In 24 hours, we built two native mobile applications - Android and iPhone - that are able to scan QR Codes printed on the story cards and, through integrating with the JIRA REST API, provide all the necessary functionality needed for:

* Assigning a story to yourself - whoever is logged in to the app
* Moving the story between states - e.g.: In Analysis, Resolve Issue, Reopen Issue etc...
* Adding comments to a given story

All code is [open source and on github](https://github.com/ninemsn/cool-story). [Check it out](https://github.com/ninemsn/cool-story)! - and bear in mind this was written in 24 hours... ;)


We didn't stop there though. We actually finished early and had a couple of hours to spare so we shot a video - yes, a video!!! - about the app. I'm telling you, this is Hollywood quality:

<iframe width="560" height="315" src="http://www.youtube.com/embed/jzhY4JHDowI" frameborder="0" allowfullscreen></iframe>

Also, checkout some screenshots of the Android version:

<p>
<div class="gallery">
    <a rel="gallery1" href="/assets/images/posts/cool-story-1.png" class="fancybox hoverZoomLink">
        <img src="/assets/images/posts/cool-story-1.png" width="120" height="213"/>
    </a>
    <a rel="gallery1" href="/assets/images/posts/cool-story-2.png" class="fancybox hoverZoomLink">
        <img src="/assets/images/posts/cool-story-2.png" width="120" height="213"/>
    </a>
    <a rel="gallery1" href="/assets/images/posts/cool-story-3.png" class="fancybox hoverZoomLink">
        <img src="/assets/images/posts/cool-story-3.png" width="120" height="213"/>
    </a>
    <a rel="gallery1" href="/assets/images/posts/cool-story-4.png" class="fancybox hoverZoomLink">
        <img src="/assets/images/posts/cool-story-4.png" width="120" height="213"/>
    </a>
    <div class="clear"></div>
</div>

{% include fancybox.html %}

Enjoy! :)