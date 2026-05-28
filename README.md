# composer-210-install-policy

## Feature exercised

Exercises Composer 2.10.0 install-policy BC breaks and how the Mend
Unified Agent handles them: `preferred-install: dist` enforced in
`config`, revised `composer audit` exit codes, and platform-requirement
enforcement via `config.platform.php`.

## Composer 2.10.0 context

Three changes in Composer 2.10.0 are SCA-relevant:

1. **Automatic source fallback disabled by default.**
   Prior to 2.10, Composer would silently fall back from `dist` to
   `source` (VCS clone) if a dist download failed.  In 2.10 this
   fallback is off by default. UA's pre-step (`composer install`)
   must work with `preferred-install: dist` to avoid install failures
   on CI. The probe sets `config.preferred-install: dist` explicitly
   so the project behaves identically on Composer >=2.10.

2. **Audit command exit-code changes.**
   `composer audit` exit codes were revised. UA interprets the exit
   code to decide whether the scan succeeded. With 2.10, a zero-
   vulnerability audit now returns 0; prior versions may have returned
   non-zero in certain states. The probe includes `config.audit.abandoned`
   to exercise audit config parsing without triggering failures.

3. **Platform-requirement enforcement / locked-file policy.**
   The probe sets `config.platform.php: 8.2.0` to simulate a locked
   target platform, which affects whether Composer accepts the lockfile
   as consistent during `composer install --check-platform-reqs`.

## Dependency structure

### Production (`require`)

```
mend-qa/composer-210-install-policy
├── guzzlehttp/guzzle 7.8.1
│   ├── guzzlehttp/promises 2.0.2
│   ├── guzzlehttp/psr7 2.6.2
│   │   ├── psr/http-factory 1.1.0
│   │   │   └── psr/http-message 2.0
│   │   ├── psr/http-message 2.0
│   │   └── ralouphie/getallheaders 3.0.3
│   ├── psr/http-client 1.0.3
│   │   └── psr/http-message 2.0
│   └── symfony/deprecation-contracts 3.5.0
├── symfony/console 7.1.5
│   ├── symfony/deprecation-contracts 3.5.0
│   ├── symfony/polyfill-mbstring 1.30.0
│   ├── symfony/service-contracts 3.5.0
│   └── symfony/string 7.1.5
│       ├── symfony/polyfill-intl-grapheme 1.30.0
│       ├── symfony/polyfill-intl-normalizer 1.30.0
│       └── symfony/polyfill-mbstring 1.30.0
├── psr/log 3.0.0
└── symfony/http-client 7.1.5
    ├── psr/http-client 1.0.3
    ├── symfony/deprecation-contracts 3.5.0
    └── symfony/http-client-contracts 3.5.0
```

### Development (`require-dev`)

```
├── phpunit/phpunit 11.2.9
│   ├── sebastian/comparator 6.1.0
│   │   ├── sebastian/diff 6.0.2
│   │   └── sebastian/exporter 6.1.3
│   │       └── sebastian/recursion-context 6.0.2
│   ├── sebastian/diff 6.0.2
│   ├── sebastian/exporter 6.1.3
│   ├── sebastian/global-state 7.0.2
│   │   ├── sebastian/object-reflector 3.0.0
│   │   └── sebastian/recursion-context 6.0.2
│   ├── sebastian/object-enumerator 6.0.1
│   │   ├── sebastian/object-reflector 3.0.0
│   │   └── sebastian/recursion-context 6.0.2
│   ├── sebastian/recursion-context 6.0.2
│   ├── sebastian/type 5.1.0
│   ├── sebastian/version 4.0.1
│   └── symfony/deprecation-contracts 3.5.0
└── symfony/var-dumper 7.1.5
    └── symfony/polyfill-mbstring 1.30.0
```

## Expected dependency tree

- All packages in `packages` keyed by `vendor/package` (Composer
  `artifactId`-equivalent).
- `source: "registry"` for all Packagist packages.
- `group: "main"` for production; `group: "dev"` for dev-only packages.
- Platform packages (`php`, `ext-*`) absent from tree per UA resolver.
- `symfony/deprecation-contracts` is a shared transitive dependency
  (diamond): pulled in by both `guzzlehttp/guzzle`, `symfony/console`,
  `symfony/http-client`, and `phpunit/phpunit`. Mend deduplicates to
  a single node.

## Mend config

Bucket B — `.whitesource` emitted with `scanSettings.versioning` because
this probe **targets a specific Composer release (2.10)** and the
manifest alone is insufficient to force the toolchain to that version.
Pin: `composer: ">=2.10 <3"`, `php: ">=8.2 <8.4"`.

A `whitesource.config` is also included in the repo root to drive the
UA resolver flags:

- `php.resolveDependencies=true` — enable the Composer resolver.
- `php.runPreStep=false` — use the `tree_command` (lockfile) path by
  default. Set to `true` to switch to the `install_command` path and
  exercise Composer 2.10's preferred-install behavior.
- `php.includeDevDependencies=true` — include `require-dev` scope.

## Probe metadata

```
pattern:           composer-210-install-policy
pm:                composer
pm_version_tested: 2.10
generated_at:      2026-05-28T11:01:43Z
resolver_sha:      80ab6f76e3cb39d4304886777cb39ee987912f20
resolver_fetched:  2026-05-28T11:01:18+00:00
```
