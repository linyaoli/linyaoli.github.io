# Short note on setting up a Shadowsocks VPS

## Step 1: Choose VPS host
I've tried vultr, bandwagon, digitalocean, azure, aws and aliyun, by far the most pleasant experience is from azure.
aws and digitalocean are usually slow, aliyun does not respect privacy agreement as far as I know.
Vultr and bandwagon are considerably cheaper than Azure, however the throughput is not satisfying.

## Step 2: Deploy shadowsocks
1. You'll first need to expose SSH's port 22, it's usually in inbound security rule.
2. Get into your server, type following commands:
```
> wget --no-check-certificate https://raw.githubusercontent.com/teddysun/shadowsocks_install/master/shadowsocks.sh
> chmod +x shadowsocks.sh
> ./shadowsocks.sh 2>&1 | tee shadowsocks.log
```
You'll need to set the password, port number and encryption algorithm during installation.
Check log in /var/log to make sure it's up and running.
3. Go back to inbound security rule page and allow the server to receive request to your Shadowsocks's port.

You can modify your config here /etc/shadowsocks.json. Need to restart the service to make it effective.

## Step 3: Install client
For Mac there is ShadowsocksX-NG. Go to the server preference and add the address, port, possaword and encryption. Leave the rest blank.

## Step 4 (Optional): Optimization
I'll bring up a few methods here and you may try them out yourself, personally I haven't found them to be very effective.
1. Enable TCP fast open in /etc/sysctl.conf.
2. Use BBR protocol to maximize bandwidth.

There are some documents out there telling you how to do the above.

If you are into this topic, you may notice some research were going on, which tried to identify shadowsocks traffic with machine learning algorithms. I've read the paper and pretty sure it's a piece of junk and currently not in GFW(because it's not working).
