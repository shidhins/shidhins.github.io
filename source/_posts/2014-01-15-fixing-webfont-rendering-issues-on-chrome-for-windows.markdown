---
layout: post
title: "Fixing webfont rendering issues on Chrome for Windows"
date: 2014-01-15 01:57
comments: true
categories: [CSS]
---
I've been working with webfonts for a while now in my local machine(with Google Chrome running on Mac). Only recently did I find how horrible the fonts rendered on Google Chrome for Windows.

A bit of googling pointed me to [this article](http://dev-metal.com/fix-ugly-font-rendering-google-chrome/) that dicusses a few workarounds for the issue. Essentially the idea is to make Chrome load the *svg* font-file inorder to render correctly. This could be achieved with a webkit specific media query or by changing the font load order and moving *svg* ahead of *woff*.

<!--more-->

The side effect and downside is that Chrome running on other OSes would also load the *svg* font-file instead of *woff*. I wanted to avoid this for performance reasons, since *woff* offers the best compression and least file size while my *svg* file was the largest among the lot (three time the size of the *woff*). Using two webfonts made my case for performance all the more pressing.

So I wanted the heavier *svg* font files to be loaded only if the browser is Chrome and the OS is Windows. As both the browser and OS have to be detected to serve the right font file, I turned to the `User-Agent` string sent with the browser request. The server can then apply the appropriate `@font-face` rule based on the UA.

This is how I did it in Rails:
{% codeblock _assets_preload.html.erb lang:erb %}
<style>
  <% if request.env['HTTP_USER_AGENT'].include? "Windows" and request.env['HTTP_USER_AGENT'].include? "Chrome" %>
    @font-face {
      font-family:your-webfont;
      font-style:bold;
      font-weight:normal;
      src:url("your-webfont.svg") format("svg")
    }
  <% else %>
    @font-face {
      font-family:your-webfont;
      font-weight:normal;
      font-style:normal;
      src:url("your-webfont.eot");
      src:url("your-webfont.eot?#iefix") format("embedded-opentype"),
          url("your-webfont.woff") format("woff"),
          url("your-webfont.ttf") format("truetype"),
          url("your-webfont.svg") format("svg")
    }
  <% end %>
</style>
{% endcodeblock %}

The `@font-face` rule has only the *svg* font file if the request is from Chrome for Windows, while the standard font-face bullet-proof syntax is used in all other cases. The conditional style should be declared before all other application specific styles dependent on the font are loaded. In Rails this would be :
{% codeblock application.html.erb lang:erb %}
  <%= render 'shared/assets_preload' %>
  <%= stylesheet_link_tag "application", :media => "all" %>
{% endcodeblock %}
