# Website Copy Rewrite — Build Notes

## What Was Changed

Replaced all generic LaunchFrame template copy on the marketing site with SleepWell-specific content. The site now accurately reflects SleepWell as an uptime monitoring SaaS.

---

## File-by-File Summary

### `website/src/app/page.tsx`
- Removed `Benefits` import and `<Benefits />` JSX node.
- Final render order: `<Hero>` → `<QuickStart>` → `<Features>` → `<CTA>`

### `website/src/components/home/Hero.tsx`
- Icon: `Sparkles` → `Radio`
- Badge: "Uptime Monitoring • Start Free"
- Headline: "Monitor your services. / Sleep well."
- Subheadline: uptime monitoring value prop
- Primary CTA: "Start Monitoring Free"
- Stat pills (replaced tech-stack badges): "Checks every minute", "Instant email alerts", "30-day uptime history" — plain text with green dot, no `font-mono`/`tech-badge`

### `website/src/components/home/QuickStart.tsx`
- Icons: `Terminal, Box, Rocket` → `Link2, Activity, BellRing`
- Badge: `<Activity>` + "How It Works"
- Headline: "Up and running / in minutes"
- Steps: CLI commands replaced with stat callout badges (no `$` prefix)
  1. Link2 — Add a URL — "Done in 30 seconds"
  2. Activity — We watch 24/7 — "1-min check intervals"
  3. BellRing — Sleep Well — "Instant email alerts"
- Bottom pills: "Monitors any public URL", "Checks every minute", "No credit card required"

### `website/src/components/home/Features.tsx`
- Icons: all replaced with `Globe, Bell, LayoutDashboard, BarChart2, Timer, FolderOpen`
- Grid: `lg:grid-cols-4` → `lg:grid-cols-3` (6 cards instead of 8)
- Subtitle: updated to monitoring-specific copy
- Features: URL Monitoring, Instant Alerts, Public Status Page, 30-Day Uptime Timeline, Response Time Tracking, Multi-Project Support

### `website/src/components/home/CTA.tsx`
- Headline: "Ready to stop worrying about downtime?"
- Subtext: "Join teams who sleep well knowing their services are monitored around the clock."
- Primary button: "Start Monitoring Free"

### `website/src/components/layout/Footer.tsx`
- Brand description: "Uptime monitoring with instant alerts and beautiful status pages. Know the moment something breaks."
- Added "Powered by LaunchFrame" attribution link below copyright

### `website/src/app/globals.css`
- Applied "Midnight Calm" palette — background shifted from neutral near-black to deep navy
- `--background`: `#050505` → `#050d12`
- `--background-alt`: `#0a0a0a` → `#070f15`
- `--surface`: `#111111` → `#0d1822`
- `--surface-elevated`: `#1a1a1a` → `#152230`
- `--border`: `#252525` → `#1a2d3d`
- `--border-subtle`: `#1a1a1a` → `#122030`
- Emerald accent unchanged — reads more vividly as "uptime green" against the navy background
