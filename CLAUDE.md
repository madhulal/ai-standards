# CLAUDE.md | FULL-STACK DEVELOPER

## High-Level Principles

- **Design First**: Prefer Architecture Decision Records (ADRs) for non-trivial changes.
- **Scale**: Analyze and document algorithmic complexity. Use caching (Redis/CDN) and connection pooling. Justify deviations from the simplest correct approach.
- **Resilience**: Implement Circuit Breakers, Retries with exponential backoff, and Graceful Degradation.
- **Security**: Zero-Trust. Assume inputs are malicious. Use Least Privilege for IAM. Secrets never in code or env vars — use a secrets manager.
- **Progressive Delivery**: New capabilities ship behind feature flags. Use trunk-based development with short-lived branches.

---

## Tech Stack & Ecosystem

- **Backend**: Java (Spring Boot), Node.js (Express/TS), Python (FastAPI).
- **Frontend**: Next.js (App Router), React, Tailwind, TypeScript (Strict).
- **Data**: PostgreSQL, MongoDB, Sanity (CMS), Akeneo (PIM).
- **Infra**: Terraform, Shell (POSIX), Docker, Kubernetes.
- **Cloud/Edge**: Azure, AWS, GCP, Cloudflare, Vercel, Netlify.
- **Secrets**: Azure Key Vault (Azure), AWS Secrets Manager (AWS), HashiCorp Vault (multi-cloud/on-prem).
- **Feature Flags**: OpenFeature SDK (provider-agnostic). Flag evaluation must be synchronous and never block the critical path.

---

## Coding & Architectural Standards

- **SRP & Modularization**: Methods < 30 lines. Separate Business Logic from Transport/Data layers.
- **Express.js**: Layered architecture (Routes → Controllers → Services). Mandatory `Zod` validation.
- **Spring Boot**: Constructor Injection, DTO pattern, `@ControllerAdvice` for global errors.
- **API Versioning**: All HTTP APIs must be versioned via URL path prefix (`/v1/`, `/v2/`). Breaking changes require a new version. Deprecation notices via `Sunset` and `Deprecation` response headers. OpenAPI/AsyncAPI specs are generated from code (not hand-authored) and published to the internal developer portal on every merge to main.
- **Dependency Management**:
  - Node.js: Use `pnpm`. Lockfile (`pnpm-lock.yaml`) must be committed. Floating version ranges (`^`, `~`) are prohibited in `package.json` for production dependencies — pin exact versions.
  - Python: Pin all dependencies in `requirements.txt` via `pip-compile`. Use `pyproject.toml` for project metadata.
  - Java: Dependency versions managed via BOM (`spring-boot-dependencies`). Do not override managed versions without a documented reason.

### Naming Conventions

| Style | Used For |
|---|---|
| `camelCase` | Java, TypeScript, React variables/methods |
| `PascalCase` | Classes, Interfaces, React Components |
| `snake_case` | Python, Terraform, Database Schema |
| `kebab-case` | CSS, URL slugs, Cloud Resource Names |
| `UPPER_SNAKE_CASE` | Global Constants, Env Vars |

---

## CI/CD & Automation (GitHub Actions)

- **Standard Workflow**: Every repo must have `.github/workflows/main.yml`.
- **Pipeline Stages**:
    1. **Lint/Format**: `prettier`, `eslint`, `checkstyle`, `flake8`, `tflint`.
    2. **Security Scan**:
       - `trivy` — container image and filesystem vulnerabilities.
       - `gitleaks` — secrets detection (hard-fail on any finding).
       - `snyk` — dependency vulnerabilities (fail on High/Critical).
       - `semgrep` or `CodeQL` — SAST for logic and injection flaws. Block merge on any Critical finding; triage High within 48h.
    3. **Test**: Parallelized Unit & Integration tests. Fail if line coverage < 80% **and** mutation score < 60% (see Testing Strategy).
    4. **Build**: Docker multi-stage builds; push to ACR/ECR/GAR. Sign images with `cosign` (Sigstore).
    5. **DAST** (Staging only): Run OWASP ZAP baseline scan against deployed staging environment. Results posted as PR comment.
    6. **Deploy**: Terraform `plan` on PR, `apply` on Merge (manual approval for Prod). Deployments use progressive rollout (canary or blue/green) gated by feature flags where applicable.
