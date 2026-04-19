# ENTERPRISE SHOPTET DEVELOPMENT PLATFORM
## Technická špecifikácia v1.0

**Pre:** Vedúci vývojového tímu  
**Dátum:** 2026-04-14  
**Klasifikácia:** Interný technický dokument  

---

## OBSAH

1. Executive Summary
2. Prehľad systémovej architektúry
3. Štruktúra repozitárov a riadenie
4. Model riadenia prístupu a bezpečnosti
5. Špecifikácie CI/CD pipeline
6. PHP framework a štandardy kódovania
7. Architektúra modulového systému
8. Integračná vrstva Shoptet API
9. Riadenie nasadenia a vydávania
10. Implementačný roadmap
11. Prílohy

---

## 1. EXECUTIVE SUMMARY

### 1.1 Rozsah projektu

| Parameter | Špecifikácia |
|-----------|-------------|
| Cieľová škála | 100+ Shoptet projektov |
| Veľkosť tímu | 15-30 vývojárov v rôznych tímoch |
| Technologický stack | PHP 8.2+, Symfony komponenty, GitHub Actions |
| Cieľ nasadenia | Shoptet Premium Partner API |
| Compliance | ISO 27001 ready, SOC 2 Type II roadmap |

### 1.2 Kľúčové požiadavky splnené

- Multi-tenant izolácia - Každý projekt izolovaný, zdieľané komponenty verzované
- Granulárne riadenie prístupu - Role-based permissions na úrovni repozitára/tímu
- Povinné code review - 2-approval gate pre core, 1-approval pre projekty
- Automatizované quality gates - PHPStan level 8, PHP-CS-Fixer, security scanning
- Reusabilita modulov - Private Composer registry pre interné balíčky
- Audit trail - Všetky zmeny logované, signed commits required

---

## 2. PREHĽAD SYSTÉMOVEJ ARCHITEKTÚRY

### 2.1 High-level architektúra

PRACOVNÉ STANICE VÝVOJÁROV
- Architekt (2-3)
- Správca modulu (3-5)
- Projektový vývojár (10-20)
- Externý/Junior vývojár (5-10)

GITHUB ENTERPRISE CLOUD
Organizácia: yourcompany-shoptet

CORE/ (5-10 repozitárov)
- api-client, utils, testing, ci-tools
- Admin: Architekti
- Merge: 2 approvals

MODULY/ (20-50 repozitárov)
- payment, shipping, heureka, inventory, ga4
- Admin: Správcovia modulov
- Merge: 1 approval

PROJEKTY/ (100+ repozitárov)
- klient-a, klient-b, klient-c...
- Admin: Projektové tímy
- Merge: 1 approval

ŠABLÓNY/ (3-5 repozitárov)
- project-base, module-base

GITHUB ACTIONS RUNNERS
- Kvalita kódu: PHPCS, Rector, PHP-CS-Fixer
- Statická analýza: PHPStan, Psalm, Deptrac
- Security scan: Dependabot, CodeQL, Semgrep, Secrets
- Deploy pipeline: Shoptet API, S3/CloudFront

PRIVATE PACKAGIST
Registr: https://packages.yourcompany.com
- yourcompany/core-api-client [v1.0.0 - v2.4.1]
- yourcompany/core-utils [v1.0.0 - v1.8.2]
- yourcompany/core-testing [v1.0.0 - v1.3.0]
- yourcompany/module-payment [v1.0.0 - v3.2.0]
- yourcompany/module-heureka [v1.0.0 - v2.1.0]
- ... (50+ balíčkov)

SHOPTET INFRAŠTRUKTÚRA
Shoptet Premium Partner API
- Eshop A, B, C... N (100+ klientov)
- Webhooky, API v2.0, Šablóny, JS/CSS
- Monitoring: Datadog/New Relic/Sentry
- Logovanie: Centralizovaný ELK stack

### 2.2 Tok dát

FÁZA 1: LOKÁLNY VÝVOJ
Vývojár → Feature Branch → Lokálne testovanie → Pre-commit hooks → Push

FÁZA 2: CONTINUOUS INTEGRATION
Push → GitHub Actions → Paralelné joby:
- Job 1: PHP-CS-Fixer (kontrola štýlu)
- Job 2: PHPStan Level 8 (statická analýza)
- Job 3: Rector (kontrola refaktoringu)
- Job 4: PHPUnit (unit testy)
- Job 5: Security scan (CodeQL, Semgrep)
- Job 6: Dependency Guard (validácia balíčkov)
- Job 7: Shoptet API Mock Tests (integračné)

