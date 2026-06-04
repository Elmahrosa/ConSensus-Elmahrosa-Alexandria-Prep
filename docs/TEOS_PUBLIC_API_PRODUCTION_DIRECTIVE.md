# TEOS SOVEREIGN PUBLIC API — PRODUCTION BUILD DIRECTIVE

**Status:** Enterprise-grade specification  
**Target:** AI Builder immediate execution  
**Ambiguity:** Minimal  
**Execution Determinism:** Maximum  

---

## OBJECTIVE

Build the TEOS Public API as the **sovereign governance layer** for the entire TEOS ecosystem.

This is NOT a simple Express backend.

The API becomes:
- ✅ The adjudication layer
- ✅ The trust boundary
- ✅ The telemetry governance layer
- ✅ The execution policy layer
- ✅ The integration surface for all products

**Architectural Law:** All downstream systems (Dashboard, CLI, GitHub Actions, VS Code, Slack, Bot, Browser Extension) MUST consume this API only. No internal bypass allowed.

---

## REQUIRED STACK (Non-negotiable)

```
✅ Node.js 20+
✅ Express 4.20+
✅ PostgreSQL 15+
✅ Redis 7+ (Upstash acceptable)
✅ JWT (jsonwebtoken)
✅ Docker + docker-compose
✅ OpenAPI 3.1
✅ Winston (structured logging)
✅ express-rate-limit
✅ helmet
✅ axios
```

No alternatives. No substitutions.

---

## MANDATORY ENDPOINTS (Executable Specification)

```javascript
// Health checks (MUST bypass rate limits)
GET    /health              → { status: 'ok' }
GET    /live                → { alive: true }

// Scan operations
POST   /api/v1/scan         → { scan_id, status }
GET    /api/v1/scans/:id    → { verdict, confidence, findings }
GET    /api/v1/reports/:id  → PDF binary

// User management
GET    /api/v1/user         → { tier, limits, usage }
POST   /api/v1/user/upgrade → { checkout_url }

// Webhooks
POST   /api/v1/webhooks     → { webhook_id }
POST   /webhooks/telegram   → { processed: true }

// Telemetry (internal only)
POST   /api/v1/telemetry    → { queued: true }

// Governance (internal only)
GET    /api/v1/governance/status → { policy_state }
POST   /api/v1/governance/policy → { updated: true }
```

**No additional endpoints permitted without governance approval.**

---

## ARCHITECTURAL RULES (Enforceable)

### Rule 1: API-First Enforcement

```
IF downstream_system accesses internal_database
  THEN FAIL build
  
IF downstream_system bypasses API layer
  THEN FAIL deployment

IF direct MCP calls exist outside API
  THEN FAIL architecture review
```

**Implementation:** All internal services must be behind API layer only.

### Rule 2: Dual-Engine Adjudication

```javascript
// Support both local MCP and remote sovereign engine
// Decision rule: STRICTEST VERDICT WINS

response = {
  verdict: 'BLOCK',           // final decision
  local_verdict: 'WARN',      
  remote_verdict: 'BLOCK',    
  confidence: 0.92,
  consensus: 0.85,
  engine_provenance: {
    local: { verdict: 'WARN', confidence: 0.8 },
    remote: { verdict: 'BLOCK', confidence: 0.95 }
  },
  triggering_rules: ['hardcoded_secret:critical'],
  policy_applied: 'production_strict'
}
```

**Constraint:** Response MUST include all fields. Never suppress engine disagreement.

### Rule 3: Epistemic Verdicts (Bounded States)

```javascript
const VERDICTS = {
  SAFE: 'no issues found',
  WARN: 'minor issues, monitor',
  BLOCK: 'critical, halt execution',
  UNKNOWN: 'insufficient data',
  LOW_CONFIDENCE: 'conflicting signals',
  CONFLICTED: 'engines disagree',
  INSUFFICIENT_CONTEXT: 'need more data'
}
```

**Constraint:** Only these 7 states allowed. No custom verdicts.

### Rule 4: Policy Layer (Machine-Readable)

```
src/governance/policies.json

{
  "environments": {
    "production": {
      "minimum_consensus": 0.75,
      "block_threshold": 0.85,
      "warn_threshold": 0.60,
      "escalation_on_conflict": true,
      "telemetry_enabled": true
    },
    "staging": {
      "minimum_consensus": 0.60,
      "block_threshold": 0.75,
      "warn_threshold": 0.50,
      "escalation_on_conflict": false,
      "telemetry_enabled": true
    },
    "development": {
      "minimum_consensus": 0.50,
      "block_threshold": 0.70,
      "warn_threshold": 0.40,
      "escalation_on_conflict": false,
      "telemetry_enabled": false
    }
  },
  "downgrade_rules": [
    { "from": "BLOCK", "to": "WARN", "condition": "consensus < 0.60" }
  ],
  "escalation_rules": [
    { "from": "WARN", "to": "BLOCK", "condition": "repeated_failure" }
  ]
}
```

