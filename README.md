# InvestIQ AI 📈

> **"Your Intelligent Investment Research Analyst"**

InvestIQ AI is a premium, production-ready AI Investment Research Agent that compiles institution-grade equity research reports from public information in seconds. Built on a modular multi-agent pipeline using **Next.js (App Router)**, **LangChain.js**, and real-time **Server-Sent Events (SSE)**, it delivers a high-fidelity SaaS experience akin to Linear, Stripe, and Vercel.

---

## 🚀 Key Features

*   **Real-time Streaming Pipeline**: Instead of blocking during multi-minute LLM operations, it streams progress checkmarks and log updates to the client in real-time.
*   **Configurable LLM Provider**: Dynamically switch between **Google Gemini (Gemini 2.5 Pro)** and **OpenAI (GPT-4o)** via settings drawer.
*   **Web Research Grounding**: Integrates **Tavily Search API** to scour the live web for financial metrics, competitors, and 2025/2026 news.
*   **Premium SaaS Aesthetics**: Responsive dark mode dashboard featuring glassmorphic navigation, Framer Motion transitions, custom SVG circular gauges, and glowing card borders.
*   **Interactive Visualizations**: Renders dual-tone gradient area charts of revenue trends using **Recharts**.
*   **Persistent Search Cache**: Automatically saves past analyses to a local browser history drawer (using synchronized LocalStorage hooks) for instant reloading.
*   **Actionable Utilities**:
    *   **Export as PDF**: Implements specific print CSS rules (`@media print`) that format the dashboard into a vector-grade print layout.
    *   **Copy Report**: Compiles the entire investment thesis and key metrics into a structured Markdown block.
*   **Zod & React Hook Form Validation**: Secure, accessible input validation for searching.

---

## 🛠 Tech Stack

*   **Framework**: Next.js 15+ (App Router)
*   **Language**: TypeScript (Strict checks)
*   **Styling**: Tailwind CSS v4 & custom scrollbar overlays
*   **Animations**: Framer Motion
*   **Visualizations**: Recharts
*   **Agent Pipeline**: LangChain.js (`@langchain/core`, `@langchain/openai`, `@langchain/google-genai`)
*   **Search Provider**: Tavily Search REST API
*   **Forms & Validation**: React Hook Form + Zod Schema Resolver
*   **UI Foundations**: Lucide React icons, Canvas Confetti particle animations

---

## 📂 Folder Structure

```
investiq-ai/
├── src/
│   ├── agents/
│   │   └── researchAgent.ts         # Agent coordinator executing Tavily search & LangChain
│   ├── app/
│   │   ├── api/
│   │   │   └── analyze/
│   │   │       └── route.ts         # SSE streaming Next.js Route Handler
│   │   ├── globals.css              # Custom styling tokens, print media & animations
│   │   ├── layout.tsx               # ToastWrapper & SEO metadata setup
│   │   └── page.tsx                 # Search form, Settings modals & Landing screen
│   ├── components/
│   │   ├── ui/                      # Base interface elements (Card, Button, Input, Toast)
│   │   ├── BusinessModel.tsx
│   │   ├── CompanyOverview.tsx
│   │   ├── CompetitiveAnalysis.tsx
│   │   ├── ExecutiveSummary.tsx
│   │   ├── FinancialSnapshot.tsx
│   │   ├── FinalDecision.tsx
│   │   ├── InvestmentScore.tsx
│   │   ├── LatestNews.tsx
│   │   ├── LoadingScreen.tsx        # Real-time multi-step checker
│   │   ├── ResearchDashboard.tsx    # Widget grid organizer & action buttons
│   │   ├── RiskAssessment.tsx
│   │   ├── SearchHistory.tsx        # LocalStorage cache drawer
│   │   └── SourcesList.tsx
│   ├── hooks/
│   │   └── useLocalStorage.ts       # Synchronized client caching hook
│   ├── lib/
│   │   ├── api-client.ts            # Client-side SSE Chunk Decoder
│   │   └── utils.ts                 # Styling utilities (cn)
│   ├── prompts/
│   │   └── analysis.ts              # Structured system & user prompt templates
│   ├── services/
│   │   ├── llm.ts                   # LangChain factory (GPT-4o & Gemini 2.5 Pro)
│   │   └── tavily.ts                # RESTful Tavily Search interface
│   └── types/
│       └── index.ts                 # Report, History, and Progress types
```