FÁZA 3: CODE REVIEW
Vytvorenie PR → Auto-assigned reviewers → Approval matrix:
- Zmeny v core: 2 schválenia Architektom
- Zmeny v module: 1 schválenie Správcom modulu + 1 Architektom
- Zmeny v projekte: 1 schválenie Projektovým leadom

FÁZA 4: STAGING NASADENIE
Merge do Staging → Auto-deploy → Smoke testy → QA validácia

FÁZA 5: PRODUKČNÉ VYDANIE
Promote do Main → 24h Staging validácia → Produkčné nasadenie → Monitoring

---

## 3. ŠTRUKTÚRA REPOZITÁROV A RIADENIE

### 3.1 Taxonómia repozitárov

| Kategória | Prefix | Počet | Viditeľnosť | Ochrana |
|-----------|--------|-------|------------|---------|
| Core | core-* | 5-10 | Private | Maximum |
| Moduly | module-* | 20-50 | Private | High |
| Projekty | project-* | 100+ | Private | Medium |
| Šablóny | template-* | 3-5 | Private | High |
| Nástroje | tool-* | 5-10 | Private | Medium |
| Dokumentácia | docs-* | 2-3 | Internal | Low |

### 3.2 Core repozitár: core-api-client

Repozitár: yourcompany-shoptet/core-api-client
Typ: Knižnica (Composer balíček)
PHP Verzia: 8.2+
Verzia: 2.4.1
Licencia: Proprietary

Správcovia:
- Primárny: Tím architektov (3 členovia)
- Záložný: Správcovia modulov (5 členov)

Ochrana vetvy - main:
- required_pull_request_reviews: 2 schválenia
- dismiss_stale_reviews: true
- require_code_owner_reviews: true
- required_status_checks: strict
- contexts: ci/codestyle, ci/static-analysis, ci/tests, ci/security-scan
- require_signed_commits: true
- include_administrators: false

Ochrana vetvy - staging:
- required_pull_request_reviews: 1 schválenie
- required_status_checks: strict

CODEOWNERS:
- *: @yourcompany-shoptet/architects
- /src/Endpoint/: @yourcompany-shoptet/module-maintainers

Štruktúra adresárov:
core-api-client/
- .github/workflows/ci.yml, release.yml, security.yml
- .github/CODEOWNERS, dependabot.yml, PULL_REQUEST_TEMPLATE.md
- config/endpoints.yaml
- src/Client/ShoptetClient.php, ShoptetClientInterface.php, RetryableClient.php
- src/Endpoint/AbstractEndpoint.php, ProductsEndpoint.php, OrdersEndpoint.php, CustomersEndpoint.php
- src/Dto/Request/, Response/
- src/Exception/ShoptetApiException.php, AuthenticationException.php, RateLimitException.php, ValidationException.php
- src/Authentication/TokenAuthentication.php, OAuth2Authentication.php
- src/Utils/ResponseValidator.php, PaginationHandler.php
- tests/Unit/, Integration/, Fixtures/
- docs/API.md, CHANGELOG.md
- .php-cs-fixer.dist.php, phpstan.neon, rector.php, composer.json, README.md

### 3.3 Šablóna repozitára modulu

Repozitár: yourcompany-shoptet/module-{názov}
Typ: Knižnica (Composer balíček)
Závislosti: core-api-client, core-utils
Verziovanie: Semantic Versioning

Štruktúra:
- src/Config/, Service/, Hook/, Component/, Template/, Asset/, Migration/
- tests/Unit/, Integration/, Functional/

Životný cyklus:
1. Návrh: RFC dokument v docs/modules/
2. Vývoj: Feature branch workflow
3. Review: 1 Správca modulu + 1 Architekt
4. Vydanie: Automatizované cez CI (tag → packagist)
5. Deprecation: 6-mesačná výpovedná lehota, migračný návod

### 3.4 Šablóna repozitára projektu

Repozitár: yourcompany-shoptet/project-{názov-klienta}
Typ: Aplikácia (Nasaditeľná)
Šablóna: template-project-base
Závislosti: core-*, module-* (špecifické verzie)

Štruktúra:
- config/shoptet.yaml, modules.yaml, parameters.yaml
- src/Controller/, Service/, EventSubscriber/, Command/
- templates/custom/
- public/assets/, .htaccess
- var/cache/, log/
- tests/, tools/
- composer.json, composer.lock, deploy.yaml

Nasadenie:
- staging: Auto-deploy pri merge do staging
- production: Manuálny trigger + schválenie

---

## 4. MODEL RIADENIA PRÍSTUPU A BEZPEČNOSTI

### 4.1 RBAC matica

