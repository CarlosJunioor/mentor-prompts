# Security & Privacy Policy

This repo is documentation and Claude Code skills — there is no running service to attack.
The realistic risk here is **a privacy leak**: this toolkit was genericized from a real
private app, and the one rule that keeps it safe is that nothing app-specific ever lands
in a tracked file. This policy covers how to report a leak or any other security concern.

## Report privately — do NOT open a public issue

If you find any of the following, **report it privately**, never in a public issue or PR
(a public report amplifies the exact exposure):

- real app data committed by mistake — entity / screen / action / module names, owners,
  teammate or person names, environment URLs, realm / tenant / environment UUIDs;
- a secret or credential of any kind in the repo or its git history;
- anything non-English or app-identifying in a tracked file;
- a skill instruction that could destroy a user's data, exfiltrate it, or run unintended
  commands.

### How to report

1. **Preferred — GitHub private vulnerability reporting.** Go to the repo's **Security**
   tab → **Report a vulnerability**. (Maintainer: enable this once under
   *Settings → Security → Private vulnerability reporting*.) It opens a private channel
   only you and the maintainer can see.
2. If that is unavailable, contact the maintainer privately through their GitHub profile
   rather than filing a public issue.

Please include where you saw it (file + commit if known) and, for a data leak, **do not
quote the private data** — just point to its location so it can be removed.

## What happens next

- Confirmed leaks are scrubbed from the working tree **and** from git history (history
  rewrite + force-push), so the data does not survive in old commits.
- You'll get an acknowledgement and a note when it's resolved.

## For contributors: prevent leaks before they happen

Before every commit, run the **"Before you commit" privacy checklist** in
[CONTRIBUTING.md](CONTRIBUTING.md). The single hard rule is: tracked files are
**app-agnostic and English-only**; your real app's facts live only in the git-ignored
`knowledge/apps/<app>/` folder.
