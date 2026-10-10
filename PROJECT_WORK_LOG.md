# Alex Project Work Log

This document summarizes the planning, implementation, iteration, debugging, and final project state for Alex, an agentic AI financial-planning platform. It is intended as an evidence-backed supplement to a resume or interview submission.

## Project Outcome

Alex is a multi-agent SaaS platform that analyzes equity portfolios and produces narrative reports, visualizations, and retirement projections. The system combines five specialized agents with AWS serverless infrastructure, retrieval-augmented market knowledge, authentication, monitoring, and validation controls.

**Primary stack:** Python, OpenAI Agents SDK, Amazon Bedrock/Nova Pro, Amazon SageMaker, AWS Lambda, SQS, Aurora Serverless PostgreSQL, S3 Vectors, API Gateway, CloudFront, Next.js, FastAPI, Terraform, Docker, Clerk, LangFuse, Playwright.

## 1. Planning and Architecture

The implementation was planned as an incremental eight-stage delivery:

1. IAM and AWS permissions
2. Serverless SageMaker embeddings
3. S3 Vectors ingestion and semantic search
4. Autonomous market researcher
5. Aurora Serverless database
6. Five-agent orchestration
7. Authenticated Next.js frontend and FastAPI API
8. Enterprise monitoring, guardrails, explainability, and observability

Key design decisions documented in the guides:

- Use specialized agents for classification, reporting, chart generation, retirement analysis, and orchestration instead of one general-purpose agent.
- Decouple long-running analysis with SQS and persist job state and agent outputs in Aurora.
- Use SageMaker for 384-dimensional embeddings and S3 Vectors for lower-cost semantic retrieval.
- Use the Aurora Data API to avoid Lambda VPC and connection-pool complexity.
- Run independent specialist agents in parallel where possible.
- Keep structured-output agents separate from tool-using agents because of the documented LiteLLM/Bedrock compatibility constraint.

References: [gameplan.md](gameplan.md), [guides/architecture.md](guides/architecture.md), [guides/agent_architecture.md](guides/agent_architecture.md), [guides/6_agents.md](guides/6_agents.md).

## 2. Implementation

- Implemented five agent roles: Planner, InstrumentTagger, Report Writer, Chart Maker, and Retirement Specialist.
- Built the ingestion path from API Gateway to Lambda, SageMaker embeddings, and S3 Vectors.
- Built the researcher workflow with Playwright-based web research, Bedrock generation, and knowledge-base population.
- Added Aurora models and migrations for users, accounts, positions, instruments, jobs, reports, charts, and retirement results.
- Built a Next.js static frontend with Clerk authentication, portfolio management, analysis status, markdown reports, charts, and retirement projections.
- Added Terraform for independently deployable infrastructure stages and S3-backed Lambda packages.
- Added CloudWatch dashboards, structured logging, Lambda metrics, Bedrock/SageMaker latency views, and LangFuse integration hooks.

References: [backend/planner/lambda_handler.py](backend/planner/lambda_handler.py), [backend/database/migrations/001_schema.sql](backend/database/migrations/001_schema.sql), [backend/api/main.py](backend/api/main.py), [terraform/6_agents/main.tf](terraform/6_agents/main.tf), [terraform/8_enterprise/main.tf](terraform/8_enterprise/main.tf).

## 3. Iteration and Refinement

Git history records several meaningful refinements:

- `68f3e0c` migrated the researcher deployment toward container-image Lambda and improved Playwright MCP reliability.
- `c3e5d73` improved Windows compatibility in local development and npm process handling.
- `a53643b` improved Windows compatibility for packaging and database scripts.
- `78a238e` fixed Windows compatibility for deployment and npm commands.
- `76df3ce` merged the Windows compatibility work into the main branch.

These changes show an iterative path from the initial architecture toward more portable deployment and more reliable research execution.

## 4. Testing and Debugging Evidence

The repository includes:

- Local tests for all five agents using `MOCK_LAMBDAS=true`; the documented harness reports `5/5` local agent flows passing.
- Full SQS integration tests that wait for job completion and verify reports, charts, and retirement output.
- Multiple-account tests covering three accounts and ten positions.
- A scale-test harness that creates five users with varied portfolio sizes, sends jobs to SQS, monitors completion, records failures/timeouts, and cleans up test data.
- Database migration, seed, connection, allocation, and reset tests.
- CloudWatch log-watching utilities for agent debugging.

