# PROJECT KNOWLEDGE BASE

**Generated:** 2026-04-27
**Type:** Obsidian Personal Knowledge Management Vault

## OVERVIEW
Obsidian vault organized with the PARA method (Tiago Forte). Contains software development best-practice notes across frontend, backend, DevOps, languages, and methodologies. No source code — pure markdown knowledge base.

## STRUCTURE
```
notes/
├── 00-Inbox/           # Capture bucket (empty)
├── 01-Project/         # Active projects (empty)
├── 03-Resource/        # Reference materials — 24 notes, 6 domains
│   ├── 01-Frontend/    # CSS, HTML5, JS, React, Tailwind, TS, Vite
│   ├── 02-Languages/   # Go, Rust
│   ├── 03-Architecture/# Backend, DB, Redis, SQL
│   ├── 04-DevOps/      # K8s, etcd, SRE
│   ├── 05-Methodologies/# TDD, BDD, DDD
│   └── 06-Agent/       # AI Skill design
├── 04-Archive/         # Completed items (empty)
└── .obsidian/          # Vault configuration
```

## WHERE TO LOOK
| Task | Location | Notes |
|------|----------|-------|
| Frontend best practices | `03-Resource/01-Frontend/` | CSS, HTML5, JS, React, Tailwind, TS, Vite |
| Language guides | `03-Resource/02-Languages/` | Go, Rust (4 files) |
| Backend architecture | `03-Resource/03-Architecture/` | DB design, Redis, SQL, backend patterns |
| DevOps | `03-Resource/04-DevOps/` | K8s, etcd, SRE operations |
| Methodologies | `03-Resource/05-Methodologies/` | TDD, BDD, DDD |
| AI Skill design | `03-Resource/06-Agent/` | Skill patterns and how-to |
| Obsidian config | `.obsidian/` | core-plugins.json, workspace state |

## CONVENTIONS
- **PARA numbering prefix**: `00-`, `01-`, `03-`, `04-` enforces sort order
- **Resource subdirs**: numbered `01-` to `06-` for consistent sorting
- **Clean filenames**: removed redundant "最佳实践" suffixes, kept concise names
- **Markdown only**: all content is `.md`; no code files, no build system

## ANTI-PATTERNS (THIS PROJECT)
- None applicable — this is a knowledge base, not a codebase

## COMMANDS
No build/test commands. This is a knowledge base.

## NOTES
- **Filename quoting required**: some filenames contain spaces; always quote in shell
- **Content moved**: all notes previously in `02-Area/` relocated to `03-Resource/` (reference materials fit PARA's Resource category)
- **02-Area empty**: ready for actual ongoing responsibilities when needed
- **Obsidian plugins enabled**: graph, canvas, daily-notes, templates, bookmarks, sync, properties