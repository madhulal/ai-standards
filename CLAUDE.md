# CLAUDE.md | SENIOR FULL-STACK & PLATFORM ARCHITECT

## 🎯 High-Level Principles
- **Design First**: Prefer Architecture Decision Records (ADRs) for non-trivial changes.
- **Scale**: Optimize for O(n) complexity. Use caching (Redis/CDN) and connection pooling.
- **Resilience**: Implement Circuit Breakers, Retries, and Graceful Degradation.
- **Security**: Zero-Trust. Assume inputs are malicious. Use Least Privilege for IAM.

## 🛠 Tech Stack & Ecosystem
- **Backend**: Java (Spring Boot), Node.js (Express/TS), Python (FastAPI).
- **Frontend**: Next.js (App Router), React, Tailwind, TypeScript (Strict).
- **Data**: PostgreSQL, MongoDB, Sanity (CMS), Akeneo (PIM).
- **Infra**: Terraform, Shell (POSIX), Docker, Kubernetes.
- **Cloud/Edge**: Azure, AWS, GCP, Cloudflare, Vercel, Netlify.

## 🏗 Coding & Architectural Standards
- **SRP & Modularization**: Methods < 30 lines. Separate Business Logic from Transport/Data layers.
- **Express.js**: Layered architecture (Routes -> Controllers -> Services). Mandatory `Zod` validation.
- **Spring Boot**: Constructor Injection, DTO pattern, `@ControllerAdvice` for global errors.
- **Naming Conventions**:
  - `camelCase`: Java, TypeScript, React variables/methods.
  - `PascalCase`: Classes, Interfaces, React Components.
  - `snake_case`: Python, Terraform, Database Schema.
  - `kebab-case`: CSS, URL slugs, Cloud Resource Names.
  - `UPPER_SNAKE_CASE`: Global Constants, Env Vars.

## 🤖 CI/CD & Automation (GitHub Actions)
- **Standard Workflow**: Every repo must have `.github/workflows/main.yml`.
- **Pipeline Stages**:
    1. **Lint/Format**: `prettier`, `eslint`, `checkstyle`, `flake8`, `tflint`.
    2. **Security Scan**: `trivy` (containers), `gitleaks` (secrets), `snyk` (deps).
    3. **Test**: Parallelized Unit & Integration tests. Fail if coverage < 80%.
    4. **Build**: Docker multi-stage builds; push to ACR/ECR/GAR.
    5. **Deploy**: Terraform `plan` on PR, `apply` on Merge (manual approval for Prod).
- **Environment Parity**: Use GitHub Environments with "Secret Protection" for Dev/Staging/Prod.

## ☁️ Infrastructure & DevOps
- **Terraform**: Modules are mandatory. Remote state locking enabled. 
- **Tagging**: `Owner`, `Project`, `Env`, `CostCenter`, `ManagedBy: Terraform`.
- **Shell**: Every script must have `set -euo pipefail` and a `usage()` function with `-h` flag.

## 📝 Documentation & Observability
- **Self-Documenting**: Clear naming > comments.
- **Javadocs/TSDoc/Docstrings**: Mandatory for public APIs. Include Big O for algorithms.
- **Observability**: Services must expose `/health`. Log in JSON with `trace_id`.

## 🧪 Testing Strategy
- **Unit**: JUnit 5 (Java), Vitest (JS/TS), Pytest (Python).
- **Integration**: Use Testcontainers for Postgres/Mongo.
- **E2E**: Playwright for critical user journeys.
- **IaC**: `terraform validate` and `tfsec` must pass before merge.

## 🚀 Execution Commands
- **Java**: `./mvnw spring-boot:run` | `./mvnw test`
- **Node**: `pnpm dev` | `pnpm test`
- **Infra**: `terraform plan` | `tflint`
- **Shell**: `shellcheck *.sh`
