# Lightweight universal DDNS Updater with Docker and web UI

*Light container updating DNS A and/or AAAA records periodically for multiple DNS providers*

[![DDNS Updater by Quentin McGaw](https://github.com/qdm12/ddns-updater/raw/master/readme/title.png)](https://hub.docker.com/r/qmcgaw/ddns-updater)

[![Build status](https://github.com/qdm12/ddns-updater/workflows/Buildx%20latest/badge.svg)](https://github.com/qdm12/ddns-updater/actions?query=workflow%3A%22Buildx+latest%22)
[![Docker Pulls](https://img.shields.io/docker/pulls/qmcgaw/ddns-updater.svg)](https://hub.docker.com/r/qmcgaw/ddns-updater)
[![Docker Stars](https://img.shields.io/docker/stars/qmcgaw/ddns-updater.svg)](https://hub.docker.com/r/qmcgaw/ddns-updater)
[![Image size](https://images.microbadger.com/badges/image/qmcgaw/ddns-updater.svg)](https://microbadger.com/images/qmcgaw/ddns-updater)
[![Image version](https://images.microbadger.com/badges/version/qmcgaw/ddns-updater.svg)](https://microbadger.com/images/qmcgaw/ddns-updater)

[![Join Slack channel](https://img.shields.io/badge/slack-@qdm12-yellow.svg?logo=slack)](https://join.slack.com/t/qdm12/shared_invite/enQtODMwMDQyMTAxMjY1LTU1YjE1MTVhNTBmNTViNzJiZmQwZWRmMDhhZjEyNjVhZGM4YmIxOTMxOTYzN2U0N2U2YjQ2MDk3YmYxN2NiNTc)
[![GitHub last commit](https://img.shields.io/github/last-commit/qdm12/ddns-updater.svg)](https://github.com/qdm12/ddns-updater/issues)
[![GitHub commit activity](https://img.shields.io/github/commit-activity/y/qdm12/ddns-updater.svg)](https://github.com/qdm12/ddns-updater/issues)
[![GitHub issues](https://img.shields.io/github/issues/qdm12/ddns-updater.svg)](https://github.com/qdm12/ddns-updater/issues)

## Features

- Updates periodically A records for different DNS providers:
    - Cloudflare
    - DDNSS.de
    - DigitalOcean
    - DonDominio
    - DNSOMatic
    - DNSPod
    - Dreamhost
    - DuckDNS
    - DynDNS
    - GoDaddy
    - Google
    - He.net
    - Infomaniak
    - LuaDNS
    - Namecheap
    - NoIP
    - OVH
    - Selfhost.de
    - Strato.de
    - **Want more?** [Create an issue for it](https://github.com/qdm12/ddns-updater/issues/new/choose)!
- Web User interface

![Web UI](https://raw.githubusercontent.com/qdm12/ddns-updater/master/readme/webui.png)

- 14MB Docker image based on a Go static binary in a Scratch Docker image with ca-certificates and timezone data
- Persistence with a JSON file *updates.json* to store old IP addresses with change times for each record
- Docker healthcheck verifying the DNS resolution of your domains
- Highly configurable
- Sends notifications to your Android phone, see the [**Gotify**](#Gotify) section (it's free, open source and self hosted 🆒)
- Compatible with `amd64`, `386`, `arm64`, `arm32v7` (Raspberry Pis) CPU architectures.

## Setup

The program reads the configuration from a JSON object, either from a file or from an environment variable.

1. Create a directory of your choice, say *data* with a file named **config.json** inside:

    ```sh
    mkdir data
    touch data/config.json
    # Owned by user ID of Docker container (1000)
    chown -R 1000 data
    # all access (for creating json database file data/updates.json)
    chmod 700 data
    # read access only
    chmod 400 data/config.json
    ```

    *(You could change the user ID, for example with `1001`, by running the container with `--user=1001`)*

1. Write a JSON configuration in *data/config.json*, for example:

    ```json
    {
        "settings": [
            {
                "provider": "namecheap",
                "domain": "example.com",
                "host": "@",
                "password": "e5322165c1d74692bfa6d807100c0310"
            }
        ]
    }
    ```

    You can find more information in the [configuration section](#configuration) to customize it.

1. Run the container with

    ```sh
    docker run -d -p 8000:8000/tcp -v "$(pwd)"/data:/updater/data qmcgaw/ddns-updater
    ```

1. (Optional) You can also set your JSON configuration as a single environment variable line (i.e. `{"settings": [{"provider": "namecheap", ...}]}`), which takes precedence over config.json. Note however that if you don't bind mount the `/updater/data` directory, there won't be a persistent database file `/updater/updates.json` but it will still work.

### Next steps

You can also use [docker-compose.yml](https://github.com/qdm12/ddns-updater/blob/master/docker-compose.yml) with:

```sh
docker-compose up -d
```

You can update the image with `docker pull qmcgaw/ddns-updater`. Other [Docker image tags are available](https://hub.docker.com/repository/docker/qmcgaw/ddns-updater/tags).

## Configuration

Start by having the following content in *config.json*, or in your `CONFIG` environment variable:

```json
{
    "settings": [
        {
            "provider": "",
        },
        {
            "provider": "",
        }
    ]
}
```

For each setting, you need to fill in parameters.
Check the documentation for your DNS provider:

- [Cloudflare](docs/cloudflare.md)
- [DDNSS.de](docs/ddnss.de.md)
- [DigitalOcean](docs/digitalocean.md)
- [DonDominio](docs/dondominio.md)
- [DNSOMatic](docs/dnsomatic.md)
- [DNSPod](docs/dnspod.md)
- [Dreamhost](docs/dreamhost.md)
- [DuckDNS](docs/duckdns.md)
- [DynDNS](docs/dyndns.md)
- [DynV6](docs/dynv6.md)
- [GoDaddy](docs/godaddy.md)
- [Google](docs/google.md)
- [He.net](docs/he.net.md)
- [Infomaniak](docs/infomaniak.md)
- [LuaDNS](docs/luadns.md)
- [Namecheap](docs/namecheap.md)
- [NoIP](docs/noip.md)
- [OVH](docs/ovh.md)
- [Selfhost.de](docs/selfhost.de.md)
- [Strato.de](docs/strato.md)

Note that:

- you can specify multiple hosts for the same domain using a comma separated list. For example with `"host": "@,subdomain1,subdomain2",`.

### Environment variables

| Environment variable | Default | Description |
| --- | --- | --- |
| `CONFIG` | | One line JSON object containing the entire config (takes precendence over config.json file) if specified |
| `PERIOD` | `5m` | Default period of IP address check, following [this format](https://golang.org/pkg/time/#ParseDuration) |
| `IP_METHOD` | `cycle` | Method to obtain the public IP address (ipv4 or ipv6). See the [IP Methods section](#IP-methods) |
| `IPV4_METHOD` | `cycle` | Method to obtain the public IPv4 address only. See the [IP Methods section](#IP-methods) |
| `IPV6_METHOD` | `cycle` | Method to obtain the public IPv6 address only. See the [IP Methods section](#IP-methods) |
| `HTTP_TIMEOUT` | `10s` | Timeout for all HTTP requests |
| `LISTENING_PORT` | `8000` | Internal TCP listening port for the web UI |
| `ROOT_URL` | `/` | URL path to append to all paths to the webUI (i.e. `/ddns` for accessing `https://example.com/ddns` through a proxy) |
| `BACKUP_PERIOD` | `0` | Set to a period (i.e. `72h15m`) to enable zip backups of data/config.json and data/updates.json in a zip file |
| `BACKUP_DIRECTORY` | `/updater/data` | Directory to write backup zip files to if `BACKUP_PERIOD` is not `0`.
| `LOG_ENCODING` | `console` | Format of logging, `json` or `console` |
| `LOG_LEVEL` | `info` | Level of logging, `info`, `warning` or `error` |
| `GOTIFY_URL` |  | (optional) HTTP(s) URL to your Gotify server |
| `GOTIFY_TOKEN` |  | (optional) Token to access your Gotify server |
| `TZ` | | Timezone to have accurate times, i.e. `America/Montreal` |

#### IP methods

By default, all ip methods are cycled through between all ip methods available for the specified ip version, if any. This allows you not to be blocked for making too many requests. You can otherwise pick one of the following.

- IPv4 or IPv6 (for most cases)
  - `opendns` using [https://diagnostic.opendns.com/myip](https://diagnostic.opendns.com/myip)
  - `ifconfig` using [https://ifconfig.io/ip](https://ifconfig.io/ip)
  - `ipinfo` using [https://ipinfo.io/ip](https://ipinfo.io/ip)
  - `ipify` using [https://api.ipify.org](https://api.ipify.org)
  - `ddnss` using [https://ddnss.de/meineip.php](https://ddnss.de/meineip.php)
  - `google` using [https://domains.google.com/checkip](https://domains.google.com/checkip)
- IPv4 only (useful for updating both ipv4 and ipv6)
  - `ipify` using [https://api.ipify.org](https://api.ipify.org)
  - `ddnss4` using [https://ip4.ddnss.de/meineip.php](https://ip4.ddnss.de/meineip.php)
  - `noip4` using [http://ip1.dynupdate.no-ip.com](http://ip1.dynupdate.no-ip.com)
  - `noip8245_4` using [http://ip1.dynupdate.no-ip.com:8245](http://ip1.dynupdate.no-ip.com:8245)
- IPv6 only
  - `ipify6` using [https://api6.ipify.org](https://api6.ipify.org)
  - `ddnss6` using [https://ip6.ddnss.de/meineip.php](https://ip6.ddnss.de/meineip.php)
  - `noip6` using [http://ip1.dynupdate6.no-ip.com](http://ip1.dynupdate6.no-ip.com)
  - `noip8245_6` using [http://ip1.dynupdate6.no-ip.com:8245](http://ip1.dynupdate6.no-ip.com:8245)

You can also specify an HTTPS URL to obtain your public IP address (i.e. `-e IPV6_METHOD=https://ipinfo.io/ip`)

### Host firewall

If you have a host firewall in place, this container needs the following ports:

- TCP 443 outbound for outbound HTTPS
- TCP 80 outbound if you use a local unsecured HTTP connection to your Gotify server
- UDP 53 outbound for outbound DNS resolution
- TCP 8000 inbound (or other) for the WebUI

## Gotify

[![Gotify](https://github.com/qdm12/ddns-updater/blob/master/readme/gotify.png?raw=true)](https://gotify.net)

[**Gotify**](https://gotify.net) is a simple server for sending and receiving messages, and it is **free**, **private** and **open source**

- It has an [Android app](https://play.google.com/store/apps/details?id=com.github.gotify) to receive notifications
- The app does not drain your battery 👍
- The notification server is self hosted, see [how to set it up with Docker](https://gotify.net/docs/install)
- The notifications only go through your own server (ideally through HTTPS though)

To set it up with DDNS updater:

1. Go to the Web GUI of Gotify
1. Login with the admin credentials
1. Create an app and copy the generated token to the environment variable `GOTIFYTOKEN` (for this container)
1. Set the `GOTIFYURL` variable to the URL of your Gotify server address (i.e. `http://127.0.0.1:8080` or `https://bla.com/gotify`)

## Testing

- The automated healthcheck verifies all your records are up to date [using DNS lookups](https://github.com/qdm12/ddns-updater/blob/master/internal/healthcheck/healthcheck.go#L15)
- You can also manually check, by:
    1. Going to your DNS management webpage
    1. Setting your record to `127.0.0.1`
    1. Run the container
    1. Refresh the DNS management webpage and verify the update happened

## Development and contributing

- [Contribute with code](docs/contributing.md)
- [Github workflows to know what's building](https://github.com/qdm12/ddns-updater/actions)
- [List of issues and feature requests](https://github.com/qdm12/ddns-updater/issues)
- [Kanban board](https://github.com/qdm12/ddns-updater/projects/1)

## License

This repository is under an [MIT license](https://github.com/qdm12/ddns-updater/master/license)

## Used in external projects

- [Starttoaster/docker-traefik](https://github.com/Starttoaster/docker-traefik#home-networks-extra-credit-dynamic-dns)

## Support

Sponsor me on [Github](https://github.com/sponsors/qdm12) or donate to [paypal.me/qmcgaw](https://www.paypal.me/qmcgaw)

[![https://github.com/sponsors/qdm12](https://raw.githubusercontent.com/qdm12/private-internet-access-docker/master/doc/sponsors.jpg)](https://github.com/sponsors/qdm12)
[![https://www.paypal.me/qmcgaw](https://raw.githubusercontent.com/qdm12/private-internet-access-docker/master/doc/paypal.jpg)](https://www.paypal.me/qmcgaw)

Many thanks to J. Famiglietti for supporting me financially 🥇👍
