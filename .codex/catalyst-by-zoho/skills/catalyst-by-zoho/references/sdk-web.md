# Catalyst Web SDK v4 Reference

> **Docs:** https://docs.catalyst.zoho.com/en/sdk/web/

---

## Setup

### Script Tags

Add both scripts to your HTML `<head>`:

```html
<script src="https://static.zohocdn.com/catalyst/sdk/js/4.6.1/catalystWebSDK.js"></script>
<script src="/__catalyst/sdk/init.js"></script>
```

The `init.js` script auto-initializes the SDK with the current project context. Always load it after `catalystWebSDK.js`.

### client-package.json

Place this in the root of your client directory:

```json
{
  "name": "my-app",
  "version": "1.0.0",
  "description": "My Catalyst application",
  "homepage": "/app/index.html",
  "login_redirect": "/app/index.html"
}
```

- `homepage` — default landing page after login
- `login_redirect` — where to redirect after successful authentication

### Response Pattern

All SDK methods return a promise that resolves to:

```js
{
  status: 200,        // HTTP status code
  content: { ... },   // response payload
  message: "OK"       // status message
}
```

### Version Compatibility

| Feature                         | Minimum SDK Version |
|---------------------------------|---------------------|
| Core SDK                        | v4.0.0              |
| `changePassword()`              | v4.3.0              |
| `isUserAuthenticated()` (local) | v4.5.0              |
| `generateAuthToken()`           | v4.6.1              |

---

## Authentication

Catalyst supports two authentication types for client apps. **Ask the user which they prefer** before recommending a pattern.

### Auth Type 1: Hosted Login (Redirect-Based)

The default approach. Catalyst automatically redirects unauthenticated users to a Zoho login page.

- No `signIn()` call needed — the redirect happens automatically when the SDK initializes
- After login, the user is redirected back to `login_redirect` from `client-package.json`
- Best for standard web apps where you want Zoho to handle the full login UI

```js
// Use isUserAuthenticated() to check login status and get the current user.
// If authenticated, it resolves with the full user object.
// If NOT authenticated, it rejects — and the platform auto-redirects to the Zoho login page.
catalyst.auth.isUserAuthenticated().then(result => {
  // result.content contains the full user object
  console.log(result.content.email_id);
  console.log(result.content.first_name);
  showApp(result.content);
}).catch(err => {
  // User is not logged in — platform will auto-redirect to login.
  // Optionally show a message before the redirect happens.
  console.log('Not authenticated, redirecting...');
});
```

> ⚠️ **`catalyst.auth.getCurrentUser()` does NOT exist** in the Web SDK. Use `isUserAuthenticated()` instead — it returns the full user object on success (see below).

### Auth Type 2: Embedded Login (iFrame)

Renders login/signup forms inside your page via an iFrame.

```js
// Sign In
catalyst.auth.signIn("login-div", {
  login_redirect: "/app/index.html"
});

// Sign Up
catalyst.auth.signUp("signup-div");

// Forgot Password
catalyst.auth.forgotPassword("forgot-div");

// Change Password (v4.3.0+)
catalyst.auth.changePassword("change-pwd-div");
```

The first argument is the `id` of a `<div>` element where the iFrame will render.

#### iFrame CSS Customization

You can customize the embedded auth iFrame appearance:

- Download the default CSS from the Catalyst console (Settings > Authentication > Customize)
- Target selectors: `.zc-login-form`, `.zc-btn-primary`, `.zc-input`, `.zc-signup-link`
- Customize palette colors, fonts, button styles, and input fields
- Upload the modified CSS back through the console

### isUserAuthenticated()

Check if the current user is authenticated and get their details (v4.5.0+):

```js
try {
  const result = await catalyst.auth.isUserAuthenticated();
  // On success: result.content is the FULL USER OBJECT (not a boolean)
  console.log(result.content.email_id);    // "user@example.com"
  console.log(result.content.first_name);  // "John"
  console.log(result.content.last_name);   // "Doe"
  console.log(result.content.user_id);     // "10103000000115057"
  console.log(result.content.time_zone);   // "Asia/Kolkata"
  console.log(result.content.created_time);// "Jul 05, 2023 10:30 AM"
} catch (err) {
  // On failure: rejects with a 401 error when user is NOT authenticated.
  // The platform will auto-redirect to the Zoho login page.
  console.log('Not authenticated');
}
```

