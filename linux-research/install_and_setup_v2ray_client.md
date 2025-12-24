there is a official v2ray core that runs as a service in your machine.
you configure it a to act as wheather server or client.
later it comes as v2fly wich is the official utility that uses v2ray core.
<br>

fetch and install by script:
`curl -fsSL https://raw.githubusercontent.com/v2fly/fhs-install-v2ray/master/install-release.sh | sudo bash` 
verify:
`v2ray version`
<br>
then the script will install the binary in `/usr/local/bin/v2ray`
<br>
now create your client configs:
`sudo nano /usr/local/etc/v2ray/config.json`
<br>
add this:
```
{
  "log": {
    "loglevel": "warning"
  },
  "inbounds": [
    {
      "port": 10808,
      "listen": "127.0.0.1",
      "protocol": "socks",
      "settings": {
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
            "address": "SERVER_IP",
            "port": 443,
            "users": [
              {
                "id": "UUID",
                "encryption": "none"
              }
            ]
          }
        ]
      },
      "streamSettings": {
        "network": "tcp",
        "security": "none"
      }
    }
  ]
}
```
in the above config replace "SERVER_IP", "UUID" the port it uses.
<br>
run service commands:
`sudo systemctl daemon-reload`
`sudo systemctl enable v2ray`
`sudo systemctl start v2ray`
<br>
check the connection by curl:
`curl --socks5 127.0.0.1:10808 https://dogapi.dog/api/v2/breeds`
> [!NOTE]
> do not set system proxy. just run `export ALL_PROXY=socks5://127.0.0.1:10808` to export the proxy for current session every time you needed and `unset ALL_PROXY` when ever you didnt need it.
<br>
[source](https://operavps.com/docs/install-v2ray-vpn/)