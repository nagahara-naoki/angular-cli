# Angular CLI font stylesheet cache reproduction

Minimal application using Angular CLI and `@angular/build` 22.2.6. Requires a compatible Node.js version (tested with 24.22.2). The shell commands below are for macOS/Linux.

## Setup

```sh
git clone --single-branch --branch repro-font-inline-cache https://github.com/nagahara-naoki/angular-cli.git font-cache-repro
cd font-cache-repro
npm ci
```

The application is named `repro`. It enables the CLI's persistent cache and font inlining, and imports Google Fonts from `src/styles.css`.

## 1. Missing cache directory

In a fresh clone, with an internet connection:

```sh
npm run build
ls .angular/cache/22.2.6/repro/angular-build-fonts
```

The build succeeds, but `ls` reports that the directory does not exist. The font stylesheet was not saved.

## 2. Valid cache is discarded

Create the missing directory and build again while online:

```sh
mkdir -p .angular/cache/22.2.6/repro/angular-build-fonts
npm run build
ls .angular/cache/22.2.6/repro/angular-build-fonts
```

The build succeeds and the directory contains one cache file.

Now run another build through an unreachable HTTPS proxy:

```sh
HTTPS_PROXY=http://127.0.0.1:9 npm run build
ls .angular/cache/22.2.6/repro/angular-build-fonts
```

The build fails while downloading the font stylesheet. The previously saved cache file has been deleted, leaving the directory empty. The proxy environment variable applies only to that command.

Expected: the directory is created automatically, and subsequent builds reuse the cached stylesheet without another request to Google Fonts.

Actual: the directory is not created; even if created manually, cached content is discarded because the read and write paths use different hash encodings.
