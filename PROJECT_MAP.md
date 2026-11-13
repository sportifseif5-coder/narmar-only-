# Narmar Platform — Project Map

```
E:\narmar one\
├── .env                         # Environment variables
├── .gitignore                   # Git ignore rules
├── .vscode/settings.json        # VS Code settings
├── Caddyfile                    # Reverse proxy config (Caddy)
├── components.json              # shadcn/ui configuration
├── env_backup.txt               # Backup of env vars
├── eslint.config.mjs            # ESLint flat config
├── next.config.ts               # Next.js configuration
├── package.json                 # Dependencies and scripts
├── postcss.config.mjs           # PostCSS config (Tailwind)
├── tailwind.config.ts           # Tailwind CSS configuration
├── tsconfig.json                # TypeScript configuration
├── bun.lock / package-lock.json # Lock files
│
├── prisma/
│   ├── schema.prisma            # Database schema (22 models, 16 enums)
│   ├── seed.ts                  # Seed 10 sample ads
│   ├── seed-ad-networks.ts     # Seed ad network configs
│   └── migrations/              # Prisma migration history
│
├── generated/prisma/            # Prisma client (auto-generated)
│
├── db/
│   └── custom.db                # SQLite database file
│
├── public/                      # Static assets (images, fonts)
│
├── examples/websocket/
│   ├── server.ts                # Example WebSocket server
│   └── frontend.tsx             # Example WebSocket client
│
├── src/
│   ├── app/
│   │   ├── globals.css          # Global styles / Tailwind
│   │   ├── layout.tsx           # Root layout (Geist font, Toaster)
│   │   ├── page.tsx             # SPA main page (routes by currentView)
│   │   └── api/
│   │       ├── route.ts         # API root
│   │       ├── auth/
│   │       │   ├── register/route.ts              # POST — user registration
│   │       │   ├── register-advertiser/route.ts   # POST — advertiser registration
│   │       │   ├── login/route.ts                 # POST — JWT login
│   │       │   ├── logout/route.ts                # POST — clear session
│   │       │   ├── me/route.ts                    # GET — current user
│   │       │   ├── forgot-password/route.ts       # POST — reset token
│   │       │   └── reset-password/route.ts        # POST — new password
│   │       ├── earn/
│   │       │   ├── ads/route.ts                   # GET — ad rotation
│   │       │   ├── ads/[id]/start/route.ts        # POST — start ad
│   │       │   ├── ads/[id]/complete/route.ts     # POST — complete ad
│   │       │   ├── ads/impression/route.ts        # POST — track impression
│   │       │   ├── ads/click/route.ts             # POST — track click
│   │       │   ├── surveys/route.ts               # GET — available surveys
│   │       │   ├── surveys/[id]/submit/route.ts   # POST — submit survey
│   │       │   ├── tasks/route.ts                 # GET — available tasks
│   │       │   ├── tasks/[id]/start/route.ts      # POST — start task
│   │       │   └── tasks/[id]/submit/route.ts     # POST — submit task
│   │       ├── earnings/
│   │       │   ├── ads/route.ts                   # GET — legacy ad listing
│   │       │   ├── ads/[id]/start/route.ts        # POST — legacy ad start
│   │       │   ├── ads/[id]/complete/route.ts     # POST — legacy ad complete
│   │       │   ├── surveys/route.ts               # GET — legacy survey listing
│   │       │   ├── surveys/[id]/submit/route.ts   # POST — legacy survey submit
│   │       │   ├── tasks/route.ts                 # GET — legacy task listing
│   │       │   ├── tasks/[id]/submit/route.ts     # POST — legacy task submit
│   │       │   └── activities/route.ts            # GET — earning activities
│   │       ├── installments/
│   │       │   ├── route.ts                       # GET/POST — list/create
│   │       │   ├── [id]/route.ts                  # GET/PUT/DELETE — by ID
│   │       │   └── [id]/payments/route.ts         # GET — payment history
│   │       ├── wallet/
│   │       │   ├── route.ts                       # GET — overview
│   │       │   ├── transactions/route.ts          # GET — transaction history
│   │       │   └── withdrawals/route.ts           # GET/POST — withdrawal requests
│   │       ├── referrals/
│   │       │   └── route.ts                       # GET — referral stats
│   │       ├── notifications/
│   │       │   ├── route.ts                       # GET — list
│   │       │   └── update/route.ts                # PUT/DELETE — mark read / delete
│   │       ├── admin/
│   │       │   ├── users/route.ts                 # GET — user list with search
│   │       │   ├── users/[id]/route.ts            # GET/PUT — user details
│   │       │   └── analytics/route.ts             # GET — platform analytics
│   │       ├── advertiser/
│   │       │   ├── videos/route.ts                # GET — list videos
│   │       │   ├── videos/[id]/route.ts           # PUT/DELETE — video
│   │       │   ├── videos/upload/route.ts         # POST — upload video
│   │       │   ├── surveys/route.ts               # GET/POST — surveys
│   │       │   └── earnings/route.ts              # GET — advertiser earnings
│   │       ├── achievements/
│   │       │   └── route.ts                       # GET — achievements + progress
│   │       ├── challenges/
│   │       │   ├── route.ts                       # GET — active challenges
│   │       │   └── [id]/join/route.ts             # POST — join challenge
│   │       └── gamification/
│   │           ├── achievements/route.ts          # GET — alias
│   │           └── challenges/route.ts            # GET/POST — alias
│   │
│   ├── components/
│   │   ├── landing-page.tsx                       # Marketing landing page
│   │   ├── auth/
│   │   │   ├── login-form.tsx                     # Email/password login
│   │   │   ├── register-form.tsx                  # Registration with referral
│   │   │   └── register-advertiser-form.tsx       # Advertiser registration
│   │   ├── dashboard/
│   │   │   ├── dashboard.tsx                      # Main user dashboard (tabs)
│   │   │   ├── sidebar.tsx                        # Collapsible sidebar nav
│   │   │   ├── dashboard-overview.tsx             # Earnings stats + activity
│   │   │   ├── earn-ads.tsx                       # Watch ads tab
│   │   │   ├── earn-surveys.tsx                   # Surveys tab
│   │   │   ├── earn-tasks.tsx                     # Tasks tab
│   │   │   ├── installments.tsx                   # Installment management tab
│   │   │   ├── wallet-page.tsx                    # Wallet + withdrawal tab
│   │   │   └── referrals.tsx                      # Referral program tab
│   │   ├── admin/
│   │   │   └── admin-dashboard.tsx                # Admin panel UI
│   │   └── advertiser/
│   │       └── dashboard.tsx                      # Advertiser portal UI
│   │
│   ├── hooks/
│   │   ├── use-mobile.ts                          # Mobile detection hook
│   │   └── use-toast.ts                           # Toast notification hook
│   │
│   ├── lib/
│   │   ├── db.ts                                  # Prisma client singleton
│   │   ├── utils.ts                               # cn() utility (clsx + twMerge)
│   │   ├── security.ts                            # Unified auth + rate + fraud middleware
│   │   ├── rate-limiter.ts                        # In-memory rate limiter class
│   │   ├── fraud-detector.ts                      # Comprehensive fraud detection (legacy)
│   │   ├── notifications.ts                       # 9 notification creator helpers
│   │   ├── achievements.ts                        # AchievementManager (15 achievements)
│   │   ├── utils/
│   │   │   ├── auth.ts                            # JWT sign/verify, referral code gen
│   │   │   └── video-earnings.ts                  # Tiered video earnings calc
│   │   ├── middleware/
│   │   │   ├── rateLimiter.ts                     # Rate limiter instances
│   │   │   ├── fraudDetection.ts                  # In-memory fraud detection middleware
│   │   │   └── advertiser-auth.ts                 # Advertiser role middleware
│   │   ├── ad-networks/
│   │   │   ├── types.ts                           # Ad network interfaces
│   │   │   ├── base-adapter.ts                    # Abstract base adapter
│   │   │   ├── ad-network-registry.ts             # Singleton registry (7 adapters)
│   │   │   ├── ad-rotation.service.ts             # Weighted random rotation algorithm
│   │   │   ├── earnings-calculator.service.ts     # Tier multipliers + auto-installment
│   │   │   └── adapters/
│   │   │       ├── google-adsense.adapter.ts      # Google AdSense adapter
│   │   │       ├── propellerads.adapter.ts        # PropellerAds adapter
│   │   │       ├── adsterra.adapter.ts            # Adsterra adapter
│   │   │       ├── medianet.adapter.ts            # Media.net adapter
│   │   │       ├── revcontent.adapter.ts          # RevContent adapter
│   │   │       ├── adgem.adapter.ts               # AdGem adapter
│   │   │       └── offertoro.adapter.ts           # OfferToro adapter
│   │   └── fraud-prevention/
│   │       ├── fraud-detector.service.ts          # Production fraud detection (DB-backed)
│   │       └── device-fingerprint.service.ts      # SHA-256 fingerprinting
│   │
│   └── store/
│       └── auth.ts                                # Zustand auth store (persisted)
│
├── mini-services/                   # Placeholder directory
│   └── .gitkeep
│
├── skills/                          # opencode skills
│
├── check_ad_content_fields.ts       # Check ad field compatibility
├── check_ad_networks.ts             # Check ad network DB records
├── check_db_ads.ts                  # Check ad content DB records
├── test-db.ts                       # Database connection test
├── test_ad_fetch.ts                 # Prisma ad query test
├── test_ads_api.js                  # API endpoint smoke test
├── test_api_fix.ts                  # API fix verification test
├── update-jwt-secret.js             # Utility to rotate JWT secret
│
├── BUG_FIXES_APPLIED.md             # 8 bugs found, 6 fixed
├── BUG_REPORT.md                    # Bug report template
├── IMPLEMENTATION_SUMMARY.md        # Full feature summary (626 lines)
├── PROJECT_DOCUMENTATION.md         # Code documentation
├── PROJECT_MAP.md                   # This file — project structure map
├── README.md                        # Original scaffold README
├── worklog.md                       # Task-by-task development log
│
├── prisma_error.log                 # Prisma error logs (debug)
├── prisma_error_2.log
├── prisma_error_final.log
├── prisma_error_generate.log
├── prisma_error_generate_retry.log
├── prisma_error_generate_utf8.log
├── prisma_out.log
└── prisma_v.log
```
