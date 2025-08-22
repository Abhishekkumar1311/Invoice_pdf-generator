# Automated Invoice Generator

A production-ready, developer-friendly tool to create, style, export (PDF), and send invoices automatically from your app or via a simple web UI.

---

## ✨ Features

* **Create & manage invoices** (clients, items, taxes, discounts, currency, due dates)
* **PDF export** via headless Chromium (Puppeteer) with clean, printable templates
* **Email delivery** with attachments (Nodemailer or SMTP)
* **Auto-numbering** with configurable prefixes (e.g., `FY25/INV-000123`)
* **Tax rules**: per-line or per-invoice VAT/GST rates
* **Discounts**: fixed or percentage
* **Multi-currency** with thousands/decimal formatting
* **Branding**: company logo, colors, header/footer
* **Web UI** for quick creation + an **HTTP API** for integrations
* **CSV import/export** for clients and items
* **Audit trail** (created/updated/sent timestamps)
* **Pluggable storage** (MongoDB by default; can swap to Postgres/MySQL)

---

## 🧱 Tech Stack

* **Backend**: Node.js, Express
* **Database**: MongoDB (Mongoose)
* **PDF**: Puppeteer (Chromium)
* **Email**: Nodemailer (SMTP/SES/SendGrid)
* **Frontend**: React + Vite (or use the bundled minimal HTML UI)
* **Auth** (optional): JWT (access/refresh)
* **Container**: Docker (multi-stage build)
* **Testing**: Jest + Supertest

> Don’t use this stack? Swap components easily—this README notes what to change.

---

## 🗂️ Repository Structure

```
.
├─ api/
│  ├─ src/
│  │  ├─ index.ts            # Express bootstrap
│  │  ├─ routes/
│  │  ├─ models/
│  │  ├─ services/
│  │  ├─ templates/          # Handlebars/HTML invoice templates
│  │  └─ utils/
│  ├─ tests/
│  └─ package.json
├─ web/
│  ├─ src/
│  └─ package.json
├─ docker/
│  ├─ api.Dockerfile
│  └─ web.Dockerfile
├─ .env.example
├─ docker-compose.yml
└─ README.md
```

---

## 🚀 Quick Start

### 1) Prerequisites

* Node.js 18+
* pnpm or npm
* MongoDB 6+ (local or Atlas)
* (Optional) Docker

### 2) Clone & install

```bash
git clone <your-repo-url> invoice-generator
cd invoice-generator
pnpm install  # or npm install
```

### 3) Configure environment

Copy `.env.example` to `.env` and fill values.

```env
# Server
PORT=4000
NODE_ENV=development
CORS_ORIGIN=http://localhost:5173

# Database
MONGODB_URI=mongodb://localhost:27017/invoices

# Auth (optional)
JWT_SECRET=change-this
JWT_EXPIRES_IN=15m
REFRESH_EXPIRES_IN=7d

# Email (use any SMTP)
MAIL_FROM="Acme Billing <no-reply@acme.com>"
SMTP_HOST=smtp.yourhost.com
SMTP_PORT=587
SMTP_USER=your_user
SMTP_PASS=your_pass

# Company branding (used in templates)
COMPANY_NAME=Acme Pvt Ltd
COMPANY_LOGO_URL=https://example.com/logo.png
COMPANY_ADDRESS="2nd Floor, MG Road, Bengaluru, KA 560001"
COMPANY_PHONE=+91-9876543210
CURRENCY=INR

# Invoice numbering
INV_PREFIX=FY25/INV-
INV_PAD=6
```

### 4) Run API + Web

```bash
# API
pnpm --filter api dev

# Web
pnpm --filter web dev
```

Open:

* API: `http://localhost:4000/health`
* Web: `http://localhost:5173`

---

## 🧩 API Overview

### Auth (optional)

```
POST /auth/login
# body: { email, password }
```

### Clients

```
GET    /clients
POST   /clients            # { name, email, address, gstin? }
GET    /clients/:id
PATCH  /clients/:id
DELETE /clients/:id
```

### Invoices

```
GET    /invoices?status=draft|sent|paid
POST   /invoices           # see payload below
GET    /invoices/:id
PATCH  /invoices/:id
DELETE /invoices/:id

# Actions
POST /invoices/:id/issue   # lock number + mark as "sent"
POST /invoices/:id/email   # send to client with PDF attached
GET  /invoices/:id/pdf     # download PDF
```

#### Sample invoice payload

