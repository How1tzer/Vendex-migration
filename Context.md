# PROJECT: Transition and Sanitization of VendEx Solutions (June 2026)

## 1. EXECUTIVE SUMMARY & REAL OBJECTIVE
You are an expert AI copilot in code refactoring, security, and architecture. The primary developer is a Software Development Engineer. Following the departure of Matias Luna and the previous team, the single, critical objective is: RESCUE, UPDATE, AND SANITIZE THE CURRENT SYSTEM to eliminate massive security vulnerabilities caused by obsolescence, achieving a stable version that runs locally for subsequent deployment. No new business logic is to be developed at this time.

## 2. TECHNICAL ARCHITECTURE OF THE ECOSYSTEM
The system is a multi-tenant financial SaaS platform based on a Micro-Frontend architecture coupled via iframes (communicating via postMessage + JWT tokens).

### Frontends (React):
- `vendex-web-client` (Port 3000): Main Shell/Application application. React 16.8, Node 14. Deploys via AWS Amplify. High-priority target for security sanitization due to being stale.
- `vkey2-frontend` (Port 3002): Document management and analytics module. React 18.2, Webpack 5, Node 16 (outdated Docker base). Uses pnpm.
- `user-registry-frontend` (Port 3002): User management interface. React 18.2, Webpack 5, Node 20. Uses npm.

### Backends (Node.js + TypeScript):
- `vendex-backend-service` (Port 3000): Legacy core (VSource/VLink). Koa 2, Knex (Query builder/Migrations), Node 20. Database: PostgreSQL + DynamoDB (Legacy VKey1 data). Uses yarn.
- `vkey2-backend` (Port 8081): Document processing and portfolio analytics (VPort) backend. Express 4, Apollo Server 4 (GraphQL), Prisma ORM, Node 18. Features a read-only direct connection to the `vendex` database.
- `user-registry-backend` (Port 8080): User and auth REST API service. Express 4, Prisma ORM, Node 20.

### Infrastructure (Manual AWS, NO IaC):
- Live workloads run on AWS ECS (managing backend/frontend tasks and service routing).
- AWS S3: Managed object storage with S3 Triggers invoking utility AWS Lambdas.
- DynamoDB: Legacy NoSQL data store containing old VKey1 tables (evaluated as dead weight).
- Secrets Management: AWS Systems Manager Parameter Store (us-east-2 region) holds all DB credentials. The `vendex-infra` repository is completely EMPTY.

## 3. AI BEHAVIORAL RULES (TOKEN EFFICIENCY & SAFETY)
- TOKEN ECONOMY PRINCIPLE: Be extremely concise. Do not include greetings, theoretical introductions, or basic explanations of technologies. Go straight to the point or the specific code diff block.
- ASYNCHRONOUS WITH CURRENT TERRAFORM: Do not suggest modifications to `.tf` files for the current deployment pipeline; all existing live infrastructure was manually provisioned via the AWS Console. Terraform implementations will take place exclusively inside the empty `vendex-infra` repo during a later phase.
- PRESERVE ARCHITECTURAL INTEGRITY: Exercise extreme caution with environment variables injected at build time (Build args) within frontends; changing target environments requires full image rebuilds in this setup.
- ECS/DOCKER COMPOSE ORCHESTRATION ORDER: When writing local startup scripts, adhere strictly to the cluster's topology dependencies: 1st DB (Postgres/Redis) -> 2nd user-registry-backend -> 3rd Core Backends -> 4th Micro-Frontends/Shell.
- SDK FRAGILITY: The `vdx-sdk` library is distributed as raw tarball (`.tgz`) files committed directly to a git release directory and fetched via SSH at build time. Do not suggest installing this package from the public npm registry.
- PACKAGE MANAGER CONSTRAINT: Do not mix commands. Use strictly: `yarn` for `vendex-backend-service`, `pnpm` for `vkey2-frontend`, and `npm` for all other repositories.
- AWS CREDENTIALS BACKUP: Create a `Password.md` file if it does not exist with the credentials for the AWS system. Important: Right now we are making the system local, but in case it's needed to use the AWS credentials in the future only if the user asks to, this is where they are stored. This will make easy to reconnect to the AWS system in case it's needed.

## 4. REPOSITORY MAP & METRICS
| Repository | Type / Role | Size | Last Commit | Status / Note |
| :--- | :--- | :--- | :--- | :--- |
| `vkey2-backend` | Backend (Contracts/Prisma) | 34.2 MB | 2026-01-22 | Active. Primary business engine. |
| `vendex-backend-service` | Backend (Koa/Legacy Core) | 11.0 MB | 2026-01-15 | Active. Catalog & core system. |
| `vkey2-frontend` | Micro-frontend (Contracts) | 6.6 MB | 2026-01-14 | Active. Analytics dashboards. |
| `user-registry-frontend` | Micro-frontend (Users) | 6.8 MB | 2026-01-13 | Active. User administration panel. |
| `user-registry-backend` | Backend (REST/Auth) | 7.9 MB | 2026-01-13 | Active. Permissions & gateway bridge. |
| `vendex-landing` | Marketing Site (Static) | 1.3 MB | 2025-12-18 | Stable. UIKit CSS website. |
| `vendex-website` | Corporate Web | 32.6 KB | 2025-12-03 | Stale. Negligible placeholder repo. |
| `vendex-web-client` | Frontend Shell (Amplify) | 43.2 MB | 2025-12-01 | Heavily outdated base dependencies. Top priority for sanitization. |
| `vendex-infra` | Infrastructure (IaC) | 53.2 KB | 2025-09-19 | Empty. Target repository for final Terraform phase. |
| `vdx-lab` | AWS Utilities & Scripts | 3.2 MB | 2024-09-19 | DB backup/restore code and standalone utilities. |
| `vdx-sdk` | Shared Internal Library | -- | 2024-06-11 | Auth/Cache logic packaged in `.tgz` format. Highly fragile. |

