# Narmar Platform — Project Documentation

> **Narmar** is a full-stack web platform that lets users earn money by watching ads, completing surveys, and doing micro-tasks, with earnings automatically applied to installment payments (credit cards, loans, etc.).

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| **Framework** | Next.js 15.3.5 (App Router) |
| **Language** | TypeScript 5 |
| **Styling** | Tailwind CSS 4, tailwindcss-animate |
| **UI** | shadcn/ui (New York style), 50+ components, Lucide icons |
| **Database** | PostgreSQL via Prisma ORM 6 |
| **Auth** | JWT (HS256, jose library) in httpOnly cookies, bcryptjs |
| **State** | Zustand 5 (client), TanStack Query 5 (server) |
| **Validation** | Zod 4 |
| **Charts** | Recharts, TanStack Table |
| **Animations** | Framer Motion 12 |
| **Forms** | React Hook Form + Zod resolvers |
| **I18n** | next-intl |
| **Image** | Sharp |

---

## Database Schema (Prisma — 22 Models, 16 Enums)

### Core Models

| Model | Fields / Purpose |
|-------|------------------|
| **User** | id, email, name, password, role, subscriptionTier, balance, totalEarnings, todaysEarnings, totalEarned, referralCode, referredBy, kycStatus, lastEarningsReset, createdAt |
| **EarningActivity** | Tracks every earning event — type (ad/survey/task/referral), status, amount, metadata |
| **Transaction** | Wallet ledger — type (earning/withdrawal/installment), amount, status, balance snapshot |
| **WithdrawalRequest** | amount, paymentMethod, status (pending/approved/processing/completed/rejected), metadata |
| **Notification** | type (9 types), title, message, read status, metadata |

### Earning Models

| Model | Purpose |
|-------|---------|
| **AdContent** | Video URL, thumbnail, budget/spent, earningsPerView, category, targeting |
| **AdView** | userId + adContentId (unique), ip, completedAt |
| **Survey** | Questions (JSON), demographics, earnings, responseLimit, expiresAt |
| **SurveyResponse** | userId + surveyId (unique), answers (JSON), disqualified |
| **Task** | Instructions, category, difficulty, earnings, expiresAt |
| **TaskAssignment** | userId + taskId, submission (JSON), reviewStatus |

### Installment Models

| Model | Purpose |
|-------|---------|
| **Installment** | title, totalAmount, paidAmount, monthlyAmount, dueDate, autoPay, status (active/paidOff/overdue) |
| **InstallmentPayment** | amount, installmentId, transactionId |

### Ad Network Models

| Model | Purpose |
|-------|---------|
| **AdNetwork** | name, type, eCPM, fillRate, reliabilityScore, priority, dailyBudget |
| **AdNetworkConfig** | encrypted API keys, zone IDs, endpoints per network |
| **AdCampaign** | budget, targeting (country, category), date range |
| **AdImpression** | fingerprint, ip, userAgent, completion rate |
| **AdClick** | impressionId, ip, userAgent, converted |
| **AdConversion** | clickId, revenue, payout |

### Fraud & Security Models

| Model | Purpose |
|-------|---------|
| **FraudDetection** | userId, type, severity (low/medium/high/critical), evidence (JSON), resolved |
| **DeviceFingerprint** | SHA-256 hash, userId, userAgent, screen, timezone, platform |

### Gamification Models

| Model | Purpose |
|-------|---------|
| **Achievement** | key, name, description, rewardAmount, icon, criteria (JSON) |
| **UserAchievement** | userId + achievementId, progress (JSON), earned |
| **Challenge** | title, description, reward, startDate, endDate, requirements (JSON) |
| **UserChallenge** | userId + challengeId, progress (JSON), completed |

### Advertiser Models

| Model | Purpose |
|-------|---------|
| **AdvertiserProfile** | companyName, website, businessType, taxId, verified |
| **AdvertiserVideo** | title, fileUrl, thumbnailUrl, status, targetCategory, budget |
| **AdvertiserVideoView** | view tracking with duration watched |

### Subscription

| Model | Purpose |
|-------|---------|
| **PremiumPlan** | name, price, tier, features (JSON), durationDays |

