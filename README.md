# Bazel Multi-Language Hello World — CI/CD Portfolio

A hands-on CI/CD engineering project demonstrating how to build, test, and package a multi-language codebase with **Bazel**, **GitHub Actions**, and **Buildkite**.

The repository contains C++, Java, and Python examples in one Bazel workspace. It explores build graphs, automated testing, incremental builds, caching, CI pipelines, release artifacts, container packaging, and remote execution architecture.

**Project goals**
- Build and test multiple languages with a unified build system.
- Automate validation on code changes with CI pipelines.
- Produce a reproducible release bundle.
- Explore CI/CD portability across GitHub Actions and Buildkite.
- Document the difference between local caching and production remote build infrastructure.

## Project status

| Capability | Status |
|---|---|
| Bazel multi-language build | Implemented |
| Automated tests | Implemented |
| GitHub Actions CI | Configured |
| Release bundle target | Implemented |
| Buildkite pipeline | Requires a non-empty pipeline configuration and Buildkite setup |
| Remote cache / execution | Examples and architecture documented; requires an external service |

> **Note:** A committed pipeline configuration is not proof that the pipeline has run successfully. Buildkite status should be reported only after a real build has completed.

## Technology stack

- **Build system:** Bazel / Bazelisk
- **Languages:** C++, Java, Python
- **CI/CD:** GitHub Actions, Buildkite
- **Packaging:** Bazel release bundle, optional Docker workflow
- **Configuration:** Starlark, YAML, Bazel module configuration
- **Build engineering topics:** dependency graphs, automated tests, incremental builds, caching, artifact validation

## Repository structure

```text
.
├── .buildkite/
│   └── pipeline.yml          # Buildkite pipeline definition
├── .github/
│   └── workflows/
│       └── bazel.yml         # GitHub Actions CI
├── cpp/                      # C++ example and tests
├── java/                     # Java example
├── python/                   # Python example
├── deploy/                    # Container/deployment scripts
├── docs/
│   └── remote-execution.md   # Remote execution architecture
├── .bazelversion              # Bazel version selected by Bazelisk
├── .bazelrc                   # Bazel build configuration
├── MODULE.bazel               # Bazel module dependencies
└── BUILD.bazel                # Root build and release bundle targets
```

## Quick start

### Prerequisites

Install the following tools:

- Git
- Bazelisk
- A supported C++ compiler and build tools
- JDK 17 for the Java target
- Python 3

On macOS, you can install Bazelisk with Homebrew:

```bash
brew install bazelisk
bazel --version
java -version
python3 --version
```

Install any missing compiler or JDK dependencies for your operating system.

### Build, test, and run

Run these commands from the repository root:

```bash
# Build all targets
bazel build //...

# Run all tests
bazel test //... --test_output=errors

# Run the language examples
bazel run //cpp:hello
bazel run //java:hello
bazel run //python:hello

# Build the release bundle
bazel build //:release_bundle
```

The examples print greetings from C++, Java, and Python. Bazelisk reads `.bazelversion` and downloads the corresponding Bazel version when necessary.

## CI/CD pipelines

### GitHub Actions

The workflow is defined in `.github/workflows/bazel.yml`.

It is configured to run on:
- Pushes to branches
- Pull requests
- Manual workflow dispatch

The workflow uses Ubuntu and macOS runners to build all targets, run tests, and smoke-test the three language examples. If a job fails, available Bazel test logs are uploaded as workflow artifacts.

To inspect results, open the repository's **Actions** tab on GitHub.

### Buildkite

The Buildkite pipeline definition belongs in `.buildkite/pipeline.yml` in this repository. The file is version-controlled in GitHub, but **Buildkite executes it** after a Buildkite pipeline has been connected and configured to upload that file.

The intended pipeline validates the same project through the following steps:

1. Prepare the build environment and install required dependencies.
2. Install or invoke Bazelisk.
3. Build all Bazel targets.
4. Run automated tests.
5. Smoke-test the C++, Java, and Python examples.
6. Build and inspect `bazel-bin/release_bundle.tar.gz`.
7. Upload test logs and the release bundle as Buildkite artifacts, where available.

To enable Buildkite CI:

1. Ensure `.buildkite/pipeline.yml` contains the actual pipeline configuration and is committed to the `main` branch.
2. Connect `scmbuildrelease/buildkite-multilang-hello-world` to Buildkite.
3. Configure the pipeline to upload steps from `.buildkite/pipeline.yml`.
4. Confirm that the configured agent queue and execution environment match the pipeline.
5. Trigger a build and review the logs and artifacts.

GitHub Actions and Buildkite are independent CI systems. A successful GitHub Actions run does not establish that Buildkite is configured or passing.

