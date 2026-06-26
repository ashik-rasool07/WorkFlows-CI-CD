# karate-todo

[![cicd](https://github.com/karatelabs/karate-todo/actions/workflows/cicd.yml/badge.svg)](https://github.com/karatelabs/karate-todo/actions/workflows/cicd.yml)

The flagship demo for [Karate](https://karatelabs.io) v2: a self-contained TODO app wired up
with runnable examples of API tests, API mocks, UI automation (Testcontainers + headless Chrome),
performance tests (Gatling), and a GitHub Actions pipeline that publishes HTML reports.

This copy is set up as a CI/CD learning repo. The app stays the same, but the GitHub Actions side
now uses one workflow with many jobs so you can learn the full pipeline from a single run page.

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

## CI/CD workflows

This repo now uses multiple workflow files that depend on each other with `workflow_run`.

| Order | File | Trigger | What it does |
|---|---|---|---|
| 1 | [`.github/workflows/01-foundation.yml`](.github/workflows/01-foundation.yml) | `push`, `pull_request`, `workflow_dispatch` | Validate the Maven project, run API tests, package the app, and snapshot dependencies |
| 2 | [`.github/workflows/02-ui-smoke.yml`](.github/workflows/02-ui-smoke.yml) | after `01 - Foundation`, or manual | Run Docker-backed UI tests and a simple app smoke test |
| 3 | [`.github/workflows/03-security.yml`](.github/workflows/03-security.yml) | after `02 - UI And Smoke`, or manual | Run CodeQL and scan tracked files for secret patterns |
| 4 | [`.github/workflows/04-performance.yml`](.github/workflows/04-performance.yml) | after `03 - Security Checks`, or manual | Start the app and run the Gatling performance smoke |
| 5 | [`.github/workflows/05-publish.yml`](.github/workflows/05-publish.yml) | after `04 - Performance`, or manual | Regenerate reports and publish them to GitHub Pages |

How the dependency chain works:

1. `01 - Foundation` is the entry point.
2. When it finishes successfully, GitHub triggers `02 - UI And Smoke`.
3. After that, GitHub triggers `03 - Security Checks`.
4. Then `04 - Performance` runs.
5. Finally `05 - Publish Reports` runs.

Each workflow file contains inline comments explaining what each task does. Open the YAML files directly if you want the explanation next to the actual step definitions.

Important limitation:

- GitHub Actions can chain workflows with `workflow_run`, but it cannot make one workflow wait on two different upstream workflows at the same time. That is why this repo uses a linear chain instead of a branching graph across files.

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
