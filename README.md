```bash
<VirtualHost *:443>
    ServerName m.youtube.com

    SSLEngine On
    SSLCertificateFile cert.pem
    SSLCertificateKeyFile key.pem

    SSLProxyEngine On
    SSLProxyVerify none
    SSLProxyCheckPeerCN off
    SSLProxyCheckPeerName off
    SSLProxyCheckPeerExpire off

    RewriteEngine On
    RewriteCond %{HTTP:Upgrade} =websocket [NC]
    RewriteRule /(.*) wss://localhost:65535/$1 [P,L]

    ProxyPass / https://127.0.0.1:65535
    ProxyPassReverse / https://127.0.0.1:65535
</VirtualHost>```

```bash
server {
    listen 443 ssl default_server;
    listen [::]:443 ssl default_server;


    server_name _;


    ssl_certificate cert.pem;
    ssl_certificate_key key.pem;


    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers HIGH:!aNULL:!MD5;

    location / {

        proxy_pass http://127.0.0.1:10000;


        proxy_ssl_verify off;
        proxy_ssl_session_reuse on;


        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";


        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}```

```bash
{
  "dns": {
    "servers": [
      "127.0.0.1"
    ],
    "queryStrategy": "UseIPv4"
  },
  "log": {
    "loglevel": "debug"
  },
  "inbounds": [
    {
      "listen": "127.0.0.1",
      "port": 10000,
      "protocol": "vless",
      "settings": {
        "clients": [
          {
            "id": "x",
            "flow": ""
          }
        ],
        "decryption": "none"
      },
      "streamSettings": {
        "network": "ws",
        "wsSettings": {
          "path": "/"
        }
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "freedom",
       "settings": {
        "domainStrategy": "ForceIPv4"
      }
    },
    {
      "protocol": "blackhole",
      "tag": "blocked"
    }
  ]
}```

```bash
{
  "log": {
    "loglevel": "debug"
  },
  "inbounds": [
    {
      "listen": "127.0.0.1",
      "port": 10808,
      "protocol": "socks",
      "settings": {
        "auth": "noauth",
        "udp": true
      }
    }
  ],
  "outbounds": [
    {
      "protocol": "vless",
      "settings": {
        "vnext": [
          {
            "address": "x",
            "port": 443,
            "users": [
              {
                "id": "x",
                "encryption": "none",
                "flow": ""
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "ws",
        "security": "tls",
        "tlsSettings": {
          "serverName": "m.youtube.com",
          "pinnedPeerCertSha256": "x"
        },
        "wsSettings": {
          "path": "/"
        }
      }
    }
  ]
}```
```bash
openssl x509 -noout -fingerprint -sha256 -in cert.pem
openssl x509 -noout -fingerprint -sha256 -in cert.pem | sed 's/:/|/g'```
```bash
openssl req -x509 -newkey rsa:2048 -keyout key.pem -out cert.pem -days 365 -nodes```
