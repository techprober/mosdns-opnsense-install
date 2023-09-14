<h1 align="center">mosdns-opnsense-deploy</h1>
<p align="center">
    <em>A generic guide to deploy mosdns to OPNSense</em>
</p>

<p align="center">
    <img src="https://custom-icon-badges.herokuapp.com/github/license/techprober/mosdns-opnsense-install?logo=law&color=critical" alt="License"/>
    <img src="https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fgithub.com%2Ftechprober%2Fmosdns-opnsense-deploy&count_bg=%231D86BF&title_bg=%23555555&icon=&icon_color=%23753131&title=hits&edge_flat=false"/>
    <img src="https://custom-icon-badges.herokuapp.com/github/v/release/IrineSistiana/mosdns?logo=rocket" alt="version">
    <img src="https://custom-icon-badges.herokuapp.com/github/issues-pr-closed/TechProber/mosdns-opnsense-install?color=purple&logo=git-pull-request&logoColor=white"/>
    <img src="https://custom-icon-badges.herokuapp.com/github/last-commit/TechProber/mosdns-opnsense-install?logo=history&logoColor=white" alt="lastcommit"/>
</p>

## Introduction

This repo provides a generic guide to deploy mosdns to OPNSense with ease. However, it requires users to have some fundamental knowledge about OPNSense and mosdns.

## Documentation

Mosdns Official Wiki: <https://irine-sistiana.gitbook.io/mosdns-wiki/>

Know DNS Providers: <https://adguard-dns.io/kb/general/dns-providers/>

## Related Projects

- [techprober/mosdns-lxc-deploy](https://github.com/techprober/mosdns-lxc-deploy) - Deploy mosdns in Proxmox LXC Container
- [IrineSistiana/mosdns](https://github.com/IrineSistiana/mosdns) - A self-hosted DNS resolver
- [tteck/Proxmox](https://github.com/tteck/Proxmox) - Proxmox Helper Scripts
- [Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat) - Enhanced edition of V2Ray rules dat files, compatible with Xray-core, Shadowsocks-windows, Trojan-Go and leaf.
- [Loyalsoldier/geoip](https://github.com/Loyalsoldier/geoip) - Enhanced edition of GeoIP files for V2Ray, Xray-core, Trojan-Go, Clash and Leaf, with replaced CN IPv4 CIDR available from ipip.net, appended CIDR lists and more.

## Steps to deploy

### Create dirs

```bash
mkdir -p /usr/local/etc/mosdns/{ips,domains,downloads,custom}
touch /usr/local/etc/mosdns/cache.dump
```

### Download binary from GitHub release page

https://github.com/IrineSistiana/mosdns/releases

```bash
cd /usr/local/etc/mosdns/downloads
curl -o mosdns.zip https://github.com/IrineSistiana/mosdns/releases/download/{VERSION}/mosdns-freebsd-amd64.zip
unzip mosdns.zip
sudo install -Dm755 mosdns /usr/bin/
```

### Create log file

```bash
touch /var/log/mosdns.log
```

### Download geodata artifacts

Reference: https://github.com/techprober/mosdns-lxc-deploy

Artifacts Source: https://github.com/techprober/v2ray-rules-dat/releases

```bash
cd /usr/local/etc/mosdns
LATEST_TAG=$(curl https://api.github.com/repos/techprober/v2ray-rules-dat/releases/latest --silent |  jq -r ".tag_name)
wget -O ./downloads/geoip.zip https://github.com/techprober/v2ray-rules-dat/releases/download/$LATEST_TAG/geoip.zip
wget -O ./downloads/geosite.zip https://github.com/techprober/v2ray-rules-dat/releases/download/$LATEST_TAG/geosite.zip
unzip ./downloads/geoip.zip -d ./ips/
unzip ./downloads/geosite.zip -d ./domains/
```

### Disable Unbound service

> [!WARNING]
> Doing so will free port `53` for mosdns to use

```bash
/usr/local/sbin/pluginctl dns stop
/usr/local/sbin/pluginctl dns disable
```

### Create mosdns rc service

Paste the content from [./rc.d/mosdns](./rc.d/mosdns) in this repo to `/usr/local/etc/rc.d/mosdns` in OPNSense.

```bash
sudo chmod +x /usr/local/etc/rc.d/mosdns
```

### Create mosdns config

> [!NOTE]
> You may start with the recommended [config](https://github.com/techprober/mosdns-lxc-deploy/blob/master/mosdns/config-v5.yml), which provides out-of-the-box ip leak prevent feature.

### Enalbe mosdns service

```bash
echo 'mosdns_enable="YES"' >> /etc/rc.conf
service mosdns start
service enable mosdns
```

### Verify running status

```bash
ps -aux | grep mosdns
service mosdns status
```

### Check journal logs

> [!IMPORTANT]
> To write logs to a file, you need to sepcify the log file destination in your config as shown in the follwing:

```yaml
## -- Log Config -- ##
log:
  level: debug # ["debug", "info", "warn", and "error"], default is set to "info"
  production: true
  file: "/var/log/mosdns.log"
```

```bash
sudo tail -f /var/log/mosdns.log
```

## Appendix

- Auto generate `geoip.txt`, `geosites.txt` (since `*.dat` are deprecated in v5) - https://github.com/techprober/v2dat
- Available Rules - https://github.com/techprober/v2ray-rules-dat/releases