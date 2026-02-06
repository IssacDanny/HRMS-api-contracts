# API Design & Documentation Tooling Standard

**Owner:** Dinh Gia Kiet
**Date:** 06/02/2026

## 1. Purpose

This document outlines the official set of tools and the standard workflow for designing, documenting, mocking, and validating our APIs. This toolchain is designed to directly support our [API Specification Standard](./api-specification-standard.md) and enable our API-first development process.

Our chosen approach is a **Best-of-Breed Stack**, which combines powerful, open-source tools with our existing Git-based workflow. This provides maximum flexibility, control, and is highly cost-effective.

## 2. The Official Toolchain

The following tools are the official standard for each part of the API lifecycle.

### A. Design & Editing

*   **Tool:** **Visual Studio Code (VS Code)**
*   **Extension:** **OpenAPI (Swagger) Editor**
*   **Why:** We will use VS Code as our primary editor for `openapi.yaml` files. This extension provides rich autocompletion, real-time validation against the OpenAPI 3.0 schema, and a live preview panel. It allows our engineers to work in a familiar, powerful environment.

### B. Documentation & Sharing

*   **Tool:** **Redoc**
*   **Why:** Redoc generates clean, modern, and highly readable three-panel documentation from an OpenAPI file. It is ideal for our API consumers.
*   **Workflow:** Redoc will be integrated into our CI/CD pipeline. On every merge to the `main` branch of our `api-contracts` repository, the pipeline will automatically generate a static HTML documentation site and deploy it to a known URL (e.g., `https://docs.api.example.com`).

### C. Mocking & Parallel Development

*   **Tool:** **Stoplight Prism**
*   **Why:** Prism is a powerful, open-source mock server that can be run from the command line. It reads our `openapi.yaml` file and instantly provides a live API that returns example responses and validates incoming requests.
*   **Workflow:** Frontend and mobile developers can run Prism locally, pointing it at any branch of the `api-contracts` repository. This completely decouples them from the backend development schedule.

### D. Validation & Governance

*   **Tool:** **Spectral**
*   **Why:** Spectral is a flexible linter for JSON/YAML that can be configured to enforce the specific rules in our [API Style Guide](./api-style-guide.md).
*   **Workflow:** Spectral will be integrated into our CI/CD pipeline. When a pull request is opened with changes to an `openapi.yaml` file, Spectral will automatically run. If the API design violates any style guide rules (e.g., incorrect naming conventions, missing fields in error responses), the build will fail, and the pull request will be blocked until it is fixed.

## 3. The Standard Workflow in Practice

1.  **Design:** An engineer creates a new branch in the `api-contracts` repository. They design the new API endpoint in VS Code, getting real-time feedback from the OpenAPI extension.
2.  **Mock & Test:** The engineer (or a frontend developer) can immediately run `prism mock <path-to-file.yaml>` to get a live mock server for local development and testing.
3.  **Review:** The engineer opens a Pull Request. The team reviews the proposed API changes directly in the `yaml` file.
4.  **Automated Validation:** The CI pipeline automatically runs **Spectral** on the Pull Request. It must pass for the PR to be mergeable.
5.  **Merge & Deploy Docs:** Upon merging, the CI pipeline automatically runs **Redoc** to build and deploy the updated documentation site. The new endpoint is now officially documented and available for all consumers.

## 4. Next Steps

*   The platform team will create a shared Spectral ruleset file in the `api-contracts` repository that codifies our API Style Guide.
*   The DevOps team will configure the CI/CD pipeline to integrate Spectral and Redoc as described above.
*   All engineers should install the recommended VS Code extension and familiarize themselves with the Prism command-line tool.