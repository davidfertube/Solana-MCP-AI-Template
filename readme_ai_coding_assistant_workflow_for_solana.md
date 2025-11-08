# AI Coding Assistant Workflow for Solana Development — Cursor 2.0 Edition

A single, opinionated **README.md** to run an AI‑assisted, reproducible Solana workflow in **Cursor 2.0** (Anchor or raw `solana-program`). Devnet‑first guardrails, Cursor Project Rules, and automated quality gates included.

---

## 🚀 Cursor 2.0 Quick Start
1. **Open the repo in Cursor 2.0.** Keep your thread short; restart often.
2. **Set Project Rules** (Settings → Rules → *Project*). Paste the rules from **“Cursor 2.0 Project Rules — Copy/Paste”** below.
3. **Add MCP servers** (Settings → Tools → MCP Servers): Solana MCP, File System MCP, Git MCP, and a Web Search MCP.
4. **Add Secrets / Env** (Settings → Secrets): `SOLANA_RPC_URL` (devnet), `SOLANA_COMMITMENT=confirmed`, `SOLANA_KEYPAIR_PATH`, optional `ANCHOR_WALLET`, `ANCHOR_PROVIDER_URL`.
5. **Toolchains**: Install Rust, Node 20+, Anchor (or use the provided **devcontainer** below).
6. **Bootstrap** your program (Anchor or raw), a TypeScript SDK, and tests. Use the **Initial Prompt** section.
7. **Run** `anchor build && anchor test` (or `cargo test` + TS tests) locally.
8. **Wire CI** via the provided GitHub Actions workflow.
9. **Iterate**: one task at a time; update `TASK.md`; keep diffs small.
10. **Guardrails**: devnet by default; require simulation success before any tx send; never expose secrets.

---

## ✨ Golden Rules
- Use markdown files to manage the project (`README.md`, `PLANNING.md`, `TASK.md`).
- Keep files under **500 lines**. Split into modules when needed.
- Start **fresh conversations** often. Long threads degrade response quality.
- Don’t overload the model. **One task per message** is ideal.
- **Test early, test often.** Every new function should have unit tests.
- Be specific in your requests. The more context, the better. Examples help a lot.
- Write docs and comments as you go. Don’t delay documentation.
- Implement environment variables yourself. **Never** paste secrets to the LLM.
- **Pin dependencies and tools**; use lockfiles and a devcontainer/Docker setup to keep builds reproducible.
- **Automate quality gates**—format, lint, type‑check, and unit tests must pass locally *and* in CI before merging.
- **Keep changes small and traceable**: commit in short increments referencing `TASK.md`, include brief rationale, and prefer diff/patch‑style edits with the AI.

---

## 🗺️ Planning & Task Management
**`PLANNING.md`**: vision, architecture, constraints, tech stack, tools, risks. Prompt: “Use the structure and decisions outlined in `PLANNING.md`.”

**`TASK.md`**: active tasks, backlog, milestones, “Discovered During Work”. Prompt: “Update `TASK.md` to mark XYZ as done and add ABC as a new task.”

`README.md` explains what the project does, who it’s for, outcomes, and how to run it.

---

## 🧠 Cursor 2.0 Project Rules — Copy/Paste
**Paste the following into Cursor → Settings → Rules → Project.**

### 🔄 Project Awareness & Context
- **Always read `PLANNING.md`** at the start of a new conversation to understand architecture, goals, style, and constraints.
- **Check `TASK.md`** before starting a task. If the task isn’t listed, add it with a brief description and today’s date.
- **Follow naming conventions, file structure, and patterns** defined in `PLANNING.md`.

### 🧱 Code Structure & Modularity
- **Never create a file > 500 LOC.** Split into modules when approaching the limit.
- Group code by **feature/responsibility** (e.g., `programs/`, `sdk/`, `web/`).
- Prefer **clear, stable imports**; keep public APIs small and documented.

### 🧪 Testing & Reliability (Solana)
- **Rust**: `cargo test`, `solana-program-test`, or `anchor test`.
- **TS client**: `vitest`/`jest` for client helpers and integration scripts.
- For each function/IX, include **1 success, 1 edge, 1 failure**.
- After logic updates, **update tests** accordingly.

### ✅ Task Completion
- **Mark completed tasks in `TASK.md` immediately.**
- Log new TODOs under **Discovered During Work**.

### 📎 Style & Conventions
- **Languages**: Rust (on‑chain), TypeScript (SDK/tests), optional minimal UI (React/Next) for demos.
- **Formatting**: `rustfmt` + `clippy -D warnings`; TS `eslint` + `prettier` + `tsc --noEmit`.
- **On‑chain**: no `panic!`; typed errors; checked math; minimal serialization; document PDAs/seeds/bumps and access control.
- **Docs**: Rust `///` for public items; JSDoc in TS; inline `# Reason:` comments for non‑obvious logic.

### 🧠 AI Behavior
- **Never assume missing context**—ask one targeted question only when necessary.
- **Never hallucinate crates/functions**—verify on official Solana/Anchor docs or `docs.rs`.
- Confirm **file paths and module names** exist before referencing.
- **Devnet by default.** Require simulation success before sending any transaction.
- **Never expose secrets**; do not print keypair material.
- **Do not delete/overwrite code** unless explicitly instructed or recorded in `TASK.md`.

---

## 🧰 Configuring **Solana MCP** (for Cursor 2.0)
**Solana MCP** lets Cursor interact with Solana safely: read on‑chain state, simulate/send **devnet** txs, inspect logs, and automate Anchor workflows.

**Add these MCP servers in Cursor → Settings → Tools → MCP Servers**
- **Solana MCP** — chain state, simulations, devnet sends, Anchor helpers.
- **Web Search MCP** — fetch official docs/examples.
- **File System MCP** — read/write, refactor, multi‑file edits.
- **Git MCP** — branching, diffing, committing.

