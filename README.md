# 📡 LiveKit Self-Hosted Server (Docker Setup)

Deploy your **own LiveKit signaling and media server** using Docker on Ubuntu.

---

## 🚀 What is LiveKit?

[LiveKit](https://livekit.io) is an open-source WebRTC platform for scalable, real-time audio and video apps.

This guide helps you:

- Host your own LiveKit signaling/media server
- Avoid third-party cloud infrastructure
- Use custom frontends (e.g., LiveKit Meet or React app)

---

## ✅ Requirements

- Ubuntu 20.04 or later (EC2/VPS recommended)
- Docker & Docker Compose
- Public IP address
- Open ports (7880, 7881, 5000–6000 UDP)
- UFW (firewall)

---

## 📦 Step 1: Install Docker & Docker Compose

```bash
sudo apt update
sudo apt install -y docker.io docker-compose curl git
sudo systemctl enable docker
sudo systemctl start docker
```

---

## 🗂️ Step 2: Create Project Directory

```bash
mkdir ~/livekit-server
cd ~/livekit-server
```

---

## 📄 Step 3: Create `livekit.yaml` Configuration

```bash
nano livekit.yaml
```

Paste:

```yaml
port: 7880
rtc:
  port_range_start: 5000
  port_range_end: 6000
  use_external_ip: true
  use_mdns: false
keys:
  devkey: secret
```

---

## ⚙️ Step 4: Create `docker-compose.yml`

```bash
nano docker-compose.yml
```

Paste:

```yaml
version: '3'
services:
  livekit:
    image: livekit/livekit-server:latest
    command: --config /etc/livekit.yaml
    volumes:
      - ./livekit.yaml:/etc/livekit.yaml
    ports:
      - "7880:7880"
      - "7881:7881"
      - "5000-6000:5000-6000/udp"
    restart: always
```

---

## 🔓 Step 5: Open Firewall Ports

### UFW:

```bash
sudo ufw allow 22/tcp
sudo ufw allow 7880/tcp
sudo ufw allow 7881/tcp
sudo ufw allow 5000:6000/udp
sudo ufw enable
```

### EC2 (AWS):

Update EC2 Security Group:

- TCP: 22, 7880, 7881 (source: 0.0.0.0/0)
- UDP: 5000–6000 (source: 0.0.0.0/0)

---

## ▶️ Step 6: Run LiveKit Server

```bash
export COMPOSE_HTTP_TIMEOUT=300
sudo docker-compose up -d --force-recreate
docker ps
```

Test:
```
http://<your-public-ip>:7880
```

---

## ✅ Step 7: Check LiveKit Is Running

```bash
curl http://localhost:7880
```

Expected output:

```json
{"ok":true,"message":"LiveKit is running"}
```

---

## 🧪 Step 8: Debugging Tips

View logs:

```bash
sudo docker-compose logs livekit --tail=50
```

Stop stuck containers:

```bash
sudo docker rm -f livekit-server_livekit_1
```

---

## 🧪 Step 9: Use LiveKit Meet with Your Server

### 1. Clone LiveKit Meet:

```bash
git clone https://github.com/livekit/livekit-meet
cd livekit-meet
curl -fsSL https://get.pnpm.io/install.sh | sh -
export PATH="$HOME/.local/share/pnpm:$PATH"
pnpm install
```

### 2. Create `.env` File:

```env
VITE_LIVEKIT_URL=wss://<your-public-ip>:7881
VITE_API_KEY=devkey
VITE_API_SECRET=secret
```

### 3. Start Development Server:

```bash
pnpm dev
```

Open in browser:

```
http://localhost:5173
```

---

## 🛡️ Production Best Practices

### 🔐 Use HTTPS (SSL)

Use Nginx or Caddy with Let's Encrypt to proxy and secure traffic:
- Ports 80/443 on your server
- Proxy to ports 7880 (HTTP) and 7881 (WS)

### 🔑 Change API Keys

Avoid using `devkey`/`secret` in production:

```yaml
keys:
  my_custom_key: my_super_secret_value
```

Update frontend `.env` accordingly.

### 🔁 Auto-Restart Server

Already handled in Docker with:

```yaml
restart: always
```

---

## 📚 LiveKit Ports Summary

| Port       | Protocol | Description           |
|------------|----------|-----------------------|
| 7880       | TCP      | HTTP API / Dashboard  |
| 7881       | TCP      | WebSocket signaling   |
| 5000–6000  | UDP      | Media transport (WebRTC) |

---

## 🧼 VS Code File Permissions Fix

If you see file permission errors:

```bash
sudo chown -R $(whoami):$(whoami) .
chmod -R u+rw .
```

---

## 🧠 Want to Build Your Own Frontend?

Use LiveKit SDKs:

### React SDK:

```bash
npm install @livekit/components-react
```

### JS SDK:

```bash
npm install @livekit/client
```

### Sample usage:

```js
import { connect } from '@livekit/client';

const room = await connect('wss://your-ip:7881', {
  token: '<generated-token>',
});
```

---

## 🧾 Helpful Commands

```bash
# Stop containers
sudo docker-compose down

# Pull latest image
sudo docker-compose pull

# Restart server
sudo docker-compose up -d

# View running containers
docker ps
```

---

## 📎 Resources

- 🌐 [LiveKit Docs](https://docs.livekit.io/)
- 💻 [LiveKit GitHub](https://github.com/livekit)
- 🧪 [LiveKit Meet](https://github.com/livekit/livekit-meet)

---

## ✅ Done!

🎉 You now have a fully working self-hosted LiveKit server, ready for real-time audio/video communication. Build your own platform confidently!