---

## ⚙️ Environment Variables

Create a `.env.local` file in the root directory:

```env
# Tavily Search API (Required)
TAVILY_API_KEY=tvly-...

# Configurable LLM Provider: 'gemini' or 'openai' (Default: 'gemini')
LLM_PROVIDER=gemini

# Google Gemini API key (Required if LLM_PROVIDER=gemini)
GEMINI_API_KEY=AIzaSy...

# OpenAI API key (Required if LLM_PROVIDER=openai)
OPENAI_API_KEY=sk-proj-...
```

---

## 🏁 How to Run

### 1. Install Dependencies
```bash
npm install
```

### 2. Run Local Dev Server
```bash
npm run dev
```
Open [http://localhost:3000](http://localhost:3000) in your browser.

### 3. Run Build & Typecheck
```bash
npm run build
```

---

## 📐 Architecture & Multi-Agent Design

The application implements a decoupled, event-driven streaming architecture.

```
[UI Search Input] --(HTTP POST)--> [API Route (/api/analyze)]
                                            │
                                 [Starts Research Agent]
                                            │
  (Step 1) Search Web  <-----------> [Tavily Search API]
  (Step 2) Filter Info
  (Step 3) Isolate Financials
  (Step 4) Map Competitors
  (Step 5) Assess Risk
  (Step 6) Run Reasoning <---------> [LangChain.js (OpenAI / Gemini)]
  (Step 7) Final Thesis
                                            │
                                (Progress Event Callback)
                                            │
                                            ▼
                    [SSE Streaming Chunks (TextEncoder)]
                                            │
                                            ▼
                  [Client ReadableStream Reader decoding chunks]
                                            │
                                            ▼
                                [Update Dashboard UI]
```

1.  **Orchestrator Pattern**: `researchAgent` manages the workflow. It invokes the search API and passes formatted contexts to the configured LLM, avoiding agent-to-agent loops which can drift in execution time.
2.  **Streaming Controller**: The Node.js Route Handler wraps execution in a `ReadableStream`. The progress callback pushes Server-Sent Events (SSE) directly to the client socket, giving instant feedback during LLM evaluation.

---

## ⚖️ Trade-offs & Decisions

1.  **JSON Mode vs. Pydantic Parser**: We configured standard JSON Mode output parameters (`responseFormat: "json_object"` for OpenAI, `json: true` for Gemini) combined with structured prompt schemas. While LangChain's `withStructuredOutput` is standard, it can fail to compile across different SDK releases or behave differently between OpenAI/Gemini under the hood. Our manual JSON parser extracts matching JSON blocks robustly and falls back gracefully.
2.  **Tavily Search API Client**: Instead of using the full Tavily SDK wrapper (which introduces extra weight and peer dependency overlaps), we implemented a direct, lightweight REST client via Next.js `fetch`. This reduces package footprints and allows better timeout handling.
3.  **PDF Generation**: Rather than rendering canvas screenshots (which can look blurry and disrupt text selection) or compiling PDFs on the server side (which adds compute cost and latency), we designed a clean `@media print` print layout. When the user clicks "Export to PDF", the browser's native print engine outputs a vector-grade, print-optimized document directly.

---

## 🔮 Future Improvements

1.  **Live Financial Data Feeds**: Integrate a market data API (like Alpha Vantage or SEC EDGAR) to augment web search results with real-time financial filings.
2.  **Vector Store (RAG) Support**: Allow uploading local investment prospectuses (PDFs), feeding them into an embedded vector store to combine live web search with custom internal company data.
3.  **Interactive SWOT/Risk details**: Enable accordion details or interactive drilldowns inside SWOT quadrants and Risk category cards.
4.  **Multi-agent Debate**: Introduce a dual-agent configuration where an "Investment Bull" and an "Investment Bear" debate the equity's value before generating the final investment rating.