## 5. MIGRATION ROADMAP & CHECKLIST
The developer will update this checklist state using `[x]` as steps are completed with the AI agent. The AI's scope of response must limit itself exclusively to the active sub-phase.
**Important Rule:** At the end of every completed step, the AI agent must append a `*Context Note (Completed):*` section detailing any key discoveries, lessons learned, and configuration changes that will be useful for future steps and repositories.

### PHASE 1: STABLE AND ISOLATED LOCAL ENVIRONMENT (THE PUZZLE)
*Objective: Get the entire ecosystem running 100% locally on the developer's machine using simulated Docker containers, fully protecting AWS live production databases.*
- [x] **Step 1.1: Authentication Foundations (user-registry-backend)**
  - Clone `user-registry-backend`. This is the lightweight user core.
  - Configure `.env` variables pointing strictly to local machine credentials (`localhost`), NEVER to AWS RDS.
  - Provision local PostgreSQL and Redis instances utilizing Docker.
  - *Context Note (Completed):*
    - The original AWS RDS credentials have been safely backed up to `Password.md` (which is now `.gitignore`d).
    - Local `database` container exposes port `5432` (Dev) and `5434` (Test).
    - `npm install` must be run *before* `npx prisma` commands to ensure the compatible Prisma v5.22.0 is used (otherwise npx fetches an incompatible v7+).
    - Added `dotenv-cli` as a dev dependency and replaced `sleep` with a cross-platform Node timeout in `package.json` to ensure Windows compatibility for tests.
- [x] **Step 1.2: The Contract Engine (vkey2-backend)**
  - Clone `vkey2-backend`.
  - Execute `npm run startLocalEnv` to spin up simulated databases and seed development test data locally and securely.
  - *Context Note (Completed):*
    - The repository was safely updated to use `.env` linked to localhost.
    - Prisma versions were strictly pinned to `5.22.0` to avoid conflicts.
    - All other dependencies were updated securely utilizing `npm update --save`.
    - AWS original credentials were saved as placeholders in `Password.md` (which is now `.gitignore`d).
    - Replaced `sleep` inside `package.json` with a cross-platform Node timeout to ensure Windows compatibility.
    - Removed `user-registry.local` AWS ECR dependency from `docker-compose-localhost.yml` as it requires AWS credentials and we are isolating the local environment.
    - Changed Redis port mapping from `6379:6379` to `6380:6379` in `docker-compose-localhost.yml` to resolve conflict with the existing `user-registry-backend` Redis container.
    - Removed the S3 AWS backup download script (`get_and_apply_latest_dev_backup.sh`) from `startLocalEnv`. The local database is now generated and seeded natively via `npx prisma migrate dev` and `npx prisma db seed`.
- [x] **Step 1.3: The Legacy Core (vendex-backend-service)**
  - Clone using Yarn.
  - Link to the existing local database and map required environment variables.
  - *Context Note (Completed):*
    - Created `local-env.env` pointing to local database via `host.docker.internal`.
    - Created `Password.md` and added to `.gitignore`.
- [ ] **Step 1.4: Frontend Layer Coupling (vkey2-frontend & user-registry-frontend)**
  - Clone `vkey2-frontend` (using pnpm) and `user-registry-frontend` (using npm).
  - Correctly map local port bindings (`3002`) to ensure iframe postMessage communication isn't blocked by CORS.
- [ ] **Step 1.5: Global Local Orchestration**
  - Consolidate a master `docker-compose.yml` file to initialize the local architecture in the precise order mandated by the ECS cluster topology.
- [x] **Step 1.6: Package Sanitization (user-registry-backend)**
  - Safely update `user-registry-backend` dependencies using `npm update --save`.
  - Maintain `prisma` strictly pinned at `5.22.0` per previous instructions.
  - Verify integrity through unit tests and local Docker deployment.
- [x] **Step 1.7: Package Sanitization (vkey2-backend)**
  - Safely update `vkey2-backend` dependencies using `npm update --save`.
  - Maintain `prisma` strictly pinned at `5.22.0` per previous instructions.
  - Verify integrity through unit tests and local Docker deployment.
  - *Context Note (Completed):*
    - Dependencies correctly updated to their latest compatible versions within semver ranges.
    - Prisma versions were successfully aligned with the cluster standard.
