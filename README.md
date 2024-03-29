# NGINX

## \etc\nginx\nginx.conf

```nginx
# For more information on configuration, see:
#   * Official English Documentation: http://nginx.org/en/docs/
#   * Official Russian Documentation: http://nginx.org/ru/docs/
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;
# Load dynamic modules. See /usr/share/nginx/README.dynamic.
include /usr/share/nginx/modules/*.conf;
events {
    worker_connections 1024;
}
http {
    log_format main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';
    access_log /var/log/nginx/access.log  main;
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;
    include /etc/nginx/mime.types;
# Load modular configuration files from the /etc/nginx/conf.d directory.
# See http://nginx.org/en/docs/ngx_core_module.html#include
# for more information.
    include /etc/nginx/conf.d/*.conf;
    default_type application/octet-stream;
}

```

## \etc\nginx\conf.d\EXAMPLESERVER.EXAMPLEDOMAIN.TLD.conf

```nginx
#
# This server block will redirect http:// to https:// AND fixes FQDN issues
#
server {
# IPv4
    listen 80 default_server;
# IPv6
    listen [::]:80 default_server;
# Define the name of the server that can be called using $server_name
    server_name _;
    return 301 https://$server_name$request_uri;
}
#
# This server block is the main config
#
server {
# IPv4
# Ensure that HTTP/2 is enabled for the server
    listen 443 ssl http2 default_server;
# IPv6
# Ensure that HTTP/2 is enabled for the server
    listen [::]:443 ssl http2 default_server;
# Define the name of the server that can be called using $server_name
    server_name _;
    include snippets/star_dot_EXAMPLE_TLD.conf;
    include snippets/ssl-params.conf;
    location / {
        proxy_redirect off;
        proxy_set_header Host                $host;
        proxy_set_header X-Real-IP           $remote_addr;
        proxy_set_header X-Forwarded-For     $proxy_add_x_forwarded_for;
        proxy_set_header Upgrade             $http_upgrade;
        proxy_set_header Connection          "upgrade";
        proxy_set_header X-Forwarded-Proto   $scheme;
        client_max_body_size 10m;
        client_body_buffer_size 128k;
        proxy_connect_timeout 90;
        proxy_send_timeout 90;
        proxy_read_timeout 90;
        proxy_buffers 32 4k;
        proxy_pass http://127.0.0.1:4434;
        proxy_http_version 1.1;
        proxy_next_upstream error timeout http_502 http_503 http_504;
    }
}

```

## \etc\nginx\snippets\ssl-params.conf

```nginx
ssl_protocols TLSv1.2;
ssl_prefer_server_ciphers on;
ssl_dhparam /etc/ssl/certs/dhparam.pem;
ssl_ciphers ECDHE-RSA-AES256-GCM-SHA512:DHE-RSA-AES256-GCM-SHA512:ECDHE-RSA-AES256-GCM-SHA384:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384;
ssl_ecdh_curve secp384r1;
ssl_session_timeout 30m;
ssl_session_cache shared:SSL:30m;
ssl_session_tickets off;
# Syntax:     ssl_stapling_verify on | off;
# Default:
# ssl_stapling_verify off;
# Context:     http, server
# Enables or disables verification of OCSP responses by the server.
# For verification to work, the certificate of the server certificate issuer, the root certificate, and all intermediate certificates should be configured as trusted using the ssl_trusted_certificate directive.
ssl_stapling on;
ssl_stapling_verify on;
# Generated by NetworkManager
# cdc-kt-ivm-o nameservers set by server team
# nameserver 192.168.10.55
# nameserver 192.168.11.55
resolver 192.168.10.55 192.168.11.55 valid=300s;
resolver_timeout 5s;
add_header X-Frame-Options DENY;
add_header X-Content-Type-Options nosniff;
add_header X-XSS-Protection "1; mode=block";

```

## \etc\nginx\snippets\star_dot_EXAMPLE_TLD.conf

```nginx
ssl_certificate /etc/ssl/certs/f13ab2a98f436f91.crt;
ssl_certificate_key /etc/ssl/certs/server.key;
# Required for setting "ssl_stapling on" & "ssl_stapling_verify on" from /etc/nginx/snippets/ssl-params.conf to work
# Otherwise NGINX will throw errors and disable the settings if set to "on"
# Syntax:     ssl_trusted_certificate file;
# Default:     —
# Context:     http, server
# This directive appeared in version 1.3.7.
# Specifies a file with trusted CA certificates in the PEM format used to verify client certificates and OCSP responses if ssl_stapling is enabled.
# In contrast to the certificate set by ssl_client_certificate, the list of these certificates will not be sent to clients.
ssl_trusted_certificate /etc/nginx/ssl/certs/chain.pem;

```