**Constraint:** All policy changes MUST be:
- Documented in GOVERNANCE.md
- Versioned
- Audit-logged
- Reversible

---

## TELEMETRY GOVERNANCE (Bounded Queuing)

Create: `src/services/telemetry.service.js`

### Queue Constraints

```javascript
MAX_QUEUE_SIZE = 200          // hard limit
FLUSH_INTERVAL = 15_000       // 15 seconds
FLUSH_THRESHOLD = 20          // events

// Queue ONLY on async path
// NEVER block decision path
// Use background job, never critical path
```

### Allowed Telemetry (Whitelist Only)

```javascript
ALLOWED_TELEMETRY = [
  'verdict_metadata',     // verdict type only
  'latency_ms',          // duration
  'scan_duration_ms',    // scan time
  'runtime_metrics',     // memory, cpu
  'confidence_score',    // confidence
  'consensus_score',     // consensus
  'replay_attempts',     // count
  'auth_failures',       // count
  'rate_limit_hits',     // count
  'startup_event',       // timestamp
  'shutdown_event'       // timestamp
]
```

### Forbidden Telemetry (Hardcoded Block)

```javascript
FORBIDDEN_TELEMETRY = [
  'raw_code',
  'repository_content',
  'conversations',
  'secrets',
  'credentials',
  'pii',
  'git_history',
  'file_contents'
]

// FAIL build if any forbidden field in telemetry
function validateTelemetry(event) {
  for (let field of FORBIDDEN_TELEMETRY) {
    if (field in event) throw new Error(`SECURITY: ${field} in telemetry`)
  }
}
```

### Retention Policy (Immutable)

```javascript
retention = {
  'runtime': '30d',
  'verdicts': '90d',
  'security': '180d',
  'governance': '90d'
}

// Auto-delete after retention period
// Append-only audit log (never delete)
```

---

## SECURITY REQUIREMENTS (Non-negotiable)

Mandatory implementations:

```javascript
// src/middleware/security.js

✅ replay protection          // nonce + timestamp validation
✅ webhook signature validation // HMAC-SHA256
✅ JWT expiration enforcement // exp claim required
✅ API key hashing           // never store plaintext
✅ CSP headers               // helmet config
✅ request sanitization      // strip dangerous chars
✅ internal auth headers     // x-internal-token
✅ memory bounds middleware  // limit request size
✅ structured errors         // no stack traces
✅ request IDs               // uuid per request
✅ trace IDs                 // parent-child tracing
✅ idempotency keys          // x-idempotency-key
```

### Forbidden Exposure

```javascript
// FAIL deployment if response contains:
❌ stack traces
❌ secrets / credentials
❌ internal engine details
❌ database errors
❌ file paths
❌ user emails in errors
```

---

## HEALTHCHECK REQUIREMENTS (Kubernetes-Compatible)

```javascript
// BOTH endpoints MUST exist
// BOTH MUST bypass rate limits

app.get('/health', (req, res) => {
  // Docker healthcheck
  // Full dependency check (DB, cache, MCP)
  // Timeout: 3 seconds
  res.json({ status: 'healthy', checks: {...} })
})

app.get('/live', (req, res) => {
  // Kubernetes liveness probe
  // Process is alive check only
  // Timeout: 1 second
  res.json({ alive: true })
})
```

**Constraint:** Rate limiter MUST skip these endpoints.

---

## WEBHOOK GOVERNANCE (Production Standard)

Production deployment MUST use webhooks, NOT polling.

```env
# Required in production
BOT_USE_WEBHOOK=true
BOT_WEBHOOK_URL=https://api.teos.ai/webhooks/telegram
NODE_ENV=production

# If polling enabled in production
FAIL deployment
```

### Webhook Requirements

```javascript
✅ auto webhook registration     // on startup
✅ replay protection            // dedup events
✅ polling conflict prevention  // no double-processing
✅ prevent 409 conflicts        // Telegram rate limit
✅ webhook validation           // signature check
✅ event ordering               // timestamp-based
✅ retry logic                  // exponential backoff
```

### Implementation

```javascript
// src/services/webhook.service.js

class WebhookService {
  constructor() {
    this.isWebhookMode = process.env.BOT_USE_WEBHOOK === 'true'
    this.webhookUrl = process.env.BOT_WEBHOOK_URL
  }

  async registerWebhook() {
    if (!this.isWebhookMode) return
    // Register with Telegram, GitHub, etc.
  }

  async validateSignature(signature, payload) {
    const expected = createHmac('sha256', SECRET)
      .update(payload)
      .digest('hex')
    return signature === expected
  }
}
```

