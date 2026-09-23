# partner-bootcamp-skilljar-lab-host

Transient **public** host for the Posit Partner Bootcamp hands-on lab ZIPs that are
published into Skilljar as `WEB_PACKAGE` lessons.

## Why this repo exists

Skilljar's web-package API (`POST /v1/web-packages`) has **no file-upload field**. It
requires a publicly-fetchable `content_url`, which Skilljar downloads **server-side and
unauthenticated**, then re-hosts on its own S3 bucket
(`everpath-course-content.s3.amazonaws.com`). That means:

- The source repo for the labs
  ([`posit/partner-bootcamp-skilljar`](https://github.com/posit/partner-bootcamp-skilljar))
  is **private**, and Skilljar cannot fetch raw URLs from a private/internal repo.
- The GitHub Actions publish workflow in that repo is **non-functional** (private repo +
  no private-Pages plan means its GitHub Pages step never yields a public URL).

So the working publish path is manual: render each lab locally, zip it, push the ZIP
**here** (public), point Skilljar at the raw URL, and let Skilljar re-host it. Once
Skilljar has copied a ZIP to its own S3, **this repo is no longer a live dependency** of
the lesson. The public host is only needed transiently during web-package creation.

## Contents

One ZIP per lab, each containing a single self-contained `index.html` at the archive root:

| ZIP | Skilljar lesson (course `1x34hm1qg9f5x`) | order |
|-----|-------------------------------------------|-------|
| `workbench_install_lab_01.zip` | Workbench Install Lab 1: Base Installation | 350 |
| `workbench_install_lab_02.zip` | Workbench Install Lab 2: SSL, OIDC & Provisioning | 360 |
| `workbench_install_lab_03.zip` | Workbench Install Lab 3: Containers (Optional) | 370 |
| `connect_install_lab_01.zip` | Connect Install Lab 1: Install & HTTPS | 450 |
| `connect_install_lab_02.zip` | Connect Install Lab 2: Auth, Publishing & OAuth | 460 |
| `connect_install_lab_03.zip` | Connect Install Lab 3: Metrics & OpenTelemetry | 470 |
| `package_manager_install_lab_01.zip` | Package Manager Install Lab: Install & Configure | 560 |
| `chronicle_install_lab_01.zip` | Chronicle Install Lab: Install & Configure | 630 |

## Security

**No secrets belong in these ZIPs.** The labs reference the bootcamp AWS Cognito OIDC
issuer and client-ids (non-sensitive) but use `<CLIENT_SECRET_PROVIDED_BY_INSTRUCTOR>`
and `<GOOGLE_OAUTH_CLIENT_ID>` / `<GOOGLE_OAUTH_CLIENT_SECRET>` placeholders for anything
sensitive. Client-secrets are distributed to participants out-of-band (email). Because
this repo is public, any ZIP pushed here is world-readable, so **never publish a lab
build that contains a live secret.** The publish pipeline includes an adversarial
secret-scan of the rendered HTML before upload for exactly this reason.

## Publishing

The end-to-end procedure (render navbar-free → verify → zip → push here → create
web-package → poll `READY` → PATCH lesson) lives in the `publish-lab-to-skilljar` skill
and `plan/10` in the source repo. Filenames here are **stable across re-publishes** so
the raw URLs never change; GitHub's raw CDN caches briefly, so the pipeline polls the raw
URL until the served `content-length` matches the new ZIP before creating the package.
