# Testing Guide

This document explains how to build and run unit and integration tests for OpenSlide.

## 1. Prerequisites

Required for all test builds:

- Meson
- Ninja
- C compiler (Clang or GCC)
- Project build dependencies listed in `README.md`

Required for the main integration driver (`test/driver.py`):

- `cjpeg`
- `djpeg`
- `xdelta3`

Optional tools used by specific test modes:

- `valgrind` for memory checking
- `clang` for sanitizer runs
- `gcov` for coverage reports

Required only for cloud integration tests:

- Docker (running locally)

## 2. Configure a Test Build

From the repository root:

```bash
meson setup builddir -Dtest=enabled
meson compile -C builddir
```

Enable cloud providers and Docker-backed cloud integration test:

```bash
meson setup builddir \
  -Dtest=enabled \
  -Ds3=enabled \
  -Dgcs=enabled \
  -Dazure=enabled \
  -Dcloud_tests=enabled
meson compile -C builddir
```

## 3. Unit/Smoke Tests (Meson Tests)

Run all Meson-registered tests:

```bash
meson test -C builddir --print-errorlogs
```

List available Meson tests:

```bash
meson test -C builddir --list
```

Run only cloud suite tests (when enabled):

```bash
meson test -C builddir --suite cloud --print-errorlogs
```

Run the cloud integration test by name:

```bash
meson test -C builddir cloud-read --print-errorlogs
```

## 4. Integration Tests (Corpus-Driven)

The main integration harness is generated at `builddir/test/driver`.

Run the full corpus integration suite:

```bash
builddir/test/driver run
```

Run a subset by pattern:

```bash
builddir/test/driver run aperio*
```

Notes:

- The driver unpacks/fetches test data as needed.
- Use `OPENSLIDE_TEST_XFAIL` (comma-separated case names) to mark expected failures for a run.

Example:

```bash
OPENSLIDE_TEST_XFAIL=example-case-1,example-case-2 builddir/test/driver run
```

## 5. Cloud Integration Tests

Run cloud integration directly through the driver:

```bash
builddir/test/driver cloud
```

Run through Meson:

```bash
meson test -C builddir cloud-read --print-errorlogs
```

Choose a specific cloud test case:

```bash
OPENSLIDE_CLOUD_TEST_CASE=trestle builddir/test/driver cloud
```

Cloud test behavior:

- Starts local emulators for S3, GCS, and Azure Blob.
- Uploads test fixtures to each emulator.
- Verifies `s3://`, `gs://`, and `az://` reads via `test/try_open`.

## 6. Advanced Test Modes

Valgrind:

```bash
builddir/test/driver valgrind
```

Sanitizers (temporary rebuild with sanitizer options):

```bash
builddir/test/driver sanitize
```

Coverage report:

```bash
builddir/test/driver coverage coverage.txt
```

## 7. Useful Maintenance Commands

Clean unpacked local test data for selected cases:

```bash
builddir/test/driver clean
```

Clean only matching cases:

```bash
builddir/test/driver clean aperio*
```

## 8. Troubleshooting

If Meson tests do not run:

- Reconfigure with `-Dtest=enabled`.
- Rebuild with `meson compile -C builddir`.

If integration tests fail early with missing tools:

- Install `cjpeg`, `djpeg`, and `xdelta3`.
- Re-run `meson setup` and `meson compile`.

If cloud tests fail immediately:

- Ensure Docker is running.
- Ensure build was configured with `-Ds3=enabled -Dgcs=enabled -Dazure=enabled -Dcloud_tests=enabled`.

If you changed provider/test code and behavior looks stale:

```bash
meson compile -C builddir --clean
meson compile -C builddir
```