```json
{
  "clientId": "66bcf3...",
  "issueDate": "2025-08-01",
  "dueDate": "2025-08-15",
  "currency": "INR",
  "notes": "Thanks for your business!",
  "items": [
    { "description": "Consulting", "qty": 10, "unitPrice": 2500, "taxRate": 18 },
    { "description": "License", "qty": 1, "unitPrice": 12000, "taxRate": 18, "discountPct": 10 }
  ],
  "shipping": 0,
  "discountPct": 0,
  "roundOff": true,
  "customFields": { "poNumber": "PO-2025-0042" }
}
```

> **Totals** are computed server-side to prevent tampering.

---

## 🧾 PDF Templates

* Templates live under `api/src/templates/*`.
* Default engine: Handlebars. You can switch to EJS/Nunjucks.
* Add custom CSS for branding. Supports page header/footer, QR for UPI, and page numbers.

### Add a new template

1. Create `api/src/templates/minimal.hbs` and `minimal.css`.
2. Register it in `templateRegistry.ts`.
3. Pass `?template=minimal` when requesting `/invoices/:id/pdf`.

---

## 🔢 Numbering Rules

* Next number = `INV_PREFIX + pad(sequence, INV_PAD)`
* Sequence increases **only** when invoice is issued (`/issue`).
* To reset annually, run the maintenance script or set `INV_PREFIX` per FY.

---

## 🇮🇳 GST/VAT Notes (India-ready)

* Per-line `taxRate` supported (e.g., 5, 12, 18).
* CGST/SGST/IGST split supported via client state vs company state.
* HSN/SAC codes can be added in line items.

> Configure company state in `.env` and client state in client record.

---

## ✉️ Email Sending

* Uses Nodemailer with your SMTP creds.
* Default subject: `Invoice {{number}} from {{company}}`.
* Retries with exponential backoff; failures logged.

---

## 🔐 Security

* Input validation with Zod/Joi.
* Rate limiting on auth + email routes.
* Helmet + CORS configured.
* Server-side total calculation; signed download URLs (optional).

---

## 🧪 Testing

```bash
pnpm --filter api test      # unit + API tests
```

---

## 🐳 Docker

```bash
# Build
docker compose build

# Run (Mongo + API + Web)
docker compose up
```

`docker-compose.yml` spins up:

* `mongodb` with a named volume
* `api` (Node 18-slim with Puppeteer deps)
* `web` (Vite dev server or Nginx for prod)

---

## 🛠️ Common Tasks

### Seed demo data

```bash
pnpm --filter api seed
```

### Regenerate PDF

```bash
curl -L http://localhost:4000/invoices/<id>/pdf -o invoice.pdf
```

### Send invoice via email

```bash
curl -X POST http://localhost:4000/invoices/<id>/email \
  -H 'Content-Type: application/json' \
  -d '{"to":"client@example.com"}'
```

---

## 🧰 Switching Components

* **DB**: Replace Mongoose models with Prisma + Postgres; update config + queries.
* **PDF**: Swap Puppeteer with `pdfmake` or `wkhtmltopdf`.
* **Templates**: Switch Handlebars → EJS/Nunjucks with adapter in `services/pdf`.

---

## 🐞 Troubleshooting

* **Chromium not launching**: Ensure `apt` dependencies installed (`libnss3`, `libatk1.0-0`, etc.). In Dockerfile, use the provided base.
* **Fonts missing**: Install Devanagari fonts for INR locales (e.g., `fonts-noto`).
* **Gmail SMTP blocks**: Use an app password or a provider like SendGrid.
* **Numbers skipped**: You issued and deleted—sequence is monotonic by design.

---

## 🗺️ Roadmap

* Recurring invoices & subscriptions
* Partial payments & reminders
* Webhooks (issued/paid)
* Multi-tenant (orgs & roles)
* QR/UPI dynamic payment links

---

## 🤝 Contributing

1. Fork & create a feature branch
2. Add tests for your change
3. Run lints + tests
4. Open a PR with context/screenshots

---

## 📄 License

MIT (see `LICENSE`)

---

## 🙋 FAQ

**Q: Can I use it without Mongo?**
Yes—swap in Prisma + Postgres; replace the repository layer.

**Q: Does it support Indian GST invoices?**
Yes—CGST/SGST/IGST split, HSN codes, place of supply.

**Q: Can I host it on free tiers?**
Yes—Render/Fly for API, Atlas for Mongo, and Netlify/Vercel for web.

**Q: How do I customize numbering?**
Change `INV_PREFIX`/`INV_PAD` and the `nextSequence()` implementation.
