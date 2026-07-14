<p align="center">
  <img src="https://raw.githubusercontent.com/Judd1zzz/Judd1zzz/refs/heads/main/assets/header.jpg" alt="Header">
</p>

<p align="center">
  <a href="README.md">🇬🇧 English</a> | <a href="README_RU.md">🇷🇺 Русский</a>
</p>

<p align="center">
  <i>Polyglot engineer. I build systems designed to outlive their first million lines.</i>
</p>

---

I pick the language after the problem, not before. Nearly eight years that started in Python now span C#/.NET, Rust, Go, TypeScript, Swift and Lua — what stays constant is the architecture: strict module boundaries, contract-first APIs, domain-driven design. I take on almost everything except gamedev itself — though game *servers* are very much my thing.

## Now

- Building **my own multiplayer game platform** — a C# modular monolith already past hundreds of thousands of lines → [details](#the-nda-project)
- Running a **multi-agent AI development pipeline** we designed from scratch — agent fleets in parallel across the team → [how it works](#ai-driven-development)
- Shipping **[yandex-music-streamdeck](https://github.com/Judd1zzz/yandex-music-streamdeck)** — a full Rust rewrite, live on two marketplaces → [below](#featured-yandex-music-for-stream-deck)

## Featured: Yandex Music for Stream Deck

[![Latest release](https://img.shields.io/github/v/release/Judd1zzz/yandex-music-streamdeck?style=flat-square&color=blue)](https://github.com/Judd1zzz/yandex-music-streamdeck/releases) [![Downloads](https://img.shields.io/github/downloads/Judd1zzz/yandex-music-streamdeck/total?style=flat-square&color=green)](https://github.com/Judd1zzz/yandex-music-streamdeck/releases) [![Stars](https://img.shields.io/github/stars/Judd1zzz/yandex-music-streamdeck?style=flat-square&color=yellow)](https://github.com/Judd1zzz/yandex-music-streamdeck/stargazers) [![License](https://img.shields.io/github/license/Judd1zzz/yandex-music-streamdeck?style=flat-square)](https://github.com/Judd1zzz/yandex-music-streamdeck/blob/main/LICENSE)

<p align="left">
  <a href="https://marketplace.elgato.com/product/yandex-music-integration-43741c24-1784-4492-be32-c631d7c55829"><img src="https://docs.elgato.com/img/badges/get-it-on-marketplace--dark.svg" alt="Get it on Elgato Marketplace" height="40"></a>
  <a href="https://space.key123.vip/product/20260706002752"><img src="https://raw.githubusercontent.com/Judd1zzz/yandex-music-streamdeck/661d7a82d46f5e45a9cd6cce5e7c086f0c0a822d/assets/badges/get-it-on-streamdock--dark.svg" alt="Get it on StreamDock Store" height="40"></a>
</p>

Stream Deck / Stream Dock plugin for Yandex Music — a complete Rust port of my original Python version.

- **13 crates, hexagonal architecture** — tokio runtime, Chrome DevTools Protocol for player control, custom key rendering on tiny-skia
- **Distributed where users are**: [Elgato Marketplace](https://marketplace.elgato.com/product/yandex-music-integration-43741c24-1784-4492-be32-c631d7c55829), [Mirabox StreamDock Store](https://space.key123.vip/product/20260706002752) and [GitHub Releases](https://github.com/Judd1zzz/yandex-music-streamdeck/releases) — 500+ downloads
- **v2.0.0 “Rust Rewrite” → v2.3.0 in one week** (July 2026): six releases, the runtime cut down to a single binary

## The NDA project

> [!NOTE]
> This one is my own venture. The product ships under NDA, so there's no name and no code here; the architecture, though, is mine to design and mine to share.

A multiplayer game platform built as a **C# modular monolith** on .NET: every feature module is a bounded context with DDD layering, **CQRS with a mediator pipeline**, domain events, and a **transactional outbox**. **PostgreSQL** is the single writer of persistent state; **Redis** is the coordination fabric — streams for commands, pub/sub for notifications, locks and idempotency keys. Contracts come first: **OpenAPI, protobuf and AsyncAPI** definitions are the single source of truth, fanning out into generated clients for web (TypeScript), native iOS (SwiftUI) and internal Python services (ML anti-cheat, bots) over gRPC. Gameplay logic lives in a **sandboxed Lua scripting layer** behind a single **hand-written REST bridge** — the one deliberate exception to client codegen, because the game runtime speaks HTTP, not gRPC.

Today it spans **hundreds of thousands of lines** across a monorepo plus dedicated game-server and iOS/Android repositories on GitLab CI, and the roadmap points at **several million lines** with real infrastructure around it. The module boundaries are extraction-ready: hot paths — the exchange engine first — have a planned extraction path into **Rust services**, with seams clean enough that even the core could follow. A system this size is exactly why the development process itself had to be engineered (next section).

<details>
<summary><b>Architecture sketch (anonymized)</b></summary>

```mermaid
flowchart LR
    subgraph hub["Platform core — C# modular monolith (.NET)"]
        direction TB
        MOD["Feature modules = bounded contexts<br/>(DDD layers, CQRS + mediator)"]
        OUT["Domain events →<br/>transactional outbox"]
        MOD --> OUT
    end
    PG[("PostgreSQL —<br/>single writer of state")]
    RD[("Redis —<br/>streams · pub/sub · locks")]
    CON["Contracts — OpenAPI · protobuf · AsyncAPI<br/>(single source of truth, client codegen)"]
    subgraph game["Game layer — sandboxed Lua runtime"]
        direction TB
        BR["Hand-written REST bridge —<br/>the one codegen exception"]
        SRV["Server-side authority modules"]
        NUI["In-game UI (React/TS)"]
        BR --> SRV
        SRV --- NUI
    end
    WEB["Web (TypeScript)"]
    IOS["iOS (SwiftUI)"]
    SVC["Internal services (Python) —<br/>ML anti-cheat, bots"]
    RUST["Rust services —<br/>hot-path extraction (planned)"]
    DEV["Agentic dev pipeline —<br/>ADR-first → gates → MR flow,<br/>parallel agent fleets (git worktrees)"]
    hub --- PG
    OUT --> RD
    RD -->|HTTP relay| BR
    RD -->|WebSocket / SSE| WEB
    RD -->|APNs| IOS
    hub <-->|REST| BR
    hub <-->|gRPC| SVC
    CON === hub
    CON -->|generated client| WEB
    CON -->|generated client| IOS
    CON -->|generated client| SVC
    hub -.->|measured extraction| RUST
    DEV -.->|builds & reviews| hub
    DEV -.->|builds & reviews| game
```

</details>

## AI-driven development

The most interesting thing we built this year isn't a service — it's a process. Together with my project partner, we designed and built **from scratch** a multi-agent development workflow where AI agents operate like a disciplined engineering organization rather than autocomplete — several engineers, each with their own fleet of agents, building one codebase in parallel:

- a **devkit** that routes every significant change through an **ADR** (architecture decision record) before any code is written
- **brain-check gates** — an agent must demonstrate correct understanding of the task, the module contracts and the constraints before it is allowed to proceed
- **merge-request flow** — agent-written, gate-checked, human-audited changes only
- **task-scoped parallelism** — one task = one issue = one branch = one git worktree = one agent, claimed before any work starts
- **agent teams × human team** — every engineer drives their own agents across parallel terminal sessions; claim-first coordination keeps entire fleets from colliding, so the process scales across people, not just within one machine

The orchestration, the gates and the conventions are our own design — born from running a codebase that no longer fits in one head, human or agent.

## Off-GitHub track record

Years of commercial work live in private repositories:

- **Payment gateway** — FastAPI, async SQLAlchemy 2, HMAC-signed webhooks, idempotency keys; Yookassa and self-employment tax service integrations
- **Game servers** — custom server-side logic and frameworks for RageMP and FiveM
- **Commercial full-stack** — warehouse management system on Next.js + FastAPI
- **Native iOS** — SwiftUI applications
- **Go microservices** (including an API) and a fleet of Discord/Telegram bots

## Tech stack

| | |
|---|---|
| **Languages** | ![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white) ![C#](https://img.shields.io/badge/C%23-512BD4?style=flat-square&logo=dotnet&logoColor=white) ![Rust](https://img.shields.io/badge/Rust-CE412B?style=flat-square&logo=rust&logoColor=white) ![Go](https://img.shields.io/badge/Go-00ADD8?style=flat-square&logo=go&logoColor=white) ![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white) ![Swift](https://img.shields.io/badge/Swift-F05138?style=flat-square&logo=swift&logoColor=white) ![Lua](https://img.shields.io/badge/Lua-2C2D72?style=flat-square&logo=lua&logoColor=white) |
| **Backend & frameworks** | ![ASP.NET Core](https://img.shields.io/badge/ASP.NET_Core-512BD4?style=flat-square&logo=dotnet&logoColor=white) ![EF Core](https://img.shields.io/badge/EF_Core-512BD4?style=flat-square&logo=dotnet&logoColor=white) ![FastAPI](https://img.shields.io/badge/FastAPI-009485?style=flat-square&logo=fastapi&logoColor=white) ![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-D71F00?style=flat-square&logo=sqlalchemy&logoColor=white) ![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=nextdotjs&logoColor=white) ![React](https://img.shields.io/badge/React-20232a?style=flat-square&logo=react&logoColor=61DAFB) ![Vue.js](https://img.shields.io/badge/Vue.js-4FC08D?style=flat-square&logo=vuedotjs&logoColor=white) ![SwiftUI](https://img.shields.io/badge/SwiftUI-007AFF?style=flat-square&logo=swift&logoColor=white) |
| **Data & contracts** | ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat-square&logo=postgresql&logoColor=white) ![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=flat-square&logo=mysql&logoColor=white) ![Redis](https://img.shields.io/badge/Redis-DD0031?style=flat-square&logo=redis&logoColor=white) ![OpenAPI](https://img.shields.io/badge/OpenAPI-6BA539?style=flat-square&logo=openapiinitiative&logoColor=white) ![protobuf](https://img.shields.io/badge/protobuf-4285F4?style=flat-square) ![gRPC](https://img.shields.io/badge/gRPC-2CA5E0?style=flat-square) |
| **Infra & delivery** | ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white) ![GitLab CI](https://img.shields.io/badge/GitLab_CI-FC6D26?style=flat-square&logo=gitlab&logoColor=white) ![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white) ![nginx](https://img.shields.io/badge/nginx-009639?style=flat-square&logo=nginx&logoColor=white) ![OpenTelemetry](https://img.shields.io/badge/OpenTelemetry-000000?style=flat-square&logo=opentelemetry&logoColor=white) |

## Activity

<!--START_SECTION:waka-->
📊 **This Week I Spent My Time On** 

```text
💬 Programming Languages: 
Rust                     10 hrs 38 mins      ⣿⣿⣿⣿⣿⣿⣿⣿⣿⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀   37.83 % 
Markdown                 10 hrs 24 mins      ⣿⣿⣿⣿⣿⣿⣿⣿⣿⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀   36.98 % 
TypeScript               1 hr 24 mins        ⣿⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀   05.01 % 
Text                     1 hr 2 mins         ⣿⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀   03.72 % 
JavaScript               1 hr 1 min          ⣿⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀⣀   03.61 % 

💻 Operating System: 
Mac                      28 hrs 9 mins       ⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿⣿   100.00 % 
```


<!--END_SECTION:waka-->

## Contact

Discord is the one channel I actually read:

[![Discord](https://img.shields.io/badge/Discord-judd1-5865F2?style=flat-square&logo=discord&logoColor=white)](https://discord.com/users/311568906307502090)
