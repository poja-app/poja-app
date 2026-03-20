# Poja — Spring Boot but cloud-native

**Deploy Spring Boot without DevOps.** Push to GitHub. Your app goes live on AWS in under 10 minutes — no infrastructure configuration, no cold starts, no surprises on your bill.

[![poja.io](https://img.shields.io/badge/website-poja.io-black)](https://poja.io)
[![docs](https://img.shields.io/badge/docs-docs.poja.io-blue)](https://docs.poja.io)
[![console](https://img.shields.io/badge/console-console.poja.io-green)](https://console.poja.io/login)
[![discord](https://img.shields.io/badge/community-Discord-5865F2)](https://discord.gg/yU3s7gyHPv)

---

## What is Poja?

Poja is a **Spring Boot PaaS** (Platform-as-a-Service) built exclusively for Java developers. It runs your Spring Boot applications on modern AWS serverless infrastructure — Lambda, API Gateway, SQS, S3, SES — without requiring you to configure, manage, or even understand any of it.

It delivers what every Java developer wants: the simplicity of Heroku with the economics and performance of AWS Lambda.

- **No DevOps expertise required** — create an app, connect your GitHub account, push code, done.
- **No cold starts** — AWS Lambda SnapStart initializes your app at deploy time and restores from a snapshot on every request.
- **No idle costs** — scale to zero when your app isn't used. Pay only for what runs.
- **No lock-in on features** — file storage, async queues, email, cron jobs, and Sentry monitoring are all togglable with a single click from the dashboard. No code changes required to enable or disable them.

---

## Performance

Poja is built on **AWS Lambda SnapStart Spring Boot** — which eliminates the JVM cold start problem entirely.

| Metric | Result |
|---|---|
| p95 response time | **34 ms** at 6,000 simultaneous requests |
| p50 (median) response time | **24.8 ms** |
| Min response time | **18 ms** |
| Concurrent requests tested | **6,000 req/s for 60 seconds** |
| HTTP 200 rate | **100%** |

> Benchmark run with [Artillery](https://www.artillery.io/) on a default-configuration Poja application. No tuning required.

---

## Pricing

Poja uses a single, transparent pricing model with no flat fees, no tiers, and no surprises.

| | |
|---|---|
| **Compute** | €0.00002 per MB of memory per minute |
| **Free tier** | €2 monthly discount — forever, for every customer |
| **Email (SES)** | Free — only compute time is charged |
| **File storage (S3)** | Free — only compute time is charged |
| **Async messaging (SQS)** | Free — only compute time is charged |

A Spring Boot app handling moderate traffic typically costs **under €2/month** — covered entirely by the free tier.

---

## AWS infrastructure stack

Poja provisions and manages all of the following on your behalf. You never interact with any of these services directly.

| AWS Service | Role in Poja |
|---|---|
| **AWS Lambda + SnapStart** | Runs your Spring Boot application. SnapStart eliminates cold starts by snapshotting the fully initialized JVM at deploy time. |
| **AWS API Gateway** | Exposes your HTTP endpoints to the internet with automatic DDoS protection. |
| **AWS SQS** | Powers event-driven async workers. Define an event class, wire a consumer — no queue infrastructure to manage. |
| **AWS EventBridge** | Routes scheduled tasks (cron) to the right worker queues. |
| **AWS S3** | File storage for your app. Upload, download, and presign files via the built-in `BucketComponent` class. |
| **AWS SES** | Email sending via the built-in `Mailer` class. Free maintenance — only compute is charged. |
| **AWS CloudWatch** | Logs and metrics, pre-configured. Available in your Poja-provided AWS credentials. |

---

## Quickstart — deploy in 5 steps

### Step 1 — Create your app

Go to [console.poja.io](https://console.poja.io/login), click **Create New**, and provide:
- Your app name
- Your GitHub account (Poja creates the repo for you)
- Your Java package name (e.g. `com.mycompany.myapp`)

Poja generates a production-ready Spring Boot starter template and deploys it automatically.
Your `/ping` endpoint is live within minutes — hit it to confirm:

```
GET https://your-app-url/ping
→ pong
```

### Step 2 — Clone and open the generated repo

```bash
git clone https://github.com/your-account/your-app-name
cd your-app-name
```

The generated project includes:
- A working Spring Boot application with REST controllers
- Two GitHub Actions workflows (`ci.yml` and `cd-compute.yml`)
- Google Java Format scripts (`format.sh` / `format.bat`)
- Testcontainers integration tests with PostgreSQL
- JaCoCo code coverage configuration

### Step 3 — Write your endpoint

```java
// src/main/java/com/mycompany/myapp/endpoint/rest/controller/HelloController.java
package com.mycompany.myapp.endpoint.rest.controller;

import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class HelloController {

  @GetMapping("/hello")
  public String hello() {
    return "Hello from Poja!";
  }
}
```

### Step 4 — Format and push

```bash
# macOS / Linux
./format.sh

# Windows
.\format.bat

git add .
git commit -m "feat: add hello endpoint"
git push origin preprod
```

Poja's CI/CD pipeline runs automatically:
1. Format check (`./format.sh` + `git diff --exit-code`)
2. Tests (`./gradlew test` — unit + integration + JaCoCo)
3. Build (`sam build` → Lambda-compatible artifact)
4. Deploy (artifact uploaded to S3, Poja API triggers deployment)

**Full pipeline completes in under 10 minutes.**

### Step 5 — Go live

```
GET https://your-app-url/hello
→ Hello from Poja!
```

When ready for production, push to the `prod` branch or click **Activate** on the prod environment in the console.

---

## Features

Every feature is pre-configured and togglable from the Poja dashboard — no code changes, no cloud expertise needed.

| Feature | How to enable | What you get in your code |
|---|---|---|
| **GitHub-native CI/CD** | Always on | Two auto-generated workflows: `ci.yml` (every branch) and `cd-compute.yml` (preprod/prod) |
| **File storage** | One click — enable File Storage | `BucketComponent` with `upload()`, `download()`, `presign()` methods. `/health/bucket` endpoint added automatically. |
| **Async queues** | Set Queues Nb ≥ 1 | Extend `PojaEvent`, implement `Consumer<T>` in `service.event` — name it `{EventName}Service` |
| **Scheduled tasks** | Configure in Scheduled Tasks UI | Any async event + a cron expression (AWS EventBridge format) |
| **Email sending** | Always available | `Mailer` class with `accept(Email)`. Works sync or async. |
| **Sentry monitoring** | One click + set `SENTRY_DSN` env var | Exception tracking + Logback `@Slf4j` error integration |
| **Auto-scaling** | Always on | Handles 10 to 10,000 simultaneous requests automatically |
| **DDoS protection** | Always on | Built into AWS API Gateway — no configuration required |
| **No cold starts** | Always on | AWS Lambda SnapStart — JVM initialized at deploy time, restored from snapshot at runtime |

---

## Project structure (generated template)

```
your-app/
├── .github/
│   └── workflows/
│       ├── ci.yml                  # Runs on every push: format + test
│       └── cd-compute.yml          # Runs on preprod/prod push: build + deploy
├── src/
│   └── main/java/com/mycompany/
│       ├── endpoint/
│       │   ├── rest/controller/    # Your REST controllers go here
│       │   └── event/model/        # Your event classes go here (extend PojaEvent)
│       ├── service/
│       │   └── event/              # Your event consumers go here ({EventName}Service)
│       ├── file/bucket/            # BucketComponent (auto-generated when File Storage is enabled)
│       └── mail/                   # Mailer and Email classes
├── format.sh                       # Google Java Format — run before every push
├── format.bat                      # Same, for Windows
└── build.gradle
```

---

## How AWS Lambda SnapStart eliminates cold starts

Traditional Spring Boot on Lambda: every cold start re-initializes the JVM from scratch — loading classes, starting the Spring context, wiring beans. This can take 5–15 seconds.

With **AWS Lambda SnapStart Spring Boot**:

1. At deploy time, Lambda fully initializes your function (JVM startup, class loading, Spring context initialization).
2. Lambda takes an encrypted snapshot of the fully initialized memory and disk state.
3. The snapshot is cached. When Lambda needs a new execution environment, it **restores from the snapshot** instead of initializing from scratch.

Result: your app responds in milliseconds on every request — including the very first request after a period of inactivity.

---

## Comparison

| | Poja | Heroku | AWS (self-managed) |
|---|---|---|---|
| Target stack | Spring Boot / Java only | Any language | Any stack |
| Pricing model | Simple usage-based | Tiered plans | Complex usage-based |
| Free tier | Forever (€2/month credit) | 550–1000 dyno-hours/month | 6 months for new customers only |
| Cold starts | None (SnapStart) | Yes (Eco dynos sleep after 30 min) | Configurable |
| CI/CD setup | Zero — auto-generated | Manual | Manual |
| Monitoring | CloudWatch pre-configured | Basic | CloudWatch (manual setup) |
| DevOps required | None | Minimal | Significant |
| Best for | Spring Boot teams of any size | Polyglot MVPs | Large, complex, heterogeneous systems |

---

## Documentation

| Resource | URL |
|---|---|
| Architecture overview (AWS Lambda, SnapStart, SAM) | [docs.poja.io/docs](https://docs.poja.io/docs) |
| CI/CD pipeline deep dive | [docs.poja.io/docs/poja-cicd-pipeline-github-actions-deployment](https://docs.poja.io/docs/poja-cicd-pipeline-github-actions-deployment) |
| Hello world — straight to the cloud | [docs.poja.io/docs/deploy-spring-boot-hello-world-application-to-the-cloud](https://docs.poja.io/docs/deploy-spring-boot-hello-world-application-to-the-cloud) |
| File storage (S3) | [docs.poja.io/docs/hello-world-but-with-persistent-memories](https://docs.poja.io/docs/hello-world-but-with-persistent-memories) |
| Async queues + email | [docs.poja.io/docs/hello-world-but-with-asynchronous-reply-by-email](https://docs.poja.io/docs/hello-world-but-with-asynchronous-reply-by-email) |
| Scheduled tasks | [docs.poja.io/docs/hello-world-but-with-scheduled-tasks](https://docs.poja.io/docs/hello-world-but-with-scheduled-tasks) |
| Load testing (6,000 req/s benchmark) | [docs.poja.io/docs/hello-world-but-with-1000-simultaneous-calls-for-10-minutes-straight](https://docs.poja.io/docs/hello-world-but-with-1000-simultaneous-calls-for-10-minutes-straight) |
| Sentry integration | [docs.poja.io/docs/monitor-errors-with-sentry](https://docs.poja.io/docs/monitor-errors-with-sentry) |
| Pricing | [poja.io/pricing](https://poja.io/pricing) |

---

## Legal

Poja is operated by **Poja SAS**, a French company registered at 6 Rue Paul Langevin, 94120 Fontenay-sous-Bois, France (SIREN: 943 914 259).

- [Terms and conditions](https://terms.poja.io)
- [Company registry](https://www.pappers.fr/entreprise/poja-943914259)

---

## Get started

**[Try Poja for free → console.poja.io](https://console.poja.io/login)**

> Free tier: €2 monthly credit, forever, for every customer. No credit card required to start.

---

*Keywords: Spring Boot PaaS, deploy Spring Boot without DevOps, AWS Lambda SnapStart Spring Boot, Spring Boot hosting, Java cloud deployment, Spring Boot serverless, Heroku alternative Spring Boot, Spring Boot CI/CD, Java PaaS, Spring Boot auto-scaling*
