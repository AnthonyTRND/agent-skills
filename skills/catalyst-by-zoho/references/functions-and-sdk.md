# Functions & SDK Reference

## Table of Contents
1. [Function Types Overview](#function-types-overview)
2. [Function Execution Limits](#function-execution-limits)
3. [Basic I/O Functions](#basic-io-functions)
4. [Advanced I/O Functions](#advanced-io-functions)
5. [Event Functions](#event-functions)
6. [Cron Functions](#cron-functions)
7. [Integration Functions](#integration-functions)
8. [Job Functions](#job-functions)
9. [Browser Logic Functions](#browser-logic-functions)
10. [Node.js SDK Setup](#nodejs-sdk-setup)
11. [Python SDK Setup](#python-sdk-setup)
12. [Java SDK Setup](#java-sdk-setup)
13. [Web SDK (Client-Side)](#web-sdk-client-side)
14. [SDK Component Access Patterns](#sdk-component-access-patterns)
15. [Error Handling Patterns](#error-handling-patterns)
16. [Security Rules](#security-rules)
17. [Retry Behavior](#retry-behavior)
18. [Cold Starts](#cold-starts)
19. [Testing](#testing)

---

## Function Types Overview

| Type | Invocation | Use Case | Handler Args (Node.js) |
|------|-----------|----------|----------------------|
| Basic I/O | HTTP GET via API/SDK | Simple request-response | `(catalystApp, context, basicIO)` |
| Advanced I/O | HTTP any method | REST APIs, webhooks | `(req, res)` — SDK via `catalyst.initialize(req)` |
| Event | Event Listeners | React to platform events | `(catalystApp, context, event)` |
| Cron | Scheduled by Cron jobs | Periodic tasks | `(catalystApp, context, cronInfo)` |
| Integration | Zoho service triggers | Zoho ecosystem integration | `(catalystApp, context, reqData)` |
| Job | Job Scheduling service | Background processing | `(catalystApp, context, jobData)` |
| Browser Logic | SmartBrowz | Headless browser scripts | `(catalystApp, context, browserData)` |

Critical: never copy code between function types. Each type has different modules initialized in the boilerplate. Always start from the correct template.

---

## Function Execution Limits

| Function Type | Timeout | Behavior on Timeout |
|---------------|---------|---------------------|
| Basic I/O | 30 seconds | Returns 504 Gateway Timeout |
| Advanced I/O | 30 seconds | Returns 504 Gateway Timeout |
| Event | 15 minutes | Silently terminated; may auto-retry |
| Cron | 15 minutes | Marked as failed; may auto-retry |
| Integration | 30 seconds | Returns error to calling Zoho service |
| Job | 15 minutes | Marked as failed; may auto-retry |
| Browser Logic | 30 seconds | Browser instance terminated |

For long-running tasks exceeding 30 seconds, use Event, Job, or Cron functions (up to 15 minutes).
For tasks exceeding 15 minutes, migrate to AppSail which has no function-level timeout.

---

## Basic I/O Functions

Simplest function type. Receives a string input, returns a string output. Invoked via GET request.

### Node.js Template
```javascript
// functions/my_basic_io/index.js
'use strict';

module.exports = (catalystApp, context, basicIO) => {
  try {
    // Get input data (sent as query parameter)
    const inputData = context.getArgument();

    // Access Catalyst components
    const dataStore = catalystApp.datastore();

    // Your business logic here
    const result = `Processed: ${inputData}`;

    // Send response (must be a string)
    basicIO.write(result);
  } catch (error) {
    console.error('Error:', error);
    basicIO.write(JSON.stringify({ error: error.message }));
  }
};
```

### package.json
```json
{
  "name": "my_basic_io",
  "version": "1.0.0",
  "main": "index.js",
  "dependencies": {
    "zcatalyst-sdk-node": "latest"
  }
}
```

Invocation: `GET /server/my_basic_io/execute?args=<input_string>`

---

## Advanced I/O Functions

Full HTTP support with native request/response objects (Express-like). Best for REST APIs.

### Node.js Template
```javascript
// functions/my_api/index.js
'use strict';
const catalyst = require('zcatalyst-sdk-node');

module.exports = async (req, res) => {
  try {
    const catalystApp = catalyst.initialize(req);
    const method = req.method;

    if (method === 'GET') {
      const queryParam = req.query.id;
      res.status(200).json({ message: 'GET request', id: queryParam });

    } else if (method === 'POST') {
      const body = req.body;
      res.status(201).json({ message: 'Created', data: body });

    } else if (method === 'PUT') {
      const body = req.body;
      res.status(200).json({ message: 'Updated', data: body });

    } else if (method === 'DELETE') {
      res.status(200).json({ message: 'Deleted' });

    } else {
      res.status(405).json({ error: 'Method not allowed' });
    }
  } catch (error) {
    console.error('Error:', error);
    res.status(500).json({ error: error.message });
  }
};
```

> **Legacy projects (node14/16/18)** may still use the 4-parameter signature:
> `module.exports = (catalystApp, context, req, res) => { ... }`
> where `catalystApp` is pre-initialized. New CLI-initialized projects (node20+)
> use the 2-parameter format shown above.

Invocation: Any HTTP method to `/server/my_api/execute`

### HTTP payload limits

| Limit | Value |
|-------|-------|
| Request body (JSON/form/multipart) | 250 MB |
| Response body | 250 MB |

Exceeding these limits returns HTTP 413 (Payload Too Large). For very large file transfers,
use Stratus presigned upload URLs instead of passing data through functions.

### Handling file uploads in Advanced I/O
```javascript
'use strict';
const catalyst = require('zcatalyst-sdk-node');

module.exports = async (req, res) => {
  const catalystApp = catalyst.initialize(req);
  const uploadedFile = req.files?.file;

  if (!uploadedFile) {
    return res.status(400).json({ error: 'No file provided' });
  }

  const stratus = catalystApp.stratus();
  const bucket = stratus.bucket('my-bucket');

  try {
    await bucket.putObject({
      key: uploadedFile.name,
      body: uploadedFile.data,
      contentType: uploadedFile.mimetype
    });
    res.status(200).json({ message: 'Uploaded', name: uploadedFile.name });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
};
```

---

## Event Functions

Triggered by Event Listeners. Cannot be invoked directly via HTTP.

```javascript
// functions/my_event_fn/index.js
'use strict';

module.exports = (catalystApp, context, event) => {
  try {
    // Get event data
    const eventData = event.getArgument();
    const parsedData = JSON.parse(eventData);

    console.log('Event received:', parsedData);

    // Process the event
    // ...

    // Must close context when done
    context.close();
  } catch (error) {
    console.error('Event processing error:', error);
    context.close();
  }
};
```

Event types that can trigger Event Functions:
- **Component Events**: Data Store row insert/update/delete, File Store upload/delete
- **Custom Events**: User-defined events triggered via SDK/API
- **Zoho Events**: Events from Zoho services (CRM, Books, etc.)

---

## Cron Functions

Invoked by Cron jobs on a schedule. Cannot be invoked directly.

```javascript
// functions/my_cron_fn/index.js
'use strict';

module.exports = async (catalystApp, context, cronInfo) => {
  try {
    const cronDetails = cronInfo.getArgument();
    console.log('Cron job triggered:', cronDetails);

    // Perform scheduled task
    // Example: Clean up old records
    const zcql = catalystApp.zcql();
    const query = "DELETE FROM Reports WHERE CREATEDTIME < '2024-01-01 00:00:00'";
    await zcql.executeZCQLQuery(query);

    // Signal success
    context.closeWithSuccess();
  } catch (error) {
    console.error('Cron error:', error);
    context.closeWithFailure();
  }
};
```

---

## Integration Functions

For integrating with other Zoho services. Note: NOT available in EU, AU, IN, or CA data centers.

```javascript
// functions/my_integration_fn/index.js
'use strict';

module.exports = (catalystApp, context, reqData) => {
  try {
    const integrationData = reqData.getArgument();
    const parsedData = JSON.parse(integrationData);

    // Access the Zoho service data
    console.log('Integration data:', parsedData);

    // Process and respond
    context.close();
  } catch (error) {
    console.error('Integration error:', error);
    context.close();
  }
};
```

---

## Job Functions

Triggered by the Job Scheduling service for background processing.

```javascript
// functions/my_job_fn/index.js
'use strict';

module.exports = async (catalystApp, context, jobData) => {
  try {
    const jobDetails = jobData.getArgument();
    console.log('Job started:', jobDetails);

    // Long-running background task
    // ...

    context.closeWithSuccess();
  } catch (error) {
    console.error('Job error:', error);
    context.closeWithFailure();
  }
};
```

---

## Browser Logic Functions

Used with SmartBrowz for headless browser automation.

```javascript
// functions/my_browser_fn/index.js
'use strict';

module.exports = (catalystApp, context, browserData) => {
  try {
    const input = browserData.getArgument();
    // Browser automation logic
    context.close();
  } catch (error) {
    console.error('Browser logic error:', error);
    context.close();
  }
};
```

---

## Node.js SDK Setup

> **Runtime support:** node20 is the only actively supported runtime — node14, 16, and 18 still work for legacy projects but receive no upstream security patches. Use node20 for all new functions.

### In Functions
The initialization pattern depends on the Node.js runtime version used when the project was created:

```javascript
// NEW (node20+ / CLI-initialized projects) — SDK must be initialized manually:
'use strict';
const catalyst = require('zcatalyst-sdk-node');

module.exports = async (req, res) => {
  const catalystApp = catalyst.initialize(req);
  const dataStore = catalystApp.datastore();
  // ...
};

// LEGACY (node14/16/18) — catalystApp is pre-initialized as first parameter:
// module.exports = (catalystApp, context, req, res) => {
//   const dataStore = catalystApp.datastore();
// };
```

### In AppSail (manual initialization)
```javascript
const catalyst = require('zcatalyst-sdk-node');

// Initialize with request context (for user-scoped operations)
app.get('/api/data', (req, res) => {
  const catalystApp = catalyst.initialize(req);
  const dataStore = catalystApp.datastore();
  // ...
});

// Initialize as admin (for system-level operations)
const catalystApp = catalyst.initialize(req, { type: catalyst.credential.admin });
```

### NPM Package
```bash
npm install zcatalyst-sdk-node
```

---

## Python SDK Setup

### In Functions
```python
# functions/my_function/main.py
import zcatalyst_sdk

def handler(context, basicIO):
    catalyst_app = zcatalyst_sdk.initialize()
    datastore = catalyst_app.datastore()

    # Business logic
    result = "Processed"
    basicIO.write(result)
```

### pip Package
```bash
pip install zcatalyst-sdk
```

---

## Java SDK Setup

### In Functions
```java
// Maven dependency: com.zoho.catalyst:zcatalyst-sdk
import com.zoho.catalyst.api.CatalystApp;
import com.zoho.catalyst.api.beans.*;

public class MainFunction implements BasicIO {
    @Override
    public void runner(CatalystApp catalystApp, Context context, BasicIOObject basicIO) {
        try {
            String input = context.getArgument();
            // Business logic
            basicIO.write("Result");
        } catch (Exception e) {
            basicIO.write("Error: " + e.getMessage());
        }
    }
}
```

---

## Web SDK (Client-Side)

The Web SDK is used in frontend code to interact with Catalyst backend services.

### Include via script tag
```html
<script src="https://static.zohocdn.com/catalyst/sdk/js/4.0.0/catalystWebSDK.js"></script>
<script>
  catalyst.auth.init("YOUR_LOGIN_REDIRECT_URL", { // Get from Catalyst Console → Authentication → Web Client
    // optional config
  });
</script>
```

### Authentication flow
```javascript
// Sign up a new user
catalyst.auth.signUp({
  email_id: "user@example.com",
  first_name: "John",
  last_name: "Doe"
});

// Login
catalyst.auth.login("user@example.com", "password");

// Check if logged in
const isLoggedIn = catalyst.auth.isUserAuthenticated();

// Logout
catalyst.auth.signOut();
```

### Calling functions from client
```javascript
// Call a Basic I/O function
catalyst.server.callFunction("my_basic_io", { args: "input_data" })
  .then(response => console.log(response))
  .catch(err => console.error(err));

// Call an Advanced I/O function
catalyst.server.callAdvancedIO("my_api", {
  method: "POST",
  body: JSON.stringify({ key: "value" }),
  headers: { "Content-Type": "application/json" }
})
  .then(response => console.log(response))
  .catch(err => console.error(err));
```

---

## SDK Component Access Patterns

All components are accessed through the initialized `catalystApp` object:

```javascript
// Data Store
const dataStore = catalystApp.datastore();
const table = dataStore.table('TableName');      // by name
const table = dataStore.table(TABLE_ID);         // by ID

// File Store
const fileStore = catalystApp.filestore();
const folder = fileStore.folder(FOLDER_ID);

// Cache
const cache = catalystApp.cache();
const segment = cache.segment(SEGMENT_ID);

// ZCQL
const zcql = catalystApp.zcql();

// Email
const email = catalystApp.email();

// Search
const search = catalystApp.search();

// User Management
const userManagement = catalystApp.userManagement();

// Push Notifications
const pushNotification = catalystApp.pushNotification();

// Connections (for third-party auth)
const connection = catalystApp.connection();
```

---

## Error Handling Patterns

### Recommended pattern for Advanced I/O functions
```javascript
'use strict';
const catalyst = require('zcatalyst-sdk-node');

module.exports = async (req, res) => {
  try {
    const catalystApp = catalyst.initialize(req);
    // Validate input
    if (!req.body || !req.body.name) {
      return res.status(400).json({
        status: 'error',
        message: 'Name is required'
      });
    }

    // Business logic
    const result = await someOperation(catalystApp, req.body);

    res.status(200).json({
      status: 'success',
      data: result
    });
  } catch (error) {
    console.error('Function error:', error);

    // Check for Catalyst-specific errors
    if (error.code === 'INVALID_DATA') {
      return res.status(400).json({ status: 'error', message: error.message });
    }

    res.status(500).json({
      status: 'error',
      message: 'Internal server error'
    });
  }
};
```

---

## Security Rules

Security Rules control who can invoke Basic I/O and Advanced I/O functions.

Access levels:
- **`no_auth`**: Anyone can invoke (public access)
- **`user_auth`**: Only authenticated users
- **`admin_auth`**: Only project admins

Security rules are configured in the Catalyst console under Serverless → Security Rules. They define
JSON-based rules that determine access per function.

Default: all functions require `user_auth` (authenticated users only). Override to `no_auth` for
public APIs, or use the API Gateway for more granular control.

---

## Retry Behavior

Background function types (Event, Cron, Job) automatically retry on failure. HTTP-facing
function types (Basic I/O, Advanced I/O) do not retry — the error is returned to the caller.

| Function Type | Auto-retry on failure? | Retry behavior |
|---------------|----------------------|----------------|
| Basic I/O | No | Error returned to caller |
| Advanced I/O | No | Error returned to caller |
| Event | Yes | Retries on failure (developer-configurable) |
| Cron | Yes | Retries on failure (developer-configurable) |
| Job | Yes | Retries on failure (developer-configurable) |
| Integration | No | Error returned to calling Zoho service |
| Browser Logic | No | Error returned to caller |

Retry logic for background functions is controlled by how you write your function and configure
the triggering service. There is no fixed platform retry count — design your handlers to be
**idempotent** (safe to run multiple times with the same input) since retries may occur.

---

## Cold Starts

When a function hasn't been invoked recently, Catalyst provisions a new execution
environment — this adds latency to the first request ("cold start").

**Typical cold-start latency:**
| Runtime | Cold start | Warm invocation |
|---------|-----------|-----------------|
| Node.js | 500ms–2s | 50–200ms |
| Java | 2–8s | 50–200ms |
| Python | 500ms–2s | 50–200ms |

Java has the longest cold starts due to JVM initialization.

**Mitigation strategies:**
- Keep function packages small — fewer dependencies = faster init
- Avoid heavy initialization outside the handler (large file reads, DB pool creation on import)
- Use a scheduled ping (via Job Scheduling) to keep critical functions warm
- For latency-sensitive endpoints, consider AppSail — it runs persistently with no cold starts

---

## Testing

### Unit testing function handlers

Mock `catalystApp` to test handler logic without connecting to Catalyst:

```javascript
// test/my_function.test.js
const handler = require('../functions/my_function/index');

const mockReq = {
  method: 'POST',
  body: { name: 'Test' },
  query: {},
  headers: { 'x-zoho-catalyst-user': 'test-user' }
};

const mockRes = {
  status: jest.fn().mockReturnThis(),
  json: jest.fn()
};

jest.mock('zcatalyst-sdk-node', () => ({
  initialize: () => ({
    datastore: () => ({
      table: () => ({
        insertRow: jest.fn().mockResolvedValue({ ROWID: '123' })
      })
    }),
    zcql: () => ({
      executeZCQLQuery: jest.fn().mockResolvedValue([])
    })
  })
}));

test('POST returns 201', async () => {
  await handler(mockReq, mockRes);
  expect(mockRes.status).toHaveBeenCalledWith(201);
});
```

### Integration testing with `catalyst serve`

`catalyst serve` runs Basic I/O and Advanced I/O functions locally, connecting to the
remote Development Data Store. Use it for integration tests:

```bash
catalyst serve &
curl -X POST http://localhost:3000/server/my_function/execute -d '{"name":"Test"}'
```

Note: Event, Cron, and Job functions cannot be tested locally via `catalyst serve`.
Deploy to Development and trigger them from the console for integration testing.
