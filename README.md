# TLyew.X. — Starter source code

Dashboard React + Vite, backend Express, registration with Brevo 6-digit email OTP verification, session login, bear mascot animation, dark/light theme, Gacha Nomor, Generate Email, and Settings.

## Important integration note
This project deliberately does **not** guess NumberPanel endpoints or pretend provider data is live. The exact official API base URL, paths, authentication method, request parameters, and response shape must be verified against provider documentation before filling in the `NUMBERPANEL_*` values. API key stays in the backend `.env` only.

## Requirements
- Node.js 20+ recommended
- npm

## Install and configure
```bash
npm install
cp .env.example .env
```
Configure Brevo transactional email first: set `BREVO_API_KEY`, `BREVO_SENDER_EMAIL` (a sender verified in your Brevo account), and optionally `BREVO_SENDER_NAME`. The registration flow sends a six-digit OTP through the official Brevo transactional email API. This sends mail; it does not create an inbox or receive incoming mail.

Create a long random session secret, then generate a password hash:
```bash
npm run make-password -- "Password-Kuat-Minimal-12"
```
Copy the printed bcrypt hash to `ADMIN_PASSWORD_HASH` in `.env`. Change `ADMIN_USERNAME` and `SESSION_SECRET`. Do not commit `.env`.

Set `PORT=3000` for local development. Do not use `PORT=587`; that is commonly an SMTP port, not an HTTP web server port.

## Run in development
```bash
npm run dev
```
Open the Vite URL shown in terminal (normally http://localhost:5173). API calls are proxied to Express on port 3000.

## Build and run production
```bash
npm run build
NODE_ENV=production npm start
```
Deploy behind HTTPS in production so the secure session cookie works. Set `PORT` to the port assigned by your host.

## Provider API setup
Fill these only after checking official documentation:
- `NUMBERPANEL_BASE_URL`
- `NUMBERPANEL_API_KEY` (only if the provider really requires one)
- `NUMBERPANEL_COUNTRIES_PATH`
- `NUMBERPANEL_SERVICES_PATH`
- `NUMBERPANEL_NUMBERS_PATH`
- `NUMBERPANEL_EMAIL_DOMAINS_PATH`
- `NUMBERPANEL_EMAIL_GENERATE_PATH`

The generic adapter sends Bearer auth only if `NUMBERPANEL_API_KEY` is configured. If the provider uses a different authentication header or request method/body, adjust `server/index.js` to match its official documentation. Do not assume the website dashboard URL is an API endpoint.

## Current scope
- Registration collects `Username account`, `Gmail terkait`, and `Password account`, then emails a six-digit code through Brevo. The `Code` field and `Verify Account` action verify the address before the account can log in.
- Registered accounts are stored in `server/data/users.json` with bcrypt password hashes. Pending OTPs are held in memory for 10 minutes and reset if the server restarts; use a durable database/OTP store for production.
- Login uses server-side bcrypt hash and session cookie. Admin login via environment variables remains available.
- Session store uses SQLite.
- Rate limits, request timeout, input validation, basic origin check, and Helmet are enabled.
- Bear's eyes-cover animation is tied to whether the password field is non-empty; the password value is not used for animation or sent anywhere except the login request.
- Dashboard stats show `—` until real provider stats are integrated; no fake results are shown.
- API result JSON is rendered as returned by the provider. Normalize fields to the provider's documented schema for a polished production result.
- `Ingat saya` is visual only in this starter; persistent login is intentionally not enabled because it needs a separate security policy.
- For production, configure HTTPS, backups, secure secrets, and review the app's security requirements before exposing it publicly.

## Gacha Nomor UI
The Gacha Nomor page includes a searchable country list with high-resolution flag images when the API supplies ISO-2 country codes, searchable service selection with recognizable app logos where available, and an SMS-style result card. Logos are loaded from public image CDNs; provide country ISO-2 codes in provider response for reliable flags. If a provider's result schema uses different fields, map them in `SmsResult` in `src/App.jsx`.
