# Enterprise Shoptet Development Platform

[![PHP Verzia](https://img.shields.io/badge/PHP-8.2%2B-777BB4?logo=php)](https://php.net)
[![Symfony](https://img.shields.io/badge/Symfony-6.4%2F7.0-000000?logo=symfony)](https://symfony.com)
[![CI/CD](https://img.shields.io/badge/GitHub%20Actions-Active-2088FF?logo=githubactions)](https://github.com/features/actions)
[![Kvalita kódu](https://img.shields.io/badge/PHPStan-Level%208-brightgreen)](https://phpstan.org)
[![Bezpečnosť](https://img.shields.io/badge/Security-ISO%2027001%20Ready-success)](https://iso.org/isoiec-27001)

> **Enterprise vývojová platforma pre správu 100+ Shoptet e-commerce projektov** s multi-tenant izoláciou, modulárnou architektúrou a automatizovanými CI/CD pipeline.

---

## Prehľad

**Enterprise Shoptet Development Platform** je komplexná infraštruktúra určená pre Shoptet Premium Partnerov, ktorí potrebujú vyvíjať, nasadzovať a udržiavať rozsiahle portfólio klientských projektov.

### Hlavné schopnosti

- **Multi-tenant architektúra** — Každý projekt je plne izolovaný, pričom zdieľa verzované core komponenty
- **Modulový systém** — Znovupoužiteľné moduly (platby, doprava, analýzy, sklady) distribuované cez privátny Composer registry
- **Automatizované quality gates** — PHPStan Level 8, PHP-CS-Fixer, Rector, PHPUnit s požiadavkou 80%+ pokrytia
- **Enterprise bezpečnosť** — Podpísané commity, CodeQL, Semgrep, Dependabot, secret scanning, povinné 2FA
- **Streamlined nasadenie** — Auto-deploy na staging, manuálne schvaľované produkčné releasy s blue/green podporou
- **Role-Based Access Control** — Granulárne oprávnenia pre architektov, správcov modulov, projektových leadrov a externých kontraktorov

---

## Systémová architektúra

```
┌─────────────────────────────────────────────────────────────┐
│                    VÝVOJOVÝ TÍM                              │
│  Architekti │ Správcovia modulov │ Projektoví devs │ Externí│
└──────────────────────┬──────────────────────────────────────┘
                       │
           ┌───────────▼───────────┐
           │  GITHUB ENTERPRISE    │
           │   yourcompany-org     │
           └───────────┬───────────┘
                       │
       ┌───────────────┼───────────────┐
       ▼               ▼               ▼
  ┌─────────┐    ┌──────────┐    ┌──────────┐
  │  CORE   │    │  MODULY  │    │ PROJEKTY │
  │ 5-10    │◄───│ 20-50    │◄───│ 100+     │
  │ repozitárov│   │ repozitárov│   │ repozitárov│
  └────┬────┘    └────┬─────┘    └────┬─────┘
       │              │               │
       └──────────────┼───────────────┘
                      ▼
           ┌────────────────────┐
           │  PRIVATE PACKAGIST │
           │ packages.company.com│
           └────────────────────┘
                      │
           ┌──────────▼──────────┐
           │   SHOPTET PREMIUM   │
           │   PARTNER API       │
           │  100+ klientských   │
           │  e-shopov           │
           └─────────────────────┘
```

### Taxonómia repozitárov

| Kategória | Prefix | Počet | Viditeľnosť | Ochrana |
|-----------|--------|-------|-------------|---------|
| Core knižnice | `core-*` | 5-10 | Private | Maximum |
| Moduly | `module-*` | 20-50 | Private | High |
| Klientské projekty | `project-*` | 100+ | Private | Medium |
| Šablóny | `template-*` | 3-5 | Private | High |
| Nástroje | `tool-*` | 5-10 | Private | Medium |

---

## Technologický stack

| Komponent | Verzia | Účel |
|-----------|--------|------|
| PHP | 8.2+ | Runtime |
| Symfony | 6.4 / 7.0 | Aplikačný framework |
| Doctrine ORM | 2.17+ | Databázová vrstva |
| Twig | 3.8+ | Templatovací engine |
| PHPUnit | 10.0+ | Unit testovanie |
| PHP-CS-Fixer | 3.41+ | Štýl kódu |
| PHPStan | 1.10+ | Statická analýza |
| Rector | 0.18+ | Automatizovaný refaktoring |

---

## CI/CD pipeline

Každý pull request prechádza **7 paralelnými quality gates**:

| Gate | Nástroj | Prah |
|------|---------|------|
| Štýl kódu | PHP-CS-Fixer | Nulové porušenia |
| Statická analýza | PHPStan + Psalm | Level 8 / Nulové chyby |
| Refaktoring | Rector | Žiadny upgradovateľný kód |
| Testovanie | PHPUnit | ≥ 80% pokrytie |
| Bezpečnosť | CodeQL + Semgrep | Nulové kritické nálezy |
| Závislosti | Composer Audit | Žiadne známe zraniteľnosti |
| Integrácia | Shoptet API Mock | Všetky testy prechádzajú |

### Pravidlá ochrany vetiev

- **`main`** — Vyžaduje 2 schválenia (Architekti), podpísané commity, všetky CI checky musia prejsť
- **`staging`** — Vyžaduje 1 schválenie, core CI checky musia prejsť
- **`feature/*`** — Pre-commit hooks vynucované lokálne

---

## Rýchly štart

### Požiadavky

- PHP 8.2+
- Composer 2.x
- Node.js 18+ (pre build assetov)
- Git s nakonfigurovaným GPG signing

### Vytvorenie nového projektu

```bash
# 1. Použi projektovú šablónu
github.com/yourcompany-org/template-project-base

# 2. Nainštaluj závislosti
composer install
npm ci

# 3. Nakonfiguruj Shoptet credentials
cp config/shoptet.yaml.dist config/shoptet.yaml
# Uprav: api_token, eshop_id

# 4. Spusti kontrolu kvality
composer cs       # PHP-CS-Fixer
composer stan     # PHPStan
composer test     # PHPUnit

# 5. Štart lokálneho vývoja
symfony server:start
```

### Vytvorenie nového modulu

```bash
# 1. Použi modulovú šablónu
github.com/yourcompany-org/template-module-base

# 2. Definuj závislosti modulu v composer.json
"require": {
    "yourcompany/core-api-client": "^2.0",
    "yourcompany/core-utils": "^1.5"
}

# 3. Implementuj funkciu podľa RFC procesu
# 4. Odošli PR na review Architektom + Správcovi modulu
```

---

## Vývojový workflow

```
Feature Branch → Lokálne testovanie → Pre-commit Hooks → PR
                                                           │
                                ┌──────────────────────────┘
                                ▼
                      ┌─────────────────┐
                      │  Code Review    │
                      │  (1-2 schválenia)│
                      └────────┬────────┘
                               ▼
                      ┌─────────────────┐
                      │  Merge do       │
                      │  Staging        │
                      └────────┬────────┘
                               ▼
                      ┌─────────────────┐
                      │  Auto-Deploy    │
                      │  + Smoke Testy  │
                      └────────┬────────┘
                               ▼
                      ┌─────────────────┐
                      │  Promote do     │
                      │  Produkčného    │
                      └─────────────────┘
```

---

## Bezpečnosť

- **Podpísané commity** — Všetky commity do `main` musia byť GPG podpísané
- **Secret scanning** — GitHub secret scanning + push protection zapnuté
- **Kontrola závislostí** — Automatické blokovanie zraniteľností na PR
- **SAST** — CodeQL a Semgrep (OWASP Top 10, CWE Top 25)
- **Rotácia tokenov** — API tokeny rotované každých 90 dní
- **Povinné 2FA** — Vyžadované pre všetkých členov organizácie

Pre bezpečnostné zraniteľnosti kontaktujte: `security@yourcompany.com`

---

## Roadmap

| Fáza | Časový rámec | Míľnik |
|------|--------------|--------|
| **Fáza 1: Základy** | Týždne 1-4 | Prvý projekt nasadený cez novú platformu |
| **Fáza 2: Škálovanie** | Týždne 5-8 | 5 projektov bežiacich na platforme |
| **Fáza 3: Optimalizácia** | Týždne 9-12 | Platforma pripravená na 100+ projektov |

### Metriky úspechu

| Metrika | Cieľ |
|---------|------|
| Frekvencia nasadenia | 10+ / deň |
| Lead time pre zmeny | < 1 hodina |
| Miera zlyhania zmien | < 5% |
| Čas obnovy po incidente | < 15 min |
| Pokrytie kódu testami | > 80% |
| Kritické bezpečnostné zraniteľnosti | 0 |

---

## Prispievanie

Vitáme príspevky podľa nášho RFC procesu:

1. **Zmeny v Core** — Odošli RFC, vyžaduje 2 schválenia Architektov
2. **Zmeny v Module** — Odošli PR, vyžaduje 1 schválenie Správcu modulu + 1 Architekt
3. **Zmeny v Projekte** — Odošli PR, vyžaduje 1 schválenie Projektového leadra

Prečítajte si náš [Contributing Guide](CONTRIBUTING.md) a [Code of Conduct](CODE_OF_CONDUCT.md).

---

## Dokumentácia

- [Prehľad architektúry](docs/ARCHITECTURE.md)
- [Dokumentácia API klienta](docs/API.md)
- [Príručka vývoja modulov](docs/MODULES.md)
- [Postupy nasadenia](docs/DEPLOYMENT.md)
- [Bezpečnostné politiky](docs/SECURITY.md)

---

## Licencia

Proprietary — Všetky práva vyhradené.

> **Poznámka:** Toto je interná enterprise platforma. Prístup je obmedzený na autorizovaný personál.

---

<p align="center">
  <sub>Vytvorené s ❤️ tímom architektov</sub><br>
  <sub>Posledná aktualizácia: Apríl 2026</sub>
</p>
