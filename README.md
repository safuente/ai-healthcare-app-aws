# MediNotes Pro - AI Healthcare Consultation Assistant

> Transform consultation notes into professional medical documentation with AI

[![Live Demo](https://img.shields.io/badge/demo-live-brightgreen)](https://ai-healthcare-g572hhcbe-safuentes-projects.vercel.app)
[![Next.js](https://img.shields.io/badge/Next.js-16-black)](https://nextjs.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115-009688)](https://fastapi.tiangolo.com/)
[![Clerk](https://img.shields.io/badge/Clerk-Auth%20%26%20Billing-6C47FF)](https://clerk.com/)

## Demo

**Live App:** [https://ai-healthcare-g572hhcbe-safuentes-projects.vercel.app](https://ai-healthcare-g572hhcbe-safuentes-projects.vercel.app)

## About

MediNotes Pro is a SaaS application for healthcare professionals that uses AI to transform raw consultation notes into structured medical documentation. Built with Next.js and FastAPI, it features secure authentication, subscription management via Clerk Billing, and real-time streaming AI responses.

## Features

- **Professional Summaries** - Generate comprehensive medical record summaries from consultation notes
- **Action Items** - Clear next steps and follow-up actions for every patient visit
- **Patient Emails** - Draft clear, patient-friendly email communications automatically
- **Real-time Streaming** - AI responses streamed via Server-Sent Events (SSE)
- **Secure Authentication** - HIPAA-conscious design with Clerk authentication
- **Subscription Management** - Premium tier with Clerk Billing

## Tech Stack

| Layer | Technology |
|-------|------------|
| **Frontend** | Next.js 16, React 19, TypeScript, Tailwind CSS 4 |
| **Backend** | FastAPI (Python), Vercel Serverless Functions |
| **AI Model** | OpenAI GPT |
| **Auth & Billing** | Clerk |
| **Deployment** | Vercel |

## Architecture

```
┌─────────────────┐         ┌─────────────────┐         ┌─────────────────┐
│                 │   JWT   │                 │         │                 │
│   Next.js       │────────▶│   FastAPI       │────────▶│   OpenAI GPT    │
│   Frontend      │         │   /api          │         │                 │
│                 │◀────────│                 │◀────────│                 │
└─────────────────┘   SSE   └─────────────────┘         └─────────────────┘
        │
        │ Auth & Billing
        ▼
┌─────────────────┐
│     Clerk       │
│  (JWT + JWKS)   │
└─────────────────┘
```

## Project Structure

```
saas/
├── api/
│   └── index.py           # FastAPI backend (serverless function)
├── pages/
│   ├── _app.tsx           # Clerk provider setup
│   ├── _document.tsx      # HTML document structure
│   ├── index.tsx          # Landing page
│   └── product.tsx        # Consultation form (subscription protected)
├── public/
├── styles/
│   └── globals.css        # Global styles + Tailwind
├── package.json
└── requirements.txt
```

## Getting Started

### Prerequisites

- Node.js 18+
- Python 3.11+
- [Clerk account](https://clerk.com)
- [OpenAI API key](https://platform.openai.com)

### Installation

```bash
# Clone the repository
git clone https://github.com/safuentes/ai-healthcare-app.git
cd ai-healthcare-app/saas

# Install frontend dependencies
npm install

# Install backend dependencies
pip install -r requirements.txt
```

### Environment Variables

Create a `.env.local` file:

```env
# Clerk Authentication
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=pk_test_xxxxx
CLERK_SECRET_KEY=sk_test_xxxxx
CLERK_JWKS_URL=https://your-app.clerk.accounts.dev/.well-known/jwks.json

# OpenAI
OPENAI_API_KEY=sk-xxxxx
```

### Run Locally

```bash
npm run dev
```

Visit [http://localhost:3000](http://localhost:3000)

## User Flow

```
1. User visits landing page (/)
2. Signs up/in via Clerk modal
3. Navigates to /product
4. Clerk checks subscription status:
   ├─ No subscription → Shows PricingTable
   └─ Has subscription → Shows ConsultationForm
5. User enters patient info and consultation notes
6. AI generates summary, action items, and patient email
```

## API Reference

### `POST /api`

Generates consultation summary using OpenAI.

**Authentication:** Required (Bearer JWT)

**Request Body:**
```json
{
  "patient_name": "John Doe",
  "date_of_visit": "2026-01-17",
  "notes": "Patient presents with..."
}
```

**Response:** `text/event-stream`

Server-Sent Events stream with Markdown-formatted:
- Summary of visit for doctor's records
- Next steps for the doctor
- Draft email to patient

## Deployment

### Vercel

```bash
cd saas
vercel --prod
```

Configure environment variables in Vercel Dashboard:
- `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`
- `CLERK_SECRET_KEY`
- `CLERK_JWKS_URL`
- `OPENAI_API_KEY`

## Subscription Tiers

| Feature | Free | Premium |
|---------|:----:|:-------:|
| View landing page | Yes | Yes |
| AI consultation summaries | No | Unlimited |
| Streaming responses | No | Yes |
| Patient email drafts | No | Yes |

## Troubleshooting

| Issue | Solution |
|-------|----------|
| PricingTable not rendering | Enable Billing in Clerk Dashboard |
| "Plan not found" error | Verify plan key is exactly `premium_subscription` |
| Auth errors | Check CLERK_JWKS_URL matches your Clerk app |
| SSE not streaming | Ensure JWT is valid and passed in Authorization header |
| OpenAI errors | Verify OPENAI_API_KEY is set in Vercel environment |

## License

MIT License

## Author

**Santiago Fuente**

---

Built with Next.js, FastAPI & OpenAI
