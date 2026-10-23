# Repository Audit — fattain-naime/fattain-naime

| Field | Value |
| --- | --- |
| Audit date | 2026-10-23 |
| Commit | `20c7357` (`main`) |
| Scope | `README.md`, `assets/`, repository metadata, outbound links, git history |
| Methods | File inspection, GitHub REST API, HTTP status checks, DNS-over-HTTPS, `markdownlint-cli2` |

## Executive summary

This repository is a GitHub profile README. At audit start it contained one 233-line `README.md` and a single asset, with no application code, build, or test surface. Its structure and rendering are sound, all GitHub-hosted identities resolve, and no secrets are present in any revision. The audit found two high-impact issues (a mislabeled orphaned image and dead venture links/icons), four medium issues (a legacy stats host, missing image alt text, unverifiable primary websites, and unlinked or unmatched open-source claims), and several low-priority hygiene gaps. Six findings were addressed directly in this branch (A2 only partially); the remaining items require owner decisions or external service changes and are documented below.

## Findings

### A1 — High — `assets/banner.png` was a mislabeled, orphaned JPEG (fixed)

- The file's magic bytes were `FF D8 FF E0 ... JFIF`, not the PNG signature `89 50 4E 47`, so it was a JPEG with a `.png` extension.
- Dimensions were 1024×1024; size 389,883 bytes.
- It was added in `c3751e2` but stopped being referenced when `20c7357` replaced the local header with a Capsule Render banner; `README.md` contained no reference to it.
- Effects: misleading extension breaks tooling that trusts file names, and the binary was dead weight in every clone.
- Action: removed with `git rm`. The file remains recoverable from history with `git show c3751e2:assets/banner.png`.

### A2 — High — Broken venture links and icons (partially fixed)

| Venture | Check | Result |
| --- | --- | --- |
| BASH website | DNS for `bashub.org` and `www.bashub.org` | NXDOMAIN (Google DoH `Status: 3`) |
| BASH icon | Google favicon service | HTTP 404 |
| SMS Hall website | `https://smshall.com/`, `https://www.smshall.com/`, `http://smshall.com/` | HTTP 404 from `nginx/1.24.0` |
| SMS Hall icon | `https://smshall.com/favicon-32x32.png` | HTTP 404 |

Actions taken:

- Replaced both broken favicon images with emoji glyphs (📨 for SMS Hall, 🚀 for BASH) so the table renders without broken-image placeholders.
- Removed the BASH hyperlink because the domain no longer resolves; the venture name and description remain.
- Kept the SMS Hall link because the domain resolves and the 404 is likely an origin/virtual-host misconfiguration rather than an abandoned domain.

Recommendations:

- Supply the current BASH URL or remove the row if the initiative is defunct.
- Fix the SMS Hall deployment or update the link to its current home.

### A3 — Medium — Deprecated streak-stats host (fixed)

`github-readme-streak-stats.herokuapp.com` is the legacy Heroku-hosted instance of the project; the upstream repository documents `streak-stats.demolab.com` as its demo and canonical host. Both hosts returned an identical 7,032-byte SVG during this audit, so there is no visual change, but the legacy host is undocumented and has no availability guarantee.

Action: switched to `https://streak-stats.demolab.com/` with all query parameters unchanged.

### A4 — Medium — Missing image alt text (fixed)

`markdownlint-cli2` reported ten `MD045/no-alt-text` violations in the "Let's Connect" badges and the footer image. The header also used the non-descriptive `alt="Header"`.

Action:

- Added descriptive alt text to all eight connect badges and the blog badge.
- Added a descriptive alt to the header banner.
- Set the decorative footer wave to `alt=""` so assistive technology skips it.

### A5 — Low — Markdown hygiene (fixed)

| Rule | Location | Issue |
| --- | --- | --- |
| MD012 | former line 83 | Two consecutive blank lines |
| MD040 | former line 88 | Fenced ASCII art had no language |
| MD060 | former line 75 | Table delimiter style inconsistent |

Action: fixed all three and added `.markdownlint-cli2.jsonc`. MD013 (long badge URLs), MD033 (inline HTML needed for centering) and MD041 (profile READMEs start with a `<p>` block) are disabled there with a comment explaining why. `npx markdownlint-cli2 README.md` now reports 0 issues.

### A6 — Low — Inconsistent Builder Hall destination (fixed)

The About section linked Builder Hall Ltd. to `https://github.com/builderhall`, while the Ventures table linked to `https://builderhall.com`; the GitHub organization API lists `https://builderhall.com` as its website.

Action: both references now use `https://builderhall.com`.

### A7 — Medium — Primary websites could not be verified from this environment (open)

`iamnaime.info.bd`, `builderhall.com`, `giglovin.com` and `shopika.com.bd` resolve in DNS but TCP/TLS requests timed out after 8–15 seconds, including with a desktop browser user agent. This environment cannot distinguish geo/bot filtering from an outage, so these are not classified as broken.

Recommendation: verify from a neutral uptime monitor and add alerting for the site used in the README header.

### A8 — Medium — Open Source Highlights were unlinked and partly unmatched (partially resolved)

