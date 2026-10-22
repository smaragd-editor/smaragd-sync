# Getting started with Smaragd Sync

Smaragd Sync keeps a [Smaragd](https://github.com/smaragd-editor/smaragd) project
identical across your own devices, end-to-end encrypted. This page is the shortest
path to a working setup. For the full picture — configuration, backups, TLS,
troubleshooting — see the server's own
[self-hosting guide](smaragd-sync-server/README.md).

> **Experimental.** Back up your projects independently; don't treat the server as
> the only copy of anything.

## What you need

- A machine you control to run the server on — a home server, a VPS, a Raspberry
  Pi. It only needs to be reachable from your own devices.
- Docker on that machine (or a Rust toolchain, if you'd rather build the binary —
  see [Running without Docker](smaragd-sync-server/README.md#running-without-docker)).
- Smaragd itself on each device you want to sync.

## 1. Start the server

Generate an admin token — you'll need it in step 2 — and run the server:

```sh
TOKEN="$(openssl rand -hex 24)"
echo "$TOKEN"   # save it somewhere safe, e.g. a password manager

docker run -d --name smaragd-sync --restart unless-stopped \
  -p 127.0.0.1:8080:8080 \
  -v smaragd-sync-data:/data \
  -e SMARAGD_SYNC_ADMIN_TOKEN="$TOKEN" \
  ghcr.io/smaragd-editor/smaragd-sync-server:latest
```

This binds to localhost only. To reach it from other devices, either put it behind
a TLS reverse proxy (recommended — see
[Putting it behind TLS](smaragd-sync-server/README.md#putting-it-behind-tls)) or,
on a network you fully trust, publish it on that network's address instead, e.g.
`-p 192.168.1.10:8080:8080`.

Check it's up:

```sh
curl http://localhost:8080/v1/health
# {"status":"ok","version":"0.1.0"}
```

### Alternative: reach it over Tailscale instead

If you'd rather not set up a reverse proxy and certificate yourself, or open a port
on your router, [Tailscale](https://tailscale.com) is a quick way to make the
server reachable from your own devices wherever they are. It puts them on one
private, WireGuard-encrypted network (a "tailnet").

1. Install Tailscale on the machine running the server and on every device that
   runs Smaragd, and sign them all into the same tailnet.
2. With the server bound to `127.0.0.1:8080` as above, give it an HTTPS address in
   one command:

   ```sh
   tailscale serve --bg --https=443 localhost:8080
   ```

   This provisions a real TLS certificate for the machine's MagicDNS name (e.g.
   `my-server.tailnet-name.ts.net`) and proxies it to the server. `tailscale serve
   status` shows what's running; `tailscale serve reset` clears it.
3. In Smaragd's Sync settings, enter that MagicDNS name as the **Host**, leave
   **Port** at `0`, and keep **Use HTTPS (TLS)** on.

Only devices on your tailnet can reach the server this way — nothing is exposed to
the public internet.

## 2. Connect your first device

In Smaragd, go to **File → Settings → Sync** and fill in the server's host, port
and whether it uses TLS, plus an encryption **passphrase** of your choosing —
you'll enter this same passphrase on every device, and it never leaves your
devices. **Test Connection** confirms Smaragd can reach the server.

Open your project, then **Tools → Sync Panel** (`Ctrl+Shift+Y`) and press
**Create Vault…**. Since the server only lets its administrator create vaults by
default, Smaragd asks for the admin token from step 1 — needed only this once.

## 3. Add more devices

On the device that's already syncing, open the Sync panel and press
**Make Pairing Ticket**. Send that ticket to your other device (chat, email,
whatever suits) — it works once and expires after 10 minutes.

On the other device: install Smaragd, repeat step 2's Settings page with the
*same* passphrase, open the project folder, open the Sync panel, press
**Join Vault…**, and paste the ticket.

## Where to go next

- The **Sync** chapter of the
  [Smaragd user manual](https://smaragd-editor.github.io/smaragd/manual/sync.html)
  covers the client side in full: the Sync panel, syncing images and PDFs, how
  edits from two devices are merged, passphrases, and troubleshooting.
- The [server README](smaragd-sync-server/README.md) covers the operator side:
  configuration options, running without Docker (including systemd), TLS, backups,
  upgrades, limits, maintenance and admin commands.
