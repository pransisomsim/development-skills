---
name: simulate-html
description: >
  Generate a single-file simulation HTML to manually test ayos-moto-server backend features.
  Matches the visual patterns, UI conventions, and JS architecture of existing simulate/ files.
  Use when asked to create a simulation, simulator, or test page for any backend feature.
---

# Simulate HTML Skill

Generate self-contained single-file HTML simulators for testing ayos-moto-server backend endpoints.
Match existing simulate/ conventions exactly.

---

## 1. Existing Files — Study Before Generating

Read at least one closely related existing file before generating:

| File | What it covers |
|------|---------------|
| `simulate/notifications/shop-owner-application.html` | Multi-role (customer + admin), login, notifications, bottom-sheet modals, event log |
| `simulate/booking/shop-owner-dashboard.html` | Token gate, tabbed list, booking actions, mechanic CRUD, bottom-sheet modals |
| `simulate/booking/create-booking.html` | Simple form, token verify debounce, dynamic selects, idempotency keys |

---

## 2. Visual Design Conventions

### CSS Variables (always use)
```css
:root {
    --primary: #FF6B35;
    --primary-dark: #e55a26;
    --bg: #f0f2f5;
    --card: #ffffff;
    --text: #1a1a1a;
    --text-secondary: #666;
    --border: #e0e0e0;
    --radius: 12px;
    --radius-sm: 8px;
}
```

### Layout
- `body`: `max-width: 430px` (phone), centered, `font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif`
- `.phone-bar`: sticky top header, `background: var(--primary)`, white text, `padding: 44px 20px 12px`
- `.container` (simple pages): `max-width: 800px`, white card, shadow

### Token Status Badge
```html
<div id="tokenBadge" class="token-status-badge badge-idle">⬤ No token</div>
```
Classes: `badge-idle`, `badge-checking`, `badge-valid`, `badge-invalid`

### Status Chips
```html
<span class="status-chip chip-pending">pending</span>
```
Chip variants: `chip-pending`, `chip-active`, `chip-accepted`, `chip-declined`, `chip-cancelled`, `chip-in_service`, `chip-completed`, `chip-rejected`

### Buttons
```css
.btn-primary { background: var(--primary); color: white; }
.btn-green   { background: #28a745; color: white; }
.btn-red     { background: #dc3545; color: white; }
.btn-gray    { background: #ececec; color: var(--text); }
.btn-blue    { background: #007bff; color: white; }
```

### Bottom Sheet / Overlay
```html
<div class="overlay" id="myOverlay" onclick="closeOverlay('myOverlay')">
    <div class="sheet" onclick="event.stopPropagation()">
        <div class="sheet-handle"></div>
        <h3>Sheet Title</h3>
        <p class="sheet-sub">Subtitle</p>
        <!-- inputs -->
        <div class="sheet-actions">
            <button class="sheet-btn sheet-btn-cancel" onclick="closeOverlay('myOverlay')">Cancel</button>
            <button class="sheet-btn sheet-btn-primary" onclick="submitAction()">Confirm</button>
        </div>
    </div>
</div>
```

```js
function openOverlay(id) {
    document.getElementById(id).classList.add('show');
    document.body.style.overflow = 'hidden';
}
function closeOverlay(id) {
    document.getElementById(id).classList.remove('show');
    document.body.style.overflow = '';
}
```

### Toast
```html
<div class="toast" id="toast"></div>
```
```js
let toastTimer = null;
function showToast(msg, type = '') {
    const t = document.getElementById('toast');
    t.textContent = msg;
    t.className = `toast ${type}`;
    void t.offsetWidth;
    t.classList.add('show');
    clearTimeout(toastTimer);
    toastTimer = setTimeout(() => t.classList.remove('show'), 3200);
}
```
Types: `'success'`, `'error'`, `''`

### Spinner / Loading
```html
<div class="loading-wrap">
    <div class="spinner"></div>
    <p>Loading...</p>
</div>
```

### Empty State
```html
<div class="empty-state">
    <div class="icon">📭</div>
    <p>No items found.</p>
</div>
```

### Tabs (status or section)
```html
<div class="tab-bar" id="statusTabBar">
    <button class="tab-btn active" onclick="switchTab('all')">All <span class="tab-badge" id="badge-all">0</span></button>
    <button class="tab-btn" onclick="switchTab('pending')">Pending <span class="tab-badge zero" id="badge-pending">0</span></button>
</div>
```

### Server Health Dot (multi-role / notification sims)
```html
<div class="base-bar">
    <span class="health-dot" id="healthDot"></span>
    <input id="baseUrl" value="http://localhost:3000" onchange="saveBase()">
    <button class="btn btn-gray btn-sm" onclick="checkHealth()">Check</button>
</div>
```
```js
async function checkHealth() {
    const dot = document.getElementById('healthDot');
    dot.className = 'health-dot';
    try {
        const res = await fetch(`${baseUrl}/health`);
        dot.className = res.ok ? 'health-dot online' : 'health-dot offline';
    } catch {
        dot.className = 'health-dot offline';
    }
}
```

