
# Frontend Integration Contract Generator

## Purpose

Generate concise, implementation-ready Markdown documentation for a frontend team based on a backend API test workflow.

The input is usually a manually created HTML API test page used during backend development.

Example:

```bash
/frontend-contract test-booking.html
```

The generated document explains **how the frontend should integrate with the backend**, not how the HTML test page itself works.

The output should allow a frontend developer to implement the corresponding React/React Native/Expo workflow without needing to inspect the backend source code or the test HTML.

---

# Core Principle

Treat the provided HTML test page as a **workflow specification**.

Extract:

* API endpoints
* HTTP methods
* authentication requirements
* request headers
* request body
* request fields
* field types
* data sources
* frontend interactions
* API dependencies
* request generation logic
* idempotency behavior
* hashing behavior
* success handling
* error handling
* retry behavior
* frontend/backend responsibility boundaries

Do NOT simply summarize the HTML source.

Translate the implementation details into a frontend-facing API contract.

---

# Accuracy Rules

## 1. Never invent backend behavior

Only document behavior that can be established from:

1. The provided HTML/test file
2. Backend API calls inside the HTML
3. Backend source code if available in the project
4. Existing API documentation
5. Explicit information provided by the user

If behavior cannot be determined, write:

> Not determined from the provided implementation.

Do not guess.

---

## 2. Distinguish test-only behavior from production behavior

The HTML may contain functionality that exists only to test the backend.

For example:

```text
Generate New Keys
Retry Same Request
Retry Different Payload
Add Example Shop
```

Do not automatically tell the frontend team to reproduce these controls.

Instead, translate them into the underlying production behavior.

Example:

```text
Test button:
"Retry Same Request"

Frontend contract:
A retry of the same booking request must reuse the original idempotency key.
```

---

## 3. Do not require the frontend to copy the HTML UI

The HTML is a backend testing tool.

The frontend team may have a completely different UI.

Document the **data flow and API contract**, not the HTML layout.

---

# Output Format

Always generate the following structure when the information is available.

````markdown
# <Feature Name> — Frontend Integration Contract

## 1. <Primary Endpoint>

### Endpoint

`METHOD /api/...`

### Authentication

...

---

## 2. Request Body

```json
...
````

### Fields

| Field | Type | Required | Frontend source |
| ----- | ---- | -------: | --------------- |
| ...   | ...  |      ... | ...             |

### Important

...

---

# 3. Frontend <Feature> Flow

```text
...
```

---

# 4. Example Frontend Request

```javascript
...
```

---

# 5. <Special Behavior>

...

---

# 6. Success

...

---

# 7. Important Error Cases

| Status | Meaning | Frontend behavior |
| -----: | ------- | ----------------- |
|    ... | ...     | ...               |

---

# 8. Frontend Responsibilities

* ...

---

# 9. Backend Responsibilities

* ...

---

# 10. Integration Checklist

* [ ] ...

````

Adapt section names when necessary.

Do not force irrelevant sections into the document.

---

# Section Requirements

## 1. Endpoint

Identify:

- HTTP method
- URL/path
- authentication requirement
- required headers

Example:

```markdown
## 1. Create Booking

### Endpoint

`POST /api/bookings`

### Authentication

Requires authenticated customer.

```http
Authorization: Bearer <access_token>
Content-Type: application/json
````

````

If the authenticated role can be determined, document it.

Do not guess the role.

---

# 2. Request Body

Extract the actual request body sent by the test.

Show a clean representative example.

Do not copy fake test values when they are clearly placeholders unless they help explain the structure.

Example:

```json
{
  "shopId": 3,
  "vehicleId": 7,
  "serviceIds": [1, 2],
  "scheduledAt": "2026-09-30T12:53:52.063Z",
  "idempotencyKey": "uuid",
  "requestHash": "sha256",
  "note": "Please check the brakes"
}
````

Then create a field table.

Determine the field type from the code.

Example:

