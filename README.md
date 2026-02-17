# Thoughtclock

A zero-friction time tracking dashboard that connects to Jira Cloud. Built for overworked employees who need to fill timesheets without it becoming another chore.

Single HTML file. No build tools. No dependencies. Just open `index.html`.

---

## Features

### 8-Slot Timer Cockpit

The main interface is a responsive 4x2 grid of timer cards. Each card is an independent stopwatch. **Click anywhere on the card body to start/stop the timer.** No tiny buttons to hunt for during a context switch.

Maximum 8 slots enforced by design — this forces you to prioritize what you're actually working on today and keeps the dashboard glanceable.

Keyboard shortcuts: press `1`–`8` to toggle the corresponding timer.

### Three Card Types

Add timers via the `+` card. Each type serves a different kind of work:

#### Jira Ticket

- Search and pick from your open Jira tickets (fetched via `assignee = currentUser()`)
- Displays ticket key (clickable link to Jira), summary, and project info
- When logging time, posts a worklog entry to Jira via REST API v3
- Optionally posts a Jira comment alongside the worklog
- Blue type indicator

#### Quick Task

- For ad-hoc work that doesn't have a Jira ticket: email, Slack, code reviews, deployments, helping a colleague
- Just type a name and hit Enter — timer starts immediately
- Time is logged locally and included in CSV exports
- Cyan type indicator

#### Meeting

- For any meeting with a known expected duration
- Presets: Standup (15m), 1:1 (30m), Sprint Planning (60m), Retrospective (60m), Workshop (90m)
- Or set a custom duration in minutes
- **SVG circular progress ring** replaces the standard timer display:
  - Green when under 80% of expected time
  - Amber when 80–100%
  - Red with pulsing border when overtime
  - Shows `+Xm over` label when exceeding expected duration
- Purple type indicator

### Voice-to-Text Comments

Each card has a microphone button that opens a voice recording modal.

- Uses the browser's [Web Speech API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Speech_API) (Chrome/Edge required)
- Continuous recording with live interim transcript
- Editable transcript — fix any recognition errors before posting
- One-click post to Jira as a comment (for Jira ticket cards)
- Useful for dictating standup updates or quick progress notes without typing

### Time Logging

When a timer has accumulated at least 1 minute, a **Log** button appears on the card.

Clicking Log:
1. Pauses the timer automatically
2. Opens a modal pre-filled with the elapsed hours and minutes
3. Add an optional work description comment
4. For Jira tickets: checkbox to also post the description as a Jira comment
5. Submit → posts worklog to Jira API, saves to local day log, resets the timer

If the Jira API call fails (network issues, auth problems), the entry is saved locally with a "Local" sync status so you don't lose the data.

### Daily Overview

#### Top Bar
- **Today timer**: Live-updating total of all active timers + logged time
- **Progress bar**: Visual percentage toward your daily target (configurable, default 8h)

#### Timeline Bar
- Horizontal stacked bar chart showing proportional time distribution
- Each segment is color-coded to match its card
- Hover to see task name and duration

#### Day Log Table
- Toggle via the **Day Log** button in the top bar
- Shows all logged entries: type, name, time, comment, sync status
- **CSV Export**: Download the day's log as a CSV file for pasting into external timesheets

---

## Jira Integration

### Setup

1. Open `index.html` in a browser
2. Enter your Jira Cloud domain (e.g. `yourcompany.atlassian.net`)
3. Enter your Atlassian account email
4. Enter a Jira API token — generate one at [id.atlassian.com/manage-profile/security/api-tokens](https://id.atlassian.com/manage-profile/security/api-tokens)
5. Set your daily hour target
6. Click **Connect & Load Tickets**

Credentials are stored in your browser's `localStorage` only. Nothing is sent anywhere except directly to your Jira instance via authenticated API calls.

### API Endpoints Used

| Action | Method | Endpoint |
|--------|--------|----------|
| Fetch assigned tickets | GET | `/rest/api/3/search?jql=assignee=currentUser()` |
| Log time (worklog) | POST | `/rest/api/3/issue/{key}/worklog` |
| Post comment | POST | `/rest/api/3/issue/{key}/comment` |

All request/response bodies use the Jira v3 [Atlassian Document Format (ADF)](https://developer.atlassian.com/cloud/jira/platform/apis/document/structure/) for comment content.

### CORS Note

Jira Cloud allows cross-origin requests with Basic auth from browser-based apps. If your instance has restrictive CORS policies, you may need to access this page from a domain allowed by your Jira admin, or use a browser extension to relax CORS during development.

---

## Data Persistence

| Data | Storage | Scope |
|------|---------|-------|
| Jira credentials & settings | `localStorage` (`tc2_config`) | Permanent until cleared |
| Dashboard slots & timer state | `localStorage` (`tc2_slots`) | Permanent, survives refresh |
| Day log entries | `localStorage` (`tc2_log_YYYY-MM-DD`) | One key per day |

Timer state includes `startedAt` timestamps, so running timers correctly accumulate elapsed time across page refreshes.

Auto-save runs every 3 seconds.

---

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `1`–`8` | Toggle timer for slot 1–8 |
| `Esc` | Close any open modal or popover |

Shortcuts are disabled when an input field is focused.

---

## Browser Support

- **Chrome / Edge** (recommended): Full support including voice-to-text
- **Firefox**: Everything works except voice-to-text (Web Speech API not supported)
- **Safari**: Works, voice-to-text may have limited support

---

## Tech Stack

- Single HTML file, zero external dependencies
- Vanilla JavaScript (no framework)
- Modern CSS: Grid, custom properties, backdrop-filter, SVG animations
- Web Speech API for voice recognition
- Jira Cloud REST API v3 with Basic auth

---

## File Structure

```
thoughts/
├── index.html    # The entire application (HTML + CSS + JS)
├── CLAUDE.md     # Claude Code project config
└── README.md     # This file
```

---

## License

MIT
