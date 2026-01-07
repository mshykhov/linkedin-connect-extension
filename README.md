# LinkedIn AI Connect

Chrome extension for personalized LinkedIn connection requests with AI-generated messages.

## Tech Stack

- **Manifest V3** (required 2025+)
- **Vite 6.0** + **TypeScript 5.9**
- **TailwindCSS 4.1**
- **AI Providers**: OpenAI (gpt-4.1-mini) + Claude API (configurable in settings)

## Project Structure

```
src/
├── manifest.json       # Extension manifest (MV3)
├── background/         # Service worker
│   └── index.ts
├── content/            # Scripts injected into LinkedIn
│   ├── index.ts
│   └── styles.css
├── popup/              # Extension popup UI
│   ├── popup.html
│   ├── popup.ts
│   └── popup.css
├── storage/            # Chrome storage wrapper
│   └── index.ts
├── api/                # AI API clients (OpenAI + Claude)
│   └── index.ts
├── types/              # TypeScript types
│   └── index.ts
└── utils/              # Utility functions
    └── index.ts
```

---

## Feature Plan

### Phase 1: Core Infrastructure

| # | Feature | Module | Description |
|---|---------|--------|-------------|
| 1.1 | Types definition | `types/` | Define interfaces for Profile, Settings, Messages, History |
| 1.2 | Storage wrapper | `storage/` | CRUD for chrome.storage.local (queue, settings, history) |
| 1.3 | Message passing | `background/` + `content/` | Communication between scripts |
| 1.4 | Basic popup UI | `popup/` | Shell with tabs: Queue, History, Settings |

### Phase 2: Profile Collection

| # | Feature | Module | Description |
|---|---------|--------|-------------|
| 2.1 | Detect LinkedIn profile | `content/` | Detect profile anywhere (search, feed, job, profile page) |
| 2.2 | Parse profile data | `content/` | Extract name, headline, company, profile URL |
| 2.3 | Right-click context menu | `background/` | "Add to Queue" via right-click on any LinkedIn page |
| 2.4 | Minimal floating button | `content/` | Small unobtrusive [+] near profile, hidden from LinkedIn detection |
| 2.5 | Bulk add on search | `content/` | "Add All Visible" button on search results page |
| 2.6 | Duplicate detection | `storage/` | Prevent adding same profile twice |
| 2.7 | Skip if connected | `content/` | Auto-skip 1st degree connections |

### Phase 3: AI Message Generation

| # | Feature | Module | Description |
|---|---------|--------|-------------|
| 3.1 | OpenAI client | `api/` | GPT-4.1-mini API wrapper |
| 3.2 | Claude client | `api/` | Claude API wrapper (alternative provider) |
| 3.3 | Provider selector | `storage/` | Setting to choose OpenAI or Claude |
| 3.4 | Prompt template | `storage/` | User-configurable prompt with variables |
| 3.5 | Generate message | `background/` | Call selected AI with profile context |
| 3.6 | User context | `storage/` | Store info about user (skills, goals, looking for) |

### Phase 4: Queue Management

| # | Feature | Module | Description |
|---|---------|--------|-------------|
| 4.1 | Queue list view | `popup/` | Show queued profiles with status (pending/sent/failed) |
| 4.2 | Queue cleanup actions | `popup/` | Clear all, Clear processed, Remove selected |
| 4.3 | Auto-clean toggle | `storage/` | Setting: auto-remove processed profiles |
| 4.4 | Profile card preview | `popup/` | Show profile info + generated message preview |
| 4.5 | Edit message | `popup/` | Edit AI-generated message before sending |
| 4.6 | Manual actions | `popup/` | Skip / Remove buttons per profile |

### Phase 5: Connection Flow

| # | Feature | Module | Description |
|---|---------|--------|-------------|
| 5.1 | Process queue | `background/` | Go to each profile, trigger connect flow |
| 5.2 | Auto-connect mode | `content/` | AI clicks Connect → fills message → (optional) sends |
| 5.3 | Review mode toggle | `storage/` | Setting: Auto-send ON/OFF (if OFF, waits for user confirm) |
| 5.4 | Fill & wait | `content/` | Fill message, highlight Send button, wait for user |
| 5.5 | Auto-send | `content/` | If enabled, automatically click Send |
| 5.6 | Update status | `storage/` | Mark profile as sent/skipped/failed |
| 5.7 | Next profile | `background/` | Auto-advance to next after delay |

### Phase 6: History & Tracking

| # | Feature | Module | Description |
|---|---------|--------|-------------|
| 6.1 | History log | `storage/` | Track all sent requests with date, profile, message |
| 6.2 | History view | `popup/` | Tab showing sent connections history |
| 6.3 | Export history | `popup/` | Download history as JSON/CSV |
| 6.4 | Daily counter | `storage/` | Track sends per day |
| 6.5 | Limit warning | `popup/` | Warn when approaching daily limit (e.g. 20/day) |