---

## GITHUB SCANNER CONSTRAINTS (Hard Limits)

```env
MAX_FILES=100
MAX_FILE_SIZE=200KB
MAX_TOTAL_SIZE=5MB
GITHUB_TIMEOUT=15s
MAX_DEPTH=3
```

### Scan Scope (Execution-Relevant Only)

```javascript
SCANNABLE = [
  '.js', '.ts', '.jsx', '.tsx',
  '.py', '.go', '.rs', '.java',
  'package.json', 'requirements.txt',
  'Cargo.toml', 'go.mod', 'pom.xml',
  'Dockerfile', '.github/workflows',
  '.env.example', 'config.js'
]

// Do NOT scan:
❌ node_modules
❌ .git
❌ dist, build
❌ vendor
❌ media files
❌ binary files
```

### What NOT to Build

```
❌ Full SAST platform (too broad)
❌ Recursive crawler (unbounded)
❌ CodeQL clone (enterprise tool)
❌ Package registry scanner (different layer)
❌ Secrets manager (separate product)
```

---

## PDF REPORT CONSTRAINTS (Bounded Output)

```javascript
MAX_REPORT_SIZE = 5 * 1024 * 1024  // 5MB
MAX_REASON_LINES = 20              // per finding
MAX_FINDINGS = 100                 // per report
```

### Restrictions

```javascript
❌ no remote URLs (no external calls)
❌ no raw HTML (no script injection)
❌ no script execution (pure rendering)
❌ no external resources (offline-safe)
✅ embedded images only (base64)
✅ bounded CSS only (inline)
✅ text + tables only (simple)
```

---

## DATABASE REQUIREMENTS (PostgreSQL)

Required tables (immutable schema):

```sql
-- Users and auth
CREATE TABLE users (
  id UUID PRIMARY KEY,
  tier VARCHAR(20),
  created_at TIMESTAMP NOT NULL,
  updated_at TIMESTAMP NOT NULL
);

CREATE TABLE api_keys (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  key_hash VARCHAR(255) NOT NULL,  -- never store plaintext
  last_used TIMESTAMP,
  created_at TIMESTAMP NOT NULL
);

-- Scans and verdicts
CREATE TABLE scans (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  type VARCHAR(20),
  target TEXT,
  verdict VARCHAR(20),
  confidence FLOAT,
  created_at TIMESTAMP NOT NULL
);

CREATE TABLE verdicts (
  id UUID PRIMARY KEY,
  scan_id UUID REFERENCES scans(id),
  engine VARCHAR(20),
  verdict VARCHAR(20),
  confidence FLOAT,
  created_at TIMESTAMP NOT NULL
);

-- Audit and governance
CREATE TABLE audit_events (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  event_type VARCHAR(50),
  payload JSONB,
  created_at TIMESTAMP NOT NULL
) WITH (OIDS = FALSE);  -- append-only

CREATE TABLE governance_events (
  id UUID PRIMARY KEY,
  policy_id VARCHAR(50),
  change JSONB,
  created_at TIMESTAMP NOT NULL
) WITH (OIDS = FALSE);  -- append-only

-- Telemetry (auto-delete by retention)
CREATE TABLE telemetry (
  id BIGSERIAL PRIMARY KEY,
  verdict_type VARCHAR(20),
  latency_ms INT,
  scan_duration_ms INT,
  runtime_metrics JSONB,
  created_at TIMESTAMP NOT NULL,
  expires_at TIMESTAMP NOT NULL
);

-- Webhooks
CREATE TABLE webhooks (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  url TEXT NOT NULL,
  events TEXT[],
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP NOT NULL
);

-- Usage tracking
CREATE TABLE usage_metrics (
  id UUID PRIMARY KEY,
  user_id UUID REFERENCES users(id),
  scans_today INT,
  scans_limit INT,
  created_at TIMESTAMP NOT NULL
);
```

**Constraint:** Audit and governance tables are APPEND-ONLY. No UPDATE or DELETE.

---

## CONFIDENCE DECAY SYSTEM (Anti-Staleness)

Create: `src/governance/confidence-decay.js`

Purpose: Prevent stale rules from becoming permanent institutional truth.

```javascript
function calculateDecayedConfidence(
  initialConfidence,
  falsePositiveRate,
  agreementFrequency,
  driftFactor,
  timeSinceCreation
) {
  const lambda = 0.01 // decay constant
  const timeDecay = Math.exp(-lambda * timeSinceCreation)
  const fpPenalty = Math.max(0, 1 - falsePositiveRate * 2)
  const aggreementBoost = agreementFrequency * 0.5

  return Math.max(0.1, initialConfidence * timeDecay * fpPenalty * aggreementBoost)
}

// Decay factors
const decayFactors = {
  falsePositives: count,      // penalizes wrong verdicts
  overrides: count,            // penalizes ignored verdicts
  disagreementFrequency: %,    // penalizes engine conflict
  timeInProduction: days       // penalizes staleness
}
```