- [x] **Step 1.8: Package Sanitization (vendex-backend-service)**
  - Safely update `vendex-backend-service` dependencies using `yarn upgrade`.
  - *Context Note (Completed):*
    - Upgraded runtime engines to Node 22 (LTS) and aligned `@types/node` dependency.
    - Moved type definition packages (`@types/*`) from dependencies to devDependencies.
    - Successfully migrated EOL `apollo-server-koa`/`apollo-server-core` to `@apollo/server` and `@as-integrations/koa` (Apollo Server v4), refactoring `src/app.ts`, `src/classes/Response.ts`, `src/resolvers/`, and `src/schema/` to use standard `GraphQLError` and `graphql-tag`.
    - Standardized `@aws-sdk/*` client libraries to a unified version `^3.600.0`.
    - Upgraded legacy `knex` (to `^3.1.0`), `axios` (to `^1.7.9`), and migrated deprecated `vimeo` package to official scoped `@vimeo/vimeo` (`^3.0.0`).
    - Restructured production `Dockerfile` into a secure, optimized multi-stage build, splitting compilation (Stage 1: builder) from execution (Stage 2: production). This removes devDependencies and type configurations from the runtime image, drastically reducing size.
    - Standardized `Dockerfile-dev` to use `node:22-alpine` as base image.
    - Bypassed host local Node engine checks during development compilation using direct `npx tsc` commands.
- [x] **Step 1.9: Initial SDK Context & Local Setup**
  - Read `Context.md` and explore `vdx-sdk` architecture.
  - Create `Password.md` placeholder (no actual RDS credentials in this repo).
  - Start local Redis container `redis.sdk.local` via `docker-compose-test.yml` for testing/development.
  - *Context Note (Completed):*
    - `vdx-sdk` uses Redis locally (port `4379`) for caching via `docker-compose-test.yml`.
    - Cognito Authentication is hardcoded for different environments in `src/cognito-authentication.ts` with no secret keys required here.
    - User Registry domain endpoints are also hardcoded in `src/cognito-authentication.ts`.
    - `release.sh` is currently packing releases into `.tgz` inside a local `./releases` directory, matching the fragile distribution pattern identified in Phase 3.1.
- [x] **Step 1.10: SDK Package Sanitization**
  - Update `vdx-sdk` dependencies safely using `npm update --save`.
  - Ensure SDK unit tests still pass without breaking the cache/authentication features locally.
  - *Context Note (Completed):*
    - Dependencies correctly updated to their latest compatible versions within semver ranges.
    - Jest test suites ran successfully with the local `redis.sdk.local` Docker container, verifying no regressions were introduced.

### PHASE 2: ENVIRONMENT DESKUPGRADES (NODE.JS & REACT)
*Objective: Elevate obsolete code runtimes to modern, security-patched versions.*
- [ ] **Step 2.1: Sanitizing the Shell Application (vendex-web-client)**
  - Upgrade the main shell client from Node 14 / React 16.8 to Node 20 / React 18.
  - Resolve breaking changes across legacy trees (specifically the AWS Amplify SDK and Material-UI v4) while preserving original visual layouts.
- [ ] **Step 2.2: Modernizing the Contracts Container (vkey2-frontend)**
  - Update the outdated `node:16-alpine` base image in the Dockerfile to `node:20-alpine` to standardize cluster deployment models.
- [ ] **Step 2.3: Cross-Version Integration Testing**
  - Verify inside local Docker containers that runtime upgrades did not break token-passing mechanisms between the shell application and embedded iframes.

### PHASE 3: CLEANUP, SECURITY PATCHING & SANITIZATION
*Objective: Remove dead code paths and address vulnerabilities in code distribution channels.*
- [ ] **Step 3.1: Stabilizing SDK Package Distribution**
  - Deprecate the fragile practice of fetching `.tgz` files via SSH from the `vdx-sdk` repo.
  - Configure a secure local or private npm package registry to stabilize builds across deployment pipelines.
- [ ] **Step 3.2: Code Dead-Weight Removal**
  - Review remaining dependencies on DynamoDB (VKey1) to establish their complete deprecation from backend initialization files.
- [ ] **Step 3.3: Resolving Critical Bug VEN-398**
  - Isolate and fix the legacy contract processing bug in production utilizing the newly modernized local architecture.

### FASE 4: AUTOMATION & INFRASTRUCTURE AS CODE (TERRAFORM)
*Objective: Translate manual AWS environments into fully repeatable code blueprints without interfering with live production systems.*
- [ ] **Step 4.1: Passive AWS Account Audit (625262942667)**
  - Map out current live cloud configurations across ECS task definitions, RDS clusters, S3 bucket triggers, and standalone Lambdas via the AWS Console.
- [ ] **Step 4.2: Writing Modular Terraform Blueprints**
  - Author reusable Infrastructure as Code (IaC) structures using HashiCorp Configuration Language (HCL) within the empty `vendex-infra` repo.
- [ ] **Step 4.3: Secure CI/CD Infrastructure Pipeline Deployment**
  - Integrate Terraform execution inside current AWS CodePipeline/CodeBuild targets to automate sandboxed environments (`develop` and `staging`) before running on live `production` layers.