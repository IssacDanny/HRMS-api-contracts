# API Contracts & Governance Repository

> **"Consistency is the last refuge of the unimaginative, but the first requirement of the professional."**

## 📖 Overview

This repository serves as the **Single Source of Truth** for all API specifications within the AI Call Center ecosystem. It embodies the "API First" philosophy, ensuring that the contract between Backend and Frontend is defined, validated, and agreed upon before a single line of implementation code is written.

**Owner:** Dinh Gia Kiet 
**Enforcement:** Automated via [Spectral](https://stoplight.io/open-source/spectral)
**Documentation:** Generated via [Redocly](https://redocly.com/)

---

## 🛠 Prerequisites

To interact with this repository, your development environment must be equipped with:

*   **Node.js** (v20 or higher recommended)
*   **NPM** (v10 or higher)
*   **Git**

---

## 🚀 Getting Started

### 1. Installation
Clone the repository and install the necessary tooling. The system will automatically initialize the **Husky** git hooks to ensure quality control.

```bash
git clone <repository-url>
cd api-contracts
npm install
```

### 2. The Directory Structure
We adhere to a strict modular organization to promote reuse and clarity.

```text
api-contracts/
├── .github/             # CI/CD Workflows (Linting & Publishing)
├── .husky/              # Git Hooks (Pre-commit enforcement)
├── docs/                # Generated HTML documentation (Artifacts)
├── rulesets/            # The Law (Spectral Linter Rules)
│   └── .spectral.yaml   # The active ruleset
├── specs/               # The Blueprints
│   ├── _shared/         # Reusable components (Security, Errors, Pagination)
│   ├── v1/              # Version 1.0 Specifications
│   │   └── hrms-api.yaml
│   └── template.yaml    # The Golden Template for new services
└── package.json         # Scripts and Dependencies
```

---

## ⚡ Available Commands

A gentleman does not memorize complex arguments. Use these standardized scripts:

| Command | Description |
| :--- | :--- |
| `npm run lint` | Checks the OpenAPI file (hrms-api.yaml) for syntax errors and style violations using Spectral rules. |
| `npm run docs:build` |Generates a standalone, fully interactive HTML documentation file in the docs/ folder. |
| `npm run docs:watch` |Starts a local development mode that automatically rebuilds your documentation every time you save changes to the YAML file. |
| `npm run bundle` | Combines all `$ref` imports into a single, massive JSON file (useful for code generation). |

---

## 📐 The Workflow

### Creating a New API
Do not start from a blank slate. We have provided a **Golden Template** pre-wired with our security and pagination standards.

1.  Copy the template:
    ```bash
    copy specs/template.yaml specs/v1/my-new-service.yaml
    ```
2.  Define your paths and schemas.
3.  Run `npm run docs:preview` to visualize your work.

### Modifying an Existing API
1.  Create a new branch (e.g., `feature/add-payroll-export`).
2.  Edit the YAML file in `specs/v1/`.
3.  **Crucial:** Run `npm run lint`. If Sterling (the linter) complains, you must fix the errors.
4.  Commit your changes. *Note: The pre-commit hook will block you if the linter fails.*
5.  Push and open a Pull Request.

---

## 🤖 CI/CD Pipeline

When code is merged into the `main` branch, the **GitHub Actions** pipeline will automatically:

1.  **Validate:** Ensure the spec is valid OpenAPI 3.0.
2.  **Bundle:** Compile the YAML and shared components into a single `openapi.json`.
3.  **Publish:** Deploy the HTML documentation to **GitHub Pages**.

**Accessing the Docs:**
*   **Human Readable:** `https://issacdanny.github.io/Human-Resoure-Management-System/`
*   **Machine Readable:** `https://issacdanny.github.io/Human-Resoure-Management-System/openapi.json`

---

## 🤝 Contribution

*   **Backend Developers:** You are responsible for defining the schema *before* you code.
*   **Frontend Developers:** You should review the PRs to ensure the data structure meets UI requirements.
*   **Architect:** Final approval on all API changes.

