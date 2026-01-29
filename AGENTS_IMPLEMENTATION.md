# Sister 2 – AI Agent Implementation Operating Rules

## 1. Purpose
This document defines how AI agents must behave during the implementation phase.

AI is allowed to generate code under strict governance rules.
Architecture decisions remain human-controlled.

---

## 2. Implementation Phase Objectives

Primary goals:
- Deliver production-ready code
- Maintain architectural integrity
- Ensure security and maintainability
- Prevent AI-induced technical debt

---

## 3. AI Responsibility Levels

### Allowed
- Generate code snippets and modules
- Write unit and integration tests
- Refactor code for readability and performance (within constraints)
- Detect bugs and security issues
- Propose optimizations with justification

### Forbidden Without Explicit Permission
- Changing architecture or infrastructure choices
- Introducing new frameworks, libraries, or services
- Modifying authentication, security, or data models
- Large-scale refactors across modules
- Deploying or executing production changes

---

## 4. Architecture Lock (Non-Negotiable)

### Core Stack
- Frontend: SvelteKit on Vercel
- Backend: FastAPI on AWS Lambda
- Batch/AI Jobs: AWS Fargate
- Database: Neon PostgreSQL
- Storage: AWS S3
- Authentication: AWS Cognito
- Notifications: AWS SES/SNS
- AI: OpenAI API

### Enforcement Rules
- Lambda = request/response APIs only
- Fargate = long-running / batch / AI workloads
- Serverless-first design

---

## 5. Coding Standards

AI must:
- Follow project coding conventions
- Write type-safe and lint-compliant code
- Add comments for non-obvious logic
- Avoid premature optimization
- Prefer explicit over implicit logic

---

## 6. Security and Compliance Rules

AI must NOT:
- Hardcode secrets or credentials
- Log sensitive data
- Bypass authentication/authorization layers
- Introduce insecure defaults

All security-relevant changes must be flagged as:
[Security Risk]

---

## 7. Change Governance

All AI-generated changes must include:
- Purpose
- Impact
- Risk assessment

Use labels:
[Implementation]
[Refactor]
[Bug Fix]
[Optimization]
[Security Risk]
[Open Question]

---

## 8. Human-in-the-Loop Requirement

AI outputs must be:
- Reviewed by a human developer
- Approved via PR or design review artifact

AI must never merge, deploy, or execute changes autonomously.

---

## 9. Logging and Traceability

AI-generated code must:
- Be traceable in commit history or PR comments
- Include AI attribution in commit/PR notes when possible

---

## 10. Long-Term Maintainability Principles

AI should:
- Avoid clever but opaque code
- Favor maintainable patterns
- Preserve extensibility for future AI-driven workflows

---

## 11. Final Rule

When uncertain:
> Ask before coding.

Correctness, maintainability, and security override speed.