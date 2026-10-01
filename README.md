# proxy-proton

An HTTP proxy that sends traffic through ProtonVPN over WireGuard, using [Gluetun](https://github.com/qdm12/gluetun). Run it on a server or another machine on your network, then point your browser at it.

## Requirements

- Docker with Docker Compose
- A ProtonVPN account (plans that allow WireGuard configs)
- Linux host with `/dev/net/tun` available

## 1. Get a WireGuard private key from ProtonVPN

1. Sign in at <https://account.protonvpn.com>.
2. Go to **Downloads → WireGuard configuration**.
3. Choose a platform (e.g. *Router* or *Linux*), pick a server, and click **Create**.
4. From the generated config, copy the value of `PrivateKey` from the `[Interface]` section.

## 2. Configure

```sh
cp .env.sample .env
```

Edit `.env`:

| Variable | Description |
| --- | --- |
| `SERVER_COUNTRIES` | Exit country, e.g. `Greece`. Comma-separated for several. |
| `WIREGUARD_PRIVATE_KEY` | The `PrivateKey` from step 1. |
| `PROXY_USER` | Username required to use the proxy. |
| `PROXY_PASSWORD` | Password required to use the proxy. |
| `PROXY_PORT` | Host port for the HTTP proxy, e.g. `8888`. |

`.env` is git-ignored. Never commit it.

## 3. Start

```sh
docker compose up -d
docker logs -f vpn
```

Wait until the logs show the VPN is connected and the public IP is reported.

Check it from another machine (replace the host, user and password):

```sh
curl -x http://USER:PASS@SERVER_IP:8888 https://ipinfo.io
```

The returned IP should be a ProtonVPN address in your chosen country.

## 4. Browser setup

Use an **HTTP** proxy, not SOCKS5. Chromium-based browsers (Chrome, Vivaldi, Edge, Brave) cannot log in to SOCKS5 proxies.

### Chrome / Vivaldi / Edge / Brave — SwitchyOmega

1. Install the **Proxy SwitchyOmega** extension.
2. Open its options and create (or edit) a proxy profile:
   - Protocol: `HTTP`
   - Server: the server's IP (e.g. `192.168.101.40`)
   - Port: your `PROXY_PORT`
3. Click the lock icon and enter `PROXY_USER` / `PROXY_PASSWORD`.
4. Click **Apply changes**, then select the profile from the extension icon.

### Firefox

Settings → **Network Settings** → **Manual proxy configuration**:

- HTTP Proxy: server IP, Port: your `PROXY_PORT`
- Tick **Also use this proxy for HTTPS**

Firefox asks for the username and password on the first request.

### macOS (system-wide)

System Settings → **Network** → your connection → **Details** → **Proxies**: enable **Web Proxy (HTTP)** and **Secure Web Proxy (HTTPS)**, enter the server IP, port and credentials.

Visit <https://ipleak.net> to confirm the IP and DNS belong to ProtonVPN.

## Troubleshooting

| Problem | Fix |
| --- | --- |
| `VPN provider name is not valid for wireguard` | `VPN_SERVICE_PROVIDER=protonvpn` must be set (it is in `docker-compose.yml`). |
| `ERR_SOCKS_CONNECTION_FAILED` | You configured SOCKS5. Switch the browser to HTTP. |
| `ERR_PROXY_CONNECTION_FAILED` | Container isn't running or the port is wrong. Check `docker compose ps` and `docker logs vpn`. |
| Browser keeps asking for a password | `PROXY_USER` / `PROXY_PASSWORD` in `.env` don't match what you entered. |
| VPN won't connect | The private key is wrong or revoked. Generate a new WireGuard config in your Proton account. |

## Update / stop

```sh
docker compose pull && docker compose up -d   # update Gluetun
docker compose down                            # stop
```
