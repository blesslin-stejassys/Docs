# Sentry Infrastructure Documentation

## 1. Introduction

Sentry is our **real-time error tracking and performance monitoring** tool used across both backend and mobile applications.  
It helps us identify issues early, trace performance bottlenecks, and maintain application stability after deployment.  
Currently, Sentry access and configuration are **managed by Varun** for our organization.

---

## 2. Prerequisites

Before configuring or accessing Sentry, ensure the following prerequisites are met:

- You have access to the organization’s **Sentry workspace**.  
  (Contact Varun for invitation or credentials.)
- Project-specific **DSN (Data Source Name)** is available.
- The relevant project (Android / iOS / Backend) is already created in Sentry.
- You have access to `.env` files or CI/CD secrets where Sentry credentials are stored.

---

## 3. Sentry Project Setup and API Credentials

### Steps to Create or Access a Sentry Project
1. Visit [https://sentry.io](https://sentry.io).
2. Log in with organizational credentials.
3. Navigate to:
   **Projects → Create Project → Select Platform (e.g., JavaScript, Android, iOS, Node.js)**
4. Once created, open **Settings → Client Keys (DSN)**.
5. Copy the **Public DSN** — this will be required in your app or backend configuration.

### Key Credentials
| Key Name | Description | Location |
|-----------|--------------|-----------|
| `SENTRY_DSN` | Unique project identifier used for sending event data | `.env` / Secret Manager |
| `SENTRY_AUTH_TOKEN` | Auth token used for release tracking | CI/CD Secret |
| `SENTRY_ORG` | Organization slug in Sentry | `.env` |
| `SENTRY_PROJECT` | Project slug name (e.g., `android-app`, `backend-api`) | `.env` |

---

## 4. Local Setup for Sentry (Backend Integration)

### Node.js / Directus Integration Example
Add the official Sentry SDK to your Directus or Node backend:
```bash
npm install @sentry/node @sentry/tracing
```

Initialize Sentry in your main file (`index.js` or `server.js`):
```js
import * as Sentry from "@sentry/node";
import { nodeProfilingIntegration } from "@sentry/profiling-node";

Sentry.init({
  dsn: process.env.SENTRY_DSN,
  integrations: [nodeProfilingIntegration()],
  tracesSampleRate: 1.0,
});
```

Verify by manually triggering an error:
```js
throw new Error("Sentry Test Error");
```

Once triggered, check the Sentry dashboard under your project to confirm event capture.

---

## 5. Android Integration with Sentry

### Steps
1. Add the dependency in `app/build.gradle`:
   ```gradle
   implementation 'io.sentry:sentry-android:6.+'
   ```
2. Initialize in your Application class:
   ```kotlin
   import io.sentry.Sentry

   class MyApp : Application() {
       override fun onCreate() {
           super.onCreate()
           Sentry.init { options ->
               options.dsn = BuildConfig.SENTRY_DSN
               options.tracesSampleRate = 1.0
           }
       }
   }
   ```
3. Add your DSN to `local.properties` or CI/CD secrets:
   ```
   SENTRY_DSN=https://<your-dsn>.ingest.sentry.io/<project-id>
   ```
4. Run a test crash to verify:
   ```kotlin
   Sentry.captureException(Exception("Test Crash"))
   ```

---

## 6. Apple (iOS) Integration with Sentry

### Steps
1. Install via CocoaPods:
   ```bash
   pod 'Sentry', '>= 8.0.0'
   ```
2. Initialize Sentry in `AppDelegate.swift`:
   ```swift
   import Sentry

   func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
       SentrySDK.start { options in
           options.dsn = "<your-dsn>"
           options.debug = true
       }
       return true
   }
   ```
3. Trigger a sample error:
   ```swift
   SentrySDK.capture(error: NSError(domain: "TestError", code: 0))
   ```

---

## 7. Deploying and Monitoring

### Environment Integration
Sentry is configured across environments:
| Environment | DSN | Log Level | Responsible |
|--------------|-----|------------|--------------|
| Development | Shared Test DSN | Debug | Developer |
| Staging | Staging DSN | Warning | DevOps |
| Production | Live DSN | Error | Varun |

### Release Tracking
- Each deployment triggers a **release version** update in Sentry via CI/CD:
  ```bash
  export SENTRY_AUTH_TOKEN=<token>
  sentry-cli releases new "v1.0.0"
  sentry-cli releases finalize "v1.0.0"
  ```
- Monitors release health (crash-free sessions, user impact).

### Issue Workflow
1. Sentry captures the issue and assigns it automatically based on tags.
2. Alerts are routed to Slack channel `#error-alerts`.
3. Developer reviews stack trace and resolves it in code.
4. After deployment, issue is marked as **Resolved** in Sentry.

---

## 8. Conclusion and Next Steps

Sentry provides a robust monitoring and debugging framework for both backend and mobile applications.  
It ensures that production issues can be traced, understood, and resolved efficiently.

**Next Steps:**
- Integrate Sentry in all new projects (Web, Android, iOS).
- Standardize `SENTRY_DSN` across environments.
- Implement **performance tracing** for API endpoints in Directus.
- Regularly review and triage Sentry alerts to maintain app health.


---

## 9. Next.js Integration with Sentry

### Overview
Learn how to set up and configure Sentry in your Next.js application using the installation wizard, capture your first errors, and view them in Sentry.

### Prerequisites
You need:
- A Sentry account and project
- Your application up and running

### Features
In addition to capturing errors, you can monitor interactions between multiple services or applications by enabling tracing.  
You can also get to the root of an error or performance issue faster by watching a video-like reproduction of a user session with **session replay**.

Select which Sentry features you'd like to install in addition to Error Monitoring to get the corresponding installation and configuration instructions below.

---

### Step 1: Install

To install Sentry using the installation wizard, run the following command within your project:

```bash
npx @sentry/wizard@latest -i nextjs
```

The wizard guides you through the setup process, asking you to enable additional (optional) Sentry features for your application beyond error monitoring.

> **Tip:** This guide assumes that you enable all features and allow the wizard to create an example page and route. You can add or remove features later, but setting them up now saves manual configuration effort.

---

### Step 2: Configure

If you prefer to configure Sentry manually, here are the configuration files the wizard would create.

#### Client-Side Configuration (`instrumentation-client.(js|ts)`)
```js
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: "https://examplePublicKey@o0.ingest.sentry.io/0",

  // Adds request headers and IP for users
  sendDefaultPii: true,

  // Replay may only be enabled for the client-side
  integrations: [
    Sentry.replayIntegration(),
    Sentry.feedbackIntegration({
      colorScheme: "system",
    }),
  ],

  // Enable logs to be sent to Sentry
  enableLogs: true,

  // Set tracesSampleRate to 1.0 to capture 100% of transactions for tracing
  tracesSampleRate: 1.0,

  // Capture Replay for 10% of all sessions + 100% of sessions with an error
  replaysSessionSampleRate: 0.1,
  replaysOnErrorSampleRate: 1.0,
});

export const onRouterTransitionStart = Sentry.captureRouterTransitionStart;
```

#### Server-Side Configuration (`sentry.server.config.(js|ts)`)
```js
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: "https://examplePublicKey@o0.ingest.sentry.io/0",
  sendDefaultPii: true,
  enableLogs: true,
  tracesSampleRate: 1.0,
});
```

> For detailed manual setup instructions, see the [Sentry Next.js manual setup guide](https://docs.sentry.io/platforms/javascript/guides/nextjs/).

---

### Step 3: Verify Your Setup

If you haven't tested your Sentry configuration yet, let's do it now.

You can confirm that Sentry is working properly and sending data to your Sentry project using the example page and route created by the installation wizard.

1. Open the example page `/sentry-example-page` in your browser (usually `http://localhost:3000/sentry-example-page`).
2. Click the **"Throw error"** button. This triggers:
   - A frontend error
   - An error within the API route

Sentry captures both of these errors. Additionally, the button click starts a performance trace to measure the time it takes for the API request to complete.

> **Tip:** Explore the example files’ code in your project to understand what happens after your button click.

#### View Captured Data in Sentry
Now, head over to your project on [Sentry.io](https://sentry.io) to view the collected data (it may take a few moments to appear).

> **Note:** Errors triggered from within your browser’s developer tools (like the console) are sandboxed, so they will not trigger Sentry’s error monitoring.
