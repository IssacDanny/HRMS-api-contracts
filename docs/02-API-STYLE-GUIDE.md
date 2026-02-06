# API Design Standard & Style Guide

**Owner:** Dinh Gia Kiet
**Date:** 06/02/2026

## Foundational Design Standards

This section defines the core, non-negotiable standards for all our APIs. These rules are the practical application of our Guiding Principles, designed to ensure consistency, predictability, and a high-quality developer experience.

### A. URL Structure & Naming
*This section supports our Principle of Predictability.*

The URL is the most fundamental part of the API's interface. A clean, logical, and consistent URL structure is essential for an intuitive API.

**Rules:**
1.  **Use Plural Nouns for Resources:** Collections of resources should be named with plural nouns.
    *   **Good:** `/users`, `/orders`, `/shipping-addresses`
    *   **Bad:** `/user`, `/orderList`, `/shippingAddress`
2.  **Use Resource IDs for Specific Instances:** To access a specific instance of a resource, append its unique identifier to the collection URL.
    *   **Good:** `/users/{userId}`, `/orders/{orderId}`
    *   **Bad:** `/getUserById/{userId}`
3.  **Use kebab-case for Path Segments:** All path segments should be lowercase and use hyphens (-) to separate words.
    *   **Good:** `/user-profiles/{profileId}/shipping-address`
    *   **Bad:** `/userProfiles`, `/user_profiles`, `/UserProfiles`
4.  **Nest Resources to Show Relationships:** For resources that have a clear parent-child relationship, express this through URL nesting. Limit nesting to one level for clarity.
    *   **Good:** `/users/{userId}/orders` (Get orders for a specific user)
    *   **Bad:** `/orders?userId={userId}` (Less discoverable)
    *   **Avoid:** `/users/{userId}/orders/{orderId}/line-items` (Overly complex)
5.  **Do Not Use Verbs in Resource URLs:** The action should be expressed by the HTTP method (GET, POST, PUT, PATCH, DELETE), not the URL.
    *   **Good:** `POST /users`
    *   **Bad:** `POST /create-user`

### B. Data Formats & JSON Properties
*This section supports our Principle of Predictability and Consumer-First Design.*

The data we exchange is the heart of the API. A consistent data format ensures that consumers can reliably work with our request and response payloads.

**Rules:**
1.  **Use `application/json` Exclusively:** All API requests and responses that contain a body MUST use JSON and set the `Content-Type` header to `application/json`.
2.  **Use camelCase for Property Keys:** All keys within JSON objects must be written in camelCase.
    *   **Good:** `{"firstName": "Alice", "orderDate": "..."}`
    *   **Bad:** `{"first_name": "Alice"}`, `{"FirstName": "Alice"}`
3.  **Use ISO 8601 for Dates and Times:** All timestamps must be strings formatted in ISO 8601 and should include the timezone designator (Z for UTC).
    *   **Good:** `{"createdAt": "2023-10-27T10:00:00Z"}`
    *   **Bad:** `{"createdAt": "10/27/2023"}`, `{"createdAt": 1698397200}`
