# Y3ti Audits — Raptoreum SmartNode Package (testnet)

This bundle lets you run a **Y3ti Audits validator node** on the Raptoreum testnet. It pairs
the **official Raptoreum Core testnet wallet binaries** with the **Y3ti Audits node tooling**
(the `y3ti-agent` sidecar + hardened scan sandbox), so a single download gives you both a
Raptoreum node and everything needed to earn testnet rewards by validating security audits.

---

## What is Y3ti Audits?

Y3ti Audits is a **decentralized, consensus-verified code-security network**. Developers submit
smart contracts, source code, or files; the central coordinator (the "hub") dispatches each job
to **three independent SmartNode validators** at random. Every validator runs the *same* scanners
in a *sealed, zero-network sandbox* and returns a SHA-256 hash of its findings. **Only when all
three hashes match byte-for-byte** is the result accepted, scored, and **anchored on the Raptoreum
blockchain** via an `OP_RETURN` transaction — a permanent, independently verifiable certificate.

Your node is one of those validators. By contributing spare CPU, you help produce audits that no
single party can forge, and you earn a share of every audit fee.

**What you earn.** Each audit fee is split **60% to the treasury / 40% to the validators**, and the
validators' 40% is divided among the nodes that agreed on the result. Rewards are paid automatically
by the coordinator to the **testnet RTM address** you configure — you never have to claim anything.
(This is a **testnet** release: rewards are testnet RTM, for testing and preparation ahead of
mainnet. Never use real funds.)

---

## About the version label

The `2.0.4.01-Y3ti-Serpent-r1` name labels **this distribution package** — it is **not** a Raptoreum
daemon version. The wallet binaries here are the **stock, unmodified official Raptoreum Core
`2.0.4.01` testnet release** (verbatim, bit-identical to upstream), which is the **latest upstream
testnet release**. No daemon-visible version/subversion string is changed, so this node speaks the
exact same testnet protocol as every other `2.0.4.01` node. **Rebuilding the node from this package
installs stock `2.0.4.01` — it does not upgrade or downgrade the Raptoreum daemon.**

---

## What's in this bundle

Flat layout (extract into a directory you create), matching the official release style:

```
raptoreumd                           # official Raptoreum Core full node/daemon (verbatim 2.0.4.01)
raptoreum-cli                        # official RPC client (verbatim)
raptoreum-qt                         # official GUI wallet (verbatim)
checksums.txt                        # sha1 + sha256 of every file in THIS bundle
checksums-upstream-2.0.4.01.txt      # upstream's original checksums (provenance)
README-Y3TI-SERPENT.md               # this file
y3ti/                                # the Y3ti Audits node tooling (identical in every OS bundle)
```

The `y3ti/` directory is **byte-identical** across the ubuntu22, ubuntu24, arm64, win, and source
archives — a hard requirement, because RF=3 consensus depends on every node building the *same* scan
sandbox from the *same* sources.

---

## What you can run

1. **A Raptoreum testnet wallet / full node** — use the native binaries directly
   (`raptoreumd` / `raptoreum-qt`) on any platform in the matrix. Testnet only.

2. **A Y3ti Audits validator node** (the point of this package) — run the tooling in `y3ti/`. This
   role needs a **Linux / amd64 host with Docker + Docker Compose v2**, because consensus determinism
   requires every validator to build the byte-identical `y3ti-sandbox` image. On Windows/arm64 the
   `y3ti/` payload is included as reference source — run the validator on a Linux/amd64 Docker host.

---

## System requirements (validator role)

| | |
|---|---|
| **OS / arch** | Linux, amd64 (x86-64) |
| **Software** | Docker Engine + Docker Compose v2 |
| **Disk** | ~6–8 GB (the scan sandbox image is large); +~5 GB if you also run the optional RTM node |
| **RAM** | ~4 GB (+~2 GB if running the optional RTM node) |
| **Network** | Outbound HTTPS to the coordinator; the scan sandbox itself runs with **no network** |

---

## What you need to participate

Before you start, get these three things:

1. **The coordinator URL** — the hub's address, e.g. `http://auditor.y3ti.uk:11337` (from whoever runs the hub).
2. **The node shared key** (`NODE_SHARED_KEY`) — a key the hub operator gives you; it must match the hub's.
3. **A Raptoreum testnet payout address** — where your rewards are sent. Either use an address from any
   testnet wallet, or mint one from the optional RTM node (step 4 below).