**Constraint:** Confidence must be recalculated daily. Rules below 0.3 confidence are auto-disabled.

---

## REQUIRED FILES (Specification Artifacts)

```
src/
├── server.js                    # main entry
├── middleware/
│   ├── auth.js
│   ├── security.js
│   ├── rateLimit.js
│   └── telemetry.js
├── routes/
│   ├── scans.js
│   ├── users.js
│   ├── reports.js
│   ├── webhooks.js
│   └── governance.js
├── services/
│   ├── scan.service.js
│   ├── verdict.service.js
│   ├── telemetry.service.js
│   ├── webhook.service.js
│   └── policy.service.js
├── governance/
│   ├── policies.json           # machine-readable policy
│   ├── confidence-decay.js
│   └── audit.js
├── models/
│   ├── user.model.js
│   ├── scan.model.js
│   ├── verdict.model.js
│   └── telemetry.model.js
└── utils/
    ├── crypto.js
    ├── logger.js
    └── errors.js

config/
├── database.js
├── redis.js
└── jwt.js

docs/
├── OPENAPI.yaml               # executable spec
├── GOVERNANCE.md              # policy documentation
├── ARCHITECTURE.md
└── SECURITY.md

docker-compose.yml
.env.example
Dockerfile
package.json
```

---

## OPERATIONAL PRIORITIES (Ranked)

### MUST Have

```
1. Explainability
   └─ Every verdict MUST be traceable to rules

2. Governance Consistency
   └─ Policy MUST be machine-readable and enforceable

3. Bounded Telemetry
   └─ Never exceed queue limits or retention policy

4. Operational Resilience
   └─ Graceful degradation when MCP/remote unavailable

5. Deterministic Behavior
   └─ Same input = same verdict always
```

### SHOULD Have

```
1. High availability
2. Low latency (<100ms)
3. Comprehensive logging
4. Easy rollback
```

### MUST NOT Have

```
❌ Excessive features
❌ Uncontrolled integrations
❌ Premature expansion
❌ Custom verdicts outside the 7 states
❌ Telemetry collection outside whitelist
❌ Policy changes without governance approval
```

---

## BUILD PHASES (Explicit Ordering)

### Phase 1: Foundation (Week 1-2)
```
✅ Database schema
✅ Health endpoints
✅ Basic auth
✅ Verdict service
✅ Docker setup
```

### Phase 2: API (Week 3-4)
```
✅ Scan endpoint
✅ Report endpoint
✅ User endpoint
✅ Rate limiting
✅ OpenAPI docs
```

### Phase 3: Governance (Week 5)
```
✅ Policy service
✅ Telemetry service
✅ Audit logging
✅ Webhook service
✅ Governance endpoints
```

### Phase 4: Production (Week 6+)
```
✅ Testing & QA
✅ Security audit
✅ Performance optimization
✅ Documentation
✅ Deployment pipeline
```

---

## FINAL ARCHITECTURE PRINCIPLE

Build TEOS as:

```
┌──────────────────────────────────────┐
│   SOVEREIGN SECURITY GOVERNANCE      │
│   PLATFORM                           │
├──────────────────────────────────────┤
│ Not: A collection of security tools │
│ But: A unified governance system     │
└──────────────────────────────────────┘
```

The system MUST remain:

```
✅ Explainable      (every decision traced)
✅ Governable       (policy-driven)
✅ Composable       (API-first)
✅ Auditable        (append-only logs)
✅ Bounded          (resource limits)
✅ Trustworthy      (at scale)
```

---

## EXECUTION CHECKLIST (For AI Builder)

```
BEFORE BUILD:
☐ Read entire directive
☐ Understand dual-engine adjudication
☐ Verify telemetry constraints understood
☐ Confirm governance layer is NOT optional
☐ Accept that API-first is mandatory

DURING BUILD:
☐ Build database schema first
☐ Implement security layer immediately
☐ Add governance layer before APIs
☐ Test telemetry boundaries
☐ Verify policy engine works
☐ Validate webhook integration

BEFORE DEPLOYMENT:
☐ Run security audit
☐ Verify no forbidden telemetry
☐ Check all endpoints bypass health checks
☐ Validate policy engine
☐ Test dual-engine adjudication
☐ Performance test rate limiting

DEPLOYMENT:
☐ Enable webhook mode
☐ Load production policies
☐ Run health checks
☐ Monitor for 24h
☐ Verify no bypass architecture
```

---

**This is the complete specification. Everything above is executable. No ambiguity. Build to spec.**

**Status: READY FOR IMMEDIATE AI BUILDER EXECUTION**