- **Environment Parity**: Use GitHub Environments with "Secret Protection" for Dev/Staging/Prod. Secrets are never stored in GitHub Secrets for production — reference them from the designated secrets manager at runtime.

---

## Infrastructure & DevOps

- **Terraform**: Modules are mandatory. Remote state locking enabled.
- **Tagging**: `Owner`, `Project`, `Env`, `CostCenter`, `ManagedBy: Terraform`.
- **Shell**: Every script must have `set -euo pipefail` and a `usage()` function with `-h` flag.
- **Secrets Management**: No credentials in Terraform state. Use data sources to reference secrets from Key Vault / Secrets Manager. Rotate secrets automatically via scheduled pipelines. Access is audited.
- **Kubernetes Standards**:
  - Every `Deployment` must declare `resources.requests` and `resources.limits` for CPU and memory.
  - `livenessProbe` and `readinessProbe` are mandatory on all containers. Probes must not share the same handler as `/health` (use `/healthz/live` and `/healthz/ready` respectively).
  - `PodDisruptionBudget` is required for all workloads with `replicas > 1`. Minimum availability: 50% of desired replicas.
  - No containers run as `root`. Set `runAsNonRoot: true` and `readOnlyRootFilesystem: true` in `securityContext`.
  - Network policies are default-deny; allowances are explicitly declared.

---

## Documentation & Observability

- **Self-Documenting**: Clear naming > comments.
- **Javadocs/TSDoc/Docstrings**: Mandatory for public APIs. Include complexity analysis (Big O) for algorithms.
- **OpenAPI/AsyncAPI**: Auto-generated specs published to the developer portal on every merge to main. Breaking changes detected by `oasdiff` in CI — block merge if backward-incompatible changes are undocumented.
- **Observability**: Services must expose `/healthz/live` and `/healthz/ready`. Log in JSON with `trace_id` and `span_id`. Emit the four golden signals (latency, traffic, errors, saturation) as metrics. Distributed tracing via OpenTelemetry (OTLP export).

---

## Testing Strategy

- **Unit**: JUnit 5 (Java), Vitest (JS/TS), Pytest (Python). Target: ≥ 80% line coverage.
- **Mutation Testing**: PIT (Java), Stryker (JS/TS). Target: ≥ 60% mutation score. Mutation testing runs on the CI pipeline. Line coverage alone is not sufficient — a green coverage report with a low mutation score indicates undertested logic.
- **Integration**: Use Testcontainers for Postgres/Mongo. Test the service boundary, not internal implementation.
- **Contract Testing**: Pact for consumer-driven contract tests between services. Contracts published to the Pact Broker. Provider verification runs in CI before any deploy.
- **E2E**: Playwright for critical user journeys. Runs in CI against staging.
- **IaC**: `terraform validate` and `tfsec` must pass before merge.
- **DAST**: OWASP ZAP baseline scan runs against staging after every deploy (see CI/CD section).

---

## Execution Commands

| Stack | Run | Test |
|---|---|---|
| Java | `./mvnw spring-boot:run` | `./mvnw test` |
| Node | `pnpm dev` | `pnpm test` |
| Infra | `terraform plan` | `tflint && tfsec` |
| Shell | — | `shellcheck *.sh` |
| Contracts | — | `pact-provider-verifier` |
| Mutation (Java) | — | `./mvnw org.pitest:pitest-maven:mutationCoverage` |
| Mutation (JS/TS) | — | `pnpm stryker run` |
