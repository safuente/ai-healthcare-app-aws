# MediNotes Pro - AI Healthcare Consultation Assistant

> Transform consultation notes into professional medical documentation with AI

[![Next.js](https://img.shields.io/badge/Next.js-16-black)](https://nextjs.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-0.115-009688)](https://fastapi.tiangolo.com/)
[![Clerk](https://img.shields.io/badge/Clerk-Auth%20%26%20Billing-6C47FF)](https://clerk.com/)
[![AWS Lambda](https://img.shields.io/badge/AWS-Lambda-FF9900)](https://aws.amazon.com/lambda/)

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
| **Backend** | FastAPI (Python) |
| **AI Model** | OpenAI GPT |
| **Auth & Billing** | Clerk |
| **Deployment** | AWS Lambda (container image) + ECR |

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
git clone https://github.com/safuentes/ai-healthcare-app-aws.git
cd ai-healthcare-app-aws/saas

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

### AWS Lambda (container image)

Deploy the app as a single container on AWS Lambda, fronted by a Function URL with response streaming. This packages the Next.js static export + FastAPI backend into one image (see [`saas/Dockerfile`](saas/Dockerfile)), pushes it to ECR, and runs it on Lambda — no load balancer, no VPC, and effectively zero cost at low traffic.

**Prerequisites:** Docker, the [AWS CLI](https://aws.amazon.com/cli/) configured with `aws configure`, and a `saas/.env` containing `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY`, `CLERK_SECRET_KEY`, `CLERK_JWKS_URL`, `OPENAI_API_KEY`, `AWS_ACCOUNT_ID`, and `DEFAULT_AWS_REGION`.

> **Architecture rule (read first):** the `--platform` you build with **must** match the *Architecture* you select for the Lambda function. Mismatch causes `exec format error` at runtime. These steps use **`arm64`** (native on Apple Silicon → faster builds, no emulation, cheaper Graviton). For Intel/x86 just swap `linux/arm64` → `linux/amd64` and select `x86_64` in Lambda.

#### 1. Create an IAM user & permissions (one-time)

Never use the root account for daily work. Create a limited IAM user with the permissions this deployment needs.

**Create the user** — AWS Console → **IAM** → **Users** → **Create user**:
- Username: `aiengineer`
- Check **Provide user access to the AWS Management Console** → **I want to create an IAM user**
- Choose **Custom password**, set a strong one, and **uncheck** *Users must create a new password at next sign-in* → **Next**

**Create a permission group** — on the permissions page choose **Add user to group** → **Create group**:
- Group name: `BroadAIEngineerAccess`
- Attach these policies (the first replaces the `AWSAppRunnerFullAccess` you'd use for App Runner):
  - **`AWSLambda_FullAccess`** — deploy and manage the Lambda function
  - **`AmazonEC2ContainerRegistryFullAccess`** — push/store Docker images in ECR
  - **`CloudWatchLogsFullAccess`** — view function logs
  - **`IAMUserChangePassword`** — manage own credentials
  - **`IAMFullAccess`** — required so Lambda can create its own service-linked execution role
- **Create user group**, make sure `BroadAIEngineerAccess` is checked → **Next** → **Create user**
- **Download the .csv** and store it securely

> If AWS later complains that `aiengineer` lacks a permission, come back here as the root user and attach the missing policy — a normal part of working with AWS.

**Sign in as the IAM user** — sign out of root, go to your account sign-in URL (in the CSV, like `https://<account-id>.signin.aws.amazon.com/console`), and log in as `aiengineer`. You should see `aiengineer @ <account-id>` in the top-right corner.

**Create access keys for the CLI** — IAM → Users → `aiengineer` → **Security credentials** → **Create access key** → **Command Line Interface (CLI)** → download the key pair, then run `aws configure` and enter the Access Key ID, Secret Access Key, your region (matching `DEFAULT_AWS_REGION`), and `json` output. This is what authenticates the `aws` / `docker` commands in the steps below.

#### 2. Create the ECR repository

AWS Console → **ECR** → confirm the region matches `DEFAULT_AWS_REGION` → **Create repository** → Visibility **Private**, name **`consultation-app`** (must match exactly) → **Create**.

#### 3. Load env vars and authenticate Docker to ECR

From the `saas/` directory:

```bash
cd saas
set -a; source .env; set +a   # export every var from .env into the shell

# sanity check — none of these should be empty
echo "ACCT=$AWS_ACCOUNT_ID  REGION=$DEFAULT_AWS_REGION  CLERK=${NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY:0:8}"

aws ecr get-login-password --region "$DEFAULT_AWS_REGION" \
  | docker login --username AWS --password-stdin \
    "$AWS_ACCOUNT_ID.dkr.ecr.$DEFAULT_AWS_REGION.amazonaws.com"
```

> The `.env` is gitignored and dockerignored, so its secrets never reach the repo or the image. `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` is the only one needed at build time (baked into the static files); the rest are runtime env vars set on the Lambda in step 5.

#### 4. Build and push the image

Use **`docker buildx`** with `--provenance=false`. A plain `docker build` (or buildx with defaults) attaches provenance/SBOM attestations that turn the push into an OCI image index, which Lambda rejects with *"The image manifest, config or layer media type ... is not supported."* `--provenance=false` produces the single manifest Lambda needs, and `--push` uploads it directly (no separate `tag`/`push`):

```bash
docker buildx build \
  --platform linux/arm64 \
  --provenance=false \
  --build-arg NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY="$NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY" \
  -t "$AWS_ACCOUNT_ID.dkr.ecr.$DEFAULT_AWS_REGION.amazonaws.com/consultation-app:latest" \
  --push .
```

The first build is slow (a full `npm ci`, several minutes); later builds reuse the cached layer unless `package*.json` changes. **Checkpoint:** the `consultation-app` repo in ECR shows an image tagged `latest`.

#### 5. Create the Lambda function

AWS Console → **Lambda** → confirm region → **Create function** → **Container image**:

- **Function name:** `consultation-app`
- **Container image URI:** Browse images → repo `consultation-app` → tag `latest`
- **Architecture:** **`arm64`** ← must match the build platform from step 4
- **Create function** (provisioning + image pull takes ~30–60s)

Then under the **Configuration** tab:

- **General configuration** → Edit → **Memory** `1024 MB`, **Timeout** `5 min 0 sec` (room for long OpenAI streams). Leave ephemeral storage at 512 MB.
- **Concurrency** → Edit → **Reserve concurrency** = `2` (a free hard ceiling so a runaway/bot can't rack up usage; excess requests get HTTP 429). Do **not** touch *Provisioned concurrency* — that's a separate paid feature.
- **Environment variables** → Edit → add (these match the `.env`; `NEXT_PUBLIC_*` is **not** needed here, it was baked in at build time):
  - `CLERK_SECRET_KEY`
  - `CLERK_JWKS_URL`
  - `OPENAI_API_KEY`

  `PORT` and `AWS_LWA_INVOKE_MODE` are already set inside the Dockerfile, so don't re-add them.

#### 6. Create the Function URL

**Configuration** → **Function URL** → Create function URL:

- **Auth type:** **`NONE`** — auth is handled by Clerk JWTs inside the app; `AWS_IAM` would block browsers from calling the endpoint with `Forbidden`.
- Additional settings → **Invoke mode:** **`RESPONSE_STREAM`** — required, or SSE responses get buffered and the streaming UI never updates.
- CORS: leave unchecked (FastAPI already sets CORS headers via its middleware).
- **Save.**

This gives a public URL like `https://<id>.lambda-url.<region>.on.aws/`. Open it — the first request may take 10–30s (cold start), then it's sub-second. Verify: page loads → Clerk sign-in → generate a summary and confirm text streams word-by-word (proves `RESPONSE_STREAM` is on).

#### Troubleshooting (AWS)

| Error | Cause & fix |
|-------|-------------|
| `Export encountered an error on /` during build | `NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY` build-arg was empty (Clerk can't prerender). Run `set -a; source .env; set +a` before building. |
| `... media type ... is not supported` when creating the Lambda | Image was pushed as an OCI index with attestations. Rebuild with `docker buildx ... --provenance=false`. |
| `fork/exec /opt/extensions/lambda-adapter: exec format error` | Image arch ≠ Lambda *Architecture*. Match `--platform linux/arm64` ↔ Lambda `arm64` (or `linux/amd64` ↔ `x86_64`). |
| `{"Message":"Forbidden..."}` from the Function URL | Function URL auth type is `AWS_IAM`. Change it to `NONE`. |
| Streaming UI never updates | Function URL **Invoke mode** isn't `RESPONSE_STREAM`. |

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
| OpenAI errors | Verify OPENAI_API_KEY is set in the Lambda environment variables |

## License

MIT License

## Author

**Santiago Fuente**

---

Built with Next.js, FastAPI & OpenAI
