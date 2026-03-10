<div align="center">
  <img src="os384-logo.svg" width="108" height="108" alt="os384" />

  <h1>os384</h1>

  <p><strong>Sovereign computing for the open web.</strong></p>

  <p>
    <a href="https://384.dev">384.dev</a> &nbsp;·&nbsp;
    <a href="https://github.com/os384/docs">Docs</a> &nbsp;·&nbsp;
    <a href="https://github.com/os384/lib384">lib384</a>
  </p>

  <p>
    <img alt="License: GPL-3.0" src="https://img.shields.io/badge/license-GPL--3.0-orange" />
    <img alt="Built on P-384" src="https://img.shields.io/badge/crypto-P--384-blue" />
    <img alt="Deno" src="https://img.shields.io/badge/runtime-Deno-black" />
  </p>
</div>

---

## What is os384?

os384 is a browser-native platform for building **private, encrypted, user-sovereign applications** — without accounts, without centralized servers, and without platform lock-in.

The name comes from **P-384**, the NIST elliptic curve at the heart of the design. Every key, every channel, every identity in the system is grounded in P-384 cryptography.

The web was supposed to be a peer-to-peer system. Instead, it became a handful of platforms that hold everyone's data, read their messages, and can revoke access at will. os384 is a technical answer to that problem: a small, auditable, self-hostable stack where the platform layer runs in the user's browser and servers are reduced to dumb encrypted storage.

---

## Two primitives, one platform

Everything in os384 is built on exactly two abstractions:

**Shards** are immutable, padded, encrypted, content-addressed blobs. A shard is identified by the hash of its content. The encryption key is derived from the content itself and stored separately — never alongside the shard. Shards can be stored on any server; the hash verifies integrity without trusting the server.

**Channels** are end-to-end encrypted communication endpoints, each identified by a P-384 public key. The owner of the corresponding private key controls the channel absolutely. Channels carry messages, but they also serve as the accounting primitive for storage budget — a channel is how the system tracks how much storage a user has paid for.

From these two primitives, os384 builds wallets, filesystems, applications, and a complete user identity model — all without a central authority.

---

## How it works

```
  You type: 384.dev/#xK9mR2...
                │
                │  (the #fragment never leaves your browser)
                ▼
         ┌─────────────┐
         │   Loader    │  resolves the channel page,
         │  (384.dev)  │  launches your app in a random subdomain
         └──────┬──────┘
                │ lib384 runtime
                ▼
         ┌─────────────┐
         │  Your App   │  runs in an isolated origin
         │  (browser)  │  keys never leave your device
         └──────┬──────┘
                │ encrypted reads/writes only
       ┌────────┴────────┐
       ▼                 ▼
  Channel Server    Storage Server
  (messages,        (shards,
   state, budget)    encrypted blobs)
```

**The loader** is a browser microkernel. It reads a channel key from the URL fragment — which is never transmitted to any server — and uses it to fetch and launch an application from that channel's page. Apps run in randomly-generated subdomains for strict origin isolation. No app can read another app's storage or intercept another app's keys.

**Identity** is derived entirely on the user's device from a passphrase and a strongpin. There is no registration, no email address, no password sent to any server. If you know your passphrase, you have your keys. The keys never leave your browser.

**The backend** is intentionally minimal. The channel server manages encrypted message routing and channel state. The storage server manages encrypted blob storage. Together they are fewer than 3,000 lines of TypeScript, deployable to Cloudflare Workers or any Docker host. They see only ciphertext — they cannot read your data.

---

## Repositories

### Core

