# security-audit

Coding-agent skill for multi-phase security audits with independently verified, machine-readable
findings. Reconnaissance, coverage-led hunting, adversarial candidate validation, structured output,
independent record verification and target-neutral reporting.

The workflow writes `run-metadata.json`, `architecture.md`, `coverage-ledger.json`, `findings.json`,
`REPORT.md`, `FINDINGS-DETAIL.md` and `NEEDS-VALIDATION.md`, and validates the two data files with
zero-dependency Node.js validators against `report-schema.json`.

## Usage

See [SKILL.md](SKILL.md) for modes, phases and rules. Guidance mode answers security questions and
focused reviews. Full audit mode runs all six phases and writes artifacts.

Typical triggers: `security audit this codebase`, `find security vulnerabilities in ./src`,
`do a security review, output to runs/security-audit/<repo>/run-1`.

### Standard template for `AGENTS.md`

```markdown
## Security Audit Skill

Dieser Workspace nutzt den Shared Skill bei `../skills/security-audit/` für Sicherheitsaudits.
Ein Voll-Audit nur auf ausdrückliche Anfrage, mit Workspace-Lock und ohne Änderung am Ziel-Quellcode.
Siehe [SKILL.md](../skills/security-audit/SKILL.md) für Modi, Phasen und Grenzen.
```

## Requirements

- A coding agent with tool use and parallel sub-agents
- Node.js for `validate-findings.cjs` and `validate-coverage-ledger.cjs` (both zero-dependency). Both
  validators require POSIX `O_NOFOLLOW`/`O_NONBLOCK`, so run them under Linux, macOS or WSL2. On plain
  Windows they refuse to read; full audit mode then reports a blocker instead of validated output.
- An OS-enforced sandbox for any target-controlled build, test, process, browser or fuzzer. Without
  that sandbox the skill executes no target code and keeps execution-dependent leads at
  `needs_validation`.

## Workspace integration

Consuming workspaces link the skill under `.agents/skills/`:

```powershell
New-Item -ItemType Junction -Path .agents/skills/security-audit -Target ..\skills\security-audit
```

On *nix:

```bash
ln -s ../skills/security-audit .agents/skills/security-audit
```

## Provenance

This repository is a fork of
[cloudflare/security-audit-skill](https://github.com/cloudflare/security-audit-skill) (MIT), pinned at
commit `c1c8a8c1471069fb0e188eeaff69b8e8db6564a8` (2026-09-14, "Clarify guidance and full audit
modes"). The upstream project lives at path `skills/security-audit/` in its own repository; this fork
carries that directory as the repository root so a consumer can mount it as one submodule.

Local deviations from the pinned upstream:

1. **Layout**: skill files moved from `skills/security-audit/` to the repository root.
2. **`SKILL.md`**: the section `Local operation (Daedalus ecosystem)` covers output location,
   subagent mapping, the sandbox boundary, the Node/POSIX validator requirement, the boundary to
   diff-scoped code review and workspace locking. One sentence in the output-directory rule points at
   that section. All other text is upstream.
3. **`README.md`**: this file replaces the upstream README; it documents requirements, integration and
   the update recipe.

## Updating from upstream

1. Pick the upstream commit to pin and note it in this README.
2. Fetch `skills/security-audit/` from that commit and copy the files to this repository root.
3. Re-apply deviation 2 in `SKILL.md` and keep deviation 1 (flat layout).
4. Run `node --test` for both validator suites. Expect full passes on Linux, macOS and WSL2; on plain
   Windows the findings suite fails by design because `O_NOFOLLOW` is unavailable, and the
   coverage-ledger suite skips those cases.
5. Record the new pin and any behaviour change in the commit message.

## License

[MIT](LICENSE) — upstream copyright Cloudflare, Inc.