```markdown
| Field | Type | Required | Frontend source |
|---|---|---:|---|
| `shopId` | number | Yes | Selected shop |
| `vehicleId` | number | Yes | Selected user's vehicle |
| `serviceIds` | number[] | Yes | Selected services |
| `scheduledAt` | ISO 8601 string | Yes | Selected date/time |
```

Do not mark a field required merely because the HTML form has `required`.

Check the actual request-building logic and backend validation when available.

---

# 3. Frontend Flow

Convert the HTML interaction into a clean application flow.

Example:

```text
1. User opens booking screen
        ↓
2. Load user's vehicles
        ↓
3. Load/select shop
        ↓
4. Load services for selected shop
        ↓
5. User selects:
   - Vehicle
   - Services
   - Date/time
   - Optional note
        ↓
6. Generate idempotencyKey
        ↓
7. Build booking payload
        ↓
8. Generate requestHash
        ↓
9. POST /api/bookings
        ↓
10. Handle response
```

Focus on business/data flow.

Do not describe HTML elements unless necessary for understanding the workflow.

---

# 4. Example Frontend Request

Generate a minimal implementation example.

Prefer modern JavaScript using `fetch`.

Example:

```javascript
const payload = {
  shopId: 3,
  vehicleId: 7,
  serviceIds: [1, 2],
  scheduledAt: new Date(selectedDateTime).toISOString(),
  idempotencyKey: crypto.randomUUID(),
  requestHash: generatedHash,
  note: note || undefined,
};

const response = await fetch(`${API_URL}/api/bookings`, {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Authorization: `Bearer ${accessToken}`,
  },
  body: JSON.stringify(payload),
});
```

The example must reflect the actual API discovered from the input.

Do not invent variables that imply unsupported behavior.

---

# 5. Special Backend Behavior

Create sections for important behavior discovered from the test.

Examples:

* Idempotency
* Pagination
* Filtering
* File upload
* OTP
* WebSocket notifications
* State transitions
* Retry behavior
* Request hashing
* Transactions
* Role-specific behavior

Only include sections that actually apply.

---

# Idempotency Rules

When idempotency behavior is detected, explicitly document:

### New operation

```text
New operation
    ↓
Generate new idempotency key
```

### Retry

```text
Same operation
    ↓
Reuse original idempotency key
```

### Changed payload

If the backend compares request hashes:

```text
Same idempotency key
Different payload/hash
    ↓
Conflict
```

Clearly explain:

> New operation = new idempotency key.
>
> Retry of the same operation = same idempotency key.

Do not tell the frontend to regenerate the key during a retry unless the backend behavior explicitly requires it.

---

# Request Hash Rules

If the HTML generates a hash:

1. Identify exactly which object is hashed.
2. Identify whether metadata is included.
3. Identify the serialization method.
4. Identify the hashing algorithm.
5. Explain when the hash must be regenerated.

Example:

```text
Booking payload
      ↓
JSON serialization
      ↓
SHA-256
      ↓
requestHash
```

If the hash excludes `idempotencyKey` and `requestHash`, explicitly state that.

Do not simplify this behavior if doing so could cause the frontend implementation to generate an incompatible hash.

---

# API Dependencies

If the test page calls other endpoints before the primary operation, document them.

Example:

```text
GET /api/vehicles
GET /api/shops/:shopId/services
POST /api/bookings
```

Explain the dependency:

```text
GET /api/vehicles
        ↓
User selects vehicle
        ↓
vehicleId
        ↓
POST /api/bookings
```

Likewise:

```text
GET /api/shops/:shopId/services
        ↓
User selects services
        ↓
serviceIds
        ↓
POST /api/bookings
```

This is important because frontend developers need to know where IDs originate.

---

# Success Response

Extract the actual response handling from the test.

If the exact response structure is known, document it.

Example:

```json
{
  "message": "Booking created successfully",
  "data": {
    "id": 123,
    "status": "PENDING"
  }
}
```

If only the existence of `data` or `message` can be determined, document only what is known.

Never invent response fields.

If the response structure cannot be determined:

```text
The exact response schema could not be determined from the provided test workflow.
```

