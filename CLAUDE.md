# CLAUDE.md

Živý dokument. Průběžně aktualizuj, udržuj krátký a přehledný.

## Projekt
- Název: Rohlíček (rohliczech-web)
- Popis: PWA na recepty a sdílený nákupní seznam (košík). Jednosouborová appka — vše v `index.html`.
- Cílová skupina: **osobní appka pro mě (a partnerku)** — žádní externí uživatelé, žádní klienti.
- Jazyk UI: čeština

## Uživatel
- Skill level: <!-- TODO: začátečník / pokročilý / expert -->
  <!-- TODO: pokud začátečník → "Neptej se na technické detaily. Rozhoduj sám, vysvětluj jednoduše." -->
  <!-- TODO: pokud pokročilý/expert → "Můžeš používat technický žargon, navrhuj alternativy." -->

## Stack
<!-- TODO: doplň technologie použité v projektu. Příklad: -->
- Framework: <!-- TODO: např. Next.js 16 (App Router) + TypeScript + React 19 -->
- UI: <!-- TODO: např. shadcn/ui + Tailwind CSS v4 + název fontu -->
- Databáze: <!-- TODO: např. PostgreSQL 17 (Docker) + Prisma ORM 7 -->
- Auth: <!-- TODO: např. Auth.js v5, Clerk, Supabase Auth... -->
- Monitoring: <!-- TODO: např. Sentry.io, LogRocket... (nebo smaž pokud není) -->
- Hosting: <!-- TODO: např. Railway, Vercel, AWS... -->
- Kontejnerizace: <!-- TODO: např. Docker (docker-compose.yml = vstupní bod) nebo smaž pokud nepoužíváš -->

## Pravidla

### Prostředí
- NIKDY needituj .env — používej pouze .env.local
- Komunikace v chatu: <!-- TODO: česky / anglicky / jinak -->
- Kód (proměnné, komentáře, commity): anglicky
<!-- TODO: pokud používáš Docker, ponech tento blok a uprav příkazy: -->
- **Vše spouštěj VÝHRADNĚ přes Docker** (`docker compose up`, `docker compose exec app ...`)
  - NIKDY nespouštěj `npm run dev`, `npm run build`, `npm run lint` apod. přímo na hostu
  - Dev server: `docker compose up` (nebo `docker compose up -d`)
  - Příkazy v kontejneru: `docker compose exec app <příkaz>`
  - Build: `docker compose exec app npm run build`
  - Lint: `docker compose exec app npm run lint`
  - Instalace balíčků: `docker compose exec app npm install <balíček>` + poté `docker compose up --build`
  <!-- TODO: pokud používáš Prisma, ponech: -->
  - **Po změně Prisma schema (migrace, prisma generate):** vždy spusť `docker compose exec app npx prisma generate` A POTÉ `docker compose restart app` — jinak běží se starým Prisma klientem a dostaneš runtime error. Dělej to VŽDY automaticky po každé migraci.
  - **Po přidání/změně env proměnných v .env.local:** `docker compose restart` NESTAČÍ — nepřenačítá env soubory. Použij `docker compose up -d --force-recreate` nebo `docker compose down && docker compose up -d`.
  - **Po větších změnách souborů (mazání/přesouvání stránek):** vždy vyčisti cache a restartuj: `docker compose exec app rm -rf .next && docker compose restart app`
<!-- TODO: pokud NEPOUŽÍVÁŠ Docker, smaž celý blok výše a napiš jak se spouští dev server, build, lint atd. -->

### Nasazování — NASAZUJ VŠE ROVNOU
Tohle je **osobní appka**, ne produkce s uživateli. Po každé změně nasaď rovnou na produkci, **bez ptaní na potvrzení**:
1. commit + `git push origin main` (pracuj přímo na `main`)
2. **bumpni `CACHE` verzi v `sw.js`** (`rohliczech-vN` → `vN+1`) — vynutí update na zařízeních
3. deploy na Cloudflare Pages (viz Deploy níže)
4. ověř živou URL (HTTP 200 + že je tam nová změna)

Commit zprávy anglicky, stručné a popisné (feat:/fix:).

### Deploy (Cloudflare Pages přes Wrangler — NE GitHub Pages)
Produkce: **https://rohliczech.pages.dev/**. Git push produkci NEAKTUALIZUJE — to dělá až Wrangler. V PowerShellu z rootu projektu:
```powershell
$env:NODE_OPTIONS = "--use-system-ca"   # NUTNÉ kvůli firemnímu proxy (cert mismatch)
$env:CLOUDFLARE_ACCOUNT_ID = "b1feeb3a5877cbd3ec146343ada4fcb1"
npx wrangler pages deploy . --project-name=rohliczech --branch=main --commit-dirty=true
```
- Když přijde `Authentication error [code: 10000]`: token vypršel → `$env:NODE_OPTIONS='--use-system-ca'; npx wrangler login` (otevře prohlížeč, klik Allow).
- DB schéma (Supabase): nový sloupec přidej v SQL editoru DŘÍV, než ho kód začne posílat — jinak spadnou všechna ukládání receptů.

