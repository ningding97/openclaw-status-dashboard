# OpenClaw Status Dashboard - Design Document

## Vision
A real-time, beautiful status visualization for OpenClaw that lets users see exactly what their agent is doing at any moment. Available as:
1. **Web Dashboard** - standalone page accessible via Control UI
2. **Telegram Mini App** - inline in Telegram for quick glances
3. **Embeddable Widget** - drop into any page

## Design Principles
- **Glanceable**: Key status visible in <1 second
- **Real-time**: WebSocket-driven live updates, no polling
- **Beautiful**: Dark theme, smooth animations, modern typography
- **Interactive**: Click to drill down, hover for details, control from the dashboard

## Architecture

### Data Sources (Gateway WebSocket API)
OpenClaw already exposes these via its WS protocol:
- **Agent Status**: current model, thinking/idle/responding, token usage
- **Sessions**: active sessions, sub-agents, their states
- **Cron Jobs**: scheduled tasks, next run, last result
- **Nodes**: connected devices, capabilities, health
- **Channels**: Telegram/WhatsApp/Discord connection status
- **Memory**: memory index status, last sync
- **Exec**: running commands, approval queue

### Tech Stack
- **Lit** (matches existing OpenClaw UI)
- **CSS Custom Properties** for theming
- **WebSocket** for real-time data
- **No heavy frameworks** - keep it lean like the existing Control UI

## Layout

```
┌────────────────────────────────────────────────┐
│  🫞 OpenClaw Status              [Live ●]  │
├────────────────────────────────────────────────┤
│                                                │
│  ┌────────────┐ ┌────────────┐ ┌────────────┐  │
│  │  🤖 Agent  │ │  📡 Nodes │ │ 🔔 Cron   │  │
│  │  Thinking │ │  2 online │ │ 3 active  │  │
│  │  opus-4   │ │  0 errors │ │ next: 5m  │  │
│  └────────────┘ └────────────┘ └────────────┘  │
│                                                │
│  ┌──────────────────────────────────────────┐  │
│  │  Active Sessions                            │  │
│  ├──────────────────────────────────────────┤  │
│  │  ● main      Telegram  Idle     2.1k tok   │  │
│  │  ● sub-1    Isolated  Running  850 tok    │  │
│  │  ○ sub-2    Isolated  Done     1.2k tok   │  │
│  └──────────────────────────────────────────┘  │
│                                                │
│  ┌──────────────────────────────────────────┐  │
│  │  Live Activity Feed                         │  │
│  ├──────────────────────────────────────────┤  │
│  │  17:52 🛠 exec: git push origin fix/...     │  │
│  │  17:51 🧠 thinking: analyzing cron code...  │  │
│  │  17:50 📩 telegram: message from Boss       │  │
│  │  17:48 ✅ cron: PR monitor completed         │  │
│  └──────────────────────────────────────────┘  │
│                                                │
│  ┌──────────────────────────────────────────┐  │
│  │  Channels                                   │  │
│  ├──────────────────────────────────────────┤  │
│  │  ● Telegram   Connected    12 chats         │  │
│  │  ● WhatsApp   Connected    3 chats          │  │
│  │  ○ Discord    Disconnected                  │  │
│  └──────────────────────────────────────────┘  │
└────────────────────────────────────────────────┘
```

## Components

### 1. StatusPulse - Agent Heartbeat
A pulsing orb that shows agent state:
- 🟢 Green pulse: Idle, ready
- 🟡 Amber pulse: Thinking/processing
- 🔵 Blue stream: Responding/streaming
- 🔴 Red: Error/disconnected
- Orbit animation shows active sub-agents as smaller dots

### 2. SessionTimeline
Horizontal timeline showing session activity:
- Messages in/out as dots on timeline
- Tool calls as colored segments
- Hover to see details
- Click to jump to session

### 3. NodeMap
Visual representation of connected nodes:
- Each node as a card with live stats
- Connection quality indicator
- Quick actions (run command, screenshot)

### 4. CronBoard
Grid of cron jobs with:
- Countdown to next run
- Success/fail history as small sparkline
- Toggle enable/disable
- Click to see last output

### 5. ActivityFeed
Real-time scrolling feed of events:
- Color-coded by type (message, tool, cron, error)
- Collapsible details
- Filter by category

### 6. TokenBudget
Visual token usage meter:
- Current session usage vs context window
- Daily/monthly usage trends
- Cost estimate

## Color Palette

```css
:root {
  --bg-primary: #0a0a0f;
  --bg-card: #12121a;
  --bg-card-hover: #1a1a25;
  --border: #2a2a3a;
  --text-primary: #e8e6f0;
  --text-secondary: #8888a0;
  --accent-green: #00d68f;
  --accent-amber: #ffaa00;
  --accent-blue: #4d9fff;
  --accent-red: #ff4d6a;
  --accent-purple: #a855f7;
  --glow-green: rgba(0, 214, 143, 0.15);
  --glow-amber: rgba(255, 170, 0, 0.15);
}
```

## Typography
- Headers: Inter or DM Sans, weight 500
- Body: Inter, weight 400
- Monospace: JetBrains Mono (for code/status)
- Status numbers: Tabular nums

## Animations
- Card entrance: fade-up with stagger (50ms between cards)
- Status pulse: CSS keyframe, 2s ease-in-out
- Data updates: CSS transition 300ms
- Activity feed: slide-in from right
- No animation if prefers-reduced-motion

## Interaction
- Click session → expand to show messages
- Click node → show capabilities, run command
- Click cron → show history, edit schedule
- Hover any element → tooltip with details
- Keyboard: Tab navigation, Esc to close panels

## Telegram Mini App
Simplified version for Telegram:
- Single column layout
- Agent status + active sessions only
- Tap to expand sections
- Native Telegram theme integration
- Send commands directly from the app

## API Endpoints Needed
All via existing Gateway WebSocket:
- `gateway.status` - overall health
- `sessions.list` - active sessions
- `cron.list` + `cron.runs` - job status
- `nodes.status` - node health
- `channels.status` - channel connections
- Real-time events via WS subscription

## Implementation Plan
1. Phase 1: Static HTML/CSS prototype with mock data
2. Phase 2: Wire up to Gateway WebSocket
3. Phase 3: Add as a new tab in existing Control UI
4. Phase 4: Telegram Mini App version
5. Phase 5: PR to openclaw/openclaw
