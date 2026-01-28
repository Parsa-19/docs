## configs

the xray config.json is in `/usr/local/etc/xray/config.json`:
```
{
  "log": {
    "loglevel": "warning"
  },
  "inbounds": [
    {
      "tag": "socks-in",
      "listen": "0.0.0.0",
      "port": 10808,
      "protocol": "socks",
      "settings": {
        "udp": true
      }
    },
    {
      "tag": "http-in",
      "listen": "0.0.0.0",
      "port": 10809,
      "protocol": "http"
    }
  ],
  "outbounds": [
    {
      "tag": "vless-out",
      "protocol": "vless",
      "settings": {
        "vnext": [
          {
            "address": "xxx.xxx.xxx",
            "port": xxx,
            "users": [
              {
                "id": "xxxxxx",
                "encryption": "none"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "xhttp",
        "security": "none",
        "xhttpSettings": {
          "path": "/path",
          "host": "hostname",
          "mode": "stream-one"
        }
      }
    }
  ],
  "routing": {
    "rules": [
      {
        "type": "field",
        "inboundTag": ["socks-in", "http-in"],
        "outboundTag": "vless-out"
      }
    ]
  }
}
```

set the docker deamon proxy in `/etc/systemd/system/docker.service.d/http-proxy.conf` (replace the HOST_IP with the ip of your vm):
```
[Service]
Environment="HTTP_PROXY=http://HOST_IP:10809"
Environment="HTTPS_PROXY=http://HOST_IP:10809"
Environment="NO_PROXY=localhost,127.0.0.1,.local"
```

then start minikube with proxy (replace the HOST_IP with the ip of your vm) variables:
```
minikube start \
  --docker-env HTTP_PROXY=http://HOST_IP:10809 \
  --docker-env HTTPS_PROXY=http://HOST_IP:10809 \
  --docker-env NO_PROXY=localhost,127.0.0.1,10.96.0.0/12,10.244.0.0/16
```

> [TIP]
> to ensure the traffic is going through xray, monitor the live logs of xray.service:
> `journalctl -efu xray`