### Knihovny a verze
- Vždy používej nejnovější stabilní verze všech knihoven a frameworků
- Před instalací ověř aktuální verzi na internetu (npm, PyPI, docs)
- Nepoužívej deprecated balíčky

### Testy
<!-- TODO: uprav dle svého testovacího setupu -->
- Ke každé nové funkčnosti piš testy
- **E2E testy: Playwright** — konfigurace v `playwright.config.ts`, testy v `e2e/`
- **Po napsání nové funkce VŽDY spusť E2E testy**: <!-- TODO: příkaz pro spuštění testů -->
- Testová data mají prefix `[E2E]` + timestamp, aby nekolidovala s dev daty
- Testovací účty: <!-- TODO: testovací přihlašovací údaje -->
- Před commitem ověř, že testy procházejí
<!-- TODO: pokud nemáš testy, smaž tuto sekci a napiš: "Ke každé nové funkčnosti piš testy. Testovací framework: TODO" -->

### Role a přístupy
<!-- TODO: uprav dle svého modelu rolí, nebo smaž pokud nemáš role -->
- <!-- TODO: vypiš role, např. ADMIN, USER, EDITOR... -->
- <!-- TODO: jak se uživatelé registrují? Veřejná registrace / pozvánka adminem / jinak -->
- <!-- TODO: ochrana na kolika úrovních? Route-level, page-level, server actions... -->
- **Při implementaci nové funkce se VŽDY zeptej uživatele:** jaká role ji může vidět/upravovat, nebo zda je veřejně dostupná

### Mazací akce
- **Všechny mazací akce musí mít potvrzovací dialog** (AlertDialog s varováním)
<!-- TODO: pokud máš sdílenou komponentu pro mazání, uveď cestu -->

### Bezpečnost
- U každé nové funkce kontroluj bezpečnost (auth, validace vstupů, SQL injection, XSS, CSRF)
- Pokud najdeš potenciální riziko, zapiš do security_warnings.md v rootu projektu
- security_warnings.md obsahuje: nechráněné endpointy, slabá hesla, chybějící rate limiting, nezašifrovaná data apod.
- Při nejistotě upozorni uživatele

### Grafika a UI
<!-- TODO: doplň brand barvy a styl -->
- Brand:
  - Primární: <!-- TODO: hex barva, např. #21337b -->
  - Sekundární: <!-- TODO: hex barva, např. #ffb700 -->
  - Doplňkové: <!-- TODO: další barvy -->
  - Font: <!-- TODO: název fontu -->
  - Styl: <!-- TODO: moderní/minimalistický/hravý, zaoblené rohy, gradienty... -->

### Vyhledávání
- Pokud si nejsi jistý aktuální verzí, best practice nebo syntaxí, vyhledej na internetu
- Nespoléhej na zastaralé znalosti, ověřuj

## Struktura projektu
Zaznamenávej složky a jejich účel. Aktualizuj při každé změně.

```
/
<!-- TODO: Claude doplní automaticky při prvním průzkumu projektu. Příklad formátu: -->
├── docker-compose.yml        # Docker orchestrace
├── Dockerfile                # Multi-stage build
├── prisma/
│   ├── schema.prisma         # Prisma schéma
│   └── seed.ts               # Seed data
└── src/
    ├── app/                  # Next.js App Router stránky
    ├── components/           # React komponenty
    ├── lib/                  # Utility, DB klient, server actions
    └── types/                # TypeScript typy
```

## Omezení agenta
<!-- TODO: co má Claude vždy dělat / na co se vždy ptát. Příklady: -->
- Při každé nové funkci se zeptej, jaká role ji smí vidět a upravovat
<!-- - Nikdy neměň soubory v adresáři X bez potvrzení -->
<!-- - Vždy navrhni UI mockup (popis) před implementací -->

## Nuance projektu
Specifická business logika, výjimky, workaroundy. Doplňuj průběžně.

<!-- TODO: Claude sem bude průběžně zapisovat důležité detaily, např.: -->
<!-- - Model.field je unikátní identifikátor — validace při create/update -->
<!-- - Soft delete vzor: deletedAt timestamp, obnovení v koši -->
<!-- - Ceny jsou bez DPH, DPH se počítá client-side -->
<!-- - API endpoint /api/xxx — auth přes API klíč, ne session -->

## Rozhodnutí
<!-- TODO: Claude sem zapisuje architektonická rozhodnutí s datem a důvodem. Příklad: -->
<!-- - 2026-01-15: Next.js + shadcn/ui — moderní stack, snadný deploy -->
<!-- - 2026-01-16: JWT sessions místo DB sessions — méně DB queries, jednodušší setup -->

## Údržba tohoto souboru
- Aktualizuj po každé strukturální změně, novém pravidlu nebo rozhodnutí
- Maximální stručnost — detaily patří do kódu nebo docs/, ne sem
- Smaž zastaralé info, nepřidávej duplicity
