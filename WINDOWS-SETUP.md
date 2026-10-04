# Pi-hole + Tailscale on Windows (no router access needed)

Flow: phone (Tailscale) -> encrypted tunnel -> laptop (Tailscale) -> Pi-hole container -> upstream DNS.

1. **Tailscale**: install on the PC and the phone, sign in with the same account.
   ```powershell
   winget install Tailscale.Tailscale
   tailscale ip -4
   ```
2. **WSL2 + Docker Desktop** (admin PowerShell, reboot after the first command):
   ```powershell
   wsl --install
   winget install Docker.DockerDesktop
   docker --version; docker compose version
   ```
   Enable "Start Docker Desktop when you sign in".
3. **Config**: `copy .env.example .env`, set `TS_IP` (from step 1) and a Pi-hole password.
4. **Start**: `docker compose up -d`, then `docker compose logs pihole`.
5. **Test locally**: `http://127.0.0.1:8080/admin`; `nslookup doubleclick.net 100.x.y.z` (your TS_IP) should return 0.0.0.0.
6. **Tailscale admin console** (login.tailscale.com/admin/dns): add a custom nameserver = `TS_IP`, turn on "Override DNS servers".
7. **Phone**: Tailscale on, mobile data. Visit an ad-heavy site, then watch the Pi-hole query log.
8. **Optional full tunnel**: `tailscale up --advertise-exit-node`, approve the node in the admin console, then pick it as the exit node on the phone.
9. **Keep it awake**: Settings > System > Power > Sleep = Never while plugged in.

## Hardening claims for the README
- No port forwarded; no inbound port open to the internet at all (Tailscale uses outbound connections).
- Pi-hole bound to the Tailscale IP only, not the LAN.
- `.env` git-ignored.
- Evidence: nmap of the public IP shows nothing open; an nmap from another LAN device against the laptop's LAN IP shows 53/8080 closed.

## Limits
- The laptop sleeping or rebooting stops ad blocking; a Pi or mini PC fixes this.
- Relies on Tailscale's coordination server (can be replaced with Headscale).
- `latest` Pi-hole tag is unpinned; pin a version on a real server.
