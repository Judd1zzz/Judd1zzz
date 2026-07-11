<p align="center">
  <img src="https://raw.githubusercontent.com/Judd1zzz/Judd1zzz/refs/heads/main/assets/header.jpg" alt="Шапка">
</p>

<p align="center">
  <a href="README.md">🇬🇧 English</a> | <a href="README_RU.md">🇷🇺 Русский</a>
</p>

<p align="center">
  <i>Инженер-полиглот. Строю системы, рассчитанные пережить свой первый миллион строк кода.</i>
</p>

---

Я выбираю язык под задачу, а не наоборот. Почти восемь лет, начавшихся с Python, теперь охватывают C#/.NET, Rust, Go, TypeScript, Swift и Lua — неизменной остаётся архитектура: жёсткие границы модулей, contract-first API, domain-driven design. Берусь почти за всё, кроме собственно геймдева — впрочем, игровые *серверы* как раз моя стихия.

## Сейчас

- Строю **собственную multiplayer-платформу** — модульный монолит на C#, уже перешагнувший сотни тысяч строк → [подробнее](#проект-под-nda)
- Развиваю **мультиагентный AI-пайплайн разработки**, который мы спроектировали с нуля, — параллельная работа групп агентов в рамках всей команды → [как это работает](#ai-driven-разработка)
- Выпускаю **[yandex-music-streamdeck](https://github.com/Judd1zzz/yandex-music-streamdeck)** — полный Rust-порт, доступен на двух маркетплейсах → [ниже](#featured-yandex-music-для-stream-deck)

## Featured: Yandex Music для Stream Deck

[![Последний релиз](https://img.shields.io/github/v/release/Judd1zzz/yandex-music-streamdeck?style=flat-square&color=blue)](https://github.com/Judd1zzz/yandex-music-streamdeck/releases) [![Загрузки](https://img.shields.io/github/downloads/Judd1zzz/yandex-music-streamdeck/total?style=flat-square&color=green)](https://github.com/Judd1zzz/yandex-music-streamdeck/releases) [![Звёзды](https://img.shields.io/github/stars/Judd1zzz/yandex-music-streamdeck?style=flat-square&color=yellow)](https://github.com/Judd1zzz/yandex-music-streamdeck/stargazers) [![Лицензия](https://img.shields.io/github/license/Judd1zzz/yandex-music-streamdeck?style=flat-square)](https://github.com/Judd1zzz/yandex-music-streamdeck/blob/main/LICENSE)

<p align="left">
  <a href="https://marketplace.elgato.com/product/yandex-music-integration-43741c24-1784-4492-be32-c631d7c55829"><img src="https://docs.elgato.com/img/badges/get-it-on-marketplace--dark.svg" alt="Get it on Elgato Marketplace" height="40"></a>
  <a href="https://space.key123.vip/product/20260706002752"><img src="https://raw.githubusercontent.com/Judd1zzz/yandex-music-streamdeck/661d7a82d46f5e45a9cd6cce5e7c086f0c0a822d/assets/badges/get-it-on-streamdock--dark.svg" alt="Get it on StreamDock Store" height="40"></a>
</p>

Плагин Stream Deck / Stream Dock для Яндекс Музыки — полный порт моей исходной Python-версии на Rust.

- **13 крейтов, гексагональная архитектура** — рантайм tokio, управление плеером через Chrome DevTools Protocol, собственный рендер клавиш на tiny-skia
- **Дистрибуция там, где пользователи**: [Elgato Marketplace](https://marketplace.elgato.com/product/yandex-music-integration-43741c24-1784-4492-be32-c631d7c55829), [StreamDock Store (Mirabox)](https://space.key123.vip/product/20260706002752) и [GitHub Releases](https://github.com/Judd1zzz/yandex-music-streamdeck/releases) — 500+ загрузок
- **v2.0.0 «Rust Rewrite» → v2.3.0 за одну неделю** (июль 2026): шесть релизов, рантайм ужат до единственного бинарника

## Проект под NDA

> [!NOTE]
> Это мой собственный проект. Продукт живёт под NDA, поэтому здесь нет ни названия, ни кода; но архитектура — мой дизайн, и делиться ею — моё решение как владельца.

Multiplayer-платформа, построенная как **модульный монолит на C#** (.NET): каждый функциональный модуль — bounded context с DDD-слоями, **CQRS через mediator-пайплайн**, доменными событиями и **transactional outbox**. **PostgreSQL** — единственный писатель персистентного состояния; **Redis** — координационная ткань: стримы для команд, pub/sub для уведомлений, локи и ключи идемпотентности. Контракты первичны: определения **OpenAPI, protobuf и AsyncAPI** — единственный источник правды, из которого кодогенерируются клиенты для веба (TypeScript), нативного iOS (SwiftUI) и внутренних Python-сервисов (ML-античит, боты) поверх gRPC. Игровая логика живёт в **изолированном Lua-слое** за единственным **рукописным REST-мостом** — осознанным исключением из кодогенерации: игровой рантайм говорит по HTTP, а не gRPC.

Сегодня это **сотни тысяч строк** в монорепо плюс отдельные репозитории игрового сервера и iOS/Android на GitLab CI, а road map ведёт к **нескольким миллионам строк** с настоящей инфраструктурой вокруг. Границы модулей готовы к извлечению: у "горячих" путей — первым движок биржи — запланирован путь выноса в **Rust-сервисы**, а швы настолько чистые, что за ними может последовать и само ядро. Система такого размера — ровно та причина, по которой сам процесс разработки пришлось проектировать как инженерную систему (следующая секция).

<details>
<summary><b>Архитектурный набросок (обезличенный)</b></summary>

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

## AI-driven разработка

Самое интересное, что мы построили в этом году, — не сервис, а процесс. Вместе с партнёром по проекту мы спроектировали и собрали **с нуля** мультиагентный workflow разработки, в котором AI-агенты работают как дисциплинированная и слаженная инженерная команда, а не как автодополнение, — несколько разработчиков, каждый со своей пачкой агентов, параллельно строят одну кодовую базу:

- **devkit**, который проводит каждое значимое изменение через **ADR** (architecture decision record) до того, как будет написана первая строка кода
- **brain-check-гейты** — агент обязан продемонстрировать верное понимание задачи, контрактов модулей и ограничений, прежде чем ему позволено продолжать
- **MR-флоу** — принимаются только изменения, написанные агентом, прошедшие гейты и проверенные человеком
- **задаче-центричный параллелизм** — одна задача = один issue = одна ветка = один git worktree = один агент, и задача закрепляется до начала работы
- **агентные команды × команда людей** — каждый разработчик ведёт своих агентов в параллельных терминальных сессиях; claim-first-координация не даёт целым группам сталкиваться, поэтому процесс масштабируется на людей, а не только внутри одной машины

Оркестрация, гейты и конвенции — наш собственный дизайн, рождённый из работы с кодовой базой, которая уже не помещается в одну голову — ни человеческую, ни агентную.

## Результаты работы вне GitHub

Годы коммерческой разработки живут в приватных репозиториях:

- **Платёжный шлюз** — FastAPI, асинхронный SQLAlchemy 2, HMAC-подписанные вебхуки, ключи идемпотентности; интеграции ЮKassa и «Мой налог»
- **Игровые серверы** — кастомная серверная логика и фреймворки для RageMP и FiveM
- **Коммерческий фулстек** — система управления складом на Next.js + FastAPI
- **Нативный iOS** — приложения на SwiftUI
- **Микросервисы на Go** (включая API) и огромная пачка Discord/Telegram-ботов

## Стек

| | |
|---|---|
| **Языки** | ![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white) ![C#](https://img.shields.io/badge/C%23-512BD4?style=flat-square&logo=dotnet&logoColor=white) ![Rust](https://img.shields.io/badge/Rust-CE412B?style=flat-square&logo=rust&logoColor=white) ![Go](https://img.shields.io/badge/Go-00ADD8?style=flat-square&logo=go&logoColor=white) ![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white) ![Swift](https://img.shields.io/badge/Swift-F05138?style=flat-square&logo=swift&logoColor=white) ![Lua](https://img.shields.io/badge/Lua-2C2D72?style=flat-square&logo=lua&logoColor=white) |
| **Бэкенд и фреймворки** | ![ASP.NET Core](https://img.shields.io/badge/ASP.NET_Core-512BD4?style=flat-square&logo=dotnet&logoColor=white) ![EF Core](https://img.shields.io/badge/EF_Core-512BD4?style=flat-square&logo=dotnet&logoColor=white) ![FastAPI](https://img.shields.io/badge/FastAPI-009485?style=flat-square&logo=fastapi&logoColor=white) ![SQLAlchemy](https://img.shields.io/badge/SQLAlchemy-D71F00?style=flat-square&logo=sqlalchemy&logoColor=white) ![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=nextdotjs&logoColor=white) ![React](https://img.shields.io/badge/React-20232a?style=flat-square&logo=react&logoColor=61DAFB) ![Vue.js](https://img.shields.io/badge/Vue.js-4FC08D?style=flat-square&logo=vuedotjs&logoColor=white) ![SwiftUI](https://img.shields.io/badge/SwiftUI-007AFF?style=flat-square&logo=swift&logoColor=white) |
| **Данные и контракты** | ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-316192?style=flat-square&logo=postgresql&logoColor=white) ![MySQL](https://img.shields.io/badge/MySQL-4479A1?style=flat-square&logo=mysql&logoColor=white) ![Redis](https://img.shields.io/badge/Redis-DD0031?style=flat-square&logo=redis&logoColor=white) ![OpenAPI](https://img.shields.io/badge/OpenAPI-6BA539?style=flat-square&logo=openapiinitiative&logoColor=white) ![protobuf](https://img.shields.io/badge/protobuf-4285F4?style=flat-square) ![gRPC](https://img.shields.io/badge/gRPC-2CA5E0?style=flat-square) |
| **Инфраструктура и доставка** | ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white) ![GitLab CI](https://img.shields.io/badge/GitLab_CI-FC6D26?style=flat-square&logo=gitlab&logoColor=white) ![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white) ![nginx](https://img.shields.io/badge/nginx-009639?style=flat-square&logo=nginx&logoColor=white) ![OpenTelemetry](https://img.shields.io/badge/OpenTelemetry-000000?style=flat-square&logo=opentelemetry&logoColor=white) |

## Активность

> Автообновляемая статистика языков за неделю (WakaTime) — в [английской версии](README.md#activity).

## Связь

Discord — единственный канал, который я действительно читаю:

[![Discord](https://img.shields.io/badge/Discord-judd1-5865F2?style=flat-square&logo=discord&logoColor=white)](https://discord.com/users/311568906307502090)
