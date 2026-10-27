# RailTrack

**Every Journey, Beautifully Tracked.**

A local-first Indian railway travel journal built with Next.js App Router and React. Search 20 recognizable train services, inspect demo timetables and schematic route maps, plan trips, follow timetable-based progress, and maintain a personal history with calculated analytics.

## Features

- Public landing page and responsive dashboard with calculated statistics, active trip, upcoming plans, charts, recent trips, and travel passport.
- Debounced train search by name, number, station code, and city; directional station filters, category, departure, operating-day, duration filters, and sorting. Recent searches are stored locally.
- Train detail pages with station timelines, overnight day numbers, running days, route favorites, and Leaflet maps.
- Six-step validated planner: train, boarding, destination, boarding date, optional class/coach/seat/notes, review and save. Destination must follow boarding; operating days are validated.
- Journey editing, deletion with confirmation, cancellation, manual completion, and restoration to timetable-derived status.
- Timetable tracking with previous/next stations, scheduled remaining time, approximate segment marker, and explicitly labelled normal/delay/early/completed demo scenarios.
- Journey cards, status tabs, sorting, search, and a monthly journey calendar.
- History in cards, timeline, and table views; date/search filters; date/distance sorting; filtered CSV and JSON exports.
- Recharts monthly activity, distance, status distribution, frequent routes and yearly activity. Travel passport and calculated milestones.
- Train and route favorites; application notifications with unread status and reminders checked while open.
- Local profile, home station, light/dark/system theme, reminder preference, default history view.
- JSON backup export/import, validation and duplicate-ID rejection before replacement; demo reset and data clearing with confirmation dialogs.
- Keyboard command palette (Cmd/Ctrl+K): pages, trains, saved journeys, new journey, settings and theme.
- Optional onboarding demo records with relative dates and clear sample labels. Start fresh for truthful zero personal statistics.

## Stack actually used

Next.js 15, React 19, JavaScript/JSX, Tailwind CSS 4 plus custom design CSS, Radix Dialog and Dropdown, shadcn-compatible local UI primitives/configuration, Lucide React, Motion for React, Zustand, Dexie/IndexedDB, React Hook Form, Zod, Leaflet/React Leaflet, OpenStreetMap-compatible CARTO tiles, Recharts, date-fns, Vitest, React Testing Library, Playwright, ESLint, and Prettier.

Core journey management needs no backend, account, or API key. An optional server-side RailRadar adapter supports provider-reported status with a user-owned free key; see LIVE-TRACKING.md.

## Getting started

Node.js 22.9 or later (Node 24 was used during development):

```sh
npm install
npm run dev
```

Open http://localhost:3000. Choose **Start fresh** or **Explore with demo journeys** on the dashboard. Data stays in the current browser origin. The hosted app and localhost have separate journals.

If macOS reports `EMFILE` file-watcher errors in a restricted environment:

```sh
WATCHPACK_POLLING=true npm run dev
```

## Commands

```sh
npm run lint
npm run test
npm run build
npm run format
npx playwright install chromium
npm run test:e2e
```

`npm run test` runs domain, component, and fake-IndexedDB integration tests. Playwright defines desktop Chromium and mobile Chromium workflows covering search, train details, route exploration, planning, saving, reloading, editing, completion, history export, analytics, deletion, favorites and themes. The standalone browser launch is blocked by this development computer's sandbox; see VERIFICATION.md for exact results. Run it in your normal terminal or CI to complete browser automation.

## Build and deployment

The app uses a Next.js static export. `npm run build` writes the deployable site to `out/`. The core app is static. Optional live status uses one server-side API endpoint and a private RAILRADAR_API_KEY; see LIVE-TRACKING.md.

**Vercel:** import the project repository and use the included `vercel.json` (framework override: Other), install with `npm install`, build with `npm run build`, and deploy the `out` directory plus the root `api/live-status.js` Node function. Set RAILRADAR_API_KEY privately in Vercel to enable live reports. Keep `trailingSlash: true` so exported routes resolve. Alternatively deploy the output to any static host that serves directory indexes. `npm start` serves the production export with the included lightweight Node preview server. Vercel serves the static assets directly.

To preview the production build locally:

```sh
npm start
```

Train pages are generated at `/trains/[trainId]/`. Browser-created journey UUIDs cannot be pre-generated at build time, so the exported detail page is `/journeys/view/?id=<UUID>` using the App Router `[journeyId]` segment. This is an intentional static-hosting tradeoff; records are looked up only in the current browser. A journey URL does not share a record with another device. Sharing copies a formatted summary instead.