Follow-up: `OwnPay` and five linked repositories (`js-web-terminal`, `rcloneCWP`, `local-dev-hub`, `rk-data-scraping`, `automatic-update-via-github`) were added to the section after the audit. The four original claims below are still unlinked and only one has an obvious match:

| Claim | Matching public repository | Recommendation |
| --- | --- | --- |
| bKash Payment Integrations | None (no `bkash` or WooCommerce repository) | Link the repository or revise the claim |
| WooCommerce SMS Automation | None (the account has PipraPay SMS plugins, not WooCommerce) | Link the repository or revise the claim |
| Currency Auto-Sync Systems | `currency-rate-auto-update` | Link it |
| WordPress Custom Execution Tools | None found | Link the repository or revise the claim |

The "Impact & Milestones" figures (2,000+ clients, 2,500+ projects, 1,000+ platform users) have no supporting evidence in the repository, and the client count exceeding the platform-user count is counterintuitive. The fourth cell, "Featured In Media", shows only an emoji instead of a metric, unlike the other three. Consider qualifying the numbers, citing a source, or replacing the fourth cell with a verifiable figure.

### A9 — Low — Repository metadata is still at defaults (open)

| Field | Current value | Suggested value |
| --- | --- | --- |
| Description | `Config files for my GitHub profile.` | A profile summary, for example `Founder & CEO at Builder Hall · Full-stack engineer & security researcher` |
| Homepage | `https://github.com/fattain-naime` | `https://iamnaime.info.bd` |
| Topics | `config`, `github-config` | `profile`, `laravel`, `php`, `cybersecurity`, `fintech` |
| License | None (community health 28%) | Optional: add a content license if reuse is intended |

Metadata changes are repository settings rather than source changes, so they were not applied here.

### A10 — Info — No CI and several third-party image dependencies

There are no workflows. The README embeds images from several third-party services (Capsule Render, Shields, Komarev, Readme Typing SVG, Skill Icons, GitHub Profile Summary Cards and GitHub avatars). Every image endpoint tested returned HTTP 200 except the broken icons listed in A2, but availability is outside the repository's control. The new markdownlint configuration can back a small CI job; a workflow was not added to avoid imposing release infrastructure on a profile repository.

## Link and asset checks

| Resource | Result |
| --- | --- |
| Capsule Render header and footer | 200, `image/svg+xml` |
| Shields.io badges (all variants) | 200, `image/svg+xml` |
| Komarev profile-views badge | 200, `image/svg+xml` |
| Readme Typing SVG | 200, `image/svg+xml` |
| Skill Icons tech stack | 200, `image/svg+xml` |
| GitHub Profile Summary Cards (5 endpoints) | 200, `image/svg+xml` |
| Streak stats, new host | 200, `image/svg+xml`, 7,032 bytes |
| GitHub avatars `u/101276209`, `u/275533731` | 200, `image/png` and `image/jpeg` |
| `github.com/fattain-naime`, `/builderhall`, `/own-pay` | 200 |
| X, Facebook, Telegram, Discord, Google search | 200 |
| LinkedIn | 999 (bot protection), not broken |
| Instagram | 429 (rate limit), not broken |
| `bashub.org` | NXDOMAIN |
| `smshall.com` | 404 |
| GigLovin, Shopika, Builder Hall, personal site | Timeout from this environment (DNS resolves) |

## What is healthy

- All GitHub identities and organizations referenced by the cards exist, and every card endpoint returned HTTP 200.
- The ASCII art is valid ANSI Shadow output for `LAB 0X4E`; it is not corrupted or truncated.
- A scan of every revision for API keys, tokens, passwords and private-key blocks found no secrets; the public contact email is intentional.
- Heading hierarchy, tables, and fenced blocks are well formed, and the README now passes the configured markdownlint rules.
- All outbound URLs are syntactically valid; reachability is reported in the link table above.

## Changes applied

1. Removed the mislabeled, unreferenced `assets/banner.png`.
2. Replaced the 404 SMS Hall and BASH icons with emoji glyphs; removed the dead BASH hyperlink.
3. Migrated the streak card to `streak-stats.demolab.com`.
4. Added alt text to every image and made the decorative footer `alt=""`.
5. Unified the Builder Hall link on `https://builderhall.com`.
6. Fixed MD012, MD040 and MD060, and added `.markdownlint-cli2.jsonc`.
7. Added OwnPay and five linked repositories to Open Source Highlights (follow-up).

Verification:

```bash
npx markdownlint-cli2 README.md   # 0 issues
git ls-files                      # README.md, AUDIT.md, .markdownlint-cli2.jsonc
```

## Recommended next steps

1. Provide a working BASH URL or remove the row; repair or replace the SMS Hall link.
2. Add uptime monitoring for the four websites that timed out during this audit.
3. Link the four original Open Source Highlights to public repositories, or rewrite those rows to match what is public.
4. Update the repository description, homepage and topics; decide whether a content license is wanted.
5. Optionally add a CI workflow that runs `markdownlint-cli2` on push and pull request.