| Rola | Core | Moduly | Projekty | Akcie |
|------|------|--------|----------|-------|
| Architekt | Admin | Admin | Read | Merge, Release, Emergency fix |
| Správca modulu | Triage | Write | Read | Module PR, Review, Release |
| Projektový lead | Read | Read | Admin | Projektové riadenie, Deploy |
| Senior vývojár | Read | Read | Write | Vývoj funkcií, PR |
| Junior vývojár | None | None | Write* | Vývoj, PR (2 schválenia) |
| Externý vývojár | None | None | Write* | Priradené projekty only |
| CI/CD Bot | Write | Write | Write | Automatizované releasy, Deploy |

*Obmedzené na priradené repozitáre

### 4.2 Konfigurácia GitHub tímov

organization: yourcompany-shoptet

tímy:
  architekti:
    popis: "Core systémoví architekti - plný prístup"
    členovia: lead-dev-1, lead-dev-2, cto (maintainer)
    repozitáre: core-*: admin, module-*: admin, project-*: admin, template-*: admin

  správcovia-modulov:
    popis: "Správcovia modulov - vývoj modulov"
    členovia: senior-dev-1, senior-dev-2, senior-dev-3, senior-dev-4, senior-dev-5
    repozitáre: core-*: triage, module-*: write, project-*: read

  projektoví-leadri:
    popis: "Projektoví team leadri"
    členovia: dev-lead-1, dev-lead-2, dev-lead-3
    repozitáre: core-*: read, module-*: read, project-*: admin

  tím-{klient-a}:
    nadradený: projektoví-leadri
    členovia: dev-1, dev-2, dev-3
    repozitáre: project-klient-a: write, core-*: read, module-*: read

  externí-kontraktori:
    popis: "Externí vývojári - obmedzený prístup"
    členovia: external-1, external-2
    repozitáre: project-assigned-*: write
    branch_protection_override: 2

### 4.3 Pravidlá ochrany vetiev

repository:
  default_branch: main
  
vetvy:
  - názov: main
    ochrana:
      required_pull_request_reviews: 2 schválenia
      dismiss_stale_reviews: true
      require_code_owner_reviews: true
      required_code_owner_review_count: 1
      dismissal_restrictions: tímy: [architekti]
      required_status_checks: strict
      contexts: ci/php-cs-fixer, ci/phpstan, ci/rector, ci/phpunit, ci/security-scan, ci/dependency-guard
      enforce_admins: true
      required_signatures: true
      required_linear_history: true
      allow_force_pushes: false
      allow_deletions: false
      required_conversation_resolution: true

  - názov: staging
    ochrana:
      required_pull_request_reviews: 1 schválenie
      required_status_checks: strict
      contexts: ci/php-cs-fixer, ci/phpstan, ci/phpunit

### 4.4 Bezpečnostné politiky

security:
  require_signed_commits: true
  gpg_signing_required_for: architekti, správcovia-modulov
  secret_scanning: enabled
  secret_scanning_push_protection: enabled
  dependabot_security_updates: enabled
  two_factor_authentication: required_for_all
  audit_log_retention: 180_days
  codeql_analysis: enabled
  semgrep_rules: p/security-audit, p/owasp-top-ten, p/cwe-top-25, custom/shoptet-security
  dependency_review: enabled
  vulnerable_dependency_blocking: enabled
  api_token_rotation: 90_dní
  webhook_secret_rotation: 90_dní

---

## 5. ŠPECIFIKÁCIE CI/CD PIPELINE

### 5.1 Hlavná CI pipeline

názov: Enterprise CI Pipeline

spustenie:
  push: vetvy: [main, staging, 'feature/**']
  pull_request: vetvy: [main, staging]

súbežnosť:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true

joby:

setup:
  runs-on: ubuntu-22.04
  výstupy:
    php-version: ${{ steps.config.outputs.php-version }}
    cache-key: ${{ steps.cache.outputs.key }}
  kroky:
    - uses: actions/checkout@v4
      with: fetch-depth: 0
    - id: config
      run: echo "php-version=$(cat .php-version || echo '8.2')" >> $GITHUB_OUTPUT
    - uses: shivammathur/setup-php@v2
      with:
        php-version: ${{ steps.config.outputs.php-version }}
        extensions: mbstring, intl, pdo_mysql, xml, zip, curl, json
        coverage: xdebug
        tools: composer:v2, cs2pr
    - uses: actions/cache@v3
      with:
        path: ~/.composer/cache
        key: ${{ runner.os }}-composer-${{ hashFiles('**/composer.lock') }}
    - run: |
        composer validate --strict
        composer install --prefer-dist --no-progress --no-scripts
        composer audit --locked

