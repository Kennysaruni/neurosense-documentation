---
sidebar_position: 6
title: Local Development & Setup
---

# Local Development & Environment Setup

Follow this guide to get the NeuroSense Africa application up and running on your local machine for development and testing.

---

## 1. Environment Variables Configuration

Create a `.env` file in the root directory of the `neurosense` project. Copy the template below and update the secret keys:

```bash
# --- DATABASE CONFIGURATION ---
# Replace with your Neon PostgreSQL connection string
DATABASE_URL="postgresql://user:password@host/neondb?sslmode=require"

# --- BETTER-AUTH CONFIGURATION ---
# A random secret string used to hash sessions
BETTER_AUTH_SECRET="neurosense-super-secret-key-123"
# The canonical base URL of your app
BETTER_AUTH_URL="http://localhost:3000"
NEXT_PUBLIC_AUTH_URL="http://localhost:3000"

# --- EMAIL CONFIGURATION (RESEND) ---
# Replace with your active Resend API key
RESEND_API_KEY="re_123456789"
EMAIL_FROM="NeuroSense Africa <noreply@yourdomain.com>"

# --- STRIPE BILLING KEYS ---
# Replace with your Stripe developer publishable and secret keys
STRIPE_PUBLISHABLE_KEY="pk_test_..."
STRIPE_SECRET_KEY="sk_test_..."
# Retrieve this by running: stripe listen --forward-to localhost:3000/api/billing/webhook
STRIPE_WEBHOOK_SECRET="whsec_..."
```

---

## 2. Database Sync & Prisma Client Setup

Prisma is configured to map your database schema dynamically. Run the following commands to generate client libraries and push schemas to Neon PostgreSQL:

```bash
# Install package dependencies
npm install

# Push structural database migrations to Neon PostgreSQL
npx prisma db push

# Regenerate local Prisma Client types
npx prisma generate
```

---

## 3. Running the Local Dev Server

Start the local development server to compile and run the application:

```bash
# Launch Next.js dev server
npm run dev
```
Open [http://localhost:3000](http://localhost:3000) in your browser to view the application.

---

## 4. Seeding Demo Admin Accounts & Testing

To test the School Admin panel and data isolation metrics without manually completing Stripe checkouts, you can use the built-in demo administrator sandbox.

1. **Seeding the Sandbox**: Click the **"Explore Demo Admin Panel"** button on the Login screen (`/login`).
2. **What gets Seeded (`/api/auth/demo-seed`)**:
   - Creates a school record named `"Demo Academy"`.
   - Creates a school administrator account (`admin@neurosense.com`).
   - Seeds teacher users, caregiver users, child profiles, baseline goals, and observation questionnaires.
   - Populates a multi-week telemetry log of completed developmental session history.
3. **Authentication**: Automatically logs you in as the administrator and redirects your session straight into the `/admin` dashboard.
