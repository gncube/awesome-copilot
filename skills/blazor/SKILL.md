---
name: blazor
description: "Guides creating and customizing Blazor applications and development workflows, including scaffolding, hosting models, debugging, testing, performance, and CI/CD best practices."
tags:
  - blazor
  - dotnet
  - wasm
  - server
  - frontend
---

## Purpose

This skill captures a reusable, step-by-step workflow for developing, iterating, and shipping Blazor applications (Blazor WebAssembly and Blazor Server). It codifies decision points, quality criteria, and example prompts so you can consistently scaffold apps, choose hosting and state strategies, set up debugging and tests, and prepare production builds and CI/CD.

## Scope

- Workspace-scoped: useful for engineering teams building Blazor apps in this repository.
- Covers project setup, architecture decisions, development practices, and release steps. Not a substitute for full framework docs, but a practical companion.

## Workflow (step-by-step)

1. Define goals and constraints
   - Clarify target: SPA, PWA, interactive admin panel, or hybrid.
   - Constraints: supported browsers, offline needs, server resources, authentication provider.

2. Choose hosting model
   - Blazor WebAssembly: client-side execution, smaller server footprint, suitable for offline or CDN hosting.
   - Blazor Server: low-download, server-rendered UI with SignalR, better for low-capable clients.
   - Hybrid (hosted WASM + Server APIs): use WASM for UI and server for heavy workloads.

3. Scaffold the project
   - Use `dotnet new blazorwasm`, `dotnet new blazorserver`, or `dotnet new blazorwasm --hosted`.
   - Add solution organization: `src/` (apps), `libs/` (shared components), `tests/`.

4. Architect components and state
   - Favor small, testable components with clear inputs/outputs.
   - Choose state strategy: cascading parameters, `IStateProvider` services, or third-party libraries (Fluxor, Redux-like patterns).

5. Styling and component library
   - Pick CSS strategy: scoped CSS, CSS isolation, utility frameworks (Tailwind), or component libraries (MudBlazor, Radzen).

6. Authentication & Authorization
   - Decide provider: Azure AD / Entra, IdentityServer, OAuth/OIDC, or custom JWT APIs.
   - Implement secure flows and token refresh strategies (for WASM use `IAccessTokenProvider`).

7. Local dev & debugging
   - Enable browser debugging for WASM: use source maps and `dotnet watch`.
   - For server, use standard ASP.NET Core debugging and SignalR diagnostics.

8. Testing
   - Unit test components with bUnit.
   - Integration tests for pages and APIs; e2e with Playwright or Selenium.

9. Performance & accessibility
   - Measure bundle size (WASM): enable trimming, AOT where appropriate.
   - Audit with Lighthouse for performance and accessibility.

10. CI/CD and publishing
    - Build pipeline: `dotnet restore`, `dotnet build`, `dotnet test`, `dotnet publish`.
    - For WASM, serve from a CDN or static site hosting (Azure Static Web Apps, S3+CloudFront). For Server, deploy to App Service, AKS, or Container Apps.

## Decision points and branching logic

- Hosting model: performance vs latency vs offline capabilities.
- State management: global store vs localized state; tradeoffs in testability and complexity.
- Component library: design consistency vs bundle size.
- Authentication choices driven by corporate identity and SSO requirements.

## Quality criteria / completion checklist

- Project builds cleanly with `dotnet build` and `dotnet publish`.
- Unit tests (`dotnet test`) pass and component tests cover critical UI paths.
- Bundle size and load times are within target budgets; A/B profiling shows acceptable latency.
- Accessibility score meets baseline (Lighthouse > 90 for accessibility or project target).
- CI pipeline builds, tests, and deploys to staging automatically on PR merge.

## Outputs

- Scaffolding commands and project layout
- A short checklist for PR reviewers to validate Blazor-specific concerns
- Example CI job snippets for building and deploying Blazor apps

## Example prompts to use this skill

- "Scaffold a Blazor WebAssembly app with authentication using Azure AD and a shared component library."
- "Recommend a state management strategy for a large Blazor app with complex forms and offline sync."
- "Generate a CI job (GitHub Actions) that builds, tests, and publishes a Blazor WebAssembly app to Azure Static Web Apps."

## Iterate and refine

1. Draft the SKILL usage in a new repository folder and try the example prompts.
2. Identify ambiguous choices or missing platform details (e.g., hosting provider, SSO type).
3. Ask targeted clarifying questions, then update this SKILL.md with concrete commands, CI snippets, or organization policies.

## Related customizations to create next

- `blazor-ci` skill: opinionated CI/CD templates for GitHub Actions and Azure Pipelines.
- `blazor-perf-audit` skill: focused checklist and automation for trimming and bundle optimizations.
- `blazor-testing` skill: templates and examples for bUnit and Playwright tests.

If any part should be more prescriptive (example pipeline YAML, exact commands, or templates), tell me which area to expand and I will add concrete artifacts.
