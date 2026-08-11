# UI Flows

Human-readable contract of the core user flows for the Vocdoni web app. Developers can follow these steps by hand in a browser, and they are compiled into an automated Playwright suite.

## Setup

- install: `pnpm install --frozen-lockfile`
- build: (none — flows run against the Vite/Vike dev server)
- start: `pnpm start`
- url: http://localhost:3000
- ready: `GET /` returns HTTP 200 (SPA shell; the app boots client-side, so wait for the `Vocdoni` text to render before interacting)
- notes: |
    No `.env.local` is required — `VOCDONI_ENVIRONMENT` defaults to `dev` and `SAAS_URL` defaults
    to the public Vocdoni dev backend (`https://saas-api-dev.vocdoni.net`). Requires network
    access to that backend.

    Authenticated flows use a dedicated, disposable test account on the DEV backend (anyone can
    self-register there; the account has no privileges beyond its own empty test organization):

    - email: `argos-uiflows-1786480980@emalupe.com`
    - password: `ArgosUITest1234`
    - organization: "Argos Flows Org" (pre-created, admin role, Free plan)

    Do NOT change this account's password or delete its organization — the flows depend on that
    state. If the account is ever lost, register a fresh one on the dev backend, create one
    organization named "Argos Flows Org", and update the credentials here.

    Determinism rules: flows never mutate backend state — the vote composer flow fills the form
    but NEVER clicks "Publish" or "Save", and no members are ever imported. A cookie-consent
    banner appears on first load and must be dismissed (click "Reject") before interacting.

## Flow: Unauthenticated landing redirects to sign in

Proves the app's root route boots and gates the dashboard behind authentication.

1. Open the app's base URL with no session.
2. Wait for the app to finish redirecting.
   - expect: the browser ends up on the sign-in screen (`/account/signin`, language-prefixed, e.g. `/en/account/signin`).
3. Look at the sign-in screen.
   - expect: a "Welcome" heading, an Email field, a Password field, and a "Log In" button are visible.

## Flow: Sign in rejects invalid credentials

Proves the sign-in form submits to the backend and surfaces a real error.

1. From the sign-in screen, dismiss the cookie-consent banner.
2. Fill the Email field with an address that has no account and the Password field with any password of 8+ characters.
3. Click "Log In".
   - expect: an error message reading "invalid login credentials" appears, and the browser stays on the sign-in screen.

## Flow: Sign in with the test account reaches the organization dashboard

Proves real authentication end-to-end: login against the live dev backend, session establishment, and the organization dashboard rendering with live plan data.

1. From the sign-in screen, dismiss the cookie-consent banner.
2. Fill Email and Password with the test account credentials from Setup and click "Log In".
   - expect: the browser lands on the admin dashboard (`/admin`).
3. Look at the dashboard.
   - expect: a "Dashboard" heading is visible and the organization name "Argos Flows Org" appears in the workspace switcher.
4. Check the plan usage panel.
   - expect: a "Plan usage" section shows "Voting processes" and "Memberbase size" counters with numeric limits (e.g. "/ 5", "/ 100").
5. Check the quick actions.
   - expect: "Create new vote", "View active votes", and "Manage team" actions are visible.

## Flow: Dashboard sidebar navigates all organization sections

Proves the authenticated app shell: every sidebar section routes correctly and renders its own management screen.

1. Sign in with the test account (as in the previous flow) and land on the dashboard.
2. Click "Voting processes" in the sidebar.
   - expect: the URL changes to the processes list (`/admin/processes/all`), a "Voting processes" heading is visible, and the "All", "Ended", and "Drafts" tabs are shown.
3. Click "Memberbase" in the sidebar.
   - expect: the URL changes to the memberbase section (`/admin/memberbase/`), a "Memberbase" heading is visible with "Members" and "Groups" tabs, and the "Add Member" and "Import" actions are shown.
4. Click "Settings" in the sidebar.
   - expect: the URL changes to `/admin/settings/organization`, an "Argos Flows Org Settings" heading is visible, and the "Organization details", "Team", and "Subscription plan" sections are shown.

## Flow: Vote composer builds a multi-option question without publishing

Proves the deepest organizer surface: the voting-process composer, its dynamic question form, and its configuration sections. This flow must NEVER click "Publish" or "Save".

1. Sign in with the test account and, from the dashboard, click "Create new vote".
   - expect: the browser lands on the composer (`/admin/processes/create`) with template shortcuts ("Annual General Meeting", "Election", "Participatory Budgeting") visible.
2. Fill the process title field with "Argos smoke vote".
3. Fill the first question's title with "Which option is best?" and its first two option fields with "Alpha" and "Beta".
4. Scroll the "Add a new option" button into view and click it.
   - expect: a third option field appears; fill it with "Gamma".
5. Review the configuration sections.
   - expect: "Basic configuration" (start/end scheduling), "Extra configuration" (result visibility, voting power), and "Census creation" sections are visible, and the census section explains a voter group must be created before starting a vote.
6. Confirm the composer's actions without using them.
   - expect: "Publish" and "Save" buttons are present. Do NOT click either — leave the page without saving.

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
