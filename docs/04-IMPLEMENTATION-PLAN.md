# API-First Implementation Plan

**Owner:** Dinh Gia Kiet
**Date:** 06/02/2026

## 1. Purpose

This document outlines the step-by-step workflow our team will follow to design, develop, and test new features using the API-first approach. The goal is to create a clear, repeatable process that maximizes our team's collaboration and efficiency.

By following this plan, we will:
*   Eliminate ambiguity by agreeing on the API contract *before* writing code.
*   Enable parallel work, allowing backend and frontend/consuming-service development to happen simultaneously.
*   Empower our QA to design and write tests earlier in the development cycle.
*   Produce higher-quality, well-documented, and consistent APIs.

## 2. Roles in the API-First Workflow

We will use the following designations for a given feature:

*   **Contract Author (2 Dev):** The developer primarily responsible for the backend implementation. Their first task is to write the initial `openapi.yaml` contract for the new feature.
*   **Contract Consumer (2 Dev):** The developer who will be using the API (e.g., building the frontend UI or integrating from another service). They are the primary reviewer of the contract.
*   **Contract Tester (QC):** The QA engineer responsible for ensuring the API is testable and meets requirements. They are a key reviewer and the creator of the test suite.

## 3. The Feature Development Workflow

This workflow is followed for every new feature or significant change.

### Phase 1: Design & Contract Review (The Blueprint Phase)

**Goal: To create and agree upon a final API contract before any implementation code is written.**

1.  **Kickoff (15 mins, All Hands):** The team meets to briefly discuss the feature requirements. The goal is to have a shared understanding of the business logic and the data involved.

2.  **Draft the Contract (Contract Author):**
    *   Creates a new feature branch in the `api-contracts` Git repository.
    *   Using VS Code, writes the `openapi.yaml` definition for the new endpoints. This includes paths, parameters, request bodies, and response schemas for **both success and error cases**.
    *   This initial draft should be based on the kickoff discussion and our API Style Guide.

3.  **The Contract Review (All Hands):**
    *   The Contract Author opens a Pull Request in the `api-contracts` repository.
    *   This PR is the central point for discussion. The team reviews the YAML file with the following focus:
        *   **Contract Consumer (Dev):** "Does this API provide all the data I need? Is the naming clear? Is the request structure easy for me to build?"
        *   **Contract Tester (QC):** "Are the error responses well-defined? Can I test all the edge cases based on this contract? Are there any potential security or performance concerns?"
        *   **Contract Author (Dev):** Guides the discussion and incorporates feedback by pushing updates to the PR.
    *   **Automated Linter:** Our CI pipeline automatically runs **Spectral** on the PR to enforce style guide rules. The linter must pass.

4.  **Approve and Merge:** Once all three team members have approved the PR, it is merged into the `main` branch.
    *   **Automation:** This merge automatically triggers the deployment of the updated documentation via **Redoc**.

### Phase 2: Parallel Development (The Efficiency Phase)

**Goal: All four developers work simultaneously, unblocked by each other.**

*   **Contract Author (Backend Dev):**
    *   Creates a feature branch in the application's code repository.
    *   Writes the server-side logic (controllers, services, database interactions) to implement the API exactly as defined in the merged contract.

*   **Contract Consumer (Frontend Dev):**
    *   Creates a feature branch in their client/application repository.
    *   Runs **Stoplight Prism** locally, pointing it at the final `openapi.yaml` file from the `main` branch: `prism mock api-contracts/your-service.yaml`.
    *   This instantly creates a mock server at `http://localhost:4010`.
    *   They build the UI or service integration by making real HTTP calls to this mock server, confident that the responses will match the final API.

*   **Contract Tester (QC):**
    *   Opens Postman (or their testing tool of choice).
    *   Using the now-final contract as a guide, they create a new test collection.
    *   They write automated tests for the happy path, all defined error codes (`400`, `404`, etc.), and any security or validation rules. These tests are written *before* the backend implementation is complete.

### Phase 3: Integration, Testing, and Deployment

**Goal: To seamlessly integrate the new components and verify the implementation against the contract.**

1.  **Backend Deployment:** The Contract Author finishes their implementation and deploys it to a `dev` or `staging` environment.

2.  **The "Switch":** The Contract Consumer updates their application's configuration to point from the local Prism mock server (`http://localhost:4010`) to the real `staging` environment URL. Since both were built from the same contract, this integration should be smooth.

3.  **Execute Tests:** The Contract Tester runs their pre-written automated test suite against the `staging` environment.
    *   The primary goal is to verify that the real implementation perfectly matches the contract. Any deviations are treated as high-priority bugs.

4.  **Review and Deploy:** Once all tests pass and the team has done a final review, the feature is approved and deployed to production.

---

### Example in Practice: Adding a User Profile

*   **Phase 1:**
    1.  We agree we need a `GET /users/{userId}/profile` endpoint.
    2.  **Dev1 (Author)** drafts the OpenAPI spec, including a `200 OK` response with `fullName` and `bio`, and a `404 Not Found` error response.
    3.  In the PR review, **Dev2 (Consumer)** requests an `avatarUrl` field be added to the `200 OK` response. **QA** asks for a `403 Forbidden` response to be defined for private profiles. Dev1 adds these and the PR is merged.

*   **Phase 2:**
    1.  **Dev1** starts writing the backend Java/Python/Node.js code to fetch the profile from the database.
    2.  **Dev2** runs `prism mock ...` and gets a mock server that returns `{"fullName": "...", "bio": "...", "avatarUrl": "..."}`. They build the React profile component against this mock data.
    3.  **QA** builds a Postman collection with requests to test the `200`, `404`, and `403` responses.

*   **Phase 3:**
    1.  Dev1 deploys the backend to staging.
    2.  Dev2 points the React app to the staging URL. The profile page now shows real data.
    3.  QA runs the Postman collection. All tests pass.
    4.  The feature is ready for release.