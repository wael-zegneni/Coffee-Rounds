# Coffee Rounds

A lightweight web app for tracking weekly coffee rounds among colleagues. Built as a single-page PWA with real-time sync across all devices.

## Features

- **Round logging** — Record who paid, who was present, and the total amount each week
- **Fair rotation system** — Automatic "next to pay" pointer with configurable order
- **Absence handling** — Absent on your turn? You still owe when you're back. The app walks the rotation to find the next available person
- **Real-time sync** — All data syncs instantly across devices via Firebase Realtime Database
- **Scoreboard** — Visual ring charts showing each person's paid/present stats
- **Fun stats** — Total spent, average per head, priciest round, most present member, biggest spender
- **Team management** — Add or remove members dynamically; historical data is preserved for removed members
- **Keyword protection** — Client-side lock screen with SHA-256 hashed password; the app and Firebase data are only accessible after entering the correct keyword
- **Installable PWA** — Add to home screen on iOS/Android for a native app feel
- **In-app help** — Rules, rotation logic, and examples accessible from the Help tab

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Frontend | Vanilla HTML, CSS, JavaScript (single file) |
| Database | Firebase Realtime Database (compat SDK v10.12.2) |
| Fonts | Inter (Google Fonts) |
| Hosting | GitHub Pages (or any static host) |
| PWA | Service Worker + Web App Manifest |

## Architecture

The entire application lives in a single `index.html` file — all HTML, CSS, and JavaScript are inline. This was a deliberate choice to keep deployment as simple as dropping files into a folder.

### Key files

```
index.html       — The complete application (HTML + CSS + JS)
manifest.json    — PWA manifest for installability
sw.js            — Service worker (network-first strategy)
icon.svg         — App icon source (SVG)
icon-192.png     — App icon 192x192
icon-512.png     — App icon 512x512
README.md        — This file
```

### Data model (Firebase)

The app uses four Firebase Realtime Database refs:

- **`coffee-rounds`** — Array of round objects, each containing:
  ```json
  {
    "id": 1713350400000,
    "date": "2025-04-17",
    "payer": "Alice",
    "amount": 18.50,
    "present": ["Alice", "Bob", "Charlie", "Dave", "Eve"]
  }
  ```
- **`members`** — Array of active member names
- **`rotation-order`** — Array defining the payment rotation sequence
- **`rotation-pointer`** — Integer index into `rotation-order` indicating who pays next

### Keyword protection

The app is gated behind a lock screen. Users must enter a keyword before any content or Firebase data loads. The keyword is stored as a SHA-256 hash in the source code — never in plain text. The hash is compared using the browser's native `crypto.subtle` API.

Once unlocked, the session is remembered via `sessionStorage` so users don't have to re-enter the keyword on every page interaction. Closing the browser tab clears the session.

To change the keyword:
1. Generate a SHA-256 hash of your new keyword (e.g. via `echo -n "yourkeyword" | sha256sum`)
2. Replace the `PASS_HASH` constant in the script section of `index.html`

> Note: This is client-side protection — it deters casual access but is not a substitute for server-side authentication. For stronger security, consider adding Firebase Authentication.

### Rotation logic

The rotation pointer advances only when:
1. The round being logged is for the **current week** (date >= most recent Thursday)
2. The pointer person is **present** and pays

This time-awareness means historical data can be entered without affecting the current pointer.

When someone is absent on their turn, a **roll call chain** walks forward through the rotation, presenting each next person with Present/Absent buttons until someone confirms they'll pay. The original pointer stays put — the absent person still owes their turn.

### Sync strategy

Firebase listeners (`on('value')`) provide real-time updates. When any device writes data, all connected devices receive the update instantly. The service worker uses a **network-first** strategy so Firebase always gets fresh data, falling back to cache only when offline.

## Setup

### Prerequisites

- A Firebase project with Realtime Database enabled
- Database rules set to allow read/write (or configure auth as needed)

### Steps

1. Clone or download this repository
2. Open `index.html` and update the Firebase config object at the bottom of the `<script>` section:
   ```javascript
   const firebaseConfig = {
     apiKey: 'YOUR_API_KEY',
     authDomain: 'YOUR_PROJECT.firebaseapp.com',
     databaseURL: 'https://YOUR_PROJECT.firebasedatabase.app',
     projectId: 'YOUR_PROJECT',
     storageBucket: 'YOUR_PROJECT.appspot.com',
     messagingSenderId: 'YOUR_SENDER_ID',
     appId: 'YOUR_APP_ID'
   };
   ```
3. Update the `PASS_HASH` constant with the SHA-256 hash of your chosen keyword
4. Update the `INITIAL_MEMBERS` array with your team members' names
5. Deploy to any static hosting (GitHub Pages, Netlify, Vercel, or just open the file locally)

### Firebase Database Rules (development)

```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

> For production, consider adding Firebase Authentication and restricting access.

## Design

The UI follows 2025-2026 design trends:

- **Glassmorphism** — Semi-transparent cards with `backdrop-filter: blur(20px)`
- **Warm coffee palette** — Deep espresso browns, warm creams, and golden accents
- **Bottom navigation** on mobile, inline tabs on desktop (≥640px)
- **Grain texture overlay** — Subtle SVG noise filter for visual warmth
- **Safe area insets** — Supports notched phones

## Tools Used

- **Claude** (Anthropic) — AI assistant used to design, build, and iterate on the entire application
- **Firebase Console** — Database setup and configuration
- **ImageMagick** — SVG to PNG icon conversion
- **Google Fonts** — Inter typeface

## License

MIT — use it for your own team's coffee rounds!