code-style:
  potrebuje: setup
  runs-on: ubuntu-22.04
  kroky:
    - uses: actions/checkout@v4
    - uses: shivammathur/setup-php@v2
      with: php-version: ${{ needs.setup.outputs.php-version }}, tools: php-cs-fixer, cs2pr
    - run: vendor/bin/php-cs-fixer fix --dry-run --diff --using-cache=no --format=checkstyle | cs2pr

static-analysis:
  potrebuje: setup
  runs-on: ubuntu-22.04
  stratégia:
    matica:
      nástroj: [phpstan, psalm]
      include:
        - nástroj: phpstan
          príkaz: vendor/bin/phpstan analyse --error-format=github --no-progress
        - nástroj: psalm
          príkaz: vendor/bin/psalm --output-format=github --no-progress
  kroky:
    - uses: actions/checkout@v4
    - uses: shivammathur/setup-php@v2
      with: php-version: ${{ needs.setup.outputs.php-version }}
    - run: ${{ matrix.príkaz }}

testing:
  potrebuje: setup
  runs-on: ubuntu-22.04
  služby:
    mysql:
      image: mysql:8.0
      env: MYSQL_ROOT_PASSWORD: root, MYSQL_DATABASE: test
      ports: 3306:3306
  kroky:
    - uses: actions/checkout@v4
    - uses: shivammathur/setup-php@v2
      with: php-version: ${{ needs.setup.outputs.php-version }}, coverage: xdebug
    - run: vendor/bin/phpunit --testsuite=unit --coverage-clover=coverage.xml
    - run: |
        COVERAGE=$(cat coverage.xml | grep -o 'lines-covered="[0-9]*"' | grep -o '[0-9]*')
        if [ "$COVERAGE" -lt "80" ]; then exit 1; fi

security:
  potrebuje: setup
  runs-on: ubuntu-22.04
  kroky:
    - uses: actions/checkout@v4
    - run: composer audit --locked
    - uses: gitleaks/gitleaks-action@v2
    - uses: returntocorp/semgrep-action@v1
      with: config: p/security-audit,p/owasp-top-ten,p/cwe-top-25

deploy-staging:
  potrebuje: [code-style, static-analysis, testing, security]
  if: github.ref == 'refs/heads/staging' && github.event_name == 'push'
  runs-on: ubuntu-22.04
  environment: staging
  kroky:
    - uses: actions/checkout@v4
    - run: |
        composer install --no-dev --optimize-autoloader
        npm ci --production && npm run build
    - uses: yourcompany-shoptet/core-ci-tools/.github/actions/shoptet-deploy@main
      with:
        environment: staging
        eshop-id: ${{ secrets.SHOPTET_STAGING_ESHOP_ID }}
        api-token: ${{ secrets.SHOPTET_STAGING_API_TOKEN }}

deploy-production:
  potrebuje: [deploy-staging]
  if: github.ref == 'refs/heads/main' && github.event_name == 'workflow_dispatch'
  runs-on: ubuntu-22.04
  environment: production
  kroky:
    - uses: actions/checkout@v4
      with: ref: ${{ github.event.inputs.version }}
    - uses: yourcompany-shoptet/core-ci-tools/.github/actions/shoptet-deploy@main
      with:
        environment: production
        eshop-id: ${{ secrets.SHOPTET_PROD_ESHOP_ID }}
        api-token: ${{ secrets.SHOPTET_PROD_API_TOKEN }}

---

## 6. PHP FRAMEWORK A ŠTANDARDY KÓDOVANIA

### 6.1 Technologický stack

| Komponent | Verzia | Účel |
|-----------|--------|------|
| PHP | 8.2+ | Runtime |
| Symfony | 6.4/7.0 | Framework |
| Doctrine ORM | 2.17+ | Databáza |
| Twig | 3.8+ | Templating |
| PHPUnit | 10.0+ | Testovanie |
| PHP-CS-Fixer | 3.41+ | Štýl kódu |
| PHPStan | 1.10+ | Statická analýza |
| Rector | 0.18+ | Refaktoring |

### 6.2 PHP-CS-Fixer konfigurácia

```
<?php
declare(strict_types=1);
use PhpCsFixer\Config;
use PhpCsFixer\Finder;

$finder = Finder::create()->in(__DIR__ . '/src')->in(__DIR__ . '/tests')->exclude('var')->exclude('vendor');

return (new Config())
    ->setRiskyAllowed(true)
    ->setUsingCache(true)
    ->setFinder($finder)
    ->setRules([
        '@PSR12' => true,
        '@PSR12:risky' => true,
        'array_syntax' => ['syntax' => 'short'],
        'ordered_imports' => ['sort_algorithm' => 'alpha'],
        'declare_strict_types' => true,
        'strict_comparison' => true,
        'strict_param' => true,
        'final_class' => true,
        'void_return' => true,
        'phpdoc_align' => true,
        'phpdoc_order' => true,
        'trailing_comma_in_multiline' => true,
        'single_quote' => true,
        'yoda_style' => ['equal' => false, 'identical' => false],
    ]);
```

