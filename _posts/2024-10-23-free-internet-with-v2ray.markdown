---
title: Free Internet with v2ray
date: 2024-10-23 14:31:00 Z
---

# What is V2ray

V2ray is core of project V, which is a set of tools to help build your own network over internet.
It is responsible for network protocols and communications. It can work alone, as well as combine
with other tools.

- Multiple inbound/outbound proxies. **_ONE V2Ray instance_** supports in parallel multiple inbound and
  outbound protocols. Each protocol works independently
- Customizable routing: incoming traffic can be sent to different outbounds based on routing
  configuration
- Multiple protocols: V2Ray supports multiple protocols, including Socks, HTTP, Shadowsocks, VMess
  VLESS, etc. Each protocol may have its own transport, such as TCP, mKCP, WebSocket etc.
- Multiple platforms. Windows, Linux, macOS, Mobile.
- Obfuscation: V2Ray has built in obfuscation to hide traffic in TLS, and can run in parallel with
  web servers

# Underlying mechanism

```
      ┌───────┐                                    ┌────────┐
      │inbound├─────────►┌─────────────┐           │outbound│
      └───────┘          │  Dispatcher │──────────►└────────┘
                         │  Router     │
      ┌───────┐          │  DNS        │──────────►┌────────┐
      │inbound├─────────►└─────────────┘           │outbound│
      └───────┘                                    └────────┘

```

- At least one inbound and one outbond protocol to make v2ray work properly
- The dispatcher is responsible for choosing an outbound for a given connection based on config
- Possible to work with CDN together

```
Illustration v2ray vless+tls+ws+CDN

                                            │ FW   │
            ┌───────────────────────────┐   │      │
            │       v2ray client        │   │      │
┌───────┐   │                           │   │      │
│browser├─► │in ─► disp/Router/DNS─► out│───┼──────┼───┐
└───────┘   │                           │   │      │   │
            └───────────────────────────┘   │      │   │
                                            │      │   │
                                                       │
   ┌───────────────────────────────────────────────────┘
   │
   │
   │   ┌────────────┐      ┌────────────────────────────┐
   │   │ CDN like   │      │    v2ray server            │
   │   │ Cloudflare │      │                            │
   └──►│            │ ───► │in ─► disp/Router/DNS ─► out│──► google.com
       │            │      │                            │
       │            │      └────────────────────────────┘
       └────────────┘
```

# High level Summary of the Setup

## domain

Apply a domain, it will be heavily used.

- [ ] Free <https://www.namesilo.com/>
- [x] Free <https://freedomain.one/>

## VPS

- [x] Block Risk <https://www.vultr.com/>

## Fake website

_[wordpress] (https://wordpress.org/documentation/article/get-started-with-wordpress/)_

## CDN(cloudflare)

Couple of things to reveal the magic

- Use CDN dns to replace the default dns in domain provider dashboard, and
  turn on domain proxy, then the domain address will be translated to AN
  IP address in pool **_randomly_**. The real destination IP is unware in the
  middle as the Traffic proxyed by CDN first. The latency is huge :(.
  Benefit is lower possibility of blocking.

- Visitor/Cloudflare/Origin Server end to end TLS encryption. Unless traffic
  presents obvious characteristic which could be easily identified by AI/ML
  stuff, it's safe to go. I don't think personal traffic deserves huge
  resources to monitor & analysis.

## GUI Client

- qv2ray old but workable and easy to setup

  - [x] macOS
  - [x] Linux
  - [x] Windows (anti virus warning)

- V2BOX
  - [x] Iphone
  - [] macOS 11.7.10

# Configuration Example

## Server Side

Nginx server accepts incoming connections, and forward those specific vist to
v2ray inbound.

```
server {
    listen 80;
    listen [::]:80;
    server_name YOU SERVER ADDRESS;
    return 301 https://$server_name:443$request_uri;
}
server {
    listen       443 ssl http2;
    listen       [::]:443 ssl http2;
    server_name www.whocares.icu;
    charset utf-8;

    # ssl配置
    ssl_protocols TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_ecdh_curve secp384r1;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_session_tickets off;
    ssl_certificate /etc/v2ray/xxx.pem;
    ssl_certificate_key /etc/v2ray/xxx.key;

    root /usr/share/nginx/html;
    location / {

    }

    location /xxxxx {
      proxy_redirect off;
      proxy_pass http://127.0.0.1:34556;
      proxy_http_version 1.1;
      proxy_set_header Upgrade $http_upgrade;
      proxy_set_header Connection "upgrade";
      proxy_set_header Host $host;
      # Show real IP in v2ray access.log
      proxy_set_header X-Real-IP $remote_addr;
      proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}

```

V2ray config to handle the real connection, which is forwarded by ngnix to 127.0.0.1

```
{
  "inbounds": [
    {
    "port": 34556,
    "listen": "127.0.0.1",
    "protocol": "vless",
    "settings": {
        "clients": [
            {
                "id": "xxxxxxxxxxxxx",
                "level": 0
            }
        ],
        "decryption": "none"
    },
    "streamSettings": {
        "network": "ws",
        "security": "none",
        "wsSettings": {
            "path": "/xxxxx",
            "headers": {
                "Host": "xxxxxxx"
            }
        }
    }
    },
    ],
  "outbounds": [{
    "protocol": "freedom",
    "settings": {}
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }]
}

```

## Client Side

```json
{
  "inbounds": [
    {
      "listen": "127.0.0.1",
      "port": 1080,
      "protocol": "socks",
      "settings": {
        "udp": true
      }
    },
    {
      "listen": "127.0.0.1",
      "port": 8888,
      "protocol": "http",
      "settings": {
        "udp": false
      }
    }
  ],
  "log": {
    "access": "/access",
    "error": "/error",
    "loglevel": "debug"
  },
  "outbounds": [
    {
      "protocol": "vless",
      "settings": {
        "vnext": [
          {
            "address": "YOUR SERVER ADDRESS URL",
            "port": 443,
            "users": [
              {
                "encryption": "none",
                "id": "xxxxxxxxxxxxxxxxxxxxxxx",
                "level": 0
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "tlsSettings": {
          "serverName": "YOUR SERVER NAME"
        },
        "wsSettings": {
          "path": "/XXXXX"
        }
      }
    }
  ]
}
```