### Phase 7: Settings & UX

| # | Feature | Module | Description |
|---|---------|--------|-------------|
| 7.1 | Settings panel | `popup/` | All settings in one place |
| 7.2 | API keys config | `popup/` | OpenAI key, Claude key, select active provider |
| 7.3 | User profile | `popup/` | Your info for AI to use in messages |
| 7.4 | Prompt editor | `popup/` | Edit AI prompt template |
| 7.5 | Auto-send toggle | `popup/` | ON = fully auto, OFF = review each message |
| 7.6 | Auto-clean toggle | `popup/` | Auto-remove sent profiles from queue |
| 7.7 | Delay setting | `popup/` | Delay between actions (30-120s) |
| 7.8 | Dark mode | `popup/` + `content/` | Match system theme |

### Phase 8: Safety & Polish

| # | Feature | Module | Description |
|---|---------|--------|-------------|
| 8.1 | Random delays | `utils/` | Humanize timing (30-60s ± random) |
| 8.2 | Daily limit | `storage/` | Hard stop at X connections/day |
| 8.3 | Error handling | all | Graceful failures, show errors in popup |
| 8.4 | LinkedIn SPA | `content/` | Re-detect profiles on navigation |
| 8.5 | Stealth mode | `content/` | Minimal DOM footprint, avoid detection |

---

## User Flow

```
┌─────────────────────────────────────────────────────────────────┐
│ 1. COLLECT PROFILES (from anywhere on LinkedIn)                 │
│                                                                 │
│    Option A: Right-click → "Add to Queue"                      │
│    Option B: Click small [+] button near profile               │
│    Option C: "Add All" on search results page                  │
│                                                                 │
│    ✓ Duplicate detection (won't add twice)                     │
│    ✓ Skip already connected (1st degree)                       │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 2. MANAGE QUEUE (Popup)                                         │
│                                                                 │
│    ┌───────────────────────────────────────────────────────┐   │
│    │ Queue (15)  │  History (47)  │  Settings              │   │
│    ├───────────────────────────────────────────────────────┤   │
│    │ [Clear All] [Clear Processed]                         │   │
│    │                                                       │   │
│    │ ○ John Smith - HR @ Google              [×]          │   │
│    │ ○ Jane Doe - Recruiter @ Meta           [×]          │   │
│    │ ✓ Bob Wilson - Sent 2 min ago           [×]          │   │
│    │                                                       │   │
│    │                              [▶ Start Processing]     │   │
│    └───────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 3. PROCESS (Auto or Review mode based on settings)              │
│                                                                 │
│    [Start] → Opens profile → AI generates message               │
│                                                                 │
│    IF auto-send = OFF (Review Mode):                           │
│    ┌───────────────────────────────────────────────────────┐   │
│    │ John Smith - HR Manager @ Google                      │   │
│    │                                                       │   │
│    │ Generated message:                                    │   │
│    │ ┌─────────────────────────────────────────────────┐  │   │
│    │ │ Hi John! I see you're hiring at Google.        │  │   │
│    │ │ I'm a DevOps engineer with 5 years exp...      │  │   │
│    │ └─────────────────────────────────────────────────┘  │   │
│    │                                                       │   │
│    │ [✓ Send & Next]  [✎ Edit]  [→ Skip]                  │   │
│    └───────────────────────────────────────────────────────┘   │
│                                                                 │
│    IF auto-send = ON:                                          │
│    → Click Connect → Fill message → Send → Next (auto)         │
└─────────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────────┐
│ 4. HISTORY                                                      │
│                                                                 │
│    All sent connections logged with:                           │
│    - Date/time                                                 │
│    - Profile name & URL                                        │
│    - Message sent                                              │
│    - Status (sent/accepted/pending)                            │
└─────────────────────────────────────────────────────────────────┘
```

---

## Settings

| Setting | Options | Default |
|---------|---------|---------|
| AI Provider | OpenAI / Claude | OpenAI |
| Auto-send | ON / OFF | OFF (review mode) |
| Auto-clean queue | ON / OFF | OFF |
| Delay between actions | 30-120 seconds | 45s |
| Daily limit | 10-50 connections | 20 |
| Dark mode | System / Light / Dark | System |

---

## Development

```bash
# Install dependencies
npm install

# Development (watch mode)
npm run dev

# Build for production
npm run build

# Load in Chrome:
# 1. Go to chrome://extensions
# 2. Enable Developer Mode
# 3. Click "Load unpacked"
# 4. Select ./dist folder
```

---

## Decisions Made

- [x] AI Provider: Both OpenAI + Claude (configurable)
- [x] Automation: Toggle between auto-send and review mode
- [x] Language: EN only
- [x] Storage: Local only (chrome.storage.local)
- [x] Theme: System theme matching (dark mode)