### Load-test investigation

The requested p95 latency, throughput, and error-rate figures were not claimed because the scale test could not reach the database. The observed failure was:

> The configured Aurora ARN referenced account `123456789012`, while the active AWS account was `209211310025`.

Therefore, the verified scale statement is:

> The project includes a five-user concurrent portfolio-analysis scale-test scenario covering portfolios with 0-10 positions across 1-3 accounts, with completion, failure, timeout, and cleanup handling.

No measured p95 latency, throughput, or production error rate is included until the Aurora configuration is corrected and the test produces timestamped results.

References: [backend/test_simple.py](backend/test_simple.py), [backend/test_full.py](backend/test_full.py), [backend/test_multiple_accounts.py](backend/test_multiple_accounts.py), [backend/test_scale.py](backend/test_scale.py), [guides/6_agents.md](guides/6_agents.md).

## 5. Enterprise Capabilities

Implemented or code-backed capabilities include:

- Clerk JWT/JWKS authentication and user-context validation
- IAM policies and service-to-service permissions
- API Gateway authentication/throttling patterns and CORS controls
- Secrets Manager integration for database credentials
- Pydantic request validation
- Structured agent outputs and output-quality checks
- Reporter evaluation with fallback behavior
- Tagger allocation validation
- Retry handling for transient agent failures
- CloudWatch dashboards for invocations, duration, errors, throttles, concurrency, and model latency
- LangFuse and Logfire instrumentation hooks for agent traces
- SQS-based asynchronous processing and failure handling

The enterprise guide also proposes WAF, VPC endpoints, GuardDuty, stronger prompt-injection sanitization, chart-output validation, response truncation, and audit logging. These should be described as planned or recommended unless deployment evidence is available.

## 6. Metrics and Claims Discipline

| Metric or claim | Evidence status |
|---|---|
| Five specialized agents | Implemented architecture |
| Five concurrent scale-test users | Executable test scenario; live run blocked by Aurora ARN mismatch |
| 384-dimensional embeddings | Model/configuration target and raw artifact present |
| 22 seeded ETFs | Implemented seed scenario |
| 500 Monte Carlo simulations | Code-configured behavior |
| Millions of vectors | Documented design target |
| 90% vector-storage savings | Documented estimate, not a measured billing result |
| p95 latency, throughput, error rate | Not yet measured successfully |
| 10,000 requests/second, 5,000 burst, 128 ACUs | Documented AWS/design targets, not observed load-test results |

## 7. Available Chat Transcript

The local Copilot session store contains the following project-specific exchange. No earlier Alex implementation chat was available in the local index, so the Git history and repository artifacts above provide the development trail.

### Resume drafting

**User:** Requested that Alex be added to an 8+ years AI/ML resume with impact and scale metrics for top product companies.

**Assistant:** Drafted an ATS-friendly project entry emphasizing five agents, AWS orchestration, RAG with SageMaker and S3 Vectors, Bedrock, enterprise controls, and documented capacity targets. It explicitly warned that capacity targets should not be presented as measured production traffic.

### Metrics correction

**User:** Requested replacing capacity numbers with p95 latency, throughput, error rate, and concurrent-user load-test results.

**Assistant:** Inspected [backend/test_scale.py](backend/test_scale.py), confirmed that the script covers five concurrent users but does not calculate p95 latency, throughput, or error rate, then ran it from the backend project. The run failed before measurement because of the Aurora ARN/account mismatch described above. The capacity claims were consequently removed from the resume wording.

## Recommended Resume Wording

> Architected a production-oriented multi-agent financial planning platform with five specialized AI agents for portfolio classification, market research, reporting, visualization, and retirement Monte Carlo simulations. Built asynchronous AWS orchestration with Lambda, SQS, Aurora Serverless, API Gateway, and CloudFront; implemented retrieval-augmented market intelligence using SageMaker embeddings and S3 Vectors; and added Clerk JWT authentication, IAM controls, validation, retries, CloudWatch dashboards, and LangFuse tracing. Developed a five-user concurrent scale-test harness covering portfolios with 0-10 positions across 1-3 accounts, including completion, failure, timeout, and cleanup monitoring.

This wording is intentionally limited to claims supported by code, documentation, Git history, or the recorded test attempt. Replace the final sentence with measured p95, throughput, and error-rate values after a successful live run.