4.  **Use `null` for Empty Values:** If a field has no value, it should be explicitly set to `null` instead of being omitted. This provides clarity to strongly-typed clients.
    *   **Good:** `{"middleName": null, "processingDate": "2023-10-27T10:00:00Z"}`
    *   **Bad:** `{"processingDate": "2023-10-27T10:00:00Z"}` (The client doesn't know if `middleName` is missing or just has no value).

### C. Standard Error Responses
*This section supports our Principle of Predictability and Consumer-First Design.*

A predictable error handling strategy is critical for a robust integration. Consumers must be able to programmatically handle our errors. Plain text or HTML error messages are forbidden.

**Rules:**
1.  **Use Standard HTTP Status Codes:** Use `4xx` codes for client errors and `5xx` codes for server errors. Be specific (e.g., use `404 Not Found` instead of a generic `400 Bad Request` if a resource doesn't exist).
2.  **Provide a Consistent JSON Error Body:** All `4xx` and `5xx` responses MUST return a JSON object that adheres to the following schema.

**Standard Error Schema:**
```json
{
  "traceId": "string",  // A unique ID for the request, for correlation in logs.
  "code": "string",     // A stable, machine-readable error code.
  "message": "string",  // A human-readable summary of the error.
  "details": [          // Optional: An array of specific field-level errors.
    {
      "field": "string", // The name of the field that failed validation.
      "issue": "string"  // The specific reason for the failure.
    }
  ]
}
```

**Example 400 Bad Request Response:**
```http
HTTP/1.1 400 Bad Request
Content-Type: application/json

{
  "traceId": "a1b2c3d4-e5f6-7890-1234-567890abcdef",
  "code": "validation_error",
  "message": "The request body failed validation.",
  "details": [
    {
      "field": "emailAddress",
      "issue": "Must be a valid email format."
    },
    {
      "field": "password",
      "issue": "Must be at least 10 characters long."
    }
  ]
}
```

### D. Versioning Strategy
*This section supports our Principle of Stability and Graceful Evolution.*

Our versioning strategy is our promise to our consumers that we will not break their applications. It allows us to evolve our APIs safely and transparently.

**Rules:**
1.  **Use URL-based Versioning:** The API version will be included as the first segment in the URL path, prefixed with a "v".
    *   **Good:** `https://api.example.com/v1/users`
    *   **Bad:** Using headers or query parameters for versioning.
2.  **A New Version is Required for Breaking Changes:** Any change that is not backward-compatible is a "breaking change" and requires a new major version (e.g., moving from `v1` to `v2`).
    *   **Definition of a Breaking Change:**
        *   Removing or renaming a field in a JSON response.
        *   Changing the data type of a field (e.g., string to number).
        *   Adding a new required field to a request body.
        *   Changing a URL path.
        *   Changing an authentication or authorization mechanism.
3.  **Non-Breaking Changes Do Not Require a New Version:** The following changes are considered backward-compatible and can be made to an existing API version:
    *   Adding a new API endpoint.
    *   Adding a new optional field to a request body.
    *   Adding a new field to a response body.

---

## Detailed Design Standards

This section builds upon our foundational standards to cover more advanced topics. Adhering to these guidelines will ensure our APIs are not just consistent, but also secure, performant, and easy to work with in complex scenarios.

### E. Authentication & Authorization
*This section supports our Principle of Security by Design.*

A clear and consistent security model is non-negotiable. Every developer, both internal and external, must know exactly how to securely interact with our APIs.

**Rules:**
1.  **Use OAuth 2.0 Bearer Tokens:** All endpoints requiring authentication MUST be secured using the OAuth 2.0 framework with Bearer Tokens.
2.  **Use the `Authorization` Header:** The access token MUST be sent in the standard `Authorization` HTTP header.
    *   **Format:** `Authorization: Bearer <your-access-token>`
    *   **Note:** API keys or tokens must not be passed in the URL as query parameters, as this is insecure.
3.  **Use Standard Security Error Responses:**
    *   If a request is made without a token or with a malformed/expired token, the API MUST return a `401 Unauthorized` status code.
    *   If a request is made with a valid token that lacks the necessary permissions (scopes) to access a resource, the API MUST return a `403 Forbidden` status code.
    *   Both responses MUST use the Standard Error Schema defined in the Foundational Standards.

### F. Handling Collections (Pagination, Filtering, Sorting)
*This section supports our Principle of Predictability and Principle of Pragmatism (by ensuring performance).*

APIs that return a list of resources must provide ways for the client to handle large datasets efficiently. Returning an entire unbounded collection is a major performance and security risk.

**Rules:**
1.  **Paginate All Collection Responses:** Every endpoint that can return a list of items (`GET /users`, `GET /users/{id}/orders`) MUST be paginated.
2.  **Use Cursor-based Pagination:** We will use cursor-based pagination as our standard. It is more performant and robust than offset-based pagination for large datasets.
    *   **Request Parameters:**
        *   `limit`: An integer specifying the maximum number of items to return. Defaults to `25`. Maximum value is `100`.
        *   `after`: An opaque cursor string from the previous response's `nextCursor` field, used to fetch the next page.
    *   **Response Body Schema:** The response for a paginated collection must be a JSON object containing the data and pagination cursors.
        ```json
        {  
           "data": [
            //... array of resource objects
          ],
          "pagination": {
            "nextCursor": "string | null", // Opaque string to get the next page, or null if it's the last page.
            "hasNextPage": "boolean"      // A boolean flag indicating if more results are available.
          }
        }
        ```
    *   **Example Request:** `GET /orders?limit=50&after=aBcDeFg12345`
3.  **Provide Standard Filtering and Sorting:**
    *   **Filtering:** Use a `filter` query parameter with bracket notation for field names.
        *   **Example:** `GET /orders?filter[status]=shipped&filter[region]=emea`
    *   **Sorting:** Use a `sort` query parameter. It should accept a comma-separated list of fields. A `-` prefix indicates descending order.
        *   **Example:** `GET /users?sort=-createdAt,lastName` (Sort by creation date descending, then last name ascending).
4.  **Allow Sparse Fieldsets (Field Selection):**
    *   To improve performance, allow clients to request only the fields they need using a `fields` query parameter.
    *   **Example:** `GET /users/{userId}?fields=id,firstName,email` (Return only the id, firstName, and email fields for the user).

### G. HTTP Methods & Idempotency
*This section supports our Principle of Predictability.*

Using HTTP methods (verbs) correctly and consistently makes our APIs more expressive and aligns with the conventions of the web.

**Rules:**
1.  **Use the Correct HTTP Method for the Action:**
    *   `GET`: Retrieve a resource or a collection of resources. Must be safe (no side-effects) and idempotent.
    *   `POST`: Create a new resource. Not idempotent by default.
    *   `PATCH`: Apply a partial update to an existing resource. Should be idempotent.
    *   `PUT`: Fully replace an existing resource. Must be idempotent. (Note: PATCH is generally preferred over PUT for updates).
    *   `DELETE`: Remove a resource. Must be idempotent.
2.  **Return Appropriate Status Codes on Success:**
    *   `GET`: `200 OK`
    *   `POST`: `201 Created`. The response MUST include a `Location` header pointing to the URL of the newly created resource (e.g., `Location: /api/v1/users/123`).
    *   `PATCH` / `PUT`: `200 OK`. The response SHOULD return the full representation of the updated resource.
    *   `DELETE`: `204 No Content`. The response MUST NOT contain a body.
3.  **Support Idempotency for POST Requests:** For critical operations that create resources (e.g., submitting a payment), the `POST` endpoint MUST support idempotency to prevent accidental duplicate creations from network retries.
    *   **Mechanism:** The client will generate a unique UUID and send it in an `Idempotency-Key` header.
    *   **Behavior:** If the server sees the same `Idempotency-Key` for a second time, it will not re-process the request. Instead, it will return the original successful response (e.g., a `201 Created` with the original resource).

### H. Asynchronous Operations
*This section supports our Principle of Pragmatism and Consumer-First Design.*

For operations that may take a long time to complete (e.g., generating a report, provisioning a service), a synchronous response is not practical. It can lead to timeouts and a poor user experience.

**Rules:**
1.  **Use Asynchronous Flows for Long-Running Operations:** Any operation expected to take longer than 2 seconds MUST be implemented as an asynchronous flow.
2.  **Use the `202 Accepted` Pattern:**
    *   The initial request (e.g., `POST /reports`) should perform basic validation, queue the job for background processing, and immediately return a `202 Accepted` status code.
    *   The response body MUST contain a `statusUrl` that the client can poll to check the progress of the job.
    *   **Example `202 Accepted` Response:**
        ```http
        HTTP/1.1 202 Accepted
        
        {
          "jobId": "a1b2c3-d4e5-f678",
          "status": "pending",
          "statusUrl": "/api/v1/report-jobs/a1b2c3-d4e5-f678"
        }
        ```
3.  **Provide a Status Endpoint:**
    *   The client can then make `GET` requests to the `statusUrl`.
    *   This endpoint will return the status of the job (`pending`, `running`, `completed`, `failed`).
    *   Once the job is `completed`, the response should include a `resultUrl` pointing to the final resource (e.g., the generated report).