> ⚠️ **This does NOT return a boolean.** It resolves with the full user object on success, and **rejects** (throws) on failure. This is the primary way to get the current user in the Web SDK.

> ⚠️ **`catalyst.auth.getCurrentUser()` does NOT exist** in the Web SDK. `isUserAuthenticated()` is the correct method — it serves both purposes (auth check + user details).

### Sign Out

Sign the user out by calling `signOut()` with a redirect URL. This is a single call — it handles session invalidation and navigation internally.

```js
// Pass the URL to redirect to after sign-out completes.
// This does NOT return a promise — it navigates away immediately.
const redirectURL = window.location.origin + '/app/index.html';
catalyst.auth.signOut(redirectURL);
```

> ⚠️ **`signOut()` requires a redirect URL argument.** Calling it with no arguments crashes because the SDK internally calls `.startsWith("/")` on `undefined`.

> ⚠️ **`constructSignOutUrl()` does NOT exist.** Do not use a two-step pattern — `signOut(redirectURL)` handles everything in one call.

> ⚠️ **This does NOT return a promise.** Do not `await` it — the browser navigates away immediately.

### generateAuthToken() (v4.6.1+)

Generate a short-lived auth token for cross-domain requests (e.g., calling AppSail from a Slate widget):

```js
const tokenResponse = await catalyst.auth.generateAuthToken();
const token = tokenResponse.content.token;
```

### JWT Sign-In

For custom authentication flows using JWT tokens:

```js
await catalyst.auth.signinWithJwt(jwtToken);
```

### Local Dev vs Production Auth

| Behavior                    | Local Dev (`catalyst serve`)       | Production (deployed)          |
|-----------------------------|------------------------------------|--------------------------------|
| Auth cookie domain          | `localhost`                        | `.catalystserverless.com`      |
| `isUserAuthenticated()`     | Checks local dev session           | Checks Catalyst auth cookie    |
| Login redirect              | Opens Zoho login in browser        | Automatic redirect             |
| Cross-domain token          | Not needed (same origin)           | Use `generateAuthToken()`      |
| CORS                        | Not enforced                       | Must whitelist in AppSail      |

### Calling AppSail from Slate (Cross-Domain Pattern)

When calling an AppSail endpoint from a Slate widget, you need to pass an auth token since they are on different subdomains.

**Helper function:**

```js
async function callAppSail(endpoint, method = "GET", body = null) {
  const tokenResponse = await catalyst.auth.generateAuthToken();
  const token = tokenResponse.content.token;

  const options = {
    method: method,
    headers: {
      "Content-Type": "application/json",
      "Authorization": `Zoho-catalyst-appsail ${token}`
    }
  };

  if (body) {
    options.body = JSON.stringify(body);
  }

  const response = await fetch(
    `https://your-app.catalystserverless.com${endpoint}`,
    options
  );
  return response.json();
}

// Usage
const data = await callAppSail("/api/items");
const result = await callAppSail("/api/items", "POST", { name: "New Item" });
```

**Required configuration:**

- **AppSail CORS whitelist:** Add your Slate domain (`*.catalystserverless.com`) in the AppSail CORS settings via the Catalyst console
- **No `cors()` middleware:** Do NOT add Express `cors()` middleware in your AppSail code — Catalyst handles CORS at the platform level. Adding middleware causes duplicate header errors.

---

## Data Store

### Table Reference

```js
const table = catalyst.table.tableId('TableName');
```

### Operations

```js
// Get all rows
const allRows = await table.getAll();

// Get paged rows
const pagedRows = await table.getPagedRows({ nextToken: null, maxRows: 100 });

// Get column metadata
const columns = await table.getColumns();

// Add a row
const newRow = await table.addRow({
  column1: "value1",
  column2: "value2"
});

// Update a row (ROWID required)
const updated = await table.updateRow({
  ROWID: "12345",
  column1: "new_value"
});

