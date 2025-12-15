# Homelab

# Cloudflare linkding tunnel

```bash
cloudflared tunnel create ld # creates the tunnel in cloudflare
cd ~/.cloudflared
k create secret generic tunnel-credentials --from-file=credentials.json={GUID}.json
```
