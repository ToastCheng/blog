---
title:  "How to deal with non-HTTP traffic in Nginx?"
date:   2020-05-08T21:02:50+0800
tags: [nginx, http]
categories: technique
---
Nginx as a web server, it's not surprising that it deals with HTTP traffic most of the time.
For example, serving static file directly can be done like this:

`nginx.conf`
```
http {
  server {
    listen 80 default_server;
    server_name localhost;
    
    location / {
      root /etc/nginx/html;
      try_files $uri $uri/ /index.html;
    }
  }
}
```
Also, if you use Nginx as a reverse proxy, then usually the config will looks like this:
```
http {
  gzip on;
  include /etc/nginx/mime.types;
  upstream backend {
    server 192.0.0.1:8080;
    server 192.0.0.2:8080;
  }
  location / {
    proxy_pass http://backend;
  }
}
```
where `proxy_pass` tells Nginx to direct the traffic to `http://backend`. The backend servers lie in the `upstream` block, recieve the incoming requests.

But, what about non-HTTP traffic?

If you take a closer look, you would find out that the above configuration is wrap by a `http` block, meaning the rules you set are tailored for HTTP protocol.

If the traffic does not look like HTTP, Nginx has no way to handle it.

Fortunately, Nginx is capable of doing L4 Load Balancing!
Even if the server you are tackling with does not speak HTTP, it's very likely that the protocol it use is still based on TCP (You can think of MySQL).

The module for this purpose is `ngx_stream_core_module`, which supports TCP/UDP Load Balancing.

There are a few examples:
```nginx
stream {
  upstream dicom_server {
    server 192.0.0.3:4242;
  }
  server {
    listen 4242;
    listen [::]:4242;
    proxy_pass dicom_server;
  }
}
```
It just looks like http block, but instead the keyword is `stream`.

First you setup your upstream server, notice that in the stream block there is no concept like `location`, you can direct traffic solely by IP and port, for it is L4.

In this example the server is at 192.0.0.3:4242, nginx will transport the tcp stream to it once someone visit the port 4242.


### case study - DICOM Protocol
The reason I research for this is for integrating PACS server with our product.

PACS is a kind of archive server that offers read/write on medical images, and it speaks a different protocol called DICOM (L7).

There is a scenario that a client PACS want to send am image to our PACS which is behind Nginx along with other microservices (we surely don't want to expose our PACS).

```
                       |
client PACS -------> nginx ------> server PACS
                       |
other client (PC) ---> | ------->  backend
                       |           
                       |             etc.
```

Here is what the `nginx.conf` looks like:
```nginx
worker_processes 4;
events { worker_connections 1024; }
http {
  gzip on;
  include /etc/nginx/mime.types;
  
  upstream backend {
    server 192.0.0.1:8080;
    server 192.0.0.2:8080;
  }
  server {
    listen 80 default_server;
    server_name localhost;
    
    location / {
      root /etc/nginx/html;
      try_files $uri $uri/ /index.html;
    }
    
    location /api {
      proxy_pass http://backend;
    }
  }
}
stream {
  upstream dicom_server {
    server 192.0.0.3:4242;
  }
  server {
    listen 4242;
    listen [::]:4242;
    proxy_pass dicom_server;
  }
}
```