**[lib384](https://github.com/os384/lib384)** — The TypeScript runtime library. This is the foundation everything else builds on. Implements the full cryptographic protocol: shard read/write, channel send/receive, wallet/identity management, SBFS (a virtual filesystem over shards via service worker), and browser helpers. Builds to an ESM bundle and an IIFE bundle (`window.__`). Distributed natively via os384 channel pages. Written for Deno, runs in any browser.

**[loader](https://github.com/os384/loader)** — The browser microkernel at [384.dev](https://384.dev). A Cloudflare Worker backed by a Vite/TypeScript frontend. Reads the URL fragment, resolves the channel page, enforces app isolation via random subdomains, and manages the user's wallet. The user's entry point to the entire system.

**[services](https://github.com/os384/services)** — The server-side infrastructure. Two Cloudflare Workers:
- `channel/` — channel server (messaging, durable objects, storage budget accounting)
- `storage/` — storage server (shard store, KV + durable objects, token validation)

A complete, auditable backend. The channel server is ~2,500 lines of TypeScript; the storage server is under 500 lines.

### Tools

**[cli](https://github.com/os384/cli)** — The `384` command-line tool. Manages channels, uploads and downloads shards, deploys lib384 artifacts to channel pages, and bootstraps wallets. Written in Deno with Cliffy for argument parsing. Install with one `deno install` command; no npm, no package manager.

**[paywall](https://github.com/os384/paywall)** — A small Cloudflare Worker that lets storage server operators sell storage tokens directly to users. Accepts Bitcoin, Ethereum, Litecoin, and PayPal. Operators pre-generate signed tokens offline with the CLI and load them into a pool; the paywall dispenses them on payment confirmation. No accounts, no token generation at runtime — the token is the receipt.

**[mirror](https://github.com/os384/mirror)** — A lightweight Python shard cache. Proxies shard requests to upstream storage servers and caches locally. Zero dependencies beyond the Python standard library. Useful for local development and self-hosted deployments.

### Applications

**[file-manager](https://github.com/os384/file-manager)** — An encrypted file manager app built with Lit web components and Vite. Upload, browse, and preview encrypted file sets stored on os384 channels. A reference implementation of a "privileged" os384 app — one that ships with the platform rather than living purely on channel pages.

**[demos](https://github.com/os384/demos)** — Eight curated demo apps illustrating lib384 usage: basic IIFE smoke test, strongpin key derivation, file encryption, wallet bootstrap, app loading, channel streaming, and user signup. Each is a standalone HTML page. The fastest way to see what the API looks like in practice.

### Documentation

**[docs](https://github.com/os384/docs)** — The documentation site, built with VitePress. Architecture, protocol reference, CLI guide, glossary, and essays on sovereign computing. Also serves as context for AI coding assistants working on the codebase.

---

## Self-hostable in under five minutes

A complete os384 backend is two commands:

```sh
git clone https://github.com/os384/services && cd services

# Terminal 1 — storage server (port 3843)
cp storage/wrangler.template.toml storage/wrangler.toml
wrangler dev --local --config storage/wrangler.toml

# Terminal 2 — channel server (port 3845)
ln -s ../storage/.wrangler channel/.wrangler
cp channel/wrangler.template.toml channel/wrangler.toml
wrangler dev --local --config channel/wrangler.toml
```

Or bring up the full stack with Docker / [OrbStack](https://orbstack.dev/):

```sh
git clone https://github.com/os384/os384
cd os384 && docker compose up
```

Ports: `3840` loader · `3841` mirror · `3843` storage · `3845` channel

---

## Install the CLI

Requires [Deno](https://deno.com):

```sh
deno install --global -n 384 \
  --allow-read --allow-write --allow-net --allow-env \
  https://os384.land/cli/384.ts
```

---

## License & community

Everything is **GPL-3.0**. os384 is a community project — 384, Inc. has open-sourced all of os384 under GPL v3. If you run a server, build an app, or contribute code, you're part of the ecosystem.

The design goal for the server side is an ecosystem of independent operators — developers, enthusiasts, small teams — running their own storage and channel servers and accepting payment directly from users, with no platform taking a cut. The paywall repo is the first step toward making that simple.

---

<div align="center">
  <sub>GPL-3.0 · P-384 cryptography · No accounts · No ads · No lock-in · Self-hostable</sub>
</div>
