# Setting Up V2Ray Service

The GFW is getting smarter, it identies the pattern of SOCKS encrypted requests and hijacks them. If you are using normal SOCKS5 to communicate, the chance of getting blocked will be much higher based on the sensitivity they set. 

So how does the GFW identify SOCKS5 requests if they are fully encrypted and almost impossible to decrypt at realtime? My guess is, which is quite naive and straight-forward, the GFW checks if the requests are completely encrypted; if so, it means they are suspicious. In addition, the GFW checks if this type of traffic continues for some time, if so, just simply blocks it. I don't think there are too much rocket science. 

Luckily, we have an alternative solution, `VMess`. `VMess` is a protocol which is specifically designed for getting thru GFW, by far, it is much harder than SOCKS to be blocked. I will only explain how to set it up in minimum steps, for the rest of topics, please check the official websites.

## Setting up server
I'd recommend Azure in region `Asia East`(Hong Kong) and `Southeast Asia`(Singapore).
1. Get machine and OS ready. Ubuntu is recommed;
2. In root mode, run `bash <(curl -L -s https://install.direct/go.sh)`. The script will automatically install everything for you.
3. In `/etc/v2ray/config.json`, e.g.
```json
{
  "inbounds": [{
    "port": 25781,
    "protocol": "vmess",
    "settings": {
      "clients": [
        {
          "id": "7e8c03c3-dc45-41e6-a4a2-b6c67c63f04f",
          "level": 1,
          "alterId": 64
        }
      ]
    }
  }],
  "outbounds": [{
    ...
  }],
  "routing": {
    ...
  }
}
```
There are two arugments you will need next, the `port` and clients `id`.
4. Now run `service v2ray start`. Then run `service v2ray status` to ensure it is running properly.

## Setting up client
I will only explain how to set up in Mac OS as I only have one machine.
1. `brew tap v2ray-core`
2. `brew install v2ray-core`
3. Now open `/usr/local/etc/v2ray/config.json`, in `outbounds` section, e.g.
```json
  // List of outbound proxy configurations.
  "outbounds": [{
    // Protocol name of the outbound proxy.
    "protocol": "vmess",

    // Settings of the protocol. Varies based on protocol.
    "settings": {
      "vnext": [{
        "address": "xx.xxx.xxx.xxx",
        "port": 25781,
        "users": [{"id": "7e8c03c3-dc45-41e6-a4a2-b6c67c63f04f"}]
      }]
    },

    // Tag of the outbound. May be used for routing.
    "tag": "direct"
  },{
    "protocol": "blackhole",
    "settings": {},
    "tag": "blocked"
  }],
```
Set up the arguments as above, make sure the protocol is vmess, and values in `vnext` are as same as what are set up in service side.

4. Finally, run `brew services start v2ray-core`, it will add this service into login item and run automatically.

## Setting up proxy
This is the final step, now do these:
1. `System Preferences` -> `Network` -> `Advanced` -> `Proxies`
2. In `Select a protocol to configure` box, select `SOCKS Proxy` and check to enable.
3. In its config, set ip address as `127.0.0.1` and port as `1080`. Save.

Now network requests will be redirected to your local v2ray proxy clients and get encrypted. Now you should be able to open 404 websites :)

## Drawbacks
1. By far I don't see smart DNS in v2ray clients, so any requests are sent via proxy, which is a bit unneccesary, and requires your server to be relatively high-performance.
2. No GUI clients. 

But these are just good-to-have. We are lucky enough to have those professionals who work on this voluntarily. Good luck! 
