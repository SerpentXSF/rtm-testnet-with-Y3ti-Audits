# Y3ti Audits — Raptoreum SmartNode Package (testnet)

This bundle lets you run a **Y3ti Audits validator node** on the Raptoreum testnet. It pairs
the **official Raptoreum Core testnet wallet binaries** with the **Y3ti Audits node tooling**
(the `y3ti-agent` sidecar + hardened scan sandbox), so a single download gives you both a
Raptoreum node and everything needed to earn testnet rewards by validating security audits.

---

> **Current release:** Raptoreum Core package `2.0.4.01-Y3ti-Serpent-r14` + Y3ti Audits
> node agent **2.1.36** (`CODE_FP 2c1f676906a769cf`, sandbox `dbb4606d60028d87` — unchanged since
> 2.1.33, so **not a flag day**).
> Every value above was read out of the release artifacts themselves.
>
> r14 adds the two licence files inside the package and moves the agent's version string.
> Measured against the published 2.1.33 archive: `LICENSE` and `LICENSE-RAPTOREUM` are new,
> `README.md` and `agent/app/build_info.py` changed, nothing was removed, and the only
> non-comment line that differs in the agent is `AGENT_VERSION`.
>
> **No sandbox rebuild is needed to take it.** `agent/sandbox/` is byte-identical to its
> 2.1.33 counterpart — all 45 files, same paths, same bytes — so the source digest is
> still `dbb4606d60028d87`, and a fleet running any mix of 2.1.33 and 2.1.36 agrees under
> RF-3, which compares scan OUTPUT and not agent versions.
>
> ⛑ **Check your own node's `sandbox=`, not a release number.** If it reads
> `dbb4606d60028d87` you owe nothing. If it reads anything else, that node is scanning with
> an older toolchain and owes `./build-sandbox.sh` — 30 to 45 minutes, and it stops your
> agent for the duration and restarts it itself. This is written as a value rather than a
> version boundary on purpose: the value is what your node reports and cannot go stale.
> The authoritative list of versions and digests is the **Releases** page and the
> `README.md` inside the node package -- deliberately not duplicated further down this file,
> because a second copy is a copy that drifts.

> ⛔ **If you installed r10 — EITHER asset — replace it.** Every r10 download ships
> `http://auditor.y3ti.uk:11337`: plain `http` on a port that serves only `https`. **A node
> installed from r10 cannot complete registration and never comes online.** The fix landed
> in agent `2.1.33`, so r11 is the first release that has it.
>
> This applies to the standalone `y3ti-audits-testnet-node-v2.1.25` archive **as well as**
> the combined bundles — an earlier version of this warning said only the combined bundles
> were affected, which was wrong.
>
> The combined bundles additionally shipped agent `2.0.9` (`CODE_FP fa89775c85e20988`), not
> the `2.1.25` those release notes claimed — the 2.1.25 was only ever in the separate
> archive, so one release offered two different agent versions and which you got depended
> on which asset you clicked. Check what you actually have; the artifact cannot be wrong
> about itself:
>
> ```
> grep AGENT_VERSION y3ti/agent/app/build_info.py
> ```

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

The `2.0.4.01-Y3ti-Serpent-rN` name (the current release is **r14**) labels
**this distribution package** — it is **not** a Raptoreum
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

## Licensing

**This repository contains two separate bodies of work under two different licences.**

* **The Y3ti Audits software** — the node agent, the hardened scan sandbox and its scanner
  adapters, the control panel, the packaging scripts and the documentation (the `y3ti/`
  directory in the combined bundles, and the whole `y3ti-audits-testnet-node` package) — is
  **© 2026 SerpentXSF (Y3ti Audits), all rights reserved**. You may download, run and
  operate it to run a validator node, inspect and reproduce the build to verify what you are
  running, and modify your own copy locally to operate it. You **may** redistribute and
  mirror a **complete, unmodified release archive** -- notices intact, with that release's
  published checksum file (`SHA256SUMS.txt`) available to whoever you give it to, so they can
  check that what reached them is what was released. You may **not** redistribute a **modified** copy,
  sublicense, sell, or use it to build a competing service. Full terms:
  [`LICENSE`](LICENSE).
  **From r12 (agent 2.1.34) onward both licence files also ship inside the release
  archives**, which is the copy you actually keep.

* **Raptoreum Core** — the `raptoreumd`, `raptoreum-cli` and `raptoreum-tx` binaries and the
  `upstream/` source — is the official upstream release, redistributed **verbatim and
  unmodified**, and remains under the **MIT licence** of the Raptoreum, Dash and Bitcoin Core
  developers. Full text: [`LICENSE-RAPTOREUM`](LICENSE-RAPTOREUM). Nothing in our terms adds
  to or subtracts from your rights in that software.

Until 2026-11-01 this repository carried only upstream's MIT licence file, which GitHub
reported as the licence for the whole repository. That was never the intent for the Y3ti
code and is corrected here.

---

## Provenance / verification

The three native binaries are the official `2.0.4.01` release artifacts for this platform.
`checksums-upstream-2.0.4.01.txt` is the checksums file that shipped inside the corresponding official
release, retained so the binaries can be cross-checked against upstream. `checksums.txt` in this bundle
is regenerated over the full contents (binaries + tooling) with correctly labelled sha1 and sha256
blocks. Verify the downloaded archives with `sha256sum -c SHA256SUMS.txt`, using the checksum
file published with the release, or check extracted files against the `sha256:` block of the
`checksums.txt` inside the bundle. Those are two different files in two different formats that
unfortunately share a name on some releases: the release-level one is plain `sha256sum` lines,
the in-bundle one has labelled `sha1:` and `sha256:` blocks. Releases before r13 published the
release-level manifest as `checksums.txt`; if that is what you downloaded, rename it or pass it
to `sha256sum -c` directly.