---

## Authentication System

- **JWT tokens** (HS256, 7-day expiry) stored in httpOnly cookie (`auth-token`)
- **Endpoints**: register, login, logout, me, forgot-password, reset-password
- **Password**: bcryptjs (12 rounds)
- **Roles**: USER, ADMIN, SUPER_ADMIN, ADVERTISER
- **Referral**: auto-generated referral code, optional referralCode on registration
- **Client**: Zustand store (`useAuthStore`) with localStorage persistence

### API Routes

| Method | Route | Description |
|--------|-------|-------------|
| POST | `/api/auth/register` | Register with email, password, name, optional referralCode |
| POST | `/api/auth/register-advertiser` | Register as advertiser with company details |
| POST | `/api/auth/login` | Login, returns JWT in cookie + user data |
| GET | `/api/auth/me` | Current user info + advertiserProfile if applicable |
| POST | `/api/auth/logout` | Clear auth cookie |
| POST | `/api/auth/forgot-password` | Generate reset token |
| POST | `/api/auth/reset-password` | Reset password with token |

---

## Earning System

### Ads (with Multi-Network Rotation)

**Architecture**: Plugin-based with 7 adapter implementations behind a common `IAdNetworkAdapter` interface:

| Adapter | Format | Notes |
|---------|--------|-------|
| Google AdSense | Display / Native | publisherId + apiKey |
| PropellerAds | Push / Interstitial / Native | apiKey + zoneId |
| Adsterra | Pop-under / Banner | publisherId + apiSecret |
| Media.net | Contextual Native | customerId + siteId |
| RevContent | Content Discovery | apiKey + publisherId |
| AdGem | Offerwall | apiKey + appId |
| OfferToro | Offerwall | apiKey + appId |

**Smart Rotation** (`ad-rotation.service.ts`):
- Weighted random selection based on eCPM, fill rate, reliability, priority, daily budget
- Premium users get 1.15x-1.3x weight boost
- Falls back to `AdContent` + `AdvertiserVideo` if network ads unavailable
- Currently all adapters return mock ads (real API integration requires keys)

**API Routes**:

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/earn/ads` | Available ads via rotation algorithm |
| POST | `/api/earn/ads/[id]/start` | Start watching ad |
| POST | `/api/earn/ads/[id]/complete` | Complete ad + earnings credit |
| POST | `/api/earn/ads/impression` | Track impression with device fingerprint |
| POST | `/api/earn/ads/click` | Track click |

### Surveys

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/earnings/surveys` | Available surveys (filtered by demographics) |
| POST | `/api/earnings/surveys/[id]/submit` | Submit answers, earn reward |

### Micro-Tasks

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/earnings/tasks` | Available tasks |
| POST | `/api/earnings/tasks/[id]/submit` | Submit task work (status: under-review) |

### Earnings Calculator (`earnings-calculator.service.ts`)

- **Tier multipliers**: FREE=1.0x, PREMIUM=1.25x, PREMIUM_PLUS=1.5x
- **Auto-installment**: 50% of earnings can auto-apply to active installment with auto-pay
- Creates `Transaction` + optional `InstallmentPayment` records atomically via Prisma transaction

---

## Installment Management

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/installments` | List user installments |
| POST | `/api/installments` | Create new installment |
| GET | `/api/installments/[id]` | Installment details |
| PUT | `/api/installments/[id]` | Update (title, amount, autoPay) |
| DELETE | `/api/installments/[id]` | Remove installment |
| GET | `/api/installments/[id]/payments` | Payment history |

Features: auto-pay toggle, progress tracking, status management (active/paidOff/overdue), automatic due date calculation.

---

## Wallet System

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/wallet` | Balance, pending withdrawals, available balance |
| GET | `/api/wallet/transactions` | Paginated transaction history with type filtering |
| GET | `/api/wallet/withdrawals` | Withdrawal history |
| POST | `/api/wallet/withdrawals` | Request withdrawal (min $10) |

Payment methods: PayPal, Bank Transfer, Crypto, Check.

---

## Referral Program

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/referrals` | Referral stats (code, link, count, earnings, list) |

