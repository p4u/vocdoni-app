# UI Flows

Human-readable contract of the core user flows for the Vocdoni web app. Developers can follow these steps by hand in a browser, and they are compiled into an automated Playwright suite.

## Setup

- install: `pnpm install --frozen-lockfile`
- build: (none — flows run against the Vite/Vike dev server)
- start: `pnpm start`
- url: http://localhost:3000
- ready: `GET /` returns HTTP 200 (SPA shell; the app boots client-side, so wait for the `Vocdoni` text to render before interacting)
- notes: |
    No `.env.local` is required — `VOCDONI_ENVIRONMENT` defaults to `dev` and `SAAS_URL` defaults to the public Vocdoni dev backend (`https://saas-api-dev.vocdoni.net`), so the app boots and serves real UI copy with zero configuration. There is no test account: the dashboard, processes, and organization admin screens all require a real login and are out of scope here. Flows instead exercise the parts of the app that are reachable and deterministic without credentials: the auth forms (rendering + client/server validation, no real account is ever created) and the public, SSR-rendered organization/process pages, which are asked for addresses/ids that intentionally do not exist so the outcome (a 404) never depends on live/mutable backend data. A cookie-consent banner appears on first load and must be dismissed (Reject) before interacting with page content.

## Flow: Unauthenticated landing redirects to sign in

Proves the app's root route boots and gates the dashboard behind authentication.

1. Open the app's base URL with no session.
2. Wait for the app to finish redirecting.
   - expect: the browser ends up on the sign-in screen (`/account/signin`, language-prefixed, e.g. `/en/account/signin`).
3. Look at the sign-in screen.
   - expect: a "Welcome" heading, an Email field, a Password field, and a "Log In" button are visible.

## Flow: Navigate between sign in and sign up

Proves the primary navigation link between the two auth forms works both ways.

1. From the sign-in screen, dismiss the cookie-consent banner (click "Reject").
2. Click the "Sign up" link.
   - expect: the URL changes to the sign-up screen (`/account/signup`) and a "Sign up" heading with First name, Last name, Email, and Password fields is visible.
3. Click the "Log In" link on the sign-up screen.
   - expect: the URL changes back to the sign-in screen (`/account/signin`) and the "Welcome" heading is visible again.

## Flow: Sign in rejects invalid credentials

Proves the sign-in form submits to the backend and surfaces a real error.

1. From the sign-in screen, dismiss the cookie-consent banner.
2. Fill the Email field with an address that has no account and the Password field with any password of 8+ characters.
3. Click "Log In".
   - expect: an error message reading "invalid login credentials" appears, and the browser stays on the sign-in screen.

## Flow: Public organization page renders a 404 and supports switching language

Proves the SSR-rendered public organization route boots, handles an unknown organization gracefully, and its navigation (language switcher) works.

1. Open the public organization page for an address that does not exist (`/organization/0x0000000000000000000000000000000000dead`).
2. Dismiss the cookie-consent banner.
   - expect: a "404" heading and "Page Not Found" message are shown, with a "Back to home" link and the app's top navigation (Login button, language switcher, theme toggle).
3. Open the language switcher in the top navigation and choose "Español".
   - expect: the URL gains an `/es/` language prefix and the page text switches to Spanish (e.g. a "Página No Encontrada" heading region).

## Flow: Public process page renders a 404 for an unknown process

Proves the second SSR-rendered public route (election/process pages) boots and handles an unknown id gracefully.

1. Open the public process page for an id that does not exist (`/processes/0x0000000000000000000000000000000000000000000000000000000000dead`).
2. Dismiss the cookie-consent banner.
   - expect: a "404" heading and "Page Not Found" message are shown, with a "Back to home" link back to the app.
