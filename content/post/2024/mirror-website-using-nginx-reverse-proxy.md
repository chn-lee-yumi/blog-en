---
title: "Mirroring a Website with Nginx"
description: "How to mirror a website using Nginx. Utilise proxy_pass and proxy_set_header Host $proxy_host to create a mirrored (reverse proxy) site."
date: 2024-01-15T22:35:00+11:00
lastmod: 2024-01-15T22:35:00+11:00
categories:
  - Learning
tags:
  - Nginx
---

## Introduction

Years ago, there were several mirror sites of Google available in China. I was always curious about how they worked. Recently, that memory resurfaced, and I decided to give it a go. In theory, you can mirror almost any website.

## Nginx Configuration

Let’s use Baidu as an example. The key configuration directives are `proxy_pass` and `proxy_set_header`. Essentially, this is just a reverse proxy setup.

To mirror a site like Baidu, you can use the following minimal configuration. It forwards all incoming requests to the real website while setting the appropriate host header.

```conf
server {
    listen 80;
    server_name example.com;
    location / {
        proxy_pass https://www.baidu.com;
        proxy_set_header Host $proxy_host;
    }
}
```

You can also use a more advanced approach. For instance, if a website requires login, you can insert a logged-in cookie directly into the configuration, restrict access by IP address, and then share this proxy with others. Here's a practical example: I’m overseas and want to let my friends in China use ChatGPT. I can mirror `chat.openai.com` on a server with external access, and inject my own logged-in cookie. This way, when my friends visit the mirrored site, they can use ChatGPT directly.

```conf
server {
    listen 80;
    server_name example.com;
    location / {
        allow 10.0.0.2;
        deny all;
        proxy_pass https://www.baidu.com;
        proxy_set_header Host $proxy_host;
        proxy_set_header Cookie "key1=value1; key2=value2";
    }
}
```