### 6.3 PHPStan konfigurácia

```
includes:
    - vendor/phpstan/phpstan/conf/bleedingEdge.neon
    - vendor/phpstan/phpstan-strict-rules/rules.neon
    - vendor/phpstan/phpstan-deprecation-rules/rules.neon

parameters:
    level: 8
    paths: [src, tests]
    checkMissingIterableValueType: true
    checkGenericClassInNonGenericObjectType: true
    checkExplicitMixed: true
```

### 6.4 Rector konfigurácia

```
<?php
declare(strict_types=1);
use Rector\Config\RectorConfig;
use Rector\Set\ValueObject\LevelSetList;
use Rector\Set\ValueObject\SetList;

return static function (RectorConfig $rectorConfig): void {
    $rectorConfig->paths([__DIR__ . '/src', __DIR__ . '/tests']);
    $rectorConfig->sets([
        LevelSetList::UP_TO_PHP_82,
        SetList::CODE_QUALITY,
        SetList::DEAD_CODE,
        SetList::EARLY_RETURN,
        SetList::TYPE_DECLARATION,
    ]);
};
```

---

## 7. ARCHITEKTÚRA MODULOVÉHO SYSTÉMU

### 7.1 Životný cyklus modulu

FÁZA 1: NÁVRH (RFC)
- Autor: Správca modulu alebo Architekt
- Výstup: RFC dokument v docs/modules/rfc-{názov}.md
- Review: 1 schválenie Architektom

FÁZA 2: VÝVOJ
- Repozitár: yourcompany-shoptet/module-{názov}
- Vetvenie: Feature branches → PR → Staging
- CI: Všetky quality gates
- Review: 1 Správca modulu + 1 Architekt

FÁZA 3: VYDANIE
- Verziovanie: Semantic Versioning
- Vydanie: Automatizované cez GitHub Actions
- Distribúcia: Private Packagist

FÁZA 4: ÚDRŽBA
- Podpora: LTS pre major verzie (12 mesiacov)
- Aktualizácie: Security patches automaticky
- Deprecation: 6-mesačná migračná lehota

FÁZA 5: UKONČENIE PODPORY
1. Označiť @deprecated v kóde
2. Vydanie minor s deprecation notice
3. Migračný návod v docs/
4. 6 mesiacov podpory
5. Major verzia s odstránením

### 7.2 Štruktúra modulu

```
module-{názov}/
- .github/workflows/ci.yml, release.yml, security.yml
- .github/CODEOWNERS
- config/services.yaml, parameters.yaml, routes.yaml
- src/Config/, Service/, Hook/, Component/, Api/, Dto/, Exception/, DependencyInjection/, {NázovModulu}Bundle.php
- templates/components/
- assets/js/, css/
- tests/Unit/, Integration/, Functional/, Fixtures/
- docs/README.md, INSTALL.md, CONFIGURATION.md, API.md
- .php-cs-fixer.dist.php, phpstan.neon, rector.php, composer.json, README.md
```

### 7.3 Module composer.json

```
{
    "name": "yourcompany/module-{názov}",
    "type": "symfony-bundle",
    "description": "Shoptet modul pre {funkcionalitu}",
    "license": "proprietary",
    "require": {
        "php": "^8.2",
        "yourcompany/core-api-client": "^2.0",
        "yourcompany/core-utils": "^1.5",
        "symfony/framework-bundle": "^6.4|^7.0"
    },
    "require-dev": {
        "yourcompany/core-testing": "^1.0",
        "phpunit/phpunit": "^10.0",
        "phpstan/phpstan": "^1.10"
    },
    "autoload": {
        "psr-4": {"YourCompany\\Module\\{Názov}\\": "src/"}
    },
    "scripts": {
        "test": "phpunit",
        "cs": "php-cs-fixer fix --dry-run --diff",
        "stan": "phpstan analyse"
    }
}
```

---

## 8. INTEGRAČNÁ VRSTVA SHOPTET API

### 8.1 Architektúra API klienta

APLIKAČNÁ VRSTVA
- ProductService, OrderService, CustomerService

API KLIENT VRSTVA (core-api-client)
- ShoptetClientInterface
- AbstractShoptetClient
- Dekorátory: RetryableClient, CachedClient, LoggingClient
- Endpoint handlery: ProductsEndpoint, OrdersEndpoint, CustomersEndpoint, SystemEndpoint

