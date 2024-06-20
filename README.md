# Cloudflare-DNS-Update

I am running a home webserver, but my ISP does not offer a static IP allocation: periodically I get a new IP address for my domain I bought through Cloudflare. Cloudflare offers a domain management service via the Web: you can add/remove/update the DNS A/AAAA records for the domains(s) you have bought. They also offer a [REST API](https://developers.cloudflare.com/api/) that can be used to do this programatically.

Here is a simple python program to use this; I run on Fedora Linux. When run, the dnsupdate.conf file is read, and the public IP addresses (v4 and v6) are obtained from the [ident.me](https://ident.me) service. The current Cloudflare A and AAAA records' IP addresses are obtained, and if necessary, the records are updated.

A log file (dnsupdate.log) is created that will show errors, and information on what IP information it has obtained, and if it has needed to update the Cloudflare record(s).

## Requirements

- Python 3
- Access to ident.me service (or similar website that returns an IP)

## Installation

The two files should be installed in the same directory (such as /root/dnsupdate):

- Main code: dnsupdate
- Configuration file: dnsupdate.conf

Ensure permissions are appropriate:

```
chmod 700 dnsupdate
chmod 600 dnsupdate.conf
```

Log onto the [Cloudflare dashboard](https://dash.cloudflare.com/), and ensure that A and/or AAAA records exist for your domain. Update the settings in the configuration file:
- key: Generate the production API token from the dashboard under Profile -> API Tokens -> Create Token. It should have Zone.DNS permissions.
- domain: The domain of the A/AAAA records.
- force_update: normally no, but "yes" can be used for testing, where and update is performed even if no change is required.

The [ident.me](ident.me) sites are recommended and are set in the service parameter, but any external site can be used as long as it returns an IP. This can be extracted with the ip_extract regex:

```
[A]
service = https://v4.ident.me
ip_extract = (.*)
```

A symlink can be added so that it is run hourly:

```
ln -s /root/dnsupdate/dnsupdate /etc/cron.hourly/dnsupdate
```

Alernatively, instead of the program checking/updating once and exiting, it can be run in a loop mode by passing the number of mins of sleep time with the "-l" flag:

```
dnsupdate -l 15
```

It may be convenient to run as a systemd service:

Create file /etc/systemd/system/dnsupdate.service:

```
[Unit]
Description=Update External DNS

[Service]
Type=simple
Restart=always
ExecStart=/usr/bin/python3 /root/dnsupdate/dnsupdate -l 15

[Install]
WantedBy=multi-user.target
```

and then enable:

```
systemctl enable dnsupdate
systemctl start dnsupdate
```
