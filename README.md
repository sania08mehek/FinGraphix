# FinGraphix — Money Muling Detection Engine

![FinGraphix Logo](frontend/public/fingraphix-logo.jpg)



> **RIFT 2026 Hackathon** | Graph Theory / Financial Crime Detection Track



A web-based **Financial Forensics Engine** that processes transaction data and exposes money muling networks through **graph analysis and interactive visualization**.

🔗 **Live Demo**: _[https://fin-graphix-79n1-5rw82uddx-irfan-irs-projects.vercel.app/]_

---

## 📋 Project Title

**FinGraphix** — Graph-Based Financial Crime Detection Engine

## 🏗 Tech Stack

| Layer      | Technology                                |
|------------|------------------------------------------|
| Frontend   | Next.js 16, React 19, TypeScript, D3.js  |
| Backend    | Python, FastAPI, Uvicorn                 |
| Engine     | NetworkX, Custom Graph Algorithms        |
| Styling    | Tailwind CSS 4, shadcn/ui               |
| Deployment | Vercel (Frontend) + Render (Backend)     |

## 🔬 System Architecture

```
┌──────────────────────────────────────────────────────────────┐
│                        FRONTEND (Next.js)                    │
│   ┌──────────┐   ┌─────────────┐   ┌────────────────────┐   │
│   │  Upload   │──▶│  Dashboard  │──▶│  Graph Visualizer  │   │
│   │  (CSV)    │   │  (Tables)   │   │  (D3.js Interactive)│  │
│   └──────────┘   └─────────────┘   └────────────────────┘   │
│        │              ▲                      ▲               │
└────────┼──────────────┼──────────────────────┼───────────────┘
         │              │                      │
    POST /api/analyze   │ GET /api/results/{id}│
         │              │                      │
┌────────▼──────────────┴──────────────────────┴───────────────┐
│                        BACKEND (FastAPI)                      │
│   ┌──────────┐   ┌─────────────┐   ┌────────────────────┐   │
│   │  Upload   │──▶│  togh.py    │──▶│  Detection Engine  │   │
│   │  Handler  │   │  CSV→Graph  │   │  (Pipeline)        │   │
│   └──────────┘   └─────────────┘   └────────────────────┘   │
│                                            │                 │
│                                     ┌──────▼──────┐         │
│                                     │ Output JSON  │         │
│                                     │ (Results)    │         │
│                                     └─────────────┘         │
└──────────────────────────────────────────────────────────────┘

                     DETECTION ENGINE PIPELINE
┌──────────────────────────────────────────────────────────────┐
│  Stage 0: Ingest (CSV parsing, validation, dedup)            │
│  Stage 1: Graph Build (adjacency lists, node stats)          │
│  Module 1: Cycle Detection (SCC + Johnson's algorithm)       │
│  Module 2: Smurfing Detection (Fan-in/Fan-out patterns)      │
│  Module 3: Shell Chain Detection (low-TX intermediaries)      │
│  Stage 4: Scoring (weighted structural + velocity + retention)│
│  Stage 5: Deduplication (overlap merging)                    │
│  Stage 6: Output Builder (ring assignment, suspicion scores)  │
└──────────────────────────────────────────────────────────────┘
```

## 🧠 Algorithm Approach

### 1. Circular Fund Routing (Cycles) — O(V + E) × cycles
- Uses **Tarjan's SCC** to find strongly connected components (min size 3)
- Runs **Johnson's Algorithm** to enumerate all simple cycles of length 3–5
- Each cycle becomes a fraud ring candidate
- **Complexity**: O((V + E)(C + 1)) where C = number of cycles found

### 2. Smurfing Patterns (Fan-in/Fan-out)
- **Fan-in**: Detects nodes receiving from ≥10 unique senders within 72h
- **Fan-out**: Detects nodes sending to ≥10 unique receivers within 72h
- **False positive suppression**: Filters out payroll patterns (low CV in amounts/gaps) and merchants (high average amounts, high variance)
- **Complexity**: O(V × D) where D is the max degree

### 3. Layered Shell Networks (Chain Detection)
- DFS-based path exploration from each node
- Identifies chains of 3+ hops where intermediate nodes have ≤4 total transactions
- Amount retention ratio check (≥55% per hop)
- Temporal ordering enforcement (each hop within 90 minutes)
- **Complexity**: O(V × P) where P = max paths per start node (capped at 500)

## 📊 Suspicion Score Methodology

The suspicion score (0–100) for each account is computed using a **weighted multi-factor formula**:

```
Base Score = 0.35 × Structural Score + 0.40 × Velocity Score + 0.25 × Retention Score
```

| Factor | Weight | Description |
|--------|--------|-------------|
| Structural | 35% | Cycle length bonus, SCC membership, topology |
| Velocity | 40% | Time span analysis — tighter cycles score higher |
| Retention | 25% | Amount preservation ratio across hops |

**Additional modifiers:**
- **Tightness Bonus**: Up to +30 points for cycles completing within 48 hours
- **Retention Floor**: Minimum 50-60 points for high-retention cycles (≥65-80%)
- **Multi-ring Bonus**: +5 points per additional ring membership
- **Suspicion cap**: Maximum 100.0

**Risk levels:**
| Score Range | Level |
|-------------|-------|
| ≥ 80 | CRITICAL |
| ≥ 60 | HIGH |
| ≥ 36 | MEDIUM |
| < 36 | LOW |

## 🚀 Installation & Setup

### Prerequisites
- Python 3.11+
- Node.js 18+ (with npm)

### Backend Setup
```bash
cd FinGraphix
pip install -r requirements.txt
python -m uvicorn backend.app:app --host 0.0.0.0 --port 8000 --reload
```

### Frontend Setup
```bash
cd frontend
npm install
npm run dev
```

The frontend runs on `http://localhost:3000` and proxies API requests to the backend on port 8000.


## 📖 Usage Instructions

1. **Upload CSV**: Drag and drop or click to upload a CSV file with columns: `transaction_id`, `sender_id`, `receiver_id`, `amount`, `timestamp`
2. **Wait for analysis**: The engine processes the data through all 6 pipeline stages
3. **View Dashboard**: Interactive graph, summary stats, fraud rings table, suspicious accounts table
4. **Download Report**: Click "Download Report" to get the JSON output file
5. **Try Sample Data**: Click "Run Sample Analysis" on the home page to test with built-in data

## 📁 Project Structure

```
FinGraphix/
├── backend/
│   └── app.py              # FastAPI endpoints
├── engine/
│   ├── togh.py              # CSV → NetworkX graph converter
│   ├── pipeline.py          # Detection pipeline orchestrator
│   ├── ingest.py            # Stage 0: Data ingestion & validation
│   ├── graph_builder.py     # Stage 1: Adjacency list construction
│   ├── cycle_detector.py    # Module 1: SCC + Johnson's cycle detection
│   ├── smurf_detector.py    # Module 2: Fan-in/Fan-out pattern detection
│   ├── chain_detector.py    # Module 3: Shell chain detection
│   ├── scorer.py            # Stage 4: Multi-factor scoring
│   ├── deduplicator.py      # Stage 5: Cluster deduplication
│   ├── output_builder.py    # Stage 6: JSON output construction
│   ├── models.py            # Data models (Edge, Cluster, FraudRing, etc.)
│   └── config.py            # Engine configuration & thresholds
├── frontend/
│   ├── app/
│   │   ├── page.tsx         # Home / Upload page
│   │   ├── dashboard/       # Dashboard with graph & tables
│   │   └── layout.tsx       # Root layout with navbar
│   ├── components/
│   │   ├── GraphVisualizer.jsx  # Interactive D3 graph
│   │   ├── upload-dropzone.tsx  # CSV upload component
│   │   └── navbar.tsx           # Navigation bar
│   └── next.config.mjs     # API proxy configuration
├── data/
│   ├── uploads/             # Uploaded CSV files
│   ├── output/              # Detection result JSON files
│   └── transactions.csv     # Sample dataset
├── requirements.txt         # Python dependencies
└── README.md
```

## ⚠ Known Limitations

1. **In-memory cache**: Results are stored in-memory on the backend; they reset on server restart. Disk fallback is available.
2. **No persistent database**: No SQL/NoSQL database for production persistence.
3. **Single-threaded engine**: Graph algorithms run synchronously; very large datasets (>10K transactions) may approach the 30-second limit.
4. **False positive tuning**: Payroll and merchant suppression heuristics may need adjustment for specific datasets.
5. **Cycle enumeration**: Johnson's algorithm is exponential in the worst case; cycles capped at length 5.

•Built with Passion & Coffee ~ Sania
