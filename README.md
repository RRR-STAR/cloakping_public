<div align="center">
  <h1>🛡️ CloakPing</h1>
  <p><strong>A Privacy-First, Real-Time Contact Widget & Inbox for Portfolios & Personal Sites</strong></p>
</div>

---

## 🌟 What is CloakPing?

**CloakPing** is a modern, decentralized contact solution designed for developers, freelancers, and creators. Instead of putting your raw email address or phone on your portfolio (and risking endl[...]

It uses [ntfy.sh](https://ntfy.sh/) for decentralized push delivery and Firebase Firestore for a reliable, real-time inbox backup.

---

## 💡 Why Use This in Your Portfolio?

If you are a developer or creator, adding CloakPing to your site offers massive benefits:

1. 🔒 **Absolute Privacy:** Never expose your phone number or raw email address to scrapers and spammers.
2. ⚡ **Instant Response:** Visitors' messages ping your phone in real time, allowing you to reply while they are still browsing your site.
3. 🎨 **Professional Aesthetics:** The customizable floating widget looks sleek, modern, and non-intrusive compared to a static `mailto:` link.
4. 🆓 **Zero Cost:** Leverages open-source `ntfy` push infrastructure instead of paid SMS or email gateway APIs.
5. 🌐 **Flexible Integration:** Supports both an embeddable one-line `<script>` badge and a dedicated hosted ping link (`/p/:channelId`).

### How to Embed in Your Site
Once you set up a channel in the dashboard, drop this one-liner into your portfolio's HTML:

```html
<script src="https://your-deployed-domain.com/widget.js" data-channel="ch_your_channel_id" async></script>
```

Or share your standalone hosted ping link directly in emails, GitHub READMEs, or social profiles:
```text
https://your-deployed-domain.com/p/ch_your_channel_id
```

---

## ⚙️ How It Works (The Mechanics)

### 1. Inbound Messaging Pipeline
When a visitor submits a message through the widget or hosted page:
* The client sends an HTTP POST request to `/api/dispatch`.
* The Express backend validates the payload, applies in-memory IP rate limiting, and performs anti-abuse sanitization.
* It constructs an urgent-priority JSON payload (including sender details, message body, priority tags, and action buttons) and pushes it to **ntfy.sh** via a native Node.js HTTPS client (`src/lib[...]
* The `ntfy` servers route the alert directly to the native ntfy mobile app on your paired device.
* Simultaneously, the backend saves a permanent backup of the message in **Firestore** under your account.

### 2. Device Pairing & Push Subscription
To receive notifications on your phone:
1. **Topic Generation:** The dashboard creates a randomized, high-entropy topic identifier (e.g., `cp_abc123_4f8b9e...`).
2. **QR Code / Universal Link:** The dashboard generates a universal pairing QR code.
3. **Scan & Subscribe:** Scanning the QR code with your phone camera opens the pairing flow, automatically launching the native `ntfy` app (iOS or Android) and subscribing it to your private topic[...]
4. **Diagnostic Verification:** Clicking "Send Live Test Alert" dispatches an immediate test push to ensure lock-screen banners and sound notifications are active.

---

## 🗂️ Dashboard Modules & Capabilities

The CloakPing management dashboard provides complete control over your contact channels:

* 📊 **Overview & Channel Management:** View channel metadata, quick embed snippets, topic details, and direct ping links.
* 📱 **QR Device Pairing & Diagnostics:** Generate pairing QR codes, run live delivery diagnostics, test priority-5 wakeup alerts, and revoke/rotate topics on demand.
* 🎨 **Widget Customization & Snippet Generator (`SnippetGenerator.tsx`):** Customize theme colors, widget position (bottom-right vs. bottom-left), greeting text, and placeholder copy with a rea[...]
* 📥 **All Messages Inbox (`AllMessagesTab.tsx`):** A real-time message center connected directly to Firestore snapshots. Search inquiries, filter by channel, reply via one-click email mailto li[...]

---

## 🛠️ Technology Stack & Specifications

CloakPing is engineered with a modern TypeScript full-stack architecture with decoupled real-time push and cloud persistence services:

### 🌐 Core Languages & Runtimes
* **TypeScript** (`~5.8`): Strict end-to-end static typing across frontend components, backend routes, and database models.
* **JavaScript (ES Modules & Vanilla JS)**: Standalone, zero-dependency embed script (`/widget.js`) with responsive CSS injection.
* **Node.js** (`v20+` / `v22`): Server runtime for background processing, rate limiting, and HTTP push dispatching.
* **HTML5 & Modern CSS3**: Semantics, responsive layouts, backdrop blur effects, and animations.

### ⚛️ Frontend Framework & UI Libraries
* **React 19** (`react`, `react-dom`): Component architecture and dashboard state management.
* **Tailwind CSS v4** (`tailwindcss`, `@tailwindcss/vite`): Utility-first CSS engine for responsive styling, dark mode, and layout math.
* **Motion** (`motion`): Hardware-accelerated entrance transitions, tab switches, and modal viewports.
* **Lucide React** (`lucide-react`): Icon set for dashboard navigation and status badges.
* **node-qrcode** (`qrcode`): Real-time client-side generation of high-resolution QR pairing codes and downloadable canvas cards.

### ⚙️ Backend & Networking
* **Express.js** (`express` `v4.21`): HTTP server handling:
  * Dynamic `/widget.js` script delivery with custom CORS headers
  * Public `/p/:channelId` hosted ping landing page
  * Secured `/api/dispatch` notification routing and rate limiting
  * Dedicated `/api/channels/:channelId/test-ping` diagnostics endpoint
* **CORS** (`cors`): Cross-origin request management for third-party portfolio widgets.
* **Native Node.js HTTPS Engine**: Direct socket-level TLS dispatch in `src/lib/ntfyHelper.ts` ensuring resilient push delivery.

### 🗄️ Cloud Database & Authentication
* **Firebase Firestore** (`firebase/firestore`): Cloud NoSQL database with real-time reactive snapshots (`onSnapshot`) for channels and messages.
* **Firebase Auth** (`firebase/auth`): Authentication and session token handling for dashboard owners.
* **Firestore Security Rules** (`firestore.rules`): Row-level security rules enforcing owner data isolation.

### 📲 Push Notification Infrastructure
* **ntfy.sh (Decentralized HTTP Push Protocol)**: Open-source, decentralized pub-sub push notification engine routing alerts to iOS/Android lock screens with urgent priority headers and interacti[...]

### 🔨 Build Tools & Developer Experience
* **Vite 6** (`vite`, `@vitejs/plugin-react`): Frontend development server and production bundler.
* **esbuild** (`esbuild`): Bundles backend TypeScript into a standalone CommonJS file (`dist/server.cjs`).
* **tsx** (`tsx`): Fast TypeScript execution engine for development runtime.

---

## 🧑‍💻 Developer Guide: Architecture & Workflows

### 🌊 In-Depth System Control Flow

```text
[ Website Visitor ] 
        │
        ├── Submits message via <script src=".../widget.js"> OR /p/:channelId
        ▼
[ Express Server (`server.ts`) ]
        │
        ├── 1. Enforces IP-based rate limiting (Anti-Spam)
        ├── 2. Sanitizes input fields (name, email, message)
        │
        ├──► [ Push Dispatcher (`src/lib/ntfyHelper.ts`) ]
        │         │
        │         └── Native Node HTTPS POST ──► [ ntfy.sh Servers ] ──► [ Owner's Phone Lock Screen ]
        │
        └──► [ Firestore Storage (`src/lib/firestoreService.ts`) ]
                  │
                  └── Document saved to `/users/{userId}/channels/{channelId}/messages`
                            │
                            └── Real-time onSnapshot sync ──► [ React Dashboard (`AllMessagesTab.tsx`) ]
```

### 📡 Backend API Endpoints

| Endpoint | Method | Purpose | Auth Required |
| :--- | :--- | :--- | :--- |
| `/api/dispatch` | `POST` | Receives incoming messages from embed widgets or hosted ping pages. Triggers ntfy push and saves to Firestore. | No (Public / Rate-limited) |
| `/widget.js` | `GET` | Dynamically serves the standalone JavaScript widget embed script with CORS enabled. | No (Public) |
| `/p/:channelId` | `GET` | Serves the standalone, responsive public ping submission web page for a channel. | No (Public) |
| `/api/channels/:channelId/test-ping` | `POST` | Dispatches an instant test notification to verify device pairing and notification permissions. | No (Protected by Channel ID) |
| `/pair/:topic` | `GET` | Universal pairing bridge route forwarding external device cameras to the pairing bridge. | No (Public) |
| `/api/health` | `GET` | Health check endpoint returning server status. | No (Public) |

### 📂 File Structure Mapping

```text
📦 cloakping
 ┣ 📂 src
 ┃ ┣ 📂 components
 ┃ ┃ ┣ 📜 AllMessagesTab.tsx      <-- 📥 Real-time message inbox, search, and email reply action
 ┃ ┃ ┣ 📜 AuthLanding.tsx         <-- 🔐 User authentication screen (Sign In & Registration)
 ┃ ┃ ┣ 📜 QrPairingModal.tsx      <-- 📱 QR device pairing, deep-linking, and live test alert diagnostics
 ┃ ┃ ┗ 📜 SnippetGenerator.tsx    <-- 🎨 Embed snippet generator with real-time visual preview
 ┃ ┣ 📂 lib
 ┃ ┃ ��� 📜 firebase.ts             <-- 🔥 Firebase SDK client initialization
 ┃ ┃ ┣ 📜 firestoreService.ts     <-- 🗃️ Firestore database operations, real-time listeners, and channel CRUD
 ┃ ┃ ┣ 📜 ntfyHelper.ts           <-- 🚀 Native Node HTTPS dispatcher for ntfy.sh push protocols
 ┃ ┃ ┗ 📜 urlHelper.ts            <-- 🌐 URL resolution, pairing parameter parsing, and bridge config
 ┃ ┣ 📜 App.tsx                   <-- 🧭 Main React application orchestrator & dashboard view manager
 ┃ ┣ 📜 main.tsx                  <-- ⚛️ React entry point
 ┃ ┣ 📜 types.ts                  <-- 📘 Shared TypeScript interfaces and domain models
 ┃ ┗ 📜 index.css                 <-- 🎨 Tailwind CSS root styles
 ┣ 📜 server.ts                   <-- ⚙️ Express server (API endpoints, widget script, rate-limiting, Vite middleware)
 ┣ 📜 firestore.rules             <-- 🛡️ Firebase security rules enforcing per-user data isolation
 ┣ 📜 firebase-blueprint.json     <-- 📐 Firestore database schema blueprint
 ┣ 📜 vite.config.ts              <-- ⚡ Vite configuration with React and Tailwind plugins
 ┣ 📜 tsconfig.json               <-- ⚙️ TypeScript configuration
 ┣ 📜 .env.example                <-- 📋 Environment variables template
 ┗ 📜 package.json                <-- 📦 Package dependencies and execution scripts
```

### 🔑 Environment Variables Configuration

Copy `.env.example` to create your local `.env` file when developing locally:

```bash
# GEMINI_API_KEY: Required for Gemini AI API calls (configured in server-side environment)
GEMINI_API_KEY="YOUR_API_KEY"

# APP_URL: The base public URL where the application is hosted
APP_URL="https://your-deployed-domain.com"

# VITE_NTFY_BRIDGE_URL: Optional static pairing bridge URL (e.g. GitHub Pages)
VITE_NTFY_BRIDGE_URL="https://rrr-star.github.io/cloakping-bridge"
```

---

## 🚀 Getting Started & Local Development

1. **Clone the Repository & Install Dependencies:**
   ```bash
   npm install
   ```

2. **Start the Development Server:**
   ```bash
   npm run dev
   ```
   *The Express server boots with `tsx`, binding API endpoints and Vite frontend middleware simultaneously on port `3000`.*

3. **Validate Types and Syntax:**
   ```bash
   npm run lint
   ```

4. **Build for Production:**
   ```bash
   npm run build
   ```
   *Compiles the frontend assets to `dist/` and bundles the backend into a standalone `dist/server.cjs` file via `esbuild`.*

5. **Start Production Server:**
   ```bash
   npm run start
   ```

---

## 📄 License & Contributing

Contributions, feedback, and feature requests are welcome! Feel free to fork the repository, open issues, or submit pull requests.
