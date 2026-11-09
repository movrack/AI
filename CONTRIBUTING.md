# Contributing

This repository holds generic agents and skills. The owner (`@movrack`) is the only one who changes it directly.
Anyone else works through **issues** and **pull requests**.

## Rules
- **Ask first, in an issue.** To request a change, a new agent or a fix, open an issue with the matching template.
  Nothing is merged without an issue the owner agrees with.
- **Change through a pull request.** Fork the repository, work on a branch, open a pull request that references the
  issue (`Closes #N`). Direct pushes to `main` are blocked; every pull request needs the owner's approval
  (code owner review) and a green CI.
- **One change per pull request**, with Conventional Commits in English (`feat(agents): …`, `fix(skills): …`,
  `docs: …`), no secrets, no personal data, no project names inside an agent (agents stay generic).
- **Agents follow the shared ground rules** written in each file (on request only, no real environment, evidence
  over promises, one severity scale, reports in `reports/ai/<agent>/`). A pull request that changes the behaviour
  of an agent says what it changes and why.
- **Skills** keep their script tested: run `python3 -m unittest discover skills/install-agents/tests -v` before
  opening the pull request.

## Security
Report a vulnerability privately (see `SECURITY.md`), not in a public issue.

*Français : toute demande passe par une issue, toute modification par une pull request ; le propriétaire est le
seul à pousser directement sur `main`.*