- 10% commission on referred user earnings
- Referral code generated at registration
- Referral link auto-generated from APP_URL

---

## Admin Dashboard

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/admin/users` | Paginated user list with search |
| GET | `/api/admin/users/[id]` | User details |
| PUT | `/api/admin/users/[id]` | Update user (role, KYC status) |
| GET | `/api/admin/analytics` | Platform metrics |

Analytics: total/active users, total earnings/withdrawals, activity breakdown, time-based (today/week), platform revenue.

---

## Notification System

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/notifications` | Paginated list with unread filter |
| PUT | `/api/notifications/update` | Mark as read (single or all) |
| DELETE | `/api/notifications/update` | Delete (single, all read, or all) |

**9 Notification Types**: EARNING, PAYMENT, WITHDRAWAL, TASK, SURVEY, REFERRAL, SYSTEM, INSTALLMENT, ACHIEVEMENT

Helper functions: `createEarningNotification`, `createWithdrawalNotification`, `createInstallmentPaymentNotification`, `createReferralNotification`, `createAchievementNotification`, `createTaskCompletedNotification`, `createSurveyCompletedNotification`, `createSystemNotification`.

---

## Gamification

### 15 Achievements

| Achievement | Requirement | Reward |
|-------------|-------------|--------|
| First Ad | Complete 1 ad | $0.50 |
| Ad Enthusiast | Complete 10 ads | $2.00 |
| Ad Master | Complete 50 ads | $10.00 |
| First Survey | Complete 1 survey | $0.50 |
| Survey Expert | Complete 10 surveys | $3.00 |
| First Task | Complete 1 task | $0.50 |
| Task Champion | Complete 10 tasks | $5.00 |
| Getting Started | Earn $10 total | $1.00 |
| Century Club | Earn $100 total | $5.00 |
| Half Grand | Earn $500 total | $10.00 |
| Thousand Dollar Club | Earn $1,000 total | $25.00 |
| First Referral | Get 1 referral | $1.00 |
| Network Builder | Get 10 referrals | $10.00 |
| Week Warrior | 7 consecutive days active | $7.00 |
| Installment Warrior | Pay off 3 installments | $5.00 |

