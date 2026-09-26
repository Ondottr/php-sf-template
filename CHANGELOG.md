# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [4.2.0] - 2026-09-26

First release in the `nations-original/sf-template` repository after migrating
from the earlier `medserv.ie/php-simple-framework-template` repository with
rewritten git history.

### Added

- **UUID primary keys** across entities. Doctrine identity strategy migrated to
  `symfony/uid` UUIDs end-to-end. Closes #53, closes #126.
- `symfony/uid` (`^8.1`) added to template `require`.
- New `App\DoctrineLifecycleCallbacks\` typed lifecycle callbacks (one class per
  event per entity, extending `AbstractDoctrineLifecycleCallback`).

### Changed

- **PHP minimum: `8.3` → `8.5`** (strict typing still required).
- **Symfony bumped to `^8.1`** across the stack (console, framework-bundle,
  messenger, twig-bundle, security-csrf, validator, asset, http-client, mime,
  form, property-access, property-info, serializer, string, translation,
  web-link, yaml, expression-language, intl, dotenv, process, runtime, ui).
- **PHP-SF framework dependency: `^3.x` → `^4.0`** (major dependency bump).
- `App\Kernal` (custom) → `App\Kernel` boot chain stabilised:
  `setHeaderTemplateClassName()` / `setFooterTemplateClassName()` are now
  `void` static calls — must not be chained.
- `AbstractEntityRepository::find()` widened to accept `int|string` to support
  non-integer primary keys (notably UUID strings).
- `r()` global helper is the canonical way to read the current
  `Symfony\Component\HttpFoundation\Request` inside controllers, middleware,
  views, listeners — `$this->request` and constructor injection were removed
  in framework `v3.0.0` and remain unsupported in `v4.x`.
- Middleware constructors take no arguments; request is fetched via `r()`;
  layout templates are swapped via `Kernel::setHeaderTemplateClassName()` /
  `Kernel::setFooterTemplateClassName()` (static).

### Fixed

- **Router headers-already-sent crash** (`src/Router.php` around the response
  fallback path): wrapped the `else` branch of `sendRouteMethodResponse()` in
  `ob_start()` + `try/catch` to recover gracefully when output has already
  begun. Covered by a subprocess fixture test. Closes #56 — PR #57.
- **Framework controllers as services** — `ServiceNotFoundException` on
  framework controllers resolved by registering custom PHP-SF controllers
  (`App\Http\Controller\`) into Symfony's service container alongside the
  existing native Symfony controllers (`App\Http\SymfonyControllers/`). PR #127.

### Documentation

- **NO-31 docs audit closed** — 4 commits, ~52 framework files cleaned,
  100 of 106 PR review comments resolved.
- Wiki corrected: php-sf vs Symfony DI distinction clarified;
  fabricated `App\Doctrine\Trait\` docs removed; dev-only `dd()` endpoint
  documented explicitly.
- `CLAUDE.md` updated to reflect the v3.0.0 / v4.x request-access contract
  (`r()` helper, no `$this->request`) and the static `Kernel` layout setters.

### Housekeeping

- Composer scripts standardised on short names: `composer stan`, `composer cs-fix`,
  `composer cs-check`.
- Test toolchain on current majors: PHPUnit `^12`, Codeception `^5.3`,
  PHPStan `^2.1` (+ `phpstan-doctrine`, `phpstan-symfony`),
  `friendsofphp/php-cs-fixer` `^3.94`, `symfony/phpunit-bridge` `^8.0`.
- `roave/security-advisories: dev-latest` retained in `require-dev`.
- Git history rewritten via `git-filter-repo` + `mailmap`; old committer
  emails mapped to the personal authoring identity.

### Upgrade notes

1. Bump your project's PHP to `8.5` and run `composer update`.
2. Replace any `$this->request` (or constructor-injected `Request`) in
   controllers, middleware, or templates with `r()`. The same applies inside
   any custom `AbstractController` / `Middleware` subclasses.
3. If you switch layouts from controllers or middleware, call the
   `Kernel::setHeaderTemplateClassName()` / `setFooterTemplateClassName()`
   setters as **separate** static statements — they return `void`.
4. UUID migration: entities persist with `symfony/uid` UuidInterface; existing
   integer-keyed rows are not auto-converted — plan a data migration before
   upgrading production data.
5. Symfony 8 upgrade: review form/validator/service config deltas. No template
   routing changes are required for the dual-controller setup.
