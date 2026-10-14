# Security Policy

This policy covers every SecuryBlack repository that does not ship its own
`SECURITY.md`: the server agents (OxiPulse, FerroSentry, Nexus, TitanVault,
CromoForge, CupraFlow), `sb-agent-core`, the install scripts and the
SecuryBlack platform (`securyblack.com`, `app.securyblack.com` and its APIs).

## Reporting a vulnerability

**Please do not open public issues, discussions or pull requests for security
problems.** Report them privately through either channel:

- **GitHub private vulnerability reporting**: open the affected repository,
  go to *Security → Report a vulnerability*.
- **Email**: security@securyblack.com

Please include:

- what is affected (repository, component or URL) and the version;
- the impact you expect;
- steps to reproduce or a minimal proof of concept;
- a suggested fix, if you have one.

### What to expect

- **Acknowledgement** within 48 hours.
- **Assessment** (reproduced or not, and severity) within 5 business days.
- **Fix and disclosure**: we release a fix, publish a GitHub Security Advisory
  and credit you, unless you prefer to stay anonymous. We will agree the
  disclosure date with you.

### Good-faith research

We will not take action against research done in good faith that stays within
these limits:

- only test against your own servers and your own SecuryBlack account;
- do not access, change or delete other users' data;
- no denial of service, spam or social engineering;
- give us reasonable time to fix the issue before making it public.

## Supported versions

Only the latest release of each agent receives security fixes. Agents update
themselves once a day unless `auto_update = false`, so most installations are
already on it.

## How releases are protected

The agents run as root and update themselves, so a release is the most
sensitive thing we publish:

- Releases are built by GitHub Actions as **drafts**. The archives are signed
  **offline** with an ed25519 key ([zipsign](https://github.com/Kijewski/zipsign))
  that never lives on GitHub, and only then published.
- The self-updater (`sb-agent-core` 0.3 and later) **only installs archives
  signed by one of these keys** and rejects anything else:

  | Key | ed25519 public key (hex) |
  |---|---|
  | `sb-release-primary` | `63bdcfefaa2dd72c1f78b2b0d95605af6338a2be62f86d0986260835de9c2b6e` |
  | `sb-release-backup` | `3dc555627fb5c079517c7f21fe3858d148aceb3fd6a1b4fb1b02f04bd0cbe17c` |

  The source of truth is `VERIFYING_KEYS` in
  [`sb-agent-core/src/updater.rs`](https://github.com/SecuryBlack/sb-agent-core/blob/master/src/updater.rs).
- Every archive has a `.sha256` next to it. The install scripts refuse to
  install if it is missing or does not match.

To check an archive yourself:

```bash
echo 63bdcfefaa2dd72c1f78b2b0d95605af6338a2be62f86d0986260835de9c2b6e | xxd -r -p > sb-release.pub
zipsign verify tar oxipulse-x86_64-unknown-linux-gnu.tar.gz sb-release.pub \
  --context oxipulse-x86_64-unknown-linux-gnu.tar.gz
```

Known limitation: the signature covers the archive and its file name, not the
version number, so an older signed build could be published again under a
newer tag. Fixing this is on our roadmap.