HTTP TRANSPORTNÁ VRSTVA
- PSR-18 HTTP Client (Symfony HttpClient)
- Connection Pooling (max 10)
- Rate Limit Handler (100 req/min)
- Circuit Breaker (fail fast)

SHOPTET API (REST)
- https://api.myshoptet.com/api/

### 8.2 Implementácia API klienta

```
<?php
declare(strict_types=1);
namespace YourCompany\Core\ApiClient;

use Psr\Http\Client\ClientInterface;
use Psr\Log\LoggerInterface;

final class ShoptetClient implements ShoptetClientInterface
{
    private const MAX_RETRIES = 3;
    private const RATE_LIMIT_REQUESTS = 100;
    private const RATE_LIMIT_WINDOW = 60;
    private array $rateLimitStore = [];

    public function __construct(
        private ClientInterface $httpClient,
        private LoggerInterface $logger,
        private RequestFactory $requestFactory,
    ) {}

    public function request(string $method, string $endpoint, string $responseClass, ?array $data = null): object
    {
        $this->checkRateLimit();
        $request = $this->requestFactory->create($method, $endpoint, $data);
        
        $attempt = 0;
        while ($attempt < self::MAX_RETRIES) {
            try {
                $response = $this->httpClient->sendRequest($request);
                return $this->deserializeResponse($response, $responseClass);
            } catch (RateLimitException $e) {
                sleep($e->getRetryAfter() ?? self::RATE_LIMIT_WINDOW);
                $attempt++;
            } catch (TransientException $e) {
                usleep(100 * (2 ** $attempt) * 1000);
                $attempt++;
            }
        }
        throw new MaxRetriesExceededException();
    }

    private function checkRateLimit(): void
    {
        $now = time();
        $this->rateLimitStore = array_filter($this->rateLimitStore, fn($t) => $t > $now - self::RATE_LIMIT_WINDOW);
        if (count($this->rateLimitStore) >= self::RATE_LIMIT_REQUESTS) {
            throw new RateLimitException();
        }
    }
}
```

### 8.3 Endpoint pattern

```
<?php
declare(strict_types=1);
namespace YourCompany\Core\ApiClient\Endpoint;

final readonly class ProductsEndpoint extends AbstractEndpoint
{
    private const BASE_PATH = 'products';

    public function list(ProductListRequest $request): ProductListResponse
    {
        return $this->client->request(
            method: 'GET',
            endpoint: self::BASE_PATH . '?' . http_build_query($request->toArray()),
            responseClass: ProductListResponse::class,
        );
    }

    public function get(string $guid): ProductDetailResponse
    {
        return $this->client->request(
            method: 'GET',
            endpoint: self::BASE_PATH . '/' . urlencode($guid),
            responseClass: ProductDetailResponse::class,
        );
    }

    public function create(ProductCreateRequest $request): ProductCreateResponse
    {
        return $this->client->request(
            method: 'POST',
            endpoint: self::BASE_PATH,
            responseClass: ProductCreateResponse::class,
            data: $request->toArray(),
        );
    }
}
```

### 8.4 Hook handler pattern

```
<?php
declare(strict_types=1);
namespace App\Hook;

use YourCompany\Core\Hook\AbstractHookHandler;
use YourCompany\Core\Hook\HookResponse;
use YourCompany\Core\Hook\Attribute\ShoptetHook;

#[ShoptetHook(event: 'order.created', priority: 100, async: false, retry: 3)]
final readonly class OrderCreatedHandler extends AbstractHookHandler
{
    public function __construct(
        private OrderProcessor $orderProcessor,
        private LoggerInterface $logger,
    ) {}

    public function handle(array $data): HookResponse
    {
        if (!isset($data['order']['guid'])) {
            return HookResponse::invalid('Missing order.guid');
        }

        try {
            if ($this->orderProcessor->wasProcessed($data['order']['guid'])) {
                return HookResponse::success('Already processed');
            }

            $result = $this->orderProcessor->process($data);
            return HookResponse::success('Order processed', ['external_id' => $result->getExternalId()]);
        } catch (\Throwable $e) {
            $this->logger->error('Order processing failed', ['error' => $e->getMessage()]);
            return HookResponse::error($e->getMessage(), retryable: true);
        }
    }
}
```

---

## 9. RIADENIE NASADENIA A VYDÁVANIA

### 9.1 Architektúra nasadenia

