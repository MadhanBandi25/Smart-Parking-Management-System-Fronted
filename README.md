# 🅿️ SmartParking — Frontend

> **Park Smart. Live Easy.**  
> A fully responsive, single-file vanilla HTML/CSS/JavaScript frontend for the Smart Parking Management System. No build tools, no frameworks — just open in a browser and it works.

---

## 📋 Table of Contents

- [Overview](#-overview)
- [Tech Stack](#-tech-stack)
- [UI Architecture](#-ui-architecture)
- [State Management](#-state-management)
- [Roles & Access Control](#-roles--access-control)
- [Pages & Modules](#-pages--modules)
- [Features](#-features)
- [API Integration](#-api-integration)
- [Authentication Flow](#-authentication-flow)
- [Theme System](#-theme-system)
- [Setup & Running](#-setup--running)
- [File Structure](#-file-structure)
- [Component Reference](#-component-reference)
- [Known Limitations & Future Improvements](#-known-limitations--future-improvements)
- [Browser Compatibility](#-browser-compatibility)
- [Backend Repository](#-backend-repository)

---

## 🌟 Overview

SmartParking is a **role-based parking management web application** that allows users to find and book parking slots, make payments, and scan QR tickets — while giving admins, parking owners, and security personnel their own dedicated dashboards and tools.

```
┌──────────────────────────────────────────────────────────────┐
│         SMARTPARKING FRONTEND (index.html)                   │
│                                                              │
│  ┌──────────┐   ┌───────────────┐   ┌────────────────────┐   │
│  │  HTML    │   │   CSS         │   │    JavaScript      │   │
│  │ Markup   │   │ (Inline)      │   │    (Inline)        │   │
│  │ (Pages)  │   │ Theming +     │   │  API + Logic       │   │
│  │          │   │ Animations    │   │  Role Routing      │   │
│  └──────────┘   └───────────────┘   └────────────────────┘   │
│                                                              │
│        Single file · No dependencies · No build step         │
└──────────────────────────────────────────────────────────────┘
                            │
                            │  fetch() REST calls (JWT)
                            ▼
              Spring Boot API (localhost:8080)
```

The entire frontend is a **single `index.html` file** with embedded CSS and JavaScript that communicates with a Spring Boot REST API backend.

---

## 🛠 Tech Stack

| Layer | Technology |
|-------|-----------|
| Markup | HTML5 |
| Styling | CSS3 (CSS Variables, Flexbox, Grid, Animations) |
| Scripting | Vanilla JavaScript (ES6+, async/await) |
| Fonts | Google Fonts — Cabinet Grotesk, Manrope, Instrument Sans, Bebas Neue |
| HTTP Client | Fetch API (native browser) |
| Storage | `sessionStorage` (JWT + user session), `localStorage` (theme, city preference) |
| No frameworks | Zero dependencies, zero build tools |
| Backend | Spring Boot REST API (`http://localhost:8080`) |

---

## 🏗 UI Architecture

### Page System

The frontend uses a **CSS display toggle system** for routing. Every top-level section is a `.page` div. Only the active page/view is visible at any time.

```
body
 ├── Auth Pages
 │     ├── #loginPage       → Login form
 │     ├── #regPage         → Registration form
 │     ├── #verifyPage      → OTP verification
 │     ├── #forgotPage      → Forgot password
 │     └── #resetPage       → Reset password with OTP
 │
 └── App Shell (post-login, role-based)
       ├── .sidebar         → Role-aware navigation
       └── .main
             ├── .topbar    → City picker, theme toggle, notifications
             └── .content
                   ├── USER PAGES
                   │     ├── #dashPage        → User dashboard
                   │     ├── #findPage        → Find & book parking
                   │     ├── #booksPage       → My bookings
                   │     ├── #vehsPage        → My vehicles
                   │     ├── #paysPage        → My payments
                   │     ├── #qrPage          → QR e-tickets
                   │     ├── #notifsPage      → Notifications
                   │     └── #profilePage     → Profile settings
                   │
                   ├── ADMIN PAGES
                   │     ├── #adminDashPage   → Admin dashboard
                   │     ├── #admUsersPage    → User management
                   │     ├── #admBksPage      → All bookings
                   │     ├── #analyticsPage   → Revenue analytics
                   │     ├── #reportsPage     → Financial reports
                   │     └── #parkAreasPage   → Areas, slots & rates
                   │
                   ├── OWNER PAGES
                   │     └── #ownerDashPage   → Owner dashboard
                   │
                   └── SECURITY PAGES
                         └── #qrScanPage      → QR scan & checkout
```

---

## 🧠 State Management

All application state lives in plain JavaScript variables (no external store):

```javascript
// Auth & Session
let TOKEN  = sessionStorage.getItem('sp_tok')  || '';
let ROLE   = sessionStorage.getItem('sp_role') || '';
let UNAME  = sessionStorage.getItem('sp_name') || '';
let UEMAIL = sessionStorage.getItem('sp_email')|| '';
let UPHONE = sessionStorage.getItem('sp_phone')|| '';

// Booking Flow
let _curPayBkId   = null;     // booking ID currently being paid
let _selSlotIdS2  = null;     // selected slot in booking step 2
let _curAreaIdS2  = null;     // selected parking area in booking
let _allMyBks     = [];       // cached personal bookings
let _curFilteredBks = [];     // filtered bookings list
let _bkPage       = 1;        // bookings pagination

// Find Parking
let _findVType    = '';        // selected vehicle type filter
let _allAreas     = [];        // cached parking areas
let _parkingRates = [];        // cached rate config
let _selectedCity = '';        // active city filter (localStorage)

// QR & Security
let _curQrB64         = null;  // base64 QR image
let _qrCurrentTab     = 'active';
let _checkoutData     = null;  // checkout response data
let _secPollInterval  = null;  // security payment polling timer
let _secPollBookingId = null;

// Payment
let _payInterval      = null;  // payment countdown timer
let _paySecsLeft      = 180;   // 3-minute payment window
let _paymentInProgress = false;

// Admin
let _allUsers       = [];      // all users from API
let _filteredUsers  = [];      // after search/role/date filters
let _admAllBks      = [];      // all bookings (admin)
let _admBksFiltered = [];      // filtered bookings
let _admBksPage     = 1;

// Analytics & Reports
let _anFrom = '', _anTo = '';  // analytics date range
let _anVtSelected  = '';       // vehicle type filter
let _arAllEntries  = [];       // revenue report rows
let _arPage        = 1;

// UI
let _isDark    = localStorage.getItem('sp_theme') !== 'light';
let _debTimer  = null;         // search debounce timer
```

---

## 👥 Roles & Access Control

The application supports **four roles**, each with a completely different navigation and page set:

### 🙋 USER
The default role for registered customers.
- Book parking slots
- Manage personal vehicles
- View and pay invoices (including extra/overtime charges)
- Access QR-based e-tickets
- View notifications and profile

### 🛡 ADMIN
Full system control.
- Manage all users (view, filter by role, soft-delete, restore)
- View and manage all bookings system-wide
- Manage parking areas, slots, and pricing rates
- Access analytics and financial reports with date range filters

### 🏢 PARKING_OWNER
Manage owned parking infrastructure.
- View personal parking areas dashboard
- Generate and configure parking slots by floor
- Set and manage parking rates per vehicle type

### 🔒 SECURITY
On-ground operations at parking facilities.
- Scan and verify QR codes
- Perform vehicle check-in and check-out
- Apply extra/overtime charges during checkout
- Monitor real-time payment status via auto-polling

---

## 📄 Pages & Modules

### Auth Pages (pre-login)

| Page ID | Purpose |
|---------|---------|
| `loginPage` | Email + password login |
| `regPage` | Registration with role selection |
| `verifyPage` | 6-digit OTP email verification |
| `forgotPage` | Request password reset OTP |
| `resetPage` | Enter OTP + new password |

### User Pages (post-login, USER role)

| Page ID | Sidebar Item | Purpose |
|---------|-------------|---------|
| `dashPage` | 🏠 Dashboard | Stats overview and recent activity |
| `findPage` | 🔍 Find Parking | Multi-step: city → area → slot → book |
| `booksPage` | 📋 My Bookings | Booking list with filters and pagination |
| `vehsPage` | 🚗 My Vehicles | Add and manage registered vehicles |
| `paysPage` | 💳 My Payments | Payment history with date filters |
| `qrPage` | 🎫 QR Tickets | Active and past e-tickets with QR codes |
| `notifsPage` | 🔔 Notifications | Read/unread notifications |
| `profilePage` | 👤 Profile | View and edit personal profile |

### Admin Pages (ADMIN role)

| Page ID | Sidebar Item | Purpose |
|---------|-------------|---------|
| `adminDashPage` | 🏠 Dashboard | System-wide stats |
| `admUsersPage` | 👥 Users | User list, search, filter by role, delete/restore |
| `admBksPage` | 📋 All Bookings | All bookings with check-in/out controls |
| `analyticsPage` | 📊 Analytics | Revenue charts by vehicle type and area |
| `reportsPage` | 📄 Reports | Financial report table with export |
| `parkAreasPage` | 🏢 Park Areas | Areas, slots, rates management |

### Owner & Security Pages

| Page ID | Role | Purpose |
|---------|------|---------|
| `ownerDashPage` | PARKING_OWNER | Owner stats, areas, slots, rates |
| `qrScanPage` | SECURITY | QR verify, check-in, checkout, extra charges |

---

## ✨ Features

### Authentication
- 📧 Email + password login with JWT
- 📝 User registration with role selection (USER / ADMIN / PARKING_OWNER / SECURITY)
- 🔢 OTP-based email verification with resend countdown
- 🔑 Forgot password & OTP-based password reset
- 💪 Live password strength checker
- 👁 Password show/hide toggle
- 🚪 Secure logout (clears all session storage)

### Booking System
- 🔍 Search parking areas by city or address
- 🏙 City picker with searchable dropdown in topbar
- 🗂 Filter available slots by vehicle type (Bike, EV Bike, Car, EV Car, Truck)
- 📊 Real-time slot availability with floor-wise visual grid
- 🕐 Hour-based booking with dynamic cost estimation
- ⚖️ Automatic weekend pricing support
- 🔄 Multi-step booking flow (Area → Slot → Confirm)
- ❌ Booking cancellation

### Payment System
- ⏱ 3-minute payment countdown timer
- 💳 Multiple payment method selection
- 💰 Extra/overtime charge payment (triggered by Security during checkout)
- 🔄 Auto-polling for pending extra charges (security-side payment wait)
- ❌ Payment failure handling with automatic slot release

### QR Ticket System
- 🎫 QR code generation and display per booking
- ⬇️ QR code download button
- ✅ Check-in / Check-out via QR scan (Security role)
- ⏰ Ticket expiry detection
- 🗂 Tab switching between Active and Past tickets
- 📋 Status badges: `ACTIVE`, `BOOKED`, `COMPLETED`, `CANCELLED`, `PENDING_PAYMENT`, `EXPIRED`

### Vehicle Management
- ➕ Add vehicles with type, brand, model, color, and registration number
- 🎨 Color-coded vehicle type cards with emoji icons
- 🗑 Remove registered vehicles

### Admin Tools
- 👤 User management — search, filter by role, paginated table, soft-delete and restore
- 📊 Revenue analytics with custom date range and vehicle type filters
- 📄 Financial reports with area-wise breakdown and load-more pagination
- 🏢 Parking area CRUD — create, delete, restore
- 🎰 Slot generation per area with floor configuration
- 🔧 Slot maintenance toggle
- 💲 Parking rate management — create and delete rates

### Notifications
- 🔔 Unread notification count badge (auto-loaded on init)
- ✅ Mark individual or all notifications as read
- 🔄 Auto-refresh count after mark-as-read

### UI / UX
- 🌙 Dark / ☀️ Light theme toggle (persisted in `localStorage`)
- 🌆 City picker with live search and emoji flags
- 💫 Animated splash/loading screen on app start
- 🍞 Toast notification system (success / error / info)
- 📱 Collapsible sidebar for narrow screens
- ⚡ Smooth page transitions (CSS fade + slide-up animations)
- 🔄 Debounced search inputs to reduce API calls
- 📄 Paginated tables with "Load More" for large datasets
- 🏷 Status and role color badges throughout all tables

---

## 🔌 API Integration

All API calls go through a single `api()` helper:

```javascript
async function api(method, path, body = null) {
  const opts = {
    method,
    headers: { 'Content-Type': 'application/json',
                'Authorization': `Bearer ${TOKEN}` }
  };
  if (body) opts.body = JSON.stringify(body);
  const res = await fetch(BASE + path, opts);
  const data = await res.json().catch(() => ({}));
  if (!res.ok) throw new Error(data.message || 'Request failed');
  return data;
}
```

### Base URL

```javascript
const BASE = 'http://localhost:8080';
```

To point to a different backend, change this one constant.

### Endpoints Used

#### Auth
| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/api/auth/login` | Login and receive JWT |
| `POST` | `/api/auth/register` | Register new user |
| `POST` | `/api/auth/verify` | Verify OTP |
| `POST` | `/api/auth/forgot` | Request password reset OTP |
| `POST` | `/api/auth/reset` | Reset password with OTP |

#### Dashboard
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/dashboard/user` | User dashboard stats |
| `GET` | `/api/dashboard/admin` | Admin dashboard stats |
| `GET` | `/api/dashboard/owner` | Owner dashboard stats |

#### Bookings
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/bookings/my` | Current user's bookings |
| `GET` | `/api/bookings` | All bookings (admin) |
| `GET` | `/api/bookings/{id}` | Booking by ID |
| `POST` | `/api/bookings` | Create booking |
| `PUT` | `/api/bookings/{id}/cancel` | Cancel booking |

#### Parking Areas & Slots
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/parking-areas/all` | All parking areas |
| `GET` | `/api/parking-areas/my` | Owner's parking areas |
| `GET` | `/api/parking-areas/search/city?city=` | Search by city |
| `GET` | `/api/parking-areas/search/address?address=` | Search by address |
| `POST` | `/api/parking-areas` | Create parking area |
| `DELETE` | `/api/parking-areas/{id}` | Delete parking area |
| `PUT` | `/api/parking-areas/{id}/restore` | Restore deleted area |
| `GET` | `/api/parking-slots/area/{id}` | Slots in an area |
| `GET` | `/api/parking-slots/available?parkingAreaId=&vehicleType=` | Available slots |
| `POST` | `/api/parking-slots/generate` | Generate slots for area |
| `GET` | `/api/parking-rates` | All parking rates |
| `POST` | `/api/parking-rates` | Create rate |
| `DELETE` | `/api/parking-rates/{id}` | Delete rate |

#### Payments
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/payments/my` | User's payment history |
| `POST` | `/api/payments` | Make a payment |
| `POST` | `/api/payments/failed/{bookingId}` | Mark payment as failed |
| `POST` | `/api/payments/extra/{bookingId}` | Pay extra/overtime charge |
| `GET` | `/api/payments/date-range?fromDate=&toDate=` | Payments in date range |

#### Vehicles
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/vehicles/my-vehicles` | User's registered vehicles |
| `POST` | `/api/vehicles` | Add vehicle |
| `DELETE` | `/api/vehicles/{id}` | Remove vehicle |

#### QR & Check-in/out
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/qr/booking/{bookingNumber}` | Get QR for booking |
| `GET` | `/api/qr/verify?bookingNumber=` | Verify QR code |
| `PUT` | `/api/qr/check-in/{id}` | Check in vehicle |
| `PUT` | `/api/qr/check-out/{id}` | Check out vehicle |

#### Users & Notifications
| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/api/users/all` | All users (admin) |
| `GET` | `/api/users/profile` | Current user profile |
| `PUT` | `/api/users/profile` | Update profile |
| `DELETE` | `/api/users/{id}` | Soft-delete user |
| `PUT` | `/api/users/{id}/restore` | Restore user |
| `GET` | `/api/notifications/my` | User notifications |
| `GET` | `/api/notifications/my/unread-count` | Unread count |
| `PUT` | `/api/notifications/{id}/read` | Mark as read |

---

## 🔐 Authentication Flow

```
FIRST VISIT
    │
    ▼
Check sessionStorage for 'sp_tok' + 'sp_role'
    │
    ├── Found → initApp() → route to role default page
    │
    └── Not found → showAuth('loginPage')


REGISTER
    │
    ├── POST /api/auth/register
    ├── On success → save pending email → showAuth('verifyPage')
    └── On error   → show inline error toast


OTP VERIFY
    │
    ├── POST /api/auth/verify
    ├── On success → showAuth('loginPage') + toast
    └── On error   → toast error message + allow resend


LOGIN
    │
    ├── POST /api/auth/login
    ├── On success → save TOKEN + ROLE + user info to sessionStorage
    │               → initApp() → route to role default page:
    │                   USER          → dashPage
    │                   ADMIN         → adminDashPage
    │                   PARKING_OWNER → ownerDashPage
    │                   SECURITY      → qrScanPage
    └── On error   → show error message


FORGOT PASSWORD
    │
    ├── POST /api/auth/forgot
    ├── On success → showAuth('resetPage') + toast
    └── On error   → toast error


RESET PASSWORD
    │
    ├── POST /api/auth/reset
    ├── On success → showAuth('loginPage') + toast
    └── On error   → toast error


LOGOUT
    │
    ├── Stop all polling intervals (payment, security)
    ├── clearAuth() — wipe TOKEN, ROLE, user info from sessionStorage
    └── showAuth('loginPage')
```

**Token Usage** — After login, every API call includes:
```
Authorization: Bearer eyJhbGciOiJIUzI1NiJ9...
```

---

## 🎨 Theme System

The entire UI is driven by **CSS custom properties** defined on `:root`. Switching between dark and light mode is a single attribute toggle on `<html>`.

**Dark Mode (default)**
```css
:root {
  --bg:      #07091a;  /* page background */
  --bg2:     #0c1022;  /* sidebar / topbar */
  --bg3:     #121830;  /* inputs / secondary surfaces */
  --card:    #0f1524;  /* card background */
  --accent:  #5b73ff;  /* primary blue-purple */
  --teal:    #06c8e8;
  --green:   #0fda97;
  --orange:  #ffab40;
  --red:     #ff4d6d;
  --text:    #eef2ff;
  --text2:   #8fa3c7;
}
```

**Light Mode**
```css
[data-theme="light"] {
  --bg:   #f0f4ff;
  --bg2:  #e4eaff;
  --card: #ffffff;
  --text: #0f1e3d;
}
```

**Toggling**
```javascript
function toggleTheme() {
  _isDark = !_isDark;
  localStorage.setItem('sp_theme', _isDark ? 'dark' : 'light');
  applyTheme();
}
```

Theme preference is persisted in `localStorage` under the key `sp_theme`.

---

## ⚙️ Setup & Running

### Prerequisites
- ✅ A modern browser (Chrome, Firefox, Edge, Safari)
- ✅ The Spring Boot backend running at `localhost:8080`
- ✅ A local HTTP server (VS Code Live Server, or any static file server)

> **Why a local server?** The browser may block `fetch()` calls from `file://` URLs due to CORS. Serving over HTTP avoids this.

### Option 1: VS Code Live Server (Recommended)
1. Install the **Live Server** extension in VS Code
2. Right-click `index.html` → **Open with Live Server**
3. Browser opens at `http://127.0.0.1:5500`

### Option 2: Python HTTP Server
```bash
# Python 3
python -m http.server 5500
# Then open: http://localhost:5500
```

### Option 3: Node.js serve
```bash
npx serve . -p 5500
# Then open: http://localhost:5500
```

### Backend CORS
If you serve the frontend on a different port, add it to your Spring Security config:
```java
config.setAllowedOrigins(List.of(
    "http://127.0.0.1:5500",
    "http://localhost:5500",
    "http://localhost:YOUR_PORT"  // add here
));
```

---

## 📁 File Structure

```
Smart-Parking-Management-System-Frontend/
└── index.html        ← Entire frontend (HTML + CSS + JS in one file)
```

The entire application — all pages, styles, and logic — lives in a single file.

### Internal Structure of `index.html`

```
index.html
│
├── <head>
│   ├── Google Fonts (Cabinet Grotesk, Manrope, Instrument Sans, Bebas Neue)
│   └── <style> ──────────────────────────────────── ~800 lines of CSS
│         ├── CSS Variables (dark + light theme)
│         ├── Base / reset styles + splash screen
│         ├── Toast notification system
│         ├── Auth page styles (.auth-wrap, .auth-card)
│         ├── App layout (.sidebar, .main, .topbar, .content)
│         ├── Component styles
│         │     ├── Stat cards, tables, badges
│         │     ├── Modals (.modal-ov, .modal)
│         │     ├── Vehicle cards (.veh-card)
│         │     ├── QR ticket cards (.qr-ticket)
│         │     ├── Parking area cards (.area-card)
│         │     └── Slot grid (.area-slot-type)
│         └── Responsive breakpoints
│
├── <body>
│   ├── #splash                        ← Animated loading screen
│   ├── .toast-wrap                    ← Global toast notifications
│   │
│   ├── Auth Pages
│   │   ├── #loginPage
│   │   ├── #regPage
│   │   ├── #verifyPage
│   │   ├── #forgotPage
│   │   └── #resetPage
│   │
│   └── App Shell (#app)
│       ├── .sidebar (role-aware nav)
│       └── .main
│           ├── .topbar (city picker, theme, notifications)
│           └── .content
│               ├── USER PAGES (dashPage, findPage, booksPage, ...)
│               ├── ADMIN PAGES (adminDashPage, admUsersPage, ...)
│               ├── OWNER PAGES (ownerDashPage)
│               └── SECURITY PAGES (qrScanPage)
│
└── <script> ──────────────────────────────────── ~1500 lines of JS
      ├── CONFIG & STATE         (BASE url, session vars, state vars)
      ├── THEME                  (applyTheme, toggleTheme)
      ├── SPLASH                 (splash screen hide on load)
      ├── LOCATION               (initLocPill, toggleLocPill, selectLocCity)
      ├── NAVIGATION             (goPage, buildNav, togSidebar)
      ├── AUTH                   (login, register, verifyOtp, forgotPwd, resetPwd, logout)
      ├── API HELPER             (api)
      ├── INIT                   (initApp, updateGreetings, updSidebarUser)
      ├── USER DASHBOARD         (loadUserDash)
      ├── FIND PARKING           (loadFindPage, loadAreasList, openAreaStep2,
      │                           refreshSlotsStep2, selectSlotS2, createBookingStep2)
      ├── MY BOOKINGS            (loadMyBks, renderBksTbl, filtBks, cancelBk)
      ├── PAYMENTS               (loadPayments, renderPaymentsUI, startPayCountdown,
      │                           stopPayCountdown, cancelPayModal)
      ├── QR TICKETS             (loadQrPage, renderQrCards, showQrForBk,
      │                           switchQrTab, dlQr)
      ├── VEHICLES               (loadVehicles, addVehicle, delVeh)
      ├── NOTIFICATIONS          (loadNotifs, loadNotifCount, tapNotif)
      ├── PROFILE                (loadProfile, saveProfile)
      ├── ADMIN DASHBOARD        (loadAdminDash)
      ├── ADMIN USERS            (loadAdmUsers, renderUsrsTbl, setUsrRole,
      │                           delUser, restoreUser, loadMoreUsers)
      ├── ADMIN BOOKINGS         (loadAdmBks, applyAdmBkFilters, admChkIn,
      │                           admChkOut, loadMoreAdmBks)
      ├── ANALYTICS              (loadAnalytics, renderVtChart, renderOwnerChart,
      │                           selectAnArea, setAnRange, applyAnCustom)
      ├── REPORTS                (loadReport, renderReportTable, loadMoreArRows)
      ├── PARK AREAS (Admin)     (loadParkAreasAdm, renderParkAreaCards,
      │                           addArea, delArea, restoreArea)
      ├── SLOTS                  (filterAreaSlots, renderAreaSlotGrid,
      │                           genSlots, toggleSlotMaintenance)
      ├── RATES                  (loadRates, filterRatesTable, addRate, delRate)
      ├── OWNER DASHBOARD        (loadOwnerDash)
      ├── SECURITY / QR SCAN     (verifyQR, checkIn, checkOut, resolveBooking,
      │                           startSecurityPayPoll, stopSecurityPayPoll)
      └── HELPERS                (toast, openM, closeM, gv, esc, fdt, badge,
                                   stColor, isWeekend, effectiveStatus,
                                   buildTicketStatusRow, btnLoad, togPw, chkStr)
```

---

## 🧩 Component Reference

### CSS Classes Quick Reference

| Class | Purpose |
|-------|---------|
| `.page` / `.page.active` | Page visibility toggle system |
| `.auth-wrap` / `.auth-card` | Auth page wrapper and card |
| `.sidebar` / `.sidebar.collapsed` | Left navigation sidebar states |
| `.main` / `.topbar` / `.content` | App shell layout regions |
| `.stats-grid` / `.sc` | Stats card grid and individual stat card |
| `.card` | Generic surface card |
| `.tw` / `table` / `th` / `td` | Styled data table components |
| `.modal-ov` / `.modal-ov.open` | Modal backdrop and visibility |
| `.modal` / `.modal.wide` | Modal dialog (standard and wide) |
| `.veh-card` | Vehicle card with color-coded type |
| `.qr-ticket` / `.qr-ticket.expired` | QR e-ticket card (active and expired) |
| `.area-card` / `.area-card-header` | Parking area browse card |
| `.area-slot-type` | Slot type count block in area card |
| `.find-topbar` / `.areas-grid` | Find parking layout components |
| `.bp` | Inline badge (`.green`, `.blue`, `.orange`, `.red`, `.teal`) |
| `.btn` / `.btn-p` / `.btn-sec` | Base button, primary, secondary |
| `.btn-sm` / `.btn-danger` / `.btn-ok` | Small, danger, success button variants |
| `.tabs` / `.tab` / `.tab.active` | Tab switcher component |
| `.fg` / `.fl` / `.fi` | Form group, label, input |
| `.toast` / `.toast.success` / `.toast.error` | Toast notification variants |
| `.loc-pill` / `.loc-pill-drop` | City picker pill and dropdown |
| `.otp-row` / `.otp-inp` | OTP input row and individual box |
| `.pb` / `.pf` | Progress bar track and fill |

### JavaScript Functions Quick Reference

| Function | Purpose |
|----------|---------|
| `api(method, path, body)` | Central authenticated API caller |
| `initApp()` | Bootstrap app after login, route by role |
| `goPage(id)` | Navigate to any `.page` by ID |
| `buildNav()` | Build role-specific sidebar navigation |
| `togSidebar()` | Collapse / expand sidebar |
| `showAuth(pg)` | Switch between auth pages |
| `login()` | POST login, save session, init app |
| `register()` | POST register, move to OTP verify |
| `verifyOtp()` | POST OTP verify |
| `forgotPwd()` / `resetPwd()` | Forgot / reset password flow |
| `logout()` | Clear all state, return to login |
| `clearAuth()` | Wipe sessionStorage and state vars |
| `toast(msg, type)` | Show success / error / info toast |
| `openM(id)` / `closeM(id)` | Open / close modal overlay |
| `loadAreasList()` | Fetch and render parking area cards |
| `openAreaStep2(areaId)` | Load slot selection for chosen area |
| `createBookingStep2()` | POST new booking, open payment modal |
| `startPayCountdown()` | Start 3-min payment timer |
| `stopPayCountdown()` | Clear payment timer |
| `loadMyBks()` | Fetch and render personal bookings |
| `cancelBk(id)` | PUT cancel a booking |
| `loadVehicles()` | Fetch and render vehicle cards |
| `addVehicle()` | POST new vehicle |
| `delVeh(id)` | DELETE vehicle |
| `loadQrPage()` | Fetch bookings and render QR tickets |
| `showQrForBk(bkNum)` | Fetch and display QR code modal |
| `dlQr()` | Download QR image |
| `switchQrTab(tab)` | Switch between active/past tickets |
| `loadPayments()` | Fetch and render payment history |
| `loadNotifs()` | Fetch notifications |
| `loadNotifCount()` | Fetch and show unread badge count |
| `loadProfile()` / `saveProfile()` | Fetch / update user profile |
| `loadAdminDash()` | Admin dashboard stats |
| `loadAdmUsers()` | Paginated user list with filters |
| `delUser(id)` / `restoreUser(id)` | Soft-delete / restore user |
| `loadAdmBks()` | All bookings for admin |
| `admChkIn(id)` / `admChkOut(id)` | Admin-triggered check-in/out |
| `loadAnalytics()` | Revenue analytics with filters |
| `loadReport()` | Financial report with pagination |
| `loadParkAreasAdm()` | Admin parking areas list |
| `addArea()` / `delArea(id)` / `restoreArea(id)` | Area CRUD |
| `genSlots()` | Generate slots for an area |
| `toggleSlotMaintenance(id)` | Toggle slot maintenance mode |
| `loadRates()` / `addRate()` / `delRate(id)` | Rate management |
| `loadOwnerDash()` | Owner dashboard |
| `verifyQR(bookingNumber)` | Security: verify QR code |
| `checkIn(id)` / `checkOut(id)` | Security: check-in / check-out |
| `startSecurityPayPoll(bookingId)` | Poll for extra payment confirmation |
| `stopSecurityPayPoll()` | Clear security payment polling |
| `effectiveStatus(booking)` | Compute display status from booking data |
| `buildTicketStatusRow(booking)` | Generate ticket status badge HTML |
| `isWeekend()` | Returns true if today is Sat/Sun |
| `badge(s, fc)` | Return HTML badge string |
| `stColor(s)` | Return CSS color for a slot status |
| `fdt(d)` | Format date to readable Indian locale string |
| `esc(s)` | HTML-escape a string |
| `gv(id)` | Get trimmed value of an input by ID |
| `chkStr(pwd)` | Live password strength checker |
| `togPw(id, btn)` | Toggle password input visibility |
| `applyTheme()` / `toggleTheme()` | Apply and toggle dark/light mode |

---

## ⚠️ Known Limitations & Future Improvements

### Current Limitations

| Area | Limitation |
|------|-----------|
| Storage | JWT stored in `sessionStorage` — clears on tab close (no "remember me") |
| Real-time | No WebSocket — data is only fresh on view load or manual action |
| Pagination | Admin user/booking lists use "Load More" — no full pagination controls |
| Activity Log | No in-app activity feed for users (admin-side only) |
| Social Login | No OAuth / Google login support |
| Offline | No service worker or offline support |
| Profile | Limited profile edit fields (name, phone) — no avatar upload |
| Notifications | No push notifications — only in-app pull-based |

### Suggested Improvements

- Add WebSocket support for real-time booking/payment status updates
- Implement full pagination with page numbers for admin tables
- Add "Remember me" with `localStorage` token persistence
- Add avatar/photo upload on the profile page
- Implement in-app activity log fed from a backend `/api/activity` endpoint
- Add Google OAuth login option
- Extract CSS into a separate `styles.css` as the project grows
- Add hash-based or History API routing for bookmarkable URLs
- Add Progressive Web App (PWA) support for mobile installation

---

## 🌐 Browser Compatibility

| Browser | Support |
|---------|---------|
| Chrome 90+ | ✅ Full support |
| Firefox 88+ | ✅ Full support |
| Edge 90+ | ✅ Full support |
| Safari 14+ | ✅ Full support |
| IE 11 | ❌ Not supported (no ES6+, no `fetch`) |

---

## 🔗 Backend Repository

The Spring Boot backend for this project is available here:

👉 **[SmartParking Backend — GitHub](https://github.com/MadhanBandi25/Smart-Parking-Management-System-Fronted)**

The backend handles:
- JWT authentication & role-based authorization (Spring Security)
- Booking and slot management
- Payment processing (standard + extra/overtime)
- QR code generation and verification
- Parking area and rate management
- Notification delivery

---