// Delete a single row
await table.delete("12345");

// Bulk delete (max 200 rows per call)
await table.deleteRows(["12345", "12346", "12347"]);
```

> **Note:** Table operations respect the permissions configured in the Catalyst console (read, write, delete) for the current user role.

---

## ZCQL

### Execute a Query

```js
const zcql = catalyst.ZCatalystQL;
const result = await zcql.executeQuery("SELECT * FROM Users WHERE age > 25");
console.log(result.content);
```

### V2 Environment

For ZCQL V2 features, set the environment:

```js
catalyst.ZCatalystQL.setCatalystEnv("V2");
const result = await catalyst.ZCatalystQL.executeQuery("SELECT * FROM Users LIMIT 10");
```

---

## File Store

```js
const fileStore = catalyst.file;

// Get all folders
const folders = await fileStore.getAllFolder();

// Get a folder reference
const folder = fileStore.folderId("folderId");

// Upload a file
const fileInput = document.getElementById("file-input");
const uploaded = await folder.uploadFile(fileInput.files[0]);

// Get download link
const downloadLink = await folder.getDownloadLink("fileId");

// Delete a file
await folder.delete("fileId");
```

---

## Stratus (Object Storage)

```js
const bucket = catalyst.stratus.bucket("bucket-name");

// Check if object exists (head)
const head = await bucket.headObject("path/to/file.txt");

// Get object (signed URL)
const obj = await bucket.getObject("path/to/file.txt", { signedUrl: true });

// Upload object (simple)
const file = document.getElementById("file-input").files[0];
await bucket.putObject("path/to/file.txt", file);

// Upload object (multipart, for large files)
await bucket.uploadObject("path/to/large-file.zip", file, {
  partSize: 5 * 1024 * 1024 // 5MB parts
});

// Delete object
await bucket.deleteObject("path/to/file.txt");
```

---

## Search

```js
const search = catalyst.search;
const results = await search.executeSearchQuery("search term");
console.log(results.content);
```

---

## Push Notifications

```js
const push = catalyst.push;
await push.sendNotification({
  message: "Hello from Catalyst!",
  recipients: ["user@example.com"]
});
```

---

## Functions

```js
const func = catalyst.function;

// Execute a function
const result = await func.execute("functionName", {
  key1: "value1",
  key2: "value2"
});
console.log(result.content);
```

---

## Environment Variables

```js
const env = catalyst.env;

// Get a variable
const value = await env.getValue("MY_ENV_VAR");

// Get all variables
const allVars = await env.getAll();
```

---

## Common Auth Errors

| Error / Symptom                              | Cause                                                      | Fix                                                                                         |
|----------------------------------------------|------------------------------------------------------------|---------------------------------------------------------------------------------------------|
| `api_domain` is empty                        | `init.js` not loaded or loaded before `catalystWebSDK.js`  | Ensure both scripts are in `<head>`, `catalystWebSDK.js` first                              |
| `isUserAuthenticated` fails locally           | SDK version below v4.5.0                                   | Upgrade to v4.5.0+                                                                          |
| `generateAuthToken is not a function`         | SDK version below v4.6.1                                   | Upgrade to v4.6.1+                                                                          |
| `NO_ACCESS` on API calls                      | User role lacks permission for the resource                | Check role permissions in Catalyst console                                                   |
| Duplicate CORS headers / preflight fails      | Express `cors()` middleware AND Catalyst CORS both active   | Remove `cors()` middleware from AppSail code; configure CORS only in Catalyst console        |
| Sign-out not working / crashes                 | `signOut()` called without redirect URL argument           | Pass a redirect URL: `catalyst.auth.signOut(redirectURL)`. `constructSignOutUrl()` does not exist. |
| `getCurrentUser is not a function`             | Method does not exist in Web SDK                           | Use `catalyst.auth.isUserAuthenticated()` — resolves with full user object                    |
| Embedded iFrame won't load                    | Div ID mismatch or CSP blocking                            | Verify the div `id` matches, check Content-Security-Policy headers allow Zoho iFrame origins |
