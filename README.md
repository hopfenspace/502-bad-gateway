# 502-bad-gateway

Our bad gateway response page.

## Setup

As this page will load a few other resources (css and images), 
a problem arises:

All requests will take a tremendous amount of time as the timeout of the
initial page have to be awaited.

So there are 2 solutions in setting this up:
1. Single bundled `.html` file
2. Another (sub)domain / cdn / .. for serving files

Given a simple site:
```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    server_name server.name;

    location / {
        proxy_pass http://10.0.0.107; 
    }
}
```

After setting up the custom error page:
```nginx
server {
    listen 443 ssl;
    listen [::]:443 ssl;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    server_name server.name;
    
    # NEW
    recursive_error_pages on;
    
    location @badgw {
        root /path/to/index.html/root/;
        try_files $uri $uri/ =404;
    }

    location / {
        proxy_pass http://10.0.0.107;

        # NEW
        error_page 502 =200 @badgw;
    }
}

# Only required if you are serving the resources with an extra domain

# Add an extra vhost:
server {
    listen 443 ssl;
    listen [::]:443 ssl;

    ssl_certificate /path/to/cert.pem;
    ssl_certificate_key /path/to/key.pem;
    
    server_name badgw.server.name;
    
    root /path/to/extra/resource/root;
    
    location / {
        try_files $uri $uri/ =404;
    }
}
```