### Event Log (multi-role sims)
```js
function logEvent(step, detail, tone = 'info') {
    state.events.unshift({ step, detail, tone, time: new Date() });
    renderEvents();
    updateTabBadges();
}
```

---

## 3. JS Architecture Conventions

### HTTP Helper (multi-role sims)
```js
async function request(path, opts = {}) {
    const headers = { 'Content-Type': 'application/json' };
    if (opts.token) headers.Authorization = `Bearer ${opts.token}`;
    try {
        const res = await fetch(`${baseUrl}${path}`, {
            method: opts.method || 'GET',
            headers,
            body: opts.body ? JSON.stringify(opts.body) : undefined,
        });
        let data = {};
        const text = await res.text();
        if (text) { try { data = JSON.parse(text); } catch { data = { message: text }; } }
        return { res, data };
    } catch (error) {
        throw new Error(`Network error: ${error.message}`);
    }
}
```

### apiFetch Helper (single-token sims)
```js
function apiFetch(path, opts = {}, tkn = null) {
    const t = tkn || token;
    return fetch(`${BASE}${path}`, {
        ...opts,
        headers: {
            'Content-Type': 'application/json',
            'Authorization': `Bearer ${t}`,
            ...(opts.headers || {}),
        },
        body: opts.body ? JSON.stringify(opts.body) : undefined,
    });
}
```

### JWT Decode (no signature validation)
```js
function decodeJwtPayload(token) {
    try {
        const parts = token.split('.');
        if (parts.length !== 3) return null;
        const payload = parts[1].replace(/-/g, '+').replace(/_/g, '/');
        const padded = payload + '=='.slice(0, (4 - payload.length % 4) % 4);
        return JSON.parse(atob(padded));
    } catch {
        return null;
    }
}
```

### Token Debounce Verify Pattern
```js
let tokenVerifyTimer = null;
function scheduleTokenVerify() {
    clearTimeout(tokenVerifyTimer);
    const token = document.getElementById('authToken').value.trim();
    if (!token) { setTokenStatus('idle', '⬤ No token'); return; }
    setTokenStatus('checking', '⏳ Checking...');
    tokenVerifyTimer = setTimeout(verifyToken, 600);
}
```

### Session Persistence (multi-role sims)
```js
function persistSession(role) {
    const key = role === 'customer' ? STORAGE_CUSTOMER : STORAGE_ADMIN;
    sessionStorage.setItem(key, JSON.stringify({ token: state[role].token, user: state[role].user }));
}
function restoreSession(role) {
    const raw = sessionStorage.getItem(role === 'customer' ? STORAGE_CUSTOMER : STORAGE_ADMIN);
    if (!raw) return;
    try {
        const saved = JSON.parse(raw);
        if (!saved.token) return;
        state[role].token = saved.token;
        state[role].user = saved.user || null;
        showWorkspace(role);
    } catch {
        sessionStorage.removeItem(role === 'customer' ? STORAGE_CUSTOMER : STORAGE_ADMIN);
    }
}
```

### escapeHtml (always use for user/API data in innerHTML)
```js
function escapeHtml(value) {
    return String(value ?? '')
        .replace(/&/g, '&amp;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;')
        .replace(/"/g, '&quot;')
        .replace(/'/g, '&#039;');
}
```

### formatDate
```js
function formatDate(value) {
    if (!value) return '—';
    const date = new Date(value);
    if (Number.isNaN(date.getTime())) return '—';
    return date.toLocaleString('en-PH', {
        month: 'short', day: 'numeric', year: 'numeric',
        hour: '2-digit', minute: '2-digit',
    });
}
```

---

## 4. Page Types

### A. Simple Form Page
**Use for:** testing a single POST/GET endpoint (e.g., create booking, submit form).

Structure:
1. `<h1>` with emoji + title
2. `<p class="subtitle">` describing purpose
3. `.info-box` with test scenarios / instructions
4. Bearer token input + token-status badge
5. Form fields relevant to the endpoint
6. Submit button(s)
7. `#response` div (shows success/error + JSON)

Reference: `create-booking.html`

### B. Dashboard Page (single-role, token-gated)
**Use for:** role-specific dashboards with list + action flows (e.g., shop owner, mechanic).

Structure:
1. `.phone-bar` sticky header
2. Token gate screen (paste JWT → verify → enter)
3. Main dashboard (hidden until token valid):
   - Owner/user info bar
   - Main section tabs (e.g., Bookings / Mechanics)
   - Status filter tab bar (All, Pending, Accepted…)
   - List with cards
   - Action buttons → bottom sheet modals

Reference: `shop-owner-dashboard.html`

### C. Multi-Role Simulator
**Use for:** flows involving multiple actors (customer + admin, owner + customer, etc.) plus notification verification.