GIT REPOZITÁR
- main (produkcia)
- staging (pre-produkcia)
- release/v1.2.3 (príprava vydania)
- feature/*, bugfix/* (vývoj)

BUILD & PACKAGE (GitHub Actions)
1. Composer install --no-dev --optimize-autoloader
2. npm ci && npm run build
3. Minifikácia a verziovanie assetov
4. Symfony cache warmup
5. Security scan (Trivy, Semgrep)
6. Balíček artifact (zip/tar)
7. Upload na S3/Artifact Registry

CIELE NASADENIA

STAGING
- Auto-deploy
- Smoke testy
- QA validácia
- 24h stabilita

PRODUKCIA
- Manuálny trigger
- Schvaľovací gate
- Blue/green
- Canary 10%→100%

ROLLBACK
- Jedným kliknutím
- Predchádzajúca verzia
- Okamžitý prepínač
- Zachovanie dát

Shoptet API nasadenie:
1. Upload šablónových súborov
2. Upload assetov (JS/CSS)
3. Registrácia/aktualizácia hookov
4. Verifikácia nasadenia
5. Smoke testy
6. Monitoring a alertovanie

### 9.2 Konfigurácia nasadenia

```
deployment:
  prostredia:
    staging:
      shoptet:
        api_url: "https://api.myshoptet.com/api"
        eshop_id: "${SHOPTET_STAGING_ESHOP_ID}"
        api_token: "${SHOPTET_STAGING_API_TOKEN}"
      features: [debug_mode, detailed_logging, test_payment_gateway]
    
    production:
      shoptet:
        api_url: "https://api.myshoptet.com/api"
        eshop_id: "${SHOPTET_PROD_ESHOP_ID}"
        api_token: "${SHOPTET_PROD_API_TOKEN}"
      features: [production_mode, error_reporting: critical_only, cache_enabled: true]

  stratégia:
    type: blue_green
    health_check: enabled: true, timeout: 300, interval: 10, endpoints: [/, /health, /api/status]
    rollback: automatic: true, conditions: [error_rate > 5%, response_time > 2000ms, health_check_failures > 3]

  assety:
    build_command: "npm run build"
    source_dir: "public/build"
    target_path: "/assets"
    versioning: true
    cdn: enabled: true, base_url: "https://cdn.yourcompany.com"

  hooky:
    registration: auto_register: true, verify_ssl: true, timeout: 30
    endpoints:
      order.created: url: "/webhook/order-created", secret: "${WEBHOOK_SECRET}", events: ["order.created"]
      product.updated: url: "/webhook/product-updated", secret: "${WEBHOOK_SECRET}", events: ["product.updated", "product.created"]

  smoke_testy:
    enabled: true
    testy:
      - názov: "Homepage Load", url: "/", expected_status: 200, expected_content: "Shoptet"
      - názov: "API Health", url: "/api/health", expected_status: 200, timeout: 5
      - názov: "Product List", url: "/api/products", expected_status: 200, validate_json: true
```
### 9.3 Deployment workflow

```
názov: Deploy to Shoptet

spustenie:
  workflow_dispatch:
    inputs:
      prostredie: type: choice, options: [staging, production]
      verzia: required: true
      skip_tests: type: boolean, default: false

joby:
  pre-deploy:
    runs-on: ubuntu-22.04
    kroky:
      - uses: actions/checkout@v4
        with: ref: ${{ inputs.verzia }}
      - run: |
          composer install --no-dev --optimize-autoloader
          npm ci && npm run build
      - uses: aquasecurity/trivy-action@master
        with: scan-type: 'fs', severity: 'CRITICAL,HIGH'

  deploy:
    potrebuje: pre-deploy
    runs-on: ubuntu-22.04
    environment: ${{ inputs.prostredie }}
    kroky:
      - uses: actions/download-artifact@v4
      - run: shoptet-deploy --environment=${{ inputs.prostredie }} --config=deploy.yaml --version=${{ inputs.verzia }}

  post-deploy:
    potrebuje: deploy
    if: success()
    kroky:
      - run: ./bin/smoke-tests.sh --environment=${{ inputs.prostredie }}
      - uses: 8398a7/action-slack@v3
        with: status: success, channel: '#deployments'
```

---

## 10. IMPLEMENTAČNÝ ROADMAP

### Fáza 1: Základy (Týždne 1-4)

| Týždeň | Úloha | Zodpovedný | Výstup |
|--------|-------|-----------|--------|
| 1 | GitHub Org Setup | Architekt | Organizácia, tímy |
| 1 | Riadenie prístupu | Architekt | RBAC matica |
| 2 | Core šablóna | Architekt | template-core-base |
| 2 | CI/CD základy | DevOps | Reusable workflows |
| 3 | API Client v1.0 | Tím architektov | Funkčný API klient |
| 3 | Core Utils v1.0 | Tím architektov | Utility knižnica |
| 4 | Projektová šablóna | Architekt | template-project-base |
| 4 | Dokumentácia | Všetci | README, CONTRIBUTING |

Míľnik: Prvý projekt nasadený cez nový systém

### Fáza 2: Škálovanie (Týždne 5-8)

| Týždeň | Úloha | Zodpovedný | Výstup |
|--------|-------|-----------|--------|
| 5 | Modulový systém | Správcovia modulov | 3 pilotné moduly |
| 5 | Private Packagist | DevOps | packages.yourcompany.com |
| 6 | Security scanning | Security | CodeQL, Semgrep, Trivy |
| 6 | Auto releasy | DevOps | Tag → Packagist automation |
| 7 | Monitoring | DevOps | Datadog/Sentry integrácia |
| 7 | Migrácia | Projektové tímy | 5 projektov na platforme |
| 8 | Školenie | Všetci | Dokumentácia, workshop |

Míľnik: 5 projektov bežiacich na platforme

### Fáza 3: Optimalizácia (Týždne 9-12)

| Týždeň | Úloha | Zodpovedný | Výstup |
|--------|-------|-----------|--------|
| 9 | Výkon | Architekt | Caching, CDN |
| 9 | Pokročilé nasadenie | DevOps | Blue/green, canary |
| 10 | Self-Service | DevOps | CLI pre nové projekty |
| 10 | Modulový marketplace | Správcovia modulov | 10+ modulov |
| 11 | Disaster Recovery | DevOps | Backup procedúry |
| 11 | Compliance | Security | ISO 27001 readiness |
| 12 | Platforma v1.0 | Všetci | Launch |

Míľnik: Platforma pripravená na 100+ projektov

### Metriky úspechu

| Metrika | Cieľ | Meranie |
|---------|------|---------|
| Frekvencia nasadenia | 10+/deň | GitHub Actions |
| Lead time for changes | < 1 hodina | PR do produkcie |
| Miera zlyhania zmien | < 5% | Rollbacky/nasadenia |
| Čas obnovy po incidente | < 15 min | Incident do fixu |
| Pokrytie kódu testami | > 80% | PHPUnit reporty |
| Security zraniteľnosti | 0 kritických | Trivy/CodeQL |
| Spokojnosť vývojárov | > 4/5 | Interný prieskum |

---

## 11. PRÍLOHY

### Príloha A: Slovník pojmov

| Pojem | Definícia |
|-------|-----------|
| Shoptet | SaaS ecommerce platforma |
| Premium Partner | Najvyššia úroveň Shoptet partnerstva |
| Modul | Znovupoužiteľný balíček rozširujúci funkcionalitu Shoptetu |
| Hook | Shoptet webhook pre event-driven integráciu |
| Komponent | Shoptet-specific UI komponent |
| Architekt | Senior vývojár so systémovou zodpovednosťou |
| Správca modulu | Vývojár zodpovedný za špecifický modul(y) |

### Príloha B: Referencia nástrojov

| Kategória | Nástroj | Účel |
|-----------|---------|------|
| VCS | GitHub Enterprise | Hosting repozitárov |
| CI/CD | GitHub Actions | Automatizácia |
| Registry | Packagist Private | PHP balíčky |
| Security | CodeQL | SAST |
| Security | Semgrep | Custom pravidlá |
| Security | Trivy | Scanning kontajnerov |
| Monitoring | Datadog | APM, logovanie |
| Error Tracking | Sentry | Sledovanie chýb |

### Príloha C: Emergency rollback

```
#!/bin/bash
# emergency-rollback.sh
PROSTREDIE=$1
VERZIA=$2
echo "EMERGENCY ROLLBACK: $PROSTREDIE → $VERZIA"
gh workflow disable deploy.yml
gh workflow run rollback.yml -f environment=$PROSTREDIE -f version=$VERZIA
slack-post "#incidents" "Rollback $PROSTREDIE to $VERZIA"
gh issue create --title "INCIDENT: Rollback $PROSTREDIE" --label "incident,p0"
```

### Príloha D: Reakcia na security incident

1. Detekcia: Automatický alert alebo manuálny report
2. Izolácia: Izolovať postihnutý projekt(y)
3. Hodnotenie: Klasifikácia závažnosti (P0-P3)
4. Náprava: Nasadenie patchu
5. Post-mortem: RCA do 48h

---

Kontrola dokumentu:
- Verzia: 1.0
- Posledná aktualizácia: 2026-04-14
- Ďalšia revízia: 2026-07-14
- Vlastník: Tím architektov
- Distribúcia: Interná
