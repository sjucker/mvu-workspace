# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

A **wrapper workspace** for the Musikverein Harmonie Urdorf (MVU) projects. It holds only
shared Claude Code configuration (this file, plus any skills/commands). The three MVU
projects are cloned in as sibling directories and are `.gitignore`d — each is its own Git
repo with its **own `CLAUDE.md`** that is the authoritative source for that project.

Read the relevant project's `CLAUDE.md` before working in it:

- `mvu-platform/CLAUDE.md` — Spring Boot / Vaadin backend + admin UI (Postgres, jOOQ, Flyway)
- `mvu-platform-app/CLAUDE.md` — Flutter mobile app
- `mvu-website/CLAUDE.md` — PHP public website (MySQL CMS + platform REST calls)

Do **not** duplicate their build/test/architecture details here — defer to those files.

## How the projects fit together (the part no single project's CLAUDE.md explains)

`mvu-platform` is the hub. It owns the domain data (concerts, events, members, absences,
repertoire, scores, supporters) in **PostgreSQL** and exposes a REST API under `/api`,
served at `https://admin.mvurdorf.ch` (also reachable as `https://platform.mvurdorf.ch`).
Both other projects are clients of that API — they do not share its database.

- **`mvu-platform-app`** (Flutter) authenticates via **Firebase JWT** and calls the secured
  REST API (`https://platform.mvurdorf.ch`) with a Bearer token.
- **`mvu-website`** (PHP) calls a handful of REST endpoints server-side via cURL, e.g.
  `https://admin.mvurdorf.ch/api/event`, `/api/event/probeplan`, `/api/supporter`
  (see `mvu-website/sql.inc.php`, `probeplan.mvurdorf.ch/index.php`, `dao/supporter.html.inc.php`).
  The website has its **own MySQL database** for page/gallery/content management — entirely
  separate from the platform's Postgres.

Practical consequence: a change to a REST endpoint in `mvu-platform` (path, request params,
or DTO shape) can break `mvu-website` and/or `mvu-platform-app`. When editing platform
endpoints, grep the other two projects for the endpoint path before assuming a change is safe.