Structure:
1. `.phone-bar` title
2. `.base-bar` (URL input + health dot)
3. Main tabs (one per role + Event Log)
4. Per-role panel:
   - Login card (email/password + manual token option)
   - Identity bar (after login)
   - Feature cards (forms, lists, notifications inbox)
5. Event Log panel
6. Bottom sheet modals per action
7. Toast

Reference: `shop-owner-application.html`

---

## 5. File Placement

```text
simulate/
├── booking/
├── notifications/
├── <feature>/
│   └── <feature-name>.html
```

Place the file inside `simulate/<module>/` matching the backend module name.

---

## 6. Generation Checklist

Before writing the file:

- [ ] Identify page type: A (simple form), B (dashboard), or C (multi-role)
- [ ] Identify endpoint(s): method, URL, auth role required
- [ ] Identify dynamic data (dropdowns, lists) that need fetching
- [ ] Identify action flows (single submit vs. multi-step vs. status transitions)
- [ ] Check if notifications / event log needed (multi-role only)
- [ ] Check existing simulate/ file for closely matching pattern

When writing:

- [ ] Single `.html` file, no external dependencies (no CDN, no libraries)
- [ ] All CSS in `<style>`, all JS in `<script>` at bottom of body
- [ ] Use CSS variables
- [ ] Use `escapeHtml()` for any API/user data rendered via `innerHTML`
- [ ] Token debounce verify for all protected endpoints
- [ ] `apiFetch` or `request` helper — never raw fetch scattered everywhere
- [ ] Bottom-sheet overlays for confirmation/form actions (not browser `confirm()`)
- [ ] Toast for success/error feedback
- [ ] Spinner while loading, empty state when no data
- [ ] `window.addEventListener('load', ...)` for initialization
- [ ] Default `baseUrl = 'http://localhost:3000'` (configurable in multi-role sims)
- [ ] Seeded test credentials pre-filled where appropriate (e.g., `customer4@gmail.com` / `Password123!`)

---

## 7. Common Endpoint Patterns in ayos-moto-server

| Feature | Endpoints |
|---------|-----------|
| Auth | `POST /api/auth/login`, `POST /api/auth/refresh` |
| Vehicles | `GET /api/vehicles` |
| Shops | `GET /api/shops`, `POST /api/shops/apply`, `PUT /api/shops/:id/verify` |
| Shop services | `GET /api/shops/:id/services` |
| Bookings | `POST /api/bookings`, `GET /api/bookings`, `POST /api/bookings/:id/accept`, `/decline`, `/start`, `/complete`, `/assign-mechanic` |
| Mechanics | `GET /api/mechanics`, `POST /api/mechanics`, `PUT /api/mechanics/:id`, `DELETE /api/mechanics/:id` |
| Notifications | `GET /api/notifications`, `GET /api/notifications/unread-count`, `PATCH /api/notifications/:id/read`, `PATCH /api/notifications/read-all` |
| Health | `GET /health` |

Auth roles: `CUSTOMER`, `SHOP_OWNER`, `SUPER_ADMIN`

---

## 8. Quick Reference — Reusable Snippets

### Idempotency Key + SHA-256 Hash (booking)
```js
function generateUUID() {
    return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, c => {
        const r = Math.random() * 16 | 0;
        const v = c === 'x' ? r : (r & 0x3 | 0x8);
        return v.toString(16);
    });
}
async function generateHash(data) {
    const encoder = new TextEncoder();
    const buffer = encoder.encode(JSON.stringify(data));
    const hashBuffer = await crypto.subtle.digest('SHA-256', buffer);
    return Array.from(new Uint8Array(hashBuffer)).map(b => b.toString(16).padStart(2, '0')).join('');
}
```

### Notification Styles Map
```js
const NOTIF_STYLES = {
    SHOP_OWNER_APPLICATION: { icon: 'APP', cls: 'application' },
    SHOP_OWNER_APPROVED:    { icon: 'OK',  cls: 'approved' },
    SHOP_OWNER_REJECTED:    { icon: 'NO',  cls: 'rejected' },
    BOOKING_REQUEST:        { icon: 'BKG', cls: 'booking' },
    BOOKING_CONFIRMED:      { icon: 'BKG', cls: 'booking' },
    BOOKING_CANCELED:       { icon: 'BKG', cls: 'booking' },
    SYSTEM:                 { icon: 'SYS', cls: 'system' },
};
```

### Mechanic Picker (radio list)
See `shop-owner-dashboard.html` → `renderMechanicPicker()` / `selectMechanicPicker()`.

### Booking Status Chip Colors
```js
const CHIP_MAP = {
    pending:    'chip-pending',
    accepted:   'chip-accepted',
    declined:   'chip-declined',
    cancelled:  'chip-cancelled',
    in_service: 'chip-in_service',
    completed:  'chip-completed',
};
```