## Learning path: Levels 1–10

### Level 1 — Hello World

Learn the basic Bazel workspace and target structure.

Key concepts: `MODULE.bazel`, `BUILD.bazel`, labels, rules, and `bazel run`.

### Level 2 — Dependency graph

Explore targets and dependencies:

```bash
bazel query //...
bazel query 'deps(//cpp:hello)'
bazel cquery //...
bazel aquery //cpp:hello
```

These commands expose different views of the build graph, configured targets, and actions.

### Level 3 — Automated testing

```bash
bazel test //...
bazel test //cpp:hello_test --test_output=all
```

Use test results and failure output to investigate regressions.

### Level 4 — Parallel builds

Bazel schedules independent build actions concurrently.

```bash
bazel clean
bazel build //... --jobs=1 --profile=/tmp/serial.profile

bazel clean
bazel build //... --jobs=auto --profile=/tmp/parallel.profile

bazel analyze-profile /tmp/parallel.profile
```

Build times depend on hardware, dependencies, and cache state. A small project may not show a meaningful parallel-build speedup.

### Level 5 — Incremental builds and local caching

```bash
bazel build //...
bazel build //...
```

Repeat the build, then modify a source file and build again. Observe which actions Bazel can reuse and which need to run again.

Local caching is available without a separate remote caching service.

### Level 6 — Remote caching

The repository includes example remote-cache settings in `.bazelrc`. They remain disabled until a compatible service is configured.

A typical configuration may look like this:

```text
# Example only — replace with a real endpoint.
# build:remote --remote_cache=grpcs://YOUR_CACHE_ENDPOINT
# build:remote --remote_upload_local_results
```

A production setup also requires appropriate authentication, access controls, and service configuration. Do not commit credentials or tokens.

### Level 7 — CI automation

Use `.github/workflows/bazel.yml` to validate changes automatically with GitHub Actions. Use `.buildkite/pipeline.yml` for the separately configured Buildkite pipeline.

Compare the two systems' pipeline configuration, logs, job environments, failure handling, and artifact collection.

### Level 8 — Release artifact

```bash
bazel build //:release_bundle
```

The target is intended to produce:

```text
bazel-bin/release_bundle.tar.gz
```

Inspect the archive:

```bash
test -s bazel-bin/release_bundle.tar.gz
tar -tzf bazel-bin/release_bundle.tar.gz
```

The bundle includes outputs for the example applications, including the Java deploy JAR. Verify its contents after building rather than assuming the artifact exists.

### Level 9 — Docker and container packaging

Docker Desktop is optional for local container workflows.

```bash
bazel build //cpp:hello
./deploy/deploy.sh
docker run --rm bazel-hello:local
```

Container publishing is a separate operation from building the release bundle. If using GitHub Container Registry, verify the publishing workflow, repository permissions, and image tags before relying on a published image.

### Level 10 — Remote execution architecture

Read [`docs/remote-execution.md`](docs/remote-execution.md).

Production remote execution requires a compatible Remote Execution API service, workers, toolchains, platform properties, storage, monitoring, and access controls. This repository documents the concepts; it does not claim that a remote execution cluster is provisioned.

## Troubleshooting and diagnostics

```bash
# Inspect Bazel environment and output directories
bazel info

# Run tests with failure details
bazel test //... --test_output=errors

# Explain build decisions
bazel build //... --explain=/tmp/bazel-explain.log

# Capture execution details
bazel build //... --execution_log_json_file=/tmp/bazel-execution.json
```

Common issues:

- **First build is slow:** Bazel may need to download dependencies.
- **Java build fails:** Check that JDK 17 is installed and available to the build.
- **C++ build fails:** Check that the compiler and required build tools are installed.
- **Buildkite does not start:** Confirm the pipeline is connected to the correct repository, the YAML file is committed, and the pipeline is configured to upload it.
- **Buildkite cannot run a job:** Check the configured queue and whether an eligible agent or hosted execution environment is available.
- **Release archive is missing:** Run `bazel build //:release_bundle` and inspect the build output.
- **Remote caching or execution fails:** Confirm the endpoint, credentials, and remote service configuration. These features are not enabled automatically by this demo.

## Engineering takeaways

This project demonstrates several practical build and release engineering principles:

- One build system can manage targets across multiple languages.
- Build and test commands should be repeatable locally and in CI.
- CI configuration should be version-controlled alongside the source code.
- A green build should be supported by actual test results and artifact validation.
- Local caching, remote caching, and remote execution are different capabilities and should be documented accurately.
- Release artifacts should be verified before being handed to downstream consumers.

## License

See the repository's `LICENSE` file.