## Architecture and files

```text
src/
  app/                  App Router pages, layout, global styles, error handling
    dashboard/          Saved-record overview
    trains/[trainId]/   Generated train pages
    routes/             Route explorer
    journeys/new/       Planner and edit form
    journeys/[journeyId]/  Static browser-record detail entry
    history/ analytics/ favorites/ notifications/ settings/
  components/
    ui/                 Button utilities and accessible Radix dialog
    layout/             Responsive shell, menus, command palette, onboarding
    dashboard/          Overview and calculated statistics
    trains/             Search, details, station timeline, favorites
    journeys/           Planner, management, tracking, history
    maps/               Client-only dynamic Leaflet map and explorer
    charts/             Recharts and analytics screen
    common/             Shared cards, notifications, settings
  data/                 Stations, 20 trains, relative-date sample journeys
  lib/
    database/           Dexie schema and initialization
    providers/          RailDataProvider and DemoRailDataProvider
    validation/         Zod journey rules
    utils/              Search, schedule, status, progress, analytics
  services/             Transactional CRUD, notifications, backup, export
  store/                Zustand UI/snapshot state
  tests/                Vitest and React Testing Library tests
e2e/                    Playwright workflows
public/icons/           Original SVG logo
```

## Persistence and data safety

Dexie tables: `trains`, `stations`, `journeys`, `favorites`, `notifications`, `preferences`, and `activity`. Schema v2 adds an indexed reminder delivery key to prevent deleted reminders from returning. Writes and their notifications use transactions. Import validates the full backup before a transaction replaces records. Duplicate IDs, unknown trains, invalid routes and malformed dates are rejected. Imports replace journeys/favorites/profile and clear old notifications/activity; they do not restore old notifications/activity. Recent search terms are kept in localStorage separately and are not part of a backup.

Storage failure is surfaced with an error panel and retry action; mutation errors show a toast. Clearing browser data or changing browser/device loses access to records unless exported. No cloud synchronization.

## Demo railway information

Train service names and station locations are provided for demonstration. **All schedules, route stop selections, operating days, distances, and classes are illustrative and unverified.** Some real-world stops are omitted. Do not rely on this dataset for travel, booking, connections, or service availability. Verify with official Indian Railways/IRCTC sources.

Journey dates mean the boarding date in **IST (UTC+05:30)**. Arrival/departure minutes include day offsets. For an intermediate station after midnight, the origin day offset is subtracted so the selected date stays the boarding date.

Tracking is calculated from schedule time. Segment positions interpolate between station coordinates and are schematic, never verified railway alignment or live GPS. Demo scenario controls change only the view; they do not mutate saved journey status. Manual completion/cancellation persists separately. Maps show attribution and offer an accessible station list when tiles are unavailable.

Reminders run on application initialization and once a minute while open. There are no push alerts or background service-worker reminders. All notifications originate within RailTrack, never from an official railway operator.

## Future authorized live data

`RailDataProvider` defines search, train lookup, and live status; `DemoRailDataProvider` supplies local data. Live status currently returns `Unavailable`. The optional `AuthorizedLiveRailDataProvider` now calls a same-origin server adapter for RailRadar; never put API secrets in browser bundles. Label results `Verified Live` only after the provider is configured and verified. The current app makes no live-data claim.

## Interview walkthrough

Read INTERVIEW.md for component structure, state/persistence boundaries, tracking math, search/analytics derivation, and the resume-feature verification table. Read VERIFICATION.md for completed checks and testing limitations.

## Optional provider-reported live status

RailRadar advertises a 100-request/month free development tier. The adapter is integrated, and a user-owned free key has been configured for this local installation and tested successfully. Downloaded copies need their own private key configuration. Train/journey details include a separate live-status panel. See [LIVE-TRACKING.md](LIVE-TRACKING.md) for research, setup, quota and accuracy limits. The current implementation displays reported stations, not exact moving GPS coordinates.

Live lookup is also available at `/live/` for any five-digit train number, outside the local demo catalogue. It requires the private key setup described above.

Automatic refresh: train 13186 refreshes every 30 minutes while visible/online. A durable local 80-request rolling 31-day budget covers manual and automatic lookups. Preserve `.railtrack/live-usage.json` across restarts. Other applications using the same API key are outside this local counter. Auto polling is disabled on stateless deployments without a shared durable guard. See LIVE-TRACKING.md.
