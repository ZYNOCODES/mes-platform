# MES Web Application

React interface of the MES platform. Each role opens its own area: management dashboards, planning, stock, quality, repair, sampling, pallet validation and label printing. The same build contains the full-screen wall displays.

![Login](../../docs/screenshots/web/login.png)

## Stack

React 19, Vite 7, React Router 7, TanStack Query 5, Axios, Socket.IO client, Chart.js bindings, XLSX export, Lucide icons. Plain CSS with shared variables.

## Pages by role

| Role | Pages |
| --- | --- |
| Production manager | Dashboards, plans, units, pallets, downtime, users, station passwords |
| Planning manager | Plans, editor for active plans, units, pallets, downtime |
| Quality inspector | Quality dashboard, units, pallets, defects, corrective actions, sources, sampling types, users |
| Warehouse configurator | Lots, models and bills of materials, components |
| Warehouse manager | Stock transactions |
| Repairer | `/repair` |
| Sampler | `/sampling` |
| Pallet validator | `/pallet-validation` |
| Printer | `/offline-printing` |

Wall displays need no account and use revocable links: `/tv-display/:token`, `/quality-display/:token`, `/pallet-display/:token`.

## Screenshots

| | |
| --- | --- |
| ![](../../docs/screenshots/web/dashboard-realtime.png) Hourly cadence | ![](../../docs/screenshots/web/quality-analytics.png) Quality analytics |
| ![](../../docs/screenshots/web/plan-details.png) Plan details | ![](../../docs/screenshots/web/product-traceability.png) Traceability |
| ![](../../docs/screenshots/web/pallets.png) Pallets | ![](../../docs/screenshots/web/downtimes.png) Downtime |
| ![](../../docs/screenshots/web/models.png) Models | ![](../../docs/screenshots/web/model-bom.png) Model and BOM |
| ![](../../docs/screenshots/web/components.png) Components | ![](../../docs/screenshots/web/stock-transactions.png) Stock |
| ![](../../docs/screenshots/web/repair-workspace.png) Repair | ![](../../docs/screenshots/web/sampling.png) Sampling |
| ![](../../docs/screenshots/web/defects.png) Defects | ![](../../docs/screenshots/web/sampling-types.png) Sampling types |
| ![](../../docs/screenshots/web/users.png) Users | ![](../../docs/screenshots/web/station-passwords.png) Station passwords |
| ![](../../docs/screenshots/web/display-tv.png) Line display | ![](../../docs/screenshots/web/display-quality.png) Quality display |

## Structure

```text
src/api/         one Axios module per API area
src/hooks/       TanStack Query hooks and sockets
src/context/     authentication and notifications
src/components/  common, layout, display and feature components
src/pages/       screens grouped by area
src/routes/      route table, protected and role routes
src/utils/       permissions, dates, durations, Excel export
```

## How it works

- Pages never call Axios directly: an API module is wrapped by a query hook that owns caching and invalidation.
- Socket hooks invalidate the relevant queries, so pages refresh when the floor changes.
- `ProtectedRoute` checks the session and `RoleRoute` the role. The API enforces the same rules.
- A server clock hook keeps displays on factory time rather than browser time.

## Run

```bash
npm ci
cp .env.example .env     # VITE_API_URL and role names
npm run dev
npm run build
npm run lint
```

In production the build is copied to `MES_server_side/public/dist`.