### Challenges

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/challenges` | Active challenges (date-filtered) |
| POST | `/api/challenges/[id]/join` | Join challenge with validation |

Achievements auto-checked after any earning activity via `AchievementManager.checkAndAwardAchievements()`.

---

## Security

### Three-Layer Fraud Detection

**Layer 1 — In-Memory** (`middleware/fraudDetection.ts`):
- Rapid activity (>10 actions/minute)
- IP multi-account (>3 users from same IP)
- High daily earnings (>$100)

**Layer 2 — Database-Backed** (`lib/fraud-detector.ts`):
- Ad views: 30/hour max, same IP >5 users/hour, fast completions
- Surveys: 10/hour max, <10s between submissions
- Tasks: 15/hour max, >50% rejection rate
- Earnings: $100/day cap, $500/week cap, >300% spike
- Withdrawals: near-full balance, >3 in 7 days, >80% of total earnings, new account <7 days

**Layer 3 — Production Service** (`fraud-prevention/fraud-detector.service.ts`):
- Rapid completion <5 seconds
- SHA-256 device fingerprint collision detection
- IP pattern: 3+ users from same IP in 24h
- Hourly rate: 20 ads/hour
- Earnings spike: 5x daily average
- Logs to `FraudDetection` table with severity
- Auto-blocks user after 3+ unresolved CRITICAL flags

### Device Fingerprinting (`device-fingerprint.service.ts`)
- SHA-256 hash from userAgent + screen resolution + timezone + language + platform
- Collision detection against existing fingerprints

### Rate Limiting (6 Tiers)

| Tier | Limit | Window |
|------|-------|--------|
| Auth | 5 req | 1 minute |
| General API | 100 req | 1 minute |
| Ad Watch | 30 | 1 day |
| Surveys | 10 | 1 day |
| Tasks | 15 | 1 day |
| Withdrawals | 3 | 1 week |

In-memory sliding window counter with automatic cleanup.

### Middleware (`security.ts`)

Unified `enforceSecurity(req, options)` function that combines auth + rate limit + fraud check:
- Options: `requireAuth`, `requireAdmin`, `rateLimit` (tier name), `fraudCheck` (type)
- Returns structured: `{ success, userId }` or `{ error, statusCode }`

Supporting middleware: `authenticate()` (JWT), `enforceRateLimit()`, `checkFraud()`, `verifyAdvertiser()`.

---

## Frontend Architecture

### SPA Routing

`src/app/page.tsx` renders views based on `useAuthStore.currentView`:

| View | Component | Description |
|------|-----------|-------------|
| `landing` | LandingPage | Marketing page (hero, features, how-it-works, testimonials, footer) |
| `login` | LoginForm | Email/password sign-in |
| `register` | RegisterForm | Registration with referral code |
| `register-advertiser` | RegisterAdvertiserForm | Advertiser registration |
| `dashboard` | Dashboard | Tabbed user dashboard |
| `advertiser-dashboard` | AdvertiserDashboard | Advertiser portal |
| `admin-dashboard` | AdminDashboard | Platform management |

### User Dashboard Tabs

1. **Overview** — Earnings stats, activity feed, installment progress
2. **Watch Ads** — Available ad grid with earnings display
3. **Surveys** — Available surveys with filtering
4. **Tasks** — Micro-tasks with difficulty levels
5. **Installments** — List with progress tracking
6. **Wallet** — Balance, transactions, withdrawal form
7. **Referrals** — Referral link, stats, earnings

### UI Components

- 50+ shadcn/ui components in `src/components/ui/`
- Auth forms: `login-form.tsx`, `register-form.tsx`, `register-advertiser-form.tsx`
- Dashboard: `dashboard.tsx`, `sidebar.tsx`, `dashboard-overview.tsx`, `earn-ads.tsx`, `earn-surveys.tsx`, `earn-tasks.tsx`, `installments.tsx`, `wallet-page.tsx`, `referrals.tsx`
- Admin: `admin-dashboard.tsx`
- Advertiser: `dashboard.tsx`

---

## Bug Fixes Applied

| Bug | Severity | Status | File |
|-----|----------|--------|------|
| Unused import in Dashboard | Low | Fixed | `dashboard.tsx` |
| Null reference in Dashboard | High | Fixed | `dashboard.tsx` |
| Wrong Prisma model name `withdrawalRequest` | Critical | Fixed | `fraud-detector.ts` |
| Import path typo (`ate-limiter`, `fraud-detector`) | Critical | Fixed | `security.ts` |
| Unused eslint-disable directive | Low | Fixed | `use-toast.ts` |
| Daily earnings not resetting | Medium | Open | ad complete route |
| Missing subscription tier multiplier | Medium | Open | ad complete route |

---

## Scripts & Utilities

| Script | Purpose |
|--------|---------|
| `prisma/seed.ts` | Seeds 10 sample ads |
| `prisma/seed-ad-networks.ts` | Seeds ad network configurations |
| `test_ads_api.js` | Tests `/api/earn/ads` endpoint |
| `test_ad_fetch.ts` | Tests Prisma ad queries |
| `test_api_fix.ts` | API fix verification |
| `test-db.ts` | Database connection test |
| `check_db_ads.ts` | Inspects ad records in DB |
| `check_ad_networks.ts` | Inspects network records |
| `check_ad_content_fields.ts` | Checks field compatibility |
| `update-jwt-secret.js` | Updates JWT secret in env |

---

## Deployment

### Commands
```bash
bun run dev          # Development (port 3000)
bun run build        # Production build + static export
bun start            # Production server
bun run db:seed      # Seed database
bun run lint         # ESLint
```

### Test Accounts
- **Admin**: admin@narmar.com / admin123
- **User**: user@narmar.com / user123

### Next Steps for Production
1. Real email service (SendGrid/AWS SES)
2. Payment gateway (Stripe/PayPal)
3. Real-time WebSocket notifications
4. Unit/integration/E2E tests
5. CI/CD pipeline
6. Production env config
7. SSL/TLS
8. CDN for video uploads
9. Ad network API keys (currently mock ads)