---

# Error Handling

Extract HTTP status handling from the HTML.

Create a table:

| Status | Meaning | Frontend behavior |
| -----: | ------- | ----------------- |
|  `400` | ...     | ...               |
|  `401` | ...     | ...               |
|  `403` | ...     | ...               |
|  `404` | ...     | ...               |
|  `409` | ...     | ...               |
|  `500` | ...     | ...               |

Only include statuses actually known.

If backend source is available, inspect it to improve accuracy.

Do not invent semantic meanings for unknown status codes.

---

# Frontend Responsibilities

Document what the frontend must do.

Typical examples:

* Collect form values
* Load dependent resources
* Convert date/time to API format
* Generate client-side UUIDs when required
* Generate request hashes when required
* Include authentication headers
* Preserve idempotency keys during retries
* Handle API errors
* Display successful results

Keep this specific to the discovered workflow.

---

# Backend Responsibilities

Document what the backend owns.

Typical examples:

* Authentication
* Validation
* Business rules
* Authorization
* Relationship validation
* Idempotency
* Database operations
* State transitions
* Response formatting

Do not claim a responsibility unless it is supported by the available implementation.

---

# Integration Checklist

Create a practical checklist for the frontend developer.

Example:

```markdown
# Integration Checklist

- [ ] Load required data
- [ ] User can select required entities
- [ ] Request body matches API contract
- [ ] Date/time uses required format
- [ ] Authentication token is included
- [ ] Client-generated values are generated correctly
- [ ] Request is sent to correct endpoint
- [ ] Success response is handled
- [ ] Validation errors are handled
- [ ] Authentication errors are handled
- [ ] Conflict errors are handled
- [ ] Retry behavior follows idempotency rules
```

Customize the checklist based on the feature.

---

# Output Quality Rules

The generated documentation should be:

* concise
* implementation-oriented
* frontend-focused
* accurate
* easy to scan
* free from backend implementation details that the frontend does not need

Avoid:

* explaining every HTML function
* copying large HTML sections
* documenting CSS
* documenting test buttons as production UI
* repeating the same information
* exposing internal implementation unnecessarily
* guessing missing API behavior

Prefer:

```text
What the frontend sends
        ↓
Why it sends it
        ↓
Where the value comes from
        ↓
What the backend returns
        ↓
What the frontend should do
```

---

# Existing Frontend Workflow Rule

The frontend team may already have a UI or workflow.

The generated document must therefore be treated as an **API integration contract**, not a required UI design.

For example:

```text
Backend requirement:
serviceIds must contain IDs belonging to the selected shop.

Frontend implementation:
The frontend may represent services using cards,
checkboxes, a bottom sheet, or another UI.
```

Document the data requirement, not the UI implementation.

---

# Input Handling

The command may receive:

```bash
/frontend-contract test-booking.html
```

If a file path is provided:

1. Read the file.
2. Identify all API calls.
3. Identify the primary workflow.
4. Identify supporting API calls.
5. Trace data from UI input → request body.
6. Trace request → response handling.
7. Identify special behavior.
8. Generate the Markdown contract.

If backend source files are available in the current project and necessary to verify behavior, inspect them.

Do not inspect the entire repository unnecessarily.

Prefer targeted searches for:

* endpoint path
* controller
* route
* schema
* service
* response structure
* error status

---

# Multiple Workflows

A single HTML file may contain multiple related workflows.

Example:

```text
Create Booking
Retry Booking
Conflict Test
```

Treat them as one feature when they represent testing scenarios for the same API.

Translate test scenarios into production behavior.

For example:

```text
"Retry Different Payload"
```

becomes:

```text
Reusing an idempotency key with a different request payload results in a conflict.
```

Do not document:

```text
Frontend needs a "Retry Different Payload" button.
```

---

# Final Output

Return only the generated Markdown document unless the user explicitly asks for an explanation.

The generated document should be ready to save as:

```text
docs/frontend/<feature>.md
```

Example:

```text
docs/frontend/booking.md
```
