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
  IP address in pool **_randomly_**. The real destination IP is unaware in the
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

### Nginx

Nginx server accepts incoming connections, and forward those specific vist to
v2ray inbound.

```
server {
    listen 80;
    listen [::]:80;
    server_name xx.yy.zz;
    return 301 https://$server_name:443$request_uri;
}

server {
    listen       443 ssl http2;
    listen       [::]:443 ssl http2;
    server_name  xx.yy.zz;
    charset utf-8;

    # ssl配置
    ssl_protocols TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_ecdh_curve secp384r1;
    ssl_prefer_server_ciphers on;
    ssl_session_cache shared:SSL:10m;
    ssl_session_timeout 10m;
    ssl_session_tickets off;
    ssl_certificate /etc/v2ray/xx.yy.zz.pem;
    ssl_certificate_key /etc/v2ray/xx.yy.zz.key;

    index index.php index.html index.htm default.php default.htm default.html;
    root /var/www/html/wordpress;

    location / {
      try_files $uri $uri/ =404;
    }

    location ~ \.php$ {
      include snippets/fastcgi-php.conf;
      fastcgi_pass unix:/run/php/php8.1-fpm.sock;
    }

    location ~ /\.ht {
      deny all;
    }

    location /accesslocation {
      proxy_redirect off;
      proxy_pass http://127.0.0.1:62162;
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

### caddy

Caddy is the first and only web server to use HTTPS automatically and by
default.Sites will be served over HTTPS automatically. You won't have to do
anything else about it. It just works!

For public domain names:

- domain's A/AAAA records point to your server
- ports 80 and 443 are open externally
- Caddy can bind to those ports (or those ports are forwarded to Caddy)
- data directory($XDG_DATA_HOME/caddy or $HOME/.local/share/caddy ) is writable
  and persistent
- domain name appears somewhere relevant in the config

```
xx.yy.zz {
    # good practice to signal on behalf of who
    # are the certs getting issue
    tls chenyanyu.ospost@@gmail.com

    # logs are optional
    log {
        output file /var/log/caddy/xx.yy.zz.log
        format console
    }

    reverse_proxy /xxxxxxx 127.0.0.1:62162

    root * /var/www/xx.yy.zz
    encode gzip
    file_server
    php_fastcgi unix//run/php/php-fpm.sock

    @disallowed {
        path /xmlrpc.php
        path *.sql
        path /wp-content/uploads/*.php
    }

    rewrite @disallowed '/index.php'
}
```

### V2ray

V2ray config to handle the real connection, which is forwarded by Nginx to 127.0.0.1

```json
{
  "inbounds": [
    {
      "port": 62162,
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
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
      "settings": {}
    },
    {
      "protocol": "blackhole",
      "settings": {},
      "tag": "blocked"
    }
  ]
}
```

### WordPress

Debian 12 records

```cmd
apt install mariadb-server
mysql_secure_installation

mysql
create database example_db default character set utf8 collate utf8_unicode_ci;
create user 'example_user'@'localhost' identified by 'example_pw';
grant all privileges on example_db.* TO 'example_user'@'localhost';
flush privileges;
exit


apt install php8.2 php8.2-cli php8.2-common php8.2-imap php8.2-redis php8.2-snmp php8.2-xml php8.2-mysqli php8.2-zip php8.2-mbstring php8.2-curl libapache2-mod-php  php-fpm -y
apt purge apache2*

cp wp-config-sample.php wp-config.php

// ** Database settings - You can get this info from your web host ** //
/** The name of the database for WordPress */
define( 'DB_NAME', 'example_db' );

/** Database username */
define( 'DB_USER', 'example_user' );

/** Database password */
define( 'DB_PASSWORD', 'example_pw' );

/** Database hostname */
define( 'DB_HOST', 'localhost' );

/** Database charset to use in creating database tables. */
define( 'DB_CHARSET', 'utf8' );

/** The database collate type. Don't change this if in doubt. */
define( 'DB_COLLATE', '' );

```

Install Starter Templates, create site with classical templates or AI builder

### Client Side

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

# Routing between multiple Server

```
                                              ┌──────────────────┐
                    ┌───────────────────────► │ server us-valley │
                    │                         └──────────────────┘
                    │                         ┌──────────────────┐
                    │               ┌───────► │ server vult-free │
          ┌─────────┴──┐────────────┘         └──────────────────┘
    app ─►│proxy client│                      ┌──────────────────┐
          └────────────┘────────────────────► │ block            │ ad ip/domain
                    │                         └──────────────────┘
                    │                         ┌──────────────────┐
                    └───────────────────────► │ direct           │ cn ip/domain
                                              └──────────────────┘


```

Client configuration example

```

{
  "log": {
    "access": "/Users/ospost/v2ray/log/access",
    "error": "/Users/ospost/v2ray/log/error",
    "loglevel": "debug"
  },
  "inbounds": [
    {
      "tag": "SOCKS-INBOUND",
      "port": 1080,
      "listen": "127.0.0.1",
      "protocol": "socks",
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"]
      },
      "settings": {
        "udp": true,
        "auth": "noauth"
      }
    },
    {
      "tag": "HTTP-INBOUND",
      "port": 8888,
      "listen": "127.0.0.1",
      "protocol": "http",
      "sniffing": {
        "enabled": true,
        "destOverride": ["http", "tls"]
      },
      "settings": {
        "udp": true,
        "auth": "noauth"
      }
    },
    {
      "tag": "API",
      "port": 53284,
      "listen": "127.0.0.1",
      "protocol": "dokodemo-door",
      "settings": {
        "udp": false,
        "address": "127.0.0.1",
        "allowTransparent": false
      }
    }
  ],

  "outbounds": [
    {
      "tag": "VULTR",
      "protocol": "vless",
      "settings": {
        "vnext": [
          {
            "address": "xxxx",
            "port": 443,
            "users": [
              {
                "encryption": "none",
                "id": "xxxx",
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
          "serverName": "xxxx"
        },
        "wsSettings": {
          "path": "xxxx"
        }
      },

      "mux": {
        "enabled": false,
        "concurrency": -1
      }
    },

    {
      "tag": "AWS",
      "protocol": "vless",
      "settings": {
        "vnext": [
          {
            "address": "xxxx",
            "port": 443,
            "users": [
              {
                "encryption": "none",
                "id": "xxxx",
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
          "serverName": "xxxx"
        },
        "wsSettings": {
          "path": "xxxx"
        }
      },

      "mux": {
        "enabled": false,
        "concurrency": -1
      }
    },

    {
      "tag": "GOOGLE",
      "protocol": "vless",
      "settings": {
        "vnext": [
          {
            "address": "xxxx",
            "port": 443,
            "users": [
              {
                "encryption": "none",
                "id": "xxxx",
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
          "serverName": "xxxx"
        },
        "wsSettings": {
          "path": "xxxx"
        }
      },

      "mux": {
        "enabled": false,
        "concurrency": -1
      }
    },
    {
      "tag": "DIRECT",
      "protocol": "freedom",
      "settings": {}
      // it is a must if you want to use V2Ray's routing feature.
      // here direct is a tag of this freedom outbound, then V2Ray will know this outbound stands for "direct".
    },
    {
      "tag": "BLOCK",
      "protocol": "blackhole",
      "settings": {
        "response": {
          "type": "http"
        }
      }
    }
  ],

  "observatory": {
    "subjectSelector": ["VULTR", "AWS", "GOOGLE"],
    "probeInterval": "5m"
  },

  "stats": {},

  "api": {
    "tag": "API",
    "services": ["StatsService"]
  },

  "dns": {
    "servers": [
      "8.8.8.8",
      "208.67.222.222",
      "114.114.114.114",
      {
        "address": "223.5.5.5",
        "port": 53,
        "domains": ["geosite:cn", "ntp.org"]
      }
    ]
  },

  "routing": {
    "domainStrategy": "IPOnDemand",

    "balancers": [
      {
        "tag": "BALANCER",
        //"selector": ["VULTR", "AWS", "GOOGLE"],
        "selector": ["VULTR"],
        //"selector": ["AWS"],
        //"selector": ["GOOGLE"],
        "strategy": {
          "type": "random"
        }
      }
    ],
    "rules": [
      {
        "type": "field",
        "ip": ["223.5.5.5", "114.114.114.114"],
        "outboundTag": "DIRECT"
      },
      {
        "type": "field",
        "ip": ["8.8.8.8", "208.67.222.222"],
        "balancerTag": "BALANCER"
      },
      {
        "type": "field",
        "domain": ["geosite:speedtest"],
        "balancerTag": "BALANCER"
      },

      {
        "type": "field",
        "outboundTag": "BLOCK",
        "domain": ["geosite:category-ads-all"],
        "enabled": true
      },
      {
        "type": "field",
        "outboundTag": "DIRECT",
        "protocol": ["bittorrent"]
      },
      {
        "type": "field",
        "outboundTag": "DIRECT",
        "domain": ["geosite:cn"],
        "enabled": true
      },

      {
        "type": "field",
        "outboundTag": "DIRECT",
        "ip": ["geoip:private", "geoip:cn"],
        "enabled": true
      },

      {
        "type": "field",
        "network": "tcp,udp",
        "balancerTag": "BALANCER"
      }
    ]
  }
}

```

# DNS

## Solution DNS routing with v2ray client

See examples in previous section.
The idea is routing dns requests with routing policy.

## Solution two dnscrypt-proxy plug dnsmasq

[dns](https://dev.to/metalage303/fan-qiang-dnswu-ran-de-yuan-li-yi-ji-ying-dui-ce-lue-3em)
[dnscrypt-proxy + dnsmasq](https://blog.linux-code.com/articles/thread-2060.html)

> dnsmasq
> dnsmasq is free software providing Domain Name System (DNS) caching, a Dynamic
> Host Configuration Protocol (DHCP) server, router advertisement and network
> boot features, intended for small computer networks

> dnscrypt-proxy
> A flexible DNS proxy, with support for modern encrypted DNS protocols such as
> DNSCrypt v2, DNS-over-HTTPS, Anonymized DNSCrypt and ODoH (Oblivious DoH).

```txt
                                               ┌───────────────────┐
                                    ┌────────► │ DNS server support│
                                    │          │ encryption        │
                      ┌─────────────┴──┐       └───────────────────┘
             ┌──────► │ dnscrypt-proxy │
             │        │ 127.0.0.1:25533│
             │        └────────────────┘
  ┌─────────────┐
  │  dnsmasq    │ cache hit return
  │ 127.0.0.1:53│ │
  └─────────────┘ │   ┌────────────────┐
       ▲     │    │   │ dnscrypt-proxy │
       │     └────┼─► │ 127.0.0.1:5533 │
       │          │   └─────────────┬──┘       ┌───────────────────┐
       │          │                 │          │ DNS server support│
       │          │                 └────────► │ encryption        │
       │          │                            └───────────────────┘
       │          │
       │   ┌──────┘
           ▼
   app dns request

```

```cmd
brew install dnscrypt-proxy
sudo brew services start dnscrypt-proxy

/opt/homebrew/opt/dnscrypt-proxy/sbin/dnscrypt-proxy -config /opt/homebrew/etc/dnscrypt-proxy.toml -resolve google.com
cp /opt/homebrew/etc/dnscrypt-proxy.toml /opt/homebrew/etc/dnscrypt-proxy-china.toml
cp ~/Library/LaunchDaemons/homebrew.mxcl.dnscrypt-proxy.plist ~/Library/LaunchDaemons/homebrew.mxcl.dnscrypt-proxy-china.plist

brew install dnsmasq

git clone https://github.com/felixonmars/dnsmasq-china-list

ln -sf bogus-nxdomain.china.conf /opt/homebrew/etc/dnsmasq.d/bogus-nxdomain.china.conf
ln -sf google.china.conf /opt/homebrew/etc/dnsmasq.d/google.china.conf
ln -sf apple.china.conf /opt/homebrew/etc/dnsmasq.d/google.china.conf
ln -sf accelerated-domains.china.conf /opt/homebrew/etc/dnsmasq.d/accelerated-domains.china.conf

```

```
ospost $ cat ~/Library/LaunchAgents/homebrew.mxcl.dnscrypt-proxy.plist
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <key>KeepAlive</key>
        <true/>
        <key>Label</key>
        <string>homebrew.mxcl.dnscrypt-proxy</string>
        <key>LimitLoadToSessionType</key>
        <array>
                <string>Aqua</string>
                <string>Background</string>
                <string>LoginWindow</string>
                <string>StandardIO</string>
                <string>System</string>
        </array>
        <key>ProcessType</key>
        <string>Background</string>
        <key>ProgramArguments</key>
        <array>
                <string>/opt/homebrew/opt/dnscrypt-proxy/sbin/dnscrypt-proxy</string>
                <string>-config</string>
                <string>/opt/homebrew/etc/dnscrypt-proxy.toml</string>
        </array>
        <key>RunAtLoad</key>
        <true/>
</dict>
</plist>
ospost $ cat ~/Library/LaunchAgents/homebrew.mxcl.dnscrypt-proxy-china.plist


ospost $ cat /opt/homebrew/etc/dnscrypt-proxy.toml | grep -v "#" | grep -v "^$"
server_names = ['google', 'yandex', 'scaleway-fr']
listen_addresses = ['127.0.0.1:25533']
...

<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN" "http://www.apple.com/DTDs/PropertyList-1.0.dtd">
<plist version="1.0">
<dict>
        <key>KeepAlive</key>
        <true/>
        <key>Label</key>
        <string>homebrew.mxcl.dnscrypt-proxy-china</string>
        <key>LimitLoadToSessionType</key>
        <array>
                <string>Aqua</string>
                <string>Background</string>
                <string>LoginWindow</string>
                <string>StandardIO</string>
                <string>System</string>
        </array>
        <key>ProcessType</key>
        <string>Background</string>
        <key>ProgramArguments</key>
        <array>
                <string>/opt/homebrew/opt/dnscrypt-proxy/sbin/dnscrypt-proxy</string>
                <string>-config</string>
                <string>/opt/homebrew/etc/dnscrypt-proxy-china.toml</string>
        </array>
        <key>RunAtLoad</key>
        <true/>
</dict>
</plist>


ospost $ cat /opt/homebrew/etc/dnscrypt-proxy-china.toml | grep -v "#" | grep -v "^$"
server_names = ['alidns-doh','tuna-doh-ipv4']
listen_addresses = ['127.0.0.1:5533']
...


ospost(stable) $ head accelerated-domains.china.conf
server=/0.zone/127.0.0.1#5533
server=/00.net/127.0.0.1#5533
server=/000.link/127.0.0.1#5533
server=/000000.net/127.0.0.1#5533
server=/00042.com/127.0.0.1#5533
server=/00058.com/127.0.0.1#5533
server=/0006266.com/127.0.0.1#5533
server=/000700.com/127.0.0.1#5533
server=/000714.xyz/127.0.0.1#5533
server=/00086.net/127.0.0.1#5533

ospost(stable) $ cat /etc/resolv.conf
#
# macOS Notice
#
# This file is not consulted for DNS hostname resolution, address
# resolution, or the DNS query routing mechanism used by most
# processes on this system.
#
# To view the DNS configuration used by this system, use:
#   scutil --dns
#
# SEE ALSO
#   dns-sd(1), scutil(8)
#
# This file is automatically generated.
#
nameserver 127.0.0.1
Mon Apr 21 19:48:41 CST 2025 ospost@M2.local:/opt/homebrew/etc/dnsmasq.d
ospost(stable) $ cat /opt/homebrew/etc/dnsmasq.conf  | grep ^server
server=127.0.0.1#25533

```

v2ray dns chagne, use 127.0.0.1

```
  "dns": {
    "servers": [
      "127.0.0.1"
      //{
      //  "address": "223.5.5.5",
      //  "port": 53,
      //  "domains": ["geosite:cn", "ntp.org"]
      //}
    ]
  },

  "routing": {
    "domainStrategy": "IPOnDemand",

    "balancers": [
      {
        "tag": "BALANCER",
        //"selector": ["VULTR", "AWS", "GOOGLE"],
        //"selector": ["VULTR"],
        //"selector": ["VULTRFREE"],
        "selector": ["RAKSMART"],
        //"selector": ["AWS"],
        //"selector": ["GOOGLE"],
        "strategy": {
          "type": "random"
        }
      }
    ],
    "rules": [
      //{
      //  "type": "field",
      //  "ip": ["223.5.5.5", "114.114.114.114"],
      //  "outboundTag": "DIRECT"
      //},
      //{
      //  "type": "field",
      //  "ip": ["8.8.8.8", "208.67.222.222"],
      //  "balancerTag": "BALANCER"
      //},


```
