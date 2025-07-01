
# 📡 LiveKit Self-Hosted Server (Docker Setup)

This guide helps you deploy your **own LiveKit signaling and media server** using Docker, without relying on LiveKit Cloud.

---

## 🚀 What is LiveKit?

[LiveKit](https://livekit.io) is an open-source WebRTC platform for building scalable, real-time audio and video applications.

With this setup, you can:

- Host your own signaling/media server.
- Use custom frontends (like LiveKit Meet or your own React app).
- Avoid third-party cloud signaling infrastructure.

---

## ⚙️ Requirements

- Ubuntu server (e.g., EC2, VPS)
- Docker & Docker Compose
- Public IP address
- Open ports (7880, 7881, 5000–6000 UDP)

---

## 📦 Step 1: Install Docker

\`\`\`bash
sudo apt update
sudo apt install docker.io docker-compose -y
sudo systemctl enable docker
\`\`\`

---

## 🛠️ Step 2: Create `docker-compose.yml`

\`\`\`yaml
version: '3'
services:
  livekit:
    image: livekit/livekit-server:latest
    command:
      - --port=7880
      - --rtc.use_mdns=false
      - --rtc.port_range_start=5000
      - --rtc.port_range_end=6000
      - --rtc.use_external_ip=true
      - --rtc.enable_stun=true
      - --keys.devkey=secret
    ports:
      - "7880:7880"                     # HTTP API
      - "7881:7881"                     # WebSocket signaling
      - "5000-6000:5000-6000/udp"       # WebRTC media transport
\`\`\`

---

## 🔓 Step 3: Open Firewall Ports

### UFW (Ubuntu):
\`\`\`bash
sudo ufw allow 7880/tcp
sudo ufw allow 7881/tcp
sudo ufw allow 5000:6000/udp
\`\`\`

### EC2 (AWS):
Add inbound rules for:
- TCP: 7880, 7881
- UDP: 5000–6000

---

## ▶️ Step 4: Run LiveKit Server

\`\`\`bash
sudo docker-compose up -d
docker ps   # Confirm container is running
\`\`\`

Access the LiveKit server at:
\`\`\`
http://<your-public-ip>:7880
\`\`\`

---

## 🧪 Step 5: Use LiveKit Meet with Your Server

### 1. Clone LiveKit Meet:
\`\`\`bash
git clone https://github.com/livekit/livekit-meet
cd livekit-meet
pnpm install  # or npm install
\`\`\`

### 2. Configure `.env`:

Create a `.env` file in the project root:

\`\`\`env
VITE_LIVEKIT_URL=wss://<your-public-ip>:7881
VITE_API_KEY=devkey
VITE_API_SECRET=secret
\`\`\`

### 3. Start LiveKit Meet:
\`\`\`bash
pnpm dev
\`\`\`

Open in browser:
\`\`\`
http://localhost:5173
\`\`\`

You’ll now be using your **own LiveKit server**!

---

## 🛡️ Production Tips

### ✅ Use SSL (HTTPS)
Use a reverse proxy like **Nginx** or **Caddy** and install SSL via Let’s Encrypt.

### ✅ Change API Keys
Do not use `devkey` and `secret` in production:
\`\`\`yaml
--keys.my_custom_key=my_super_secret_value
\`\`\`

### ✅ Auto-Restart
For uptime:
\`\`\`yaml
restart: always
\`\`\`

---

## 📚 LiveKit Ports Summary

| Port | Protocol | Description |
|------|----------|-------------|
| 7880 | TCP      | HTTP API / dashboard |
| 7881 | TCP      | WebSocket signaling |
| 5000–6000 | UDP | Media (WebRTC) |

---

## 🧼 Fixing File Permissions (VS Code Issue)

If you get permission denied errors in VS Code after cloning or editing:

\`\`\`bash
sudo chown -R $(whoami):$(whoami) .
chmod -R u+rw .
\`\`\`

---

## 📎 Resources

- [LiveKit Docs](https://docs.livekit.io/)
- [LiveKit GitHub](https://github.com/livekit)
- [LiveKit Meet](https://github.com/livekit/livekit-meet)

---

## 🧠 Want to Build Your Own Frontend?

Use LiveKit SDKs:

- React: [\`@livekit/components-react\`](https://docs.livekit.io/client-sdk/react/)
- JS/TS: [\`@livekit/client\`](https://docs.livekit.io/client-sdk/js/)

\`\`\`js
const room = await connect('wss://your-ip:7881', {
  token: '<generated-token>',
});
\`\`\`

Need help building it? Ping me anytime!

---

## ✅ Done!

You now have a fully working self-hosted LiveKit server ready to handle scalable, low-latency real-time audio/video streams 🚀