You will also pick a **unique node name** for yourself, and — once your node first connects — the **hub
operator approves your node** (a one-time step, explained under *Getting approved* below).

---

## Deploy (quick start)

```bash
cd y3ti/

# 1. Configure — set COORDINATOR_URL, NODE_SHARED_KEY, NODE_NAME (unique!), NODE_PAYOUT_ADDRESS
cp .env.example .env
nano .env

# 2. Build the scan sandbox image (once, and after any engine change)
./build-sandbox.sh

# 3. Start the validator (the agent begins polling the coordinator)
docker compose up -d --build

# 4. Watch it connect
docker logs -f y3ti-agent      # you should see it polling the coordinator's /next
```

That's the whole validator. The full configuration reference, the optional own-RTM-node profile
(`--profile rtmnode`), and the optional tray/menu-bar **Control Panel** (an On/Off toggle + settings
dashboard) are documented in **`y3ti/README.md`**.

### Optional: run your own RTM testnet node

Only needed if you want this machine to *be* its own Raptoreum node (e.g. to mint its own payout
address, or to register as an RTM SmartNode later). It is **not required** to validate audits — the
coordinator handles all payments; your node only needs a payout *address*.

```bash
cp config/raptoreum.env.example config/raptoreum.env   # set RTM_RPC_USER / RTM_RPC_PASSWORD
docker compose --profile rtmnode up -d --build
# once synced, mint an address for .env's NODE_PAYOUT_ADDRESS:
docker exec y3ti-raptoreum-testnet raptoreum-cli -datadir=/data -conf=/data/raptoreum.conf getnewaddress
```

---

## Getting approved (one-time)

Your node automatically generates its own **Ed25519 identity key** and cryptographically signs every
request to the hub (each request carries a fresh nonce, so signatures can't be replayed). You don't
configure this — but two things matter:

- **Approval:** on first connect, the hub registers your node's public key as **`pending`**. The hub
  operator approves it (Admin → **Node Keys** → your `NODE_NAME` → **Approve**). The hub currently runs
  in **dual-accept** mode, so your node already contributes via `NODE_SHARED_KEY` even before approval —
  approval is what lets it authenticate by its own key and is required before the hub ever switches to
  signature-enforced mode. Ask the hub operator to approve your node after it first appears.
- **Keep your identity key persistent.** It lives at `NODE_KEY_PATH` (default `/data/node_ed25519.key`,
  on the `y3ti_node_identity` Docker volume). If you lose it, your node mints a **new** identity that
  must be re-approved. The key also **auto-rotates** every ~30 days with a continuity proof the hub
  verifies automatically — no re-approval, no downtime.

---

## Confirm your node is healthy

- `docker logs -f y3ti-agent` shows it polling `/next` and, when assigned, running scans and submitting results.
- Your `NODE_NAME` appears in the hub's **Node Keys** panel (awaiting/after approval).
- When your node participates in a completed audit, its `NODE_PAYOUT_ADDRESS` receives the 40% reward split.
- Your node reports lightweight health telemetry (CPU / memory / response latency) to the hub's operator
  dashboard so the operator can monitor fleet health. Node locations are **not** exposed — the hub's map
  shows nodes at anonymized positions only.

---

## Notes & safety

- **Determinism:** every node must build `y3ti-sandbox:latest` from the same Dockerfile in this package,
  so scan outputs hash-match across validators. Keep it Linux/amd64; rebuild the sandbox after any update.
- **The scan sandbox is sealed:** untrusted code runs with `network=none`, read-only root filesystem,
  non-root user, all Linux capabilities dropped, `no-new-privileges`, and a PID cap.
- **Testnet only.** Never point this at mainnet or use real funds.
- **Secrets:** only `*.example` templates ship in this bundle. Copy them to their live names
  (`.env`, `config/secrets.env`, `config/raptoreum.env`) on your own host and fill them in. **Never
  share or redistribute the filled-in files** — they contain your keys and shared secret.

---

## Provenance / verification

The three native binaries are the official `2.0.4.01` release artifacts for this platform.
`checksums-upstream-2.0.4.01.txt` is the checksums file that shipped inside the corresponding official
release, retained so the binaries can be cross-checked against upstream. `checksums.txt` in this bundle
is regenerated over the full contents (binaries + tooling) with correctly labelled sha1 and sha256
blocks. Verify with `sha256sum -c SHA256SUMS.txt` (archives) or the `sha256:` block of `checksums.txt`
(extracted files).
