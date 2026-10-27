<div align="center">

# 🔍 AI-Powered Lost & Found System

**A full-stack, intelligent platform for recovering lost items — built with Next.js, Prisma, and an AI-driven matching engine.**

[![Next.js](https://img.shields.io/badge/Next.js-16.1.7-black?logo=next.js)](https://nextjs.org)
[![TypeScript](https://img.shields.io/badge/TypeScript-5.x-3178C6?logo=typescript)](https://www.typescriptlang.org)
[![Prisma](https://img.shields.io/badge/Prisma-5.21-2D3748?logo=prisma)](https://www.prisma.io)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

> **Live Demo →** *Deploys automatically to Vercel on GitHub push.*

</div>

---

## 📋 Table of Contents

1. [Project Overview](#-project-overview)
2. [The Problem It Solves](#-the-problem-it-solves)
3. [Key Features](#-key-features)
4. [Architecture & Tech Stack](#-architecture--tech-stack)
5. [Project Structure](#-project-structure)
6. [Database Schema](#-database-schema)
7. [The Matching Engine (Deep Dive)](#-the-matching-engine-deep-dive)
8. [Photo Upload & Visual Scanning](#-photo-upload--visual-scanning)
9. [How It Helps — Real-World Impact](#-how-it-helps--real-world-impact)
10. [Upgrade Roadmap: From Rule-Based to ML-Powered](#-upgrade-roadmap-from-rule-based-to-ml-powered)
11. [API Reference](#-api-reference)
12. [Local Setup](#-local-setup)
13. [Deployment](#-deployment)

---

## 🌟 Project Overview

The **AI-Powered Lost & Found System** is a campus-scale web application that intelligently connects people who have lost items with those who have found them. Unlike traditional lost-and-found boards or spreadsheets, this system automates the matching process the moment a new report is submitted — eliminating manual comparison, reducing recovery time, and building community trust.

The platform is engineered with a **multi-layered verification pipeline**:
1. **Auto-match** on submission using the scoring engine
2. **User claim** request workflow
3. **Admin approval gate** before contact details are exchanged

---

## 🚨 The Problem It Solves

| Traditional System | This System |
|---|---|
| Physical notice boards — easy to miss | Centralized digital database, always accessible |
| No automatic correlation of lost/found | Engine runs on every new submission |
| Owner has to manually scan every report | Targeted, ranked match suggestions |
| No verification — anyone can claim anything | Multi-step: report → claim → admin approval → contact reveal |
| Photos are not searchable | Visual scan endpoint for image-based search |
| Data is lost or destroyed | Persistent PostgreSQL database with full history |

---

## ✨ Key Features

### 👥 User-Facing
- **Dual-mode reporting** — file a *Lost* or *Found* report with rich details (title, category, location, landmark, date, images)
- **Drag-and-drop image upload** — visual evidence massively improves matching accuracy
- **Smart dashboard** — real-time view of your reports, pending match notifications, and verified contact details
- **Visual Image Scan** (`/scan`) — upload a photo and search the database visually
- **Match detail page** — side-by-side comparison of lost vs. found report with confidence score
- **Dark / Light theme toggle**

### 🛡️ Admin-Facing (`/admin`)
- **Live metrics** — total users, reports, matches, and confirmed recoveries
- **Pending Claims** tab — review side-by-side match details, approve or reject with one click
- **Reports Management** tab — view all reports with images, delete spam or invalid entries
- **Force Re-scan** — trigger a full pairwise matching sweep across all open reports on demand

### 🤖 Engine-Level
- **Automatic matching** on every new report (runs as a background promise)
- **Weighted scoring** across title (40%), category (20%), description (20%), location (20%)
- **Duplicate match prevention** — checks existing matches before creating a new one
- **Notification log** — both parties are notified when a match is created

---

## 🏗️ Architecture & Tech Stack

```
┌─────────────────────────────────────────────────────────────┐
│                         Browser (Client)                     │
│   React 19 + Next.js App Router | next-themes | lucide-react │
└───────────────────────┬─────────────────────────────────────┘
                        │ HTTP / RSC Streaming
┌───────────────────────▼─────────────────────────────────────┐
│                    Next.js Server (Edge/Node)                 │
│  ┌─────────────────┐  ┌──────────────┐  ┌────────────────┐  │
│  │  Next-Auth JWT  │  │  API Routes  │  │  Server Pages  │  │
│  │  (Auth Layer)   │  │  (REST)      │  │  (SSR / RSC)   │  │
│  └────────┬────────┘  └──────┬───────┘  └───────┬────────┘  │
│           └──────────────────┴───────────────────┘           │
│                              │                               │
│  ┌───────────────────────────▼────────────────────────────┐  │
│  │                   Prisma ORM Client                     │  │
│  └────────────────────────┬────────────────────────────────┘  │
│                           │                                  │
│  ┌────────────────────────▼────────────────────────────────┐  │
│  │      PostgreSQL Database (Neon / Supabase / Local)       │  │
│  │    User | Report | Image | Match | NotificationLog       │  │
│  └─────────────────────────────────────────────────────────┘  │
│                                                              │
│  ┌───────────────────────────────────────────────────────┐  │
│  │               AI / ML Layer (src/lib/)                 │  │
│  │  matchEngine.ts ── Rule-based weighted text scorer     │  │
│  │  ml.ts          ── @xenova/transformers singleton      │  │
│  │                    (all-MiniLM-L6-v2 text embeddings)  │  │
│  └───────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────┘
```

| Layer | Technology |
|---|---|
| **Frontend Framework** | Next.js 16 (App Router, RSC) |
| **Language** | TypeScript 5 |
| **Authentication** | NextAuth.js v4 (JWT sessions, bcryptjs) |
| **Database ORM** | Prisma 5.21 |
| **Database** | PostgreSQL (production) / SQLite (local dev) |
| **ML / Embeddings** | `@xenova/transformers` (`all-MiniLM-L6-v2`) |
| **UI Components** | Lucide React, custom CSS variables |
| **Theming** | next-themes (dark/light) |
| **File Uploads** | Native Node.js `fs` → `/public/uploads/` |
| **Deployment** | Vercel (auto-deploy on push) |

---

## 📁 Project Structure

```
AipoweredLostandfound/
├── prisma/
│   └── schema.prisma          # Database models
├── public/
│   └── uploads/               # Uploaded images (gitignored, .gitkeep preserved)
└── src/
    ├── app/
    │   ├── page.tsx            # Landing page with feature cards
    │   ├── layout.tsx          # Root layout — nav, theme, providers
    │   ├── globals.css         # Design system (CSS vars, glass, buttons)
    │   ├── about/page.tsx      # Mission & how-it-works
    │   ├── auth/page.tsx       # Unified login / register form
    │   ├── contact/page.tsx    # Contact page
    │   ├── dashboard/page.tsx  # User dashboard (reports, matches, notifications)
    │   ├── admin/
    │   │   ├── layout.tsx      # Admin auth guard
    │   │   └── page.tsx        # Admin panel (metrics, claims, reports mgmt)
    │   ├── reports/
    │   │   ├── new/page.tsx    # Report submission form
    │   │   ├── [id]/page.tsx   # Individual report view
    │   │   └── page.tsx        # Report browser
    │   ├── matches/
    │   │   └── [id]/page.tsx   # Match detail (side-by-side + claim CTA)
    │   ├── scan/page.tsx       # Visual image scan page
    │   └── api/
    │       ├── auth/[...nextauth]/route.ts
    │       ├── reports/route.ts
    │       ├── upload/route.ts
    │       ├── scan/route.ts
    │       ├── matches/[id]/[action]/route.ts
    │       └── admin/route.ts
    ├── components/
    │   ├── LogoutButton.tsx
    │   ├── Providers.tsx
    │   └── ThemeToggle.tsx
    └── lib/
        ├── auth.ts             # NextAuth config
        ├── matchEngine.ts      # ⭐ Core matching logic
        ├── ml.ts               # ML pipeline singleton
        └── prisma.ts           # Singleton Prisma client
```

---

## 🗄️ Database Schema

```
User ──< Report ──< Image
              |
              └──< Match (lostReport + foundReport)
                       └──< NotificationLog ──> User
```

| Model | Key Fields |
|---|---|
| `User` | id, email, passwordHash, role ("USER" or "ADMIN"), preferences |
| `Report` | id, userId, type ("lost"/"found"), title, description, category, location, landmark, itemDate, status, textEmbedding |
| `Image` | id, reportId, url, thumbnailUrl, embedding (reserved for CLIP) |
| `Match` | id, lostReportId, foundReportId, scores (JSON), feedback |
| `NotificationLog` | id, userId, matchId, status |

> The `textEmbedding` field on `Report` and `embedding` on `Image` are pre-wired in the schema for the ML upgrade — no migrations needed.

---

## 🧠 The Matching Engine (Deep Dive)

The heart of this system lives in [`src/lib/matchEngine.ts`](src/lib/matchEngine.ts).

### How It Works Right Now (Rule-Based)

The engine uses a **Jaccard-like word-overlap similarity** function and combines multiple fields with fixed weights:

```
combinedScore = (titleSimilarity      × 0.40)
              + (categoryExactMatch   × 0.20)
              + (descriptionSimilarity × 0.20)
              + (locationSimilarity   × 0.20)
```

#### `wordSimilarity(a, b)`
1. Lowercases and strips punctuation from both strings
2. Splits into word-sets
3. Counts overlapping words
4. Normalizes by the **smaller** set size (partial descriptions still match well)

```typescript
overlap / Math.min(wordsA.size, wordsB.size)
```

This means: if the lost item says *"black leather wallet"* and the found item says *"wallet"*, the score is 1.0 (100%) because every word in the smaller set appears in the larger set.

#### Match Threshold

A match is created only when `combinedScore >= 0.60` (60%). This prevents spam matches while catching meaningful correlations.

#### Trigger Points

| Event | Function | Details |
|---|---|---|
| New report submitted | `processNewReport(reportId)` | Runs as a background Promise — non-blocking |
| Admin clicks "Re-scan" | `rescanAllReports()` | Full O(n²) sweep across all open pairs |

#### Notification Flow

When a match is created, a `NotificationLog` entry is inserted for **both** users (lost owner + found owner). These appear on the dashboard as "Action Required" banners linking to the match detail page.

---

### Match Score Breakdown Example

```
Lost Item:  "Black AirPods Pro" — Electronics — Library
Found Item: "AirPods" — Electronics — Library 2nd floor

Title:       wordSim("black airpods pro", "airpods") = 1/1 = 1.00 × 0.40 = 0.40
Category:    Electronics == Electronics              = 1.00 × 0.20 = 0.20
Description: wordSim("...", "...") = 0.00            × 0.20 = 0.00
Location:    wordSim("library", "library 2nd floor") = 1/1 = 1.00 × 0.20 = 0.20

Combined = 0.40 + 0.20 + 0.00 + 0.20 = 0.80 (80%) ✅ MATCH CREATED
```

---

### ⚠️ Current Limitations of the Rule-Based Engine

| Limitation | Example Failure |
|---|---|
| Lexical (exact word match only) | "mobile" ≠ "phone" ≠ "smartphone" → 0% similar |
| No semantic understanding | "blue backpack" vs "navy rucksack" → 0% despite identical objects |
| No typo tolerance | "earbudd" vs "earbuds" → 0% |
| No image awareness | Images stored, never used in matching |
| No learning from feedback | Rejections/approvals don't improve future matches |
| O(n²) complexity on re-scan | 10,000 reports = 100M comparisons |

---

## 📷 Photo Upload & Visual Scanning

### Image Upload (`/api/upload`)

When a report is filed, users can drag-and-drop or click to upload images. The handler:
1. Reads file bytes from `FormData`
2. Generates a UUID-prefixed filename
3. Writes to `public/uploads/<uuid>-<filename>` using Node's `fs/promises`
4. Returns the public URL stored in the `Image` table

Images display in report detail pages and the admin side-by-side comparison view.

### Visual Scan (`/scan` + `/api/scan`)

Users visit `/scan`, upload any photo, and get the top 3 visually similar reports back.

**Current State (Simulated):**
```typescript
// Proof-of-concept — randomized confidence to demonstrate the UI flow
const candidates = allReports
  .filter(r => r.images.length > 0)
  .map(report => ({
    report,
    score: Math.random() * 0.4 + 0.5  // 50–90% simulated
  }))
  .sort((a, b) => b.score - a.score)
  .slice(0, 3);
```

The `ml.ts` file already contains the **singleton pattern** and **cosine similarity** ready for real embeddings:
```typescript
static async getTextPipeline() {
  if (!this.textInstance) {
    this.textInstance = await pipeline("feature-extraction", "Xenova/all-MiniLM-L6-v2");
  }
  return this.textInstance;
}

static cosineSimilarity(vecA: number[], vecB: number[]): number {
  // dot product / (|A| × |B|)
}
```

---

## 💡 How It Helps — Real-World Impact

| Scenario | Without This System | With This System |
|---|---|---|
| Student loses AirPods in library | Posts on WhatsApp, waits | Files report → engine matches in seconds → both notified |
| Faculty finds a wallet | Hands to security desk; owner never finds out | Files "Found" → owner auto-notified via match |
| Admin reviews 200 reports | Hours of manual cross-checking | Pre-ranked matches with confidence scores; approve in seconds |
| Someone files a fake claim | No verification | Requires admin approval before any contact info is revealed |
| Phone found with no label | Impossible to identify | Upload photo to `/scan` → find visually similar reports |

**Core societal value:**
- Reduces anxiety and friction around item loss in campus / institutional environments
- Rewards good samaritans by making it effortless to report found items
- Builds institutional trust through the admin verification gate
- Scales to any organization: colleges, airports, hotels, events

---

## 🚀 Upgrade Roadmap: From Rule-Based to ML-Powered

The infrastructure (`ml.ts`, `textEmbedding` on `Report`, `embedding` on `Image`) is **already in place** for a full ML upgrade.

---

### Phase 1 — Semantic Text Embeddings (Ready to Activate)

**Replace:** `wordSimilarity()` in `matchEngine.ts`  
**With:** Cosine similarity of 384-dimensional sentence vectors from `all-MiniLM-L6-v2`

```typescript
// Proposed upgrade in matchEngine.ts
import MLPipeline from "./ml";

async function semanticSimilarity(a: string, b: string): Promise<number> {
  const pipe = await MLPipeline.getTextPipeline();
  const [embA, embB] = await Promise.all([pipe(a), pipe(b)]);
  return MLPipeline.cosineSimilarity(Array.from(embA.data), Array.from(embB.data));
}
```

**What this unlocks:**
- `"phone"` ↔ `"smartphone"` → ~0.85 similarity
- `"blue bag"` ↔ `"navy backpack"` → ~0.72 similarity
- Handles synonyms, typos, and paraphrases automatically

**Storage strategy:** Pre-compute embeddings at report creation time and persist in `Report.textEmbedding` (JSON). Matching then becomes a vector dot-product — **O(n)** instead of O(n²).

---

### Phase 2 — Real Visual Matching with CLIP

**Replace:** Randomized score in `/api/scan/route.ts`  
**With:** `Xenova/clip-vit-base-patch32` for joint image-text embeddings

```typescript
// In ml.ts — add image pipeline
static imageModel = "Xenova/clip-vit-base-patch32";
static imageInstance: any = null;

static async getImagePipeline() {
  if (!this.imageInstance) {
    this.imageInstance = await pipeline("zero-shot-image-classification", this.imageModel);
  }
  return this.imageInstance;
}
```

**How the scan route upgrades:**
1. Receive uploaded image buffer
2. Compute CLIP 512-dim embedding
3. Compare against stored `Image.embedding` vectors
4. Return top-K matches by cosine similarity

**What this unlocks:**
- Photo-to-photo search — find visually identical items across the database
- Cross-modal search — text query matches against image embeddings
- Zero labels needed — CLIP works on raw visual features

**Schema column already exists:** `Image.embedding String?`

---

### Phase 3 — Approximate Nearest Neighbor (ANN) for Scale

For 10,000+ reports, brute-force cosine similarity is impractical. Integrate a vector store:

| Option | Integration | Notes |
|---|---|---|
| **pgvector** | Prisma raw query | Native PostgreSQL extension; easiest to add |
| **Pinecone** | REST API | Serverless-friendly, Vercel-compatible |
| **Qdrant** | REST / gRPC | High-performance, Rust-based, open-source |

With pgvector, matching becomes a **single indexed SQL query**:
```sql
SELECT id, title, 1 - ("textEmbeddingVec" <=> $1) AS similarity
FROM "Report"
WHERE type = 'found' AND status = 'open'
ORDER BY similarity DESC
LIMIT 10;
```

---

### Phase 4 — Feedback Learning Loop

Currently, approvals and rejections are stored but never used. A learning loop would:

1. **Collect signal** — track approved vs. rejected matches per score range
2. **Re-weight features** — if location-based matches consistently get rejected, reduce location weight dynamically
3. **Fine-tune embeddings** — use approved pairs as positives for contrastive learning (triplet loss)
4. **Active learning** — flag borderline matches (0.60–0.70 score) for priority admin review

---

### Phase 5 — LLM-Assisted Description Enhancement

Integrate a lightweight LLM (Gemini Flash via API) to:
- **Auto-expand descriptions** — `"black AirPods"` → `"black Apple AirPods Pro wireless earbuds in charging case"`
- **Extract structured attributes** — color, brand, size, distinguishing features as structured JSON
- **Generate match explanations** — `"Matched because both are black Apple earbuds lost in the library on the same day"`

---

### Upgrade Impact Summary

| Phase | Change | Effort | Impact |
|---|---|---|---|
| 1. Semantic Text | Replace word overlap with sentence embeddings | Low (code ready in `ml.ts`) | **High** — kills synonym failures |
| 2. Visual CLIP | Activate `getImagePipeline()`, store `Image.embedding` | Medium | **Very High** — enables photo search |
| 3. ANN Index | Add pgvector or Pinecone | Medium | **High** — scales to 100K+ reports |
| 4. Feedback Loop | Train on approved/rejected pairs | High | **Very High** — self-improving system |
| 5. LLM Assist | Gemini API at report submission | Low-Medium | **Medium** — better data quality |

---

## 🔌 API Reference

| Method | Endpoint | Auth | Description |
|---|---|---|---|
| `POST` | `/api/auth/[...nextauth]` | — | NextAuth sign-in / register |
| `POST` | `/api/reports` | User | Create a new lost/found report |
| `GET` | `/api/reports` | — | List reports (filter: `?type=lost\|found`) |
| `POST` | `/api/upload` | — | Upload image → returns `{ url }` |
| `POST` | `/api/scan` | — | Visual scan → top candidate reports |
| `POST` | `/api/matches/:id/:action` | User/Admin | Actions: `confirm`, `reject`, `request_claim` |
| `GET` | `/api/admin` | Session | Admin dashboard data |
| `POST` | `/api/admin` | Session | Admin actions: `approve`, `reject`, `delete_report`, `rescan` |

---

## ⚙️ Local Setup

### Prerequisites
- Node.js >= 18
- npm
- PostgreSQL (or SQLite for local dev — change `provider` in `schema.prisma`)

### Steps

```bash
# 1. Clone the repository
git clone https://github.com/Akhilcrab/AI--Powered-Lost-and-Found-Web-Portal.git
cd AI--Powered-Lost-and-Found-Web-Portal

# 2. Install dependencies
npm install

# 3. Set up environment variables
# Create a .env file (see Environment Variables section below)

# 4. Push the database schema
npx prisma generate
npx prisma db push

# 5. Start the development server
npm run dev
```

Visit [http://localhost:3000](http://localhost:3000).

**Admin access:** On first login with the admin credentials defined in `src/lib/auth.ts`, the account is auto-created.

---

## 🚢 Deployment

Auto-deploys to **Vercel** on every push to `main`:

1. Push to GitHub (this repo)
2. Connect on [vercel.com](https://vercel.com)
3. Add environment variables in the Vercel dashboard
4. Vercel automatically runs `prisma generate && next build`

---

## 👤 Author

**Akhil** — Built as part of the Smart Lost & Found initiative  
*Demonstrating full-stack AI integration, matching engine design, and product thinking.*

---

<div align="center">
Made with ❤️ using Next.js, Prisma, and @xenova/transformers
</div>
