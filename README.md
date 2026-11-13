# MVU Workspace

Shared developer configuration (Claude Code `CLAUDE.md`, skills, and commands) for the **Musikverein Harmonie Urdorf (MVU)** projects.

This repository is a *wrapper*. It does not contain application code — the three MVU
projects are cloned in as sibling directories and are excluded via `.gitignore` (each
has its own Git repository). Only the shared tooling lives here.

## Getting started

Clone the projects into this directory:

```bash
git clone git@github.com:sjucker/mvu-platform.git
git clone git@github.com:sjucker/mvu-website.git
git clone https://github.com/sjucker/mvu-platform-app.git
```

Resulting layout:

```
mvu-workspace/          # this repo (shared CLAUDE.md, skills, commands)
├── mvu-platform/       # Spring Boot + Vaadin admin UI, Postgres, REST API
├── mvu-website/        # PHP public website (mvurdorf.ch)
└── mvu-platform-app/   # Flutter mobile app
```

> Note: the `mvu-website` checkout's `origin` currently points at this workspace repo;
> confirm the correct upstream when pushing.

## The projects

| Project            | Stack                                                                 | Role                                              |
|--------------------|-----------------------------------------------------------------------|---------------------------------------------------|
| `mvu-platform`     | Spring Boot 3.5 / Java 25, Vaadin Flow 24, PostgreSQL (jOOQ + Flyway) | Admin UI + the REST API consumed by the other two |
| `mvu-website`      | PHP + MySQL                                                           | Public website at mvurdorf.ch                     |
| `mvu-platform-app` | Flutter                                                               | Mobile app (iOS / Android)                        |

Each project has its own `CLAUDE.md` with build, test, and architecture details.
