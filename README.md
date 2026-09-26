FinWare is a full-stack analytics platform built for a Data Warehousing and Mining (DWM) lab practical. It turns a Star / Snowflake / Galaxy schema design into a real, queryable application instead of just a diagram: a Node.js/Express/SQLite backend serves aggregated warehouse queries to a 16-page dashboard covering transactions, CA (Chartered Accountant) advisory sessions, and cross-process analysis between the two.

Table of contents
Overview
Features
Tech stack
The data model
Project structure
Getting started
Demo login
API reference
Known limitations
Extending this project
Academic context
Overview

The warehouse models five conformed dimensions — User, Bank, Category, Date, CA Advisor — and two fact tables, Fact_Transactions and Fact_CA_Sessions, which share the User and Date dimensions. That shared pair is what makes the Galaxy (fact constellation) schema work: it lets a single query compare a user's spending against their CA session activity for the same date.

The app demonstrates all three schema shapes side by side:

Schema	Where it shows up
Star	Dashboard — Fact_Transactions joined straight to each dimension
Snowflake	User/Bank Analysis — Dim_User and Dim_Bank normalized into City, Income Bracket, Account Type, Bank Type
Galaxy	Custom Analysis — both fact tables queried through the conformed Dim_User + Dim_Date
Features
🔐 JWT authentication — real login against a hashed password in SQLite, not a hardcoded check
📊 16-page dashboard — Dashboard, Source Tables, Dimension Tables, Fact Tables, Star/Snowflake/Galaxy schema diagrams, Transactions, CA Sessions, User/Bank/Category/Time Analysis, Custom Analysis, Reports, Profile
🔎 Live filtering on transactions and CA sessions (user, bank, category, type, date range)
🧮 Custom Analysis — an ad-hoc cross-fact query builder comparing spend vs. CA fees per user
📤 CSV report exports (Transaction Summary, User Spending, Bank-wise, CA Session, Tax Advisory Insights)
🗄️ Real SQL endpoints demonstrating Star/Snowflake/Galaxy query patterns directly, separate from the UI's main data path
🌗 Light/dark theme toggle
📱 Responsive layout
Tech stack
Layer	Technology
Backend	Node.js, Express
Database	SQLite (better-sqlite3) — zero external setup
Auth	JWT (jsonwebtoken) + bcryptjs password hashing
Frontend	Vanilla HTML/CSS/JS, Chart.js
Fonts	Sora (display), Inter (body) via Google Fonts

No build step, no framework tooling — clone it, npm install, npm start.

The data model
database/
├── schema.sqlite.sql          # DDL the app actually runs (Snowflake + Galaxy combined)
├── seed-data.js                # The 5-user / 5-bank / 5-category / 5-transaction / 5-session sample set
└── dwm-schema-reference.sql    # Star, Snowflake, and Galaxy written out SEPARATELY (Postgres-flavoured),
                                 # for a lab report — not executed by the app itself
Project structure
finware-project/
├── server.js                    Express entry point — serves the API + the frontend
├── package.json
├── .env.example
├── src/
│   ├── db.js                    Opens/creates data/finware.sqlite, seeds it on first run
│   ├── middleware/
│   │   └── auth.js              JWT verification
│   └── routes/
│       ├── auth.routes.js       POST /api/auth/login, GET /api/auth/me
│       ├── warehouse.routes.js  GET /api/warehouse/all, /api/warehouse/source/:table
│       └── analytics.routes.js  Star / Snowflake / Galaxy query endpoints
├── database/
│   ├── schema.sqlite.sql
│   ├── seed-data.js
│   └── dwm-schema-reference.sql
└── public/                       Static frontend
    ├── index.html
    ├── css/styles.css
    └── js/app.js
Getting started
Prerequisites
Node.js 18 or later
No separate database server — SQLite runs as a single file, created automatically
Installation
bash
git clone https://github.com/<your-username>/finware-project.git
cd finware-project
npm install
Run it
bash
npm start

You should see:

Seeded the database with the sample DWM dataset.
FinWare running at http://localhost:4000
Demo login: admin@finware.com / finware2026

Open http://localhost:4000 in your browser.

Use npm run dev instead of npm start if you want the server to auto-restart on file changes.

Resetting the data

Delete data/finware.sqlite (and any -shm/-wal files next to it) and restart the server — it reseeds automatically from database/seed-data.js.

Demo login
	
Email	admin@finware.com
Password	finware2026
API reference

All routes except /api/auth/login and /health require an Authorization: Bearer <token> header.

Method	Endpoint	Description
POST	/api/auth/login	Authenticate, returns a JWT
GET	/api/auth/me	Returns the decoded token payload
GET	/api/warehouse/all	Users, banks, categories, transactions, CA sessions — what the UI hydrates from
GET	/api/warehouse/source/:table	Raw OLTP mirror (transaction-raw, user-master, bank-master)
GET	/api/analytics/star/category-summary	Star: fact joined straight to one dimension
GET	/api/analytics/star/bank-summary	Star schema, by bank
GET	/api/analytics/snowflake/user-profile	Snowflake: walks City/Income/Account Type hops
GET	/api/analytics/snowflake/bank-profile	Snowflake schema, by bank type
GET	/api/analytics/galaxy/cross-process	Galaxy: both fact tables via conformed dimensions
GET	/api/analytics/dashboard-summary	Aggregate KPIs computed in SQL

Example:

bash
TOKEN=$(curl -s -X POST http://localhost:4000/api/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"admin@finware.com","password":"finware2026"}' \
  | python3 -c "import sys,json;print(json.load(sys.stdin)['token'])")

curl http://localhost:4000/api/analytics/galaxy/cross-process \
  -H "Authorization: Bearer $TOKEN"
Known limitations
The sample dataset intentionally mirrors the lab practical's 5 rows per table, so month-over-month/seasonal trend views only cover one week — the Time Analysis page says so directly rather than faking history.
Single seeded admin account; no sign-up flow.
JWT_SECRET defaults to a placeholder — set a real one in .env before deploying this anywhere beyond your own machine.
Extending this project
Swap SQLite for Postgres/MySQL — database/dwm-schema-reference.sql is already written in Postgres-flavoured DDL; point src/db.js at pg or mysql2 instead of better-sqlite3.
More sample data — add rows to database/seed-data.js, delete data/finware.sqlite, restart.
Real password reset — the Login page's "Forgot password?" link is a placeholder today.
Academic context

Built for the Data Warehousing and Mining (DWM) Lab practical: identify OLTP source tables, populate sample data, and design Star, Snowflake, and Galaxy (Fact Constellation) schemas for a banking and fintech application — then demonstrate it with a working, clickable UI rather than only a diagram.