**Environment** (set via Cursor Secrets or MCP env manager)
- `SOLANA_RPC_URL` = `https://api.devnet.solana.com`
- `SOLANA_COMMITMENT` = `confirmed`
- `SOLANA_KEYPAIR_PATH` = local path to your devnet keypair
- Optional: `ANCHOR_WALLET`, `ANCHOR_PROVIDER_URL`

**Suggested tools**
- Read‑only: `get_balance`, `get_account_info`, `get_program_accounts`, `get_signatures_for_address`, `get_latest_blockhash`, `get_slot`
- Sim & devnet‑only mutation: `simulate_tx`, `send_tx` (require prior simulation pass), `request_airdrop`
- Dev ergonomics: `decode_logs`, `fetch_idl`, `anchor_build`, `anchor_test`, `anchor_deploy_devnet`, `token_accounts_by_owner`

**Guardrails**: devnet default; require simulation before send; block mainnet writes unless `ALLOW_MAINNET=true`; never return secrets.

**Docker (Solana MCP)**
```dockerfile
FROM python:3.12-slim
WORKDIR /app
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt
COPY . .
CMD ["python", "server.py"]
```
Build: `docker build -t mcp/solana .`

Run (devnet):
```bash
docker run --rm -it \
  -e SOLANA_RPC_URL=https://api.devnet.solana.com \
  -e SOLANA_COMMITMENT=confirmed \
  -e SOLANA_KEYPAIR_PATH=/keys/devnet.json \
  -v $HOME/.config/solana:/keys:ro \
  mcp/solana
```

---

## 💬 Initial Prompt to Start a Solana Project
> “Create an Anchor program named `vault` that lets a user deposit SOL into a PDA and withdraw later with a 0.3% fee to a treasury. Provide: (1) on‑chain Rust with account constraints and checked math; (2) an IDL; (3) a TypeScript client SDK in `/sdk` with functions `initVault`, `deposit`, `withdraw`; (4) `anchor test` integration tests (success, edge, failure); (5) CLI scripts in `/scripts`; (6) README sections with setup/run. Use **devnet** by default, and require a successful simulation before sending txs. Follow repo conventions in `PLANNING.md`.”

(If not using Anchor, replace with `solana-program-test` + `@solana/web3.js`.)

---

## 🧩 Modular Prompting Process (after Initial Prompt)
- Request **one focused change** at a time.
- Only update the **minimal set of files**.
- After changes, update **`README.md`**, **`PLANNING.md`**, **`TASK.md`**.

---

## ✅ Test After Every Feature
- **Rust**: `anchor test` or `solana-program-test` with happy path, edge (0 lamports/max u64), and failures (wrong signer, PDA seeds, unauthorized CPI). Validate events, balances, logs.
- **TypeScript**: unit test client helpers; integration tests may target local validator or devnet with ephemeral accounts.
- Mirror structure in `/tests`; gate expensive tests via `--ignored` or `CI_SOLANA`.

---

## 🧪 CI: Quality Gates (GitHub Actions sample)
```yaml
name: ci
on: [push, pull_request]
jobs:
  lint-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 20 }
      - uses: dtolnay/rust-toolchain@stable
      - run: rustup component add clippy rustfmt
      - run: npm ci --workspace sdk --if-present
      - run: cargo fmt --all -- --check
      - run: cargo clippy --all-targets -- -D warnings
      - run: cargo test --all --locked
      - run: npm -w sdk run lint --if-present
      - run: npm -w sdk run test --if-present
```

---

## 🔐 Security & Keys
- **Never paste keys or secrets** to the LLM.
- Default to **devnet**; require **simulation pass** before any send.
- Document PDAs, seeds, signers, access control in code + README.
- Use typed errors; avoid `panic!`; prefer explicit early returns.

---

## 🔧 Common Commands & Scripts
```bash
# Devnet config
solana config set --url https://api.devnet.solana.com
solana-keygen new --outfile ~/.config/solana/devnet.json
solana airdrop 2 $(solana address)

# Anchor flow
anchor build
anchor test
anchor deploy

# TypeScript SDK
npm -w sdk ci
npm -w sdk run build
npm -w sdk test

# Raw program test
cargo test -p my-program -- --nocapture
```

---

## 📁 Suggested Repo Layout
```
.
├─ programs/
│  └─ <program-name>/
│     ├─ Cargo.toml
│     └─ src/
├─ sdk/
│  ├─ src/
│  └─ package.json
├─ scripts/
├─ tests/
├─ PLANNING.md
├─ TASK.md
└─ README.md
```

---

## 🧱 Devcontainer (optional, recommended)
```json
{
  "name": "solana-dev",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "features": {
    "ghcr.io/devcontainers/features/node:1": { "version": "20" },
    "ghcr.io/devcontainers/features/rust:1": {},
    "ghcr.io/devcontainers/features/common-utils:2": { "installZsh": true }
  },
  "postCreateCommand": "npm -g i pnpm && rustup component add clippy rustfmt"
}
```

---

## ✅ Example Day‑1 Task Flow
1. Create `PLANNING.md` with goals, constraints, module layout, and dependencies.
2. Bootstrap skeleton (Anchor or raw program + SDK + tests).
3. Commit baseline; open a new short Cursor thread.
4. Implement a single instruction; add tests; run `anchor test` or `cargo test`.
5. Update `TASK.md` (done + discovered). Push PR. CI must pass.
6. If the thread drifts, **restart** and re‑read `PLANNING.md` and `TASK.md`.

---

## 📝 License
MIT (or your choice). Keep this README versioned; evolve rules with the codebase.

