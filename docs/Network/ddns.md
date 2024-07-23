---
layout: default
title: DDNS
parent: 网络知识
nav_order: 1
has_children: true
permalink: /docs/net/ddns
---

# DDNS 软件推荐

## ddns-go

同事发现一个很好的 DDNS 软件，挺好用的，在次记录下。

支持 Mac、Windows、Linux 系统，支持ARM、x86架构，支持的域名服务商 `阿里云` `腾讯云` `Dnspod` `Cloudflare` `华为云` `Callback` `百度云` `Porkbun` `GoDaddy` `Namecheap` `NameSilo` `Dynadot`

项目地址：

[https://github.com/jeessy2/ddns-go](https://github.com/jeessy2/ddns-go)



## Cloudflare 脚本

在此之前，一直用别人贡献的 DDNS 脚本，调用 Cloudflare 的 API 来完成此操作，此处也记录一下：

```shell
#!/bin/bash

# A bash script to update a Cloudflare DNS A record with the external IP of the source machine
# Used to provide DDNS service for my home
# Needs the DNS record pre-creating on Cloudflare

# Proxy - uncomment and provide details if using a proxy
#export https_proxy=http://<proxyuser>:<proxypassword>@<proxyip>:<proxyport>

# Cloudflare zone is the zone which holds the record
zone=XXX.com
# dnsrecord is the A record which will be updated
fqdnlist="vpn.XXX.com music.XXX.com cloud.XXX.com"

## Cloudflare authentication details
## keep these private
cloudflare_auth_email=XXX@gmail.com
cloudflare_auth_key=XXX

#  -H "X-Auth-Email: $cloudflare_auth_email" \
#  -H "X-Auth-Key: $cloudflare_auth_key" \


# Get the current external IP address
ip=$(curl -s -X GET https://checkip.amazonaws.com)
echo "Current IP is $ip"


# get the zone id for the requested zone
zoneid=$(curl -s -X GET "https://api.cloudflare.com/client/v4/zones?name=$zone&status=active" \
  -H "Authorization: Bearer $cloudflare_auth_key" \
  -H "Content-Type: application/json" | jq -r '{"result"}[] | .[0] | .id')

echo "Zoneid for $zone is $zoneid"


for dnsrecord in $fqdnlist
 do

# yum install -y bind-utils
#if host $dnsrecord | grep "has address" | grep "$ip"; then
#  echo "$dnsrecord is currently set to $ip; no changes needed"
#  else
   # if here, the dns record needs updating


# get current dns A record
currentdnsip=$(curl -s -X GET "https://api.cloudflare.com/client/v4/zones/$zoneid/dns_records?type=A&name=$dnsrecord" \
  -H "Authorization: Bearer $cloudflare_auth_key" \
  -H "Content-Type: application/json" | jq -r '{"result"}[] | .[0] | .content')
echo "Current DNS record is $currentdnsip"

if [ $currentdnsip == $ip ]; then
  echo "$dnsrecord is currently set to $ip; no changes needed"
else

# get the dns record id
dnsrecordid=$(curl -s -X GET "https://api.cloudflare.com/client/v4/zones/$zoneid/dns_records?type=A&name=$dnsrecord" \
  -H "Authorization: Bearer $cloudflare_auth_key" \
  -H "Content-Type: application/json" | jq -r '{"result"}[] | .[0] | .id')

echo "DNSrecordid for $dnsrecord is $dnsrecordid"

# update the record
curl -s -X PUT "https://api.cloudflare.com/client/v4/zones/$zoneid/dns_records/$dnsrecordid" \
  -H "Authorization: Bearer $cloudflare_auth_key" \
  -H "Content-Type: application/json" \
  --data "{\"type\":\"A\",\"name\":\"$dnsrecord\",\"content\":\"$ip\",\"ttl\":1,\"proxied\":false}" | jq
echo "DNS update completed"

fi
#fi

done

```

