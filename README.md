# karate-todo

[![cicd](https://github.com/karatelabs/karate-todo/actions/workflows/cicd.yml/badge.svg)](https://github.com/karatelabs/karate-todo/actions/workflows/cicd.yml)

The flagship demo for [Karate](https://karatelabs.io) v2: a self-contained TODO app wired up
with runnable examples of API tests, API mocks, UI automation (Testcontainers + headless Chrome),
performance tests (Gatling), and GitHub Actions pipelines that publish HTML reports.
[03-ui-tests.yml](.github/workflows/03-ui-tests.yml)
This copy is set up as a CI/CD learning repo. The app stays the same, but the GitHub Actions side
now includes 10 workflows that cover common pipeline patterns one file at a time.

## See it live

These are produced fresh on every green push to `main`:

| Report | Link |
|---|---|
| Karate summary (API + UI) | [karate-summary.html](https://karatelabs.github.io/karate-todo/latest/karate/karate-summary.html) |
| Parallel timeline | [karate-timeline.html](https://karatelabs.github.io/karate-todo/latest/karate/karate-timeline.html) |
| UI feature with embedded screenshots | [simple.feature](https://karatelabs.github.io/karate-todo/latest/karate/feature-html/target.test-classes.app.ui.simple.simple.html) |
| API CRUD feature (request / response log) | [simple.feature](https://karatelabs.github.io/karate-todo/latest/karate/feature-html/target.test-classes.app.api.simple.simple.html) |
| Gatling performance report | [index.html](https://karatelabs.github.io/karate-todo/latest/gatling/index.html) |

In the Karate summary, try the tag filter (top right) to narrow down to `@smoke`, `@crud`, `@api`,
`@ui`, `@data-driven`, `@java-interop`, `@match`, or `@call`.

## Quickstart

```bash
git clone https://github.com/karatelabs/karate-todo.git
cd karate-todo
./mvnw test
```

If `ApiTest` passes, the full API suite ran against an in-process app. No Docker needed.

## Run the app

```bash
./mvnw test -Dtest=LocalRunner
```

Open [http://localhost:8080](http://localhost:8080) for a working TODO app backed by the same API the tests hit.
Stop with `Ctrl+C`.

The app uses a singleton session so tests mutate the same state you see in the browser. Run
`LocalRunner` in one terminal and `ApiTest` in another to watch API-driven mutations land in the UI.

## What's in the box

| Type | Entry point | Docs |
|---|---|---|
| API tests | [`app/api/`](src/test/java/app/api/) run by [`ApiTest.java`](src/test/java/app/api/ApiTest.java) | [Making Requests](https://docs.karatelabs.io/http-requests/making-requests) |
| UI tests (Testcontainers) | [`app/ui/`](src/test/java/app/ui/) run by [`UiTest.java`](src/test/java/app/ui/UiTest.java) with `-Pui` | [UI Testing](https://docs.karatelabs.io/extensions/ui-testing) |
| API mock | [`app/mock/`](src/test/java/app/mock/) started locally via [`MockRunner.java`](src/test/java/app/mock/MockRunner.java) | [Test Doubles](https://docs.karatelabs.io/extensions/test-doubles) |
| Performance | [`app/perf/TodoSimulation.java`](src/test/java/app/perf/TodoSimulation.java) | [Performance Testing](https://docs.karatelabs.io/extensions/performance-testing) |

Feature files tagged `@external` hit real external hosts and are excluded from CI by `~@external`.

## UI tests with Testcontainers

UI tests run a containerized `chromedp/headless-shell` via
[`ContainerDriverProvider`](src/test/java/app/ui/support/ContainerDriverProvider.java), with the in-process app reachable
from the browser at `host.docker.internal`:

```bash
./mvnw verify -Pui
```

This needs Docker running. The profile is opt-in so `./mvnw test` stays fast and Docker-free.

## Performance (Gatling)

Use two terminals:

```bash
# terminal 1
./mvnw test -Dtest=LocalRunner

# terminal 2
./mvnw test -P gatling
```

The HTML report is written to `target/gatling/todosimulation-*/index.html`.

## GitHub Codespaces

[![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/karatelabs/karate-todo)

The devcontainer ships with JDK 21, Maven, and Docker-in-Docker so every command above works on a fresh codespace.
Port 8080 auto-forwards for `LocalRunner`. After `./mvnw verify -Pui`, right-click any report `.html`
and open it with Live Server.

## CI/CD learning set

This repo includes 10 GitHub Actions workflows so you can learn CI/CD in small steps and then compare that to
one integrated pipeline. Every workflow can also be started manually from the GitHub Actions UI with `workflow_dispatch`.

| Workflow | File | What it teaches |
|---|---|---|
| 1 | [`.github/workflows/01-validate.yml`](.github/workflows/01-validate.yml) | Smallest useful CI job: checkout, Java setup, Maven cache, `validate` |
| 2 | [`.github/workflows/02-api-tests.yml`](.github/workflows/02-api-tests.yml) | Run API tests on push / PR and upload reports as artifacts |
| 3 | [`.github/workflows/03-ui-tests.yml`](.github/workflows/03-ui-tests.yml) | Run Docker-backed UI tests with Testcontainers |
| 4 | [`.github/workflows/04-package.yml`](.github/workflows/04-package.yml) | Build and store a package artifact on tag creation |
| 5 | [`.github/workflows/05-smoke-test.yml`](.github/workflows/05-smoke-test.yml) | Start the app in the background and verify it responds |
| 6 | [`.github/workflows/06-gatling.yml`](.github/workflows/06-gatling.yml) | Scheduled performance smoke testing with Gatling |
| 7 | [`.github/workflows/07-dependency-review.yml`](.github/workflows/07-dependency-review.yml) | PR dependency review, plus a manual dependency snapshot run |
| 8 | [`.github/workflows/08-codeql.yml`](.github/workflows/08-codeql.yml) | Static security analysis with CodeQL |
| 9 | [`.github/workflows/09-secret-scan.yml`](.github/workflows/09-secret-scan.yml) | Secret detection against tracked source files |
| 10 | [`.github/workflows/cicd.yml`](.github/workflows/cicd.yml) | An end-to-end pipeline that chains tests, performance, report scanning, and publish |

Suggested order:

1. Start with workflow 1 to understand the minimum GitHub Actions structure.
2. Add workflows 2 to 5 to learn test execution, artifacts, and smoke checks.
3. Add workflows 6 to 9 to cover performance, supply-chain review, static analysis, and secret scanning.
4. Read workflow 10 last to see how separate ideas compose into a release-style pipeline.

## End-to-end pipeline

[`.github/workflows/cicd.yml`](.github/workflows/cicd.yml) stages jobs as `tests` -> `gatling` -> `secret-scan` -> `publish`,
runs on every push to `main` and on manual dispatch, and publishes the assembled reports to GitHub Pages.

| Stage | What it does |
|---|---|
| `tests` | `./mvnw verify -Pui` for one hybrid suite (API + UI) via Testcontainers |
| `gatling` | Starts `LocalRunner` in the background and runs `TodoSimulation` against it |
| `secret-scan` | Greps report artifacts for common token and private-key patterns and fails the build on hit |
| `publish` | On `main` only: assembles `latest/{karate,gatling}/` and pushes to `gh-pages` |

Live reports: [karatelabs.github.io/karate-todo/latest/](https://karatelabs.github.io/karate-todo/latest/)

## IDE plugins

* [Karate for VS Code](https://marketplace.visualstudio.com/items?itemName=karatelabs.karate)
* [Karate for IntelliJ IDEA](https://plugins.jetbrains.com/plugin/19232-karate)

## Prerequisites

* [Java 21+](https://www.oracle.com/java/technologies/downloads) with `JAVA_HOME` set
* [Git](https://git-scm.com/download) or [download as ZIP](https://github.com/karatelabs/karate-todo/archive/refs/heads/main.zip)
* [Docker](https://docs.docker.com/get-docker/) only for `./mvnw verify -Pui`

Codespaces and VS Code devcontainer users already have the required tooling preinstalled.

## Gradle

See [Install & Get Started](https://docs.karatelabs.io/getting-started/install-dependencies) for Gradle setup.
