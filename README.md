<div align="center">

# MediaStream

### Real-Time Video Conferencing with Mediasoup SFU

A production-ready WebRTC conferencing application built on [mediasoup](https://mediasoup.org) — a Selective Forwarding Unit (SFU) that delivers low-latency, high-quality audio/video streaming for multi-party rooms.

[![Mediasoup](https://img.shields.io/badge/mediasoup-3.26.0-009688?logo=data:image/svg+xml;base64,&logoColor=white)](https://mediasoup.org)
[![Node.js](https://img.shields.io/badge/Node.js-%3E%3D22-339933?logo=node.js&logoColor=white)](https://nodejs.org)
[![Socket.IO](https://img.shields.io/badge/Socket.IO-4.8.3-010101?logo=socket.io&logoColor=white)](https://socket.io)
[![License](https://img.shields.io/badge/license-ISC-blue.svg)](LICENSE)

</div>

---

## Overview

MediaStream is a full-featured video conferencing platform that leverages the **mediasoup SFU architecture** to route real-time audio and video streams between participants. Unlike peer-to-peer mesh networks that struggle beyond 3–4 participants, the SFU model efficiently scales to larger rooms by receiving each participant's stream once and forwarding it to all others.

### Key Capabilities

| Feature | Description |
|---|---|
| **Multi-party Video** | HD video conferencing with dynamic participant grids (up to 6 per page, paginated carousel) |
| **Audio Streaming** | Crystal-clear Opus codec audio with mute/unmute controls |
| **Screen Sharing** | Share your entire screen or a specific application window with the room |
| **Simulcast** | VP8 simulcast with 3 spatial layers (r0/r1/r2) for adaptive bitrate streaming |
| **Dynamic Room Creation** | Rooms are created on-demand and automatically cleaned up when empty |
| **Responsive UI** | Bootstrap 5-powered interface that adapts to desktop and mobile |
| **Secure by Design** | HTTPS with self-signed SSL certificates; AES-GCM encrypted room links |
| **Social Sharing** | One-click sharing to WhatsApp, Facebook, Twitter, LinkedIn, and Gmail |

---

## Architecture

```
                    ┌─────────────────────────────────────────┐
                    │              MediaStream Server           │
                    │                                          │
                    │   ┌──────────┐   ┌──────────────────┐   │
                    │   │ Express  │   │   Socket.IO      │   │
                    │   │ (HTTPS)  │   │   (Signaling)     │   │
                    │   └────┬─────┘   └────────┬─────────┘   │
                    │        │                  │              │
                    │   ┌────▼──────────────────▼─────────┐   │
                    │   │        Mediasoup Worker          │   │
                    │   │   ┌──────────────────────────┐   │   │
                    │   │   │       Router (Room)      │   │   │
                    │   │   │  ┌─────────┐ ┌────────┐  │   │   │
                    │   │   │  │ Producer│ │Consumer│  │   │   │
                    │   │   │  │Transport│ │Transport│ │   │   │
                    │   │   │  └────┬────┘ └───┬────┘  │   │   │
                    │   │   │       │          │        │   │   │
                    │   │   │  ┌────▼────┐ ┌───▼────┐   │   │   │
                    │   │   │  │Producer │ │Consumer│   │   │   │
                    │   │   │  └─────────┘ └────────┘   │   │   │
                    │   │   └──────────────────────────┘   │   │
                    │   └──────────────────────────────────┘   │
                    └─────────────────────────────────────────┘
                         │                        │
                    ┌────▼────┐              ┌────▼────┐
                    │ Peer A  │              │ Peer B  │
                    │ (Web)   │              │ (Web)   │
                    └─────────┘              └─────────┘
```

### Tech Stack

- **SFU Engine:** [mediasoup 3.26](https://github.com/versatica/mediasoup) — High-performance WebRTC SFU in C++/Rust with Node.js bindings
- **Signaling:** [Socket.IO 4.8](https://socket.io) — Real-time bidirectional event-based communication
- **Web Server:** [Express 4.22](https://expressjs.com) with HTTPS via [httpolyglot](https://github.com/http-party/node-httpolyglot)
- **Client Bundler:** [esbuild](https://esbuild.github.io) — Lightning-fast JS bundler
- **Templating:** [EJS](https://ejs.co) for landing pages, Terms & Privacy Policy
- **Frontend:** Bootstrap 5, Font Awesome, vanilla JavaScript with mediasoup-client 3.23

---

## Getting Started

### Prerequisites

| Requirement | Version | Notes |
|---|---|---|
| **Node.js** | >= 22.x | Required by mediasoup 3.26 and mediasoup-client 3.23 |
| **npm** | >= 10.x | Comes bundled with Node.js |
| **Python 3** | >= 3.8 | Only needed if mediasoup builds from source (prebuilt binary preferred) |
| **C Compiler** | gcc/clang | Only needed for source builds |

> **Tip:** Use [nvm](https://github.com/nvm-sh/nvm) to manage Node versions:
> ```bash
> nvm install 22 && nvm use 22
> ```

### Installation

```bash
# Clone the repository
git clone https://github.com/Sam15b/mediasoup.git
cd mediasoup

# Install dependencies
npm install

# Generate SSL certificates (required for WebRTC)
mkdir -p server/ssl
openssl req -x509 -newkey rsa:2048 -keyout server/ssl/key.pem \
  -out server/ssl/cert.pem -days 365 -nodes \
  -subj "/CN=localhost"
```

### Configuration

Update the `announcedIp` in `app.js` to match your machine's IP address:

```js
// app.js → createWebRtcTransport()
listenIps: [
  {
    ip: '0.0.0.0',
    announcedIp: 'YOUR_LOCAL_IP',  // e.g., '192.168.1.100' or '127.0.0.1'
  }
]
```

> Find your IP with `ifconfig` (macOS/Linux) or `ipconfig` (Windows).

### Running

```bash
# Start the server (with auto-reload via nodemon)
npm start

# In a separate terminal, build the client bundle (watch mode)
npm run watch

# Or build once
npm run build
```

Open your browser at:

```
https://localhost:5000
```

> Your browser will warn about the self-signed certificate. Click **Advanced > Proceed** to continue.

---

## Usage Guide

### Creating a Room

1. Navigate to `https://localhost:5000`
2. Enter your name and a room ID (or use the auto-generated link)
3. Share the invitation link via WhatsApp, Facebook, Twitter, LinkedIn, or Email

### In-Call Controls

| Control | Icon | Action |
|---|---|---|
| **Microphone** | `fa-microphone` | Toggle audio on/off (pause/resume producer) |
| **Camera** | `fa-video` | Toggle video on/off (pause/resume producer) |
| **Screen Share** | `fa-desktop` | Share your screen with the room |
| **End Call** | `fa-phone-slash` | Leave the conference |

### How Camera Toggle Works

The application uses mediasoup's **pause/resume + replaceTrack** pattern instead of close/recreate:

- **Turn off camera:** The video producer is `pause()`d and the camera track is `stop()`ped (releases the device). The WebRTC transport and m-section stay stable.
- **Turn on camera:** A new `getUserMedia` track is created and swapped in via `replaceTrack()`, then the producer is `resume()`d. No SDP renegotiation occurs, avoiding RTP extension conflicts.

This is the recommended mediasoup pattern for toggling media without destabilizing the WebRTC connection.

---

## Project Structure

```
mediasoup/
├── app.js                  # Server entry point — Express + Socket.IO + Mediasoup
├── package.json            # Dependencies and scripts
├── .gitignore
│
├── server/
│   └── ssl/                # SSL certificates (gitignored — generate your own)
│       ├── key.pem
│       └── cert.pem
│
├── public/                 # Static assets served to the client
│   ├── index.js            # Client-side application source (mediasoup-client logic)
│   ├── bundle.js           # Built bundle (generated by esbuild — do not edit)
│   ├── index.html          # Conference room UI
│   ├── site.webmanifest    # PWA manifest
│   └── *.png / *.ico       # Favicons and icons
│
└── views/                  # EJS templates
    ├── main_page.ejs       # Landing page — create/join a room
    ├── Terms&Conditions.ejs
    └── PrivacyPolicy.ejs
```

---

## API Reference

### Socket.IO Events

#### Client → Server

| Event | Payload | Description |
|---|---|---|
| `joinRoom` | `{ roomName, username, devices }` | Join or create a room |
| `createWebRtcTransport` | `{ consumer: boolean }` | Request a new WebRTC transport |
| `transport-connect` | `{ dtlsParameters }` | Connect a producer transport |
| `transport-produce` | `{ kind, rtpParameters, appData }` | Create a server-side producer |
| `transport-recv-connect` | `{ dtlsParameters, serverConsumerTransportId }` | Connect a consumer transport |
| `consume` | `{ rtpCapabilities, remoteProducerId, serverConsumerTransportId }` | Create a server-side consumer |
| `consumer-resume` | `{ serverConsumerId }` | Resume a paused consumer |
| `AlertServertoRemoveMap` | `{ source, id, roomName, socketid }` | Notify peers of a closed producer |

#### Server → Client

| Event | Payload | Description |
|---|---|---|
| `connection-success` | `{ socketId }` | Connection established |
| `FlutterjoinRoomSuccess` | `{ rtpCapabilities, peerlength, existingPeers }` | Room joined — includes RTP capabilities and existing peers |
| `newPeerJoined` | `{ socketId, peerlength, peerDetails, producerIds }` | A new peer joined the room |
| `new-producer` | `{ producerId, socketId, producerDevice }` | A new producer is available to consume |
| `producer-closed` | `{ remoteProducerId, socketId, source }` | A producer was closed — clean up its consumer |
| `alert-socket` | `socketId` | A peer disconnected — remove their UI |

---

## Scripts

| Command | Description |
|---|---|
| `npm start` | Start the server with nodemon (auto-reload on changes) |
| `npm run watch` | Watch `public/index.js` and rebuild `bundle.js` on save |
| `npm run build` | One-time build of the client bundle |

---

## Troubleshooting

### Camera shows black screen / "Could not start video source"

- **Another app is using the camera.** Close other browser tabs, Zoom, Teams, or OBS Studio that may hold the camera.
- **Browser permissions.** Ensure `https://localhost:5000` has camera/mic permissions in your browser settings.
- **HTTPS required.** WebRTC requires HTTPS (or localhost). The self-signed cert must be accepted.

### `WorkerClosedError: Channel closed`

- The mediasoup worker process crashed. Check the server console for details.
- Ensure the `announcedIp` in `app.js` matches your actual local IP.
- If running in WSL2, use your WSL2 IP (`ip addr show eth0`), not the Windows IP.

### `Cannot read properties of undefined (reading 'produce')`

- The `producerTransport` wasn't created. Check that the `createWebRtcTransport` callback succeeded — look for `params.error` in the client console.
- Ensure the server is running and the socket connection is established before clicking camera/mic buttons.

### Bundle not updating

- Run `npm run build` to rebuild manually, or `npm run watch` for auto-rebuild on save.
- Hard-refresh the browser (Ctrl+Shift+R) to bypass cache.

---

## Tech Stack Versions

| Package | Version |
|---|---|
| mediasoup | ^3.26.0 |
| mediasoup-client | ^3.23.1 |
| socket.io | ^4.8.3 |
| socket.io-client | ^4.8.3 |
| express | ^4.22.2 |
| ejs | ^3.1.10 |
| esbuild | ^0.25.0 |
| nodemon | ^3.1.14 |

---

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add some amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## License

This project is licensed under the **ISC License** — see the [LICENSE](LICENSE) file for details.

---

## Acknowledgments

- [mediasoup](https://github.com/versatica/mediasoup) by [Versatica](https://versatica.com) — The SFU engine powering this project
- [Socket.IO](https://socket.io) — Real-time communication layer
- [Bootstrap](https://getbootstrap.com) — Responsive UI framework
- [Font Awesome](https://fontawesome.com) — Icon toolkit

---

<div align="center">

**Built with [mediasoup](https://mediasoup.org)**

[Report Bug](https://github.com/Sam15b/mediasoup/issues) · [Request Feature](https://github.com/Sam15b/mediasoup/issues) · [⭐ Star on GitHub](https://github.com/Sam15b/mediasoup)

</div>
