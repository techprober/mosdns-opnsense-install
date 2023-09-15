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

## Project Owner

Copyright 2023 @TechProber. All rights reserved.

Maintainer: [Kevin Yu (@yqlbu)](https://github.com/yqlbu)

## Related Projects

- [techprober/mosdns-lxc-deploy](https://github.com/techprober/mosdns-lxc-deploy) - Deploy mosdns in Proxmox LXC Container
- [IrineSistiana/mosdns](https://github.com/IrineSistiana/mosdns) - A self-hosted DNS resolver
- [tteck/Proxmox](https://github.com/tteck/Proxmox) - Proxmox Helper Scripts
- [Loyalsoldier/v2ray-rules-dat](https://github.com/Loyalsoldier/v2ray-rules-dat) - Enhanced edition of V2Ray rules dat files, compatible with Xray-core, Shadowsocks-windows, Trojan-Go and leaf.
- [Loyalsoldier/geoip](https://github.com/Loyalsoldier/geoip) - Enhanced edition of GeoIP files for V2Ray, Xray-core, Trojan-Go, Clash and Leaf, with replaced CN IPv4 CIDR available from ipip.net, appended CIDR lists and more.

## Steps to deploy

### Preparation

Create a new directory for mosdns

```bash
mkdir -p /etc/usr/local/mosdns
```

Create sub directories

```bash
mkdir -p /usr/local/etc/mosdns/{ips,domains,downloads,custom}
touch /usr/local/etc/mosdns/cache.dump
```

Make sure you have the following file structure present on your host:

```
# /usr/local/etc/mosdns
./
|-- cache.dump
|-- config.yml
|-- custom
|-- domains
|-- downloads
|-- scripts
`-- ips

5 directories, 2 files
```

Install Vim (Optional)

```bash
sudo pkg install vim
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

> [!NOTE]
> You may selectively download the rule lists you need from the [release branch](https://github.com/techprober/v2ray-rules-dat/tree/release) from [@techprober/v2ray-rules-dat](https://github.com/techprober/v2ray-rules-dat/releases).

```bash
cd /usr/local/etc/mosdns
curl --progress-bar -JL -o $MOSDNS_PATH/downloads/geoip.zip https://github.com/techprober/v2ray-rules-dat/raw/release/geoip.zip
curl --progress-bar -JL -o $MOSDNS_PATH/downloads/geosite.zip https://github.com/techprober/v2ray-rules-dat/raw/release/geosite.zip
unzip -o $MOSDNS_PATH/downloads/geoip.zip -d $MOSDNS_PATH/ips
unzip -o $MOSDNS_PATH/downloads/geosite.zip -d $MOSDNS_PATH/domains
```

> [!NOTE]
> Alternatively, you may use a dedicated script to automatically download and extract the geodata artifacts. See [./scripts/geodata-update.sh](./scripts/geodata-update.sh)

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

### Enable mosdns service

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
> To write logs to a file, you need to specify the log file destination in your config as shown in the following:

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

## Set up cron job to update geodata artifacts

### Add geodata-update script

The script is available in [./scripts/geodata-update.sh](./scripts/geodata-update.sh).

Download save it in `/usr/local/etc/mosdns/scripts/`

```bash
curl -L -o /usr/local/etc/mosdns/scripts/geodata-update.sh https://github.com/techprober/mosdns-opnsense-install/raw/geodata-update/scripts/geodata-update.sh
```

Set permission

```bash
chmod +x /usr/local/etc/mosdns/scripts/geodata-update.sh
```

### Add a new cron command available under OPNsense GUI

Create a `.conf `file in `/usr/local/opnsense/service/conf/actions.d/` (your file must start with `actions_`)
`vi /usr/local/opnsense/service/conf/actions.d/actions_mosdns-geodata-update.conf`

```conf
[reload]
command:/bin/sh /usr/local/etc/mosdns/scripts/geodata-update.sh
parameter:
type:script_output
message: Mosdns Geodata Update
description: Centralized Geodata Update for Mosdns DNS Service
```

Restart and reload

```bash
service configd restart
configctl mosdns-geodata-update reload
```

### Add a cron job

Go to `System` > `Settings` > `Cron` and `Add a Job`
You can show your cron command in dropdown Command. Plan your cron schedule as you wish.

## Forward requests to designated gateways

> [!NOTE]
> For those who would like to further forward DNS requests to designated gateways, depending on the DNS provider of choice, you may achieve so following the route setting below.

![CleanShot 2023-09-14 at 22 58 10@2x](https://github.com/techprober/mosdns-opnsense-install/assets/31861128/c681317c-ecd1-43a9-b441-8a56be95f6da)

## Appendix

- Auto-generate `geoip.txt`, `geosites.txt` (since `*.dat` are deprecated in v5) - https://github.com/techprober/v2dat
- Available Rules - https://github.com/techprober/v2ray-rules-dat/releases
