# Getting Support

## Before asking for help

1. **Search existing issues** — [github.com/jayanti-prajapati/model-meter/issues](https://github.com/jayanti-prajapati/model-meter/issues)
2. **Read the docs** — especially [docs/FAQ.md](docs/FAQ.md) and [docs/GETTING-STARTED.md](docs/GETTING-STARTED.md)
3. **Check the API health** — `curl http://localhost:3000/health` should return `{"status":"ok","db":"ok"}`

## Where to get help

| Channel | Best for |
|---|---|
| [GitHub Issues](https://github.com/jayanti-prajapati/model-meter/issues/new/choose) | Bug reports and feature requests |
| [GitHub Discussions](https://github.com/jayanti-prajapati/model-meter/discussions) | Questions, ideas, show-and-tell |
| [CONTRIBUTING.md](CONTRIBUTING.md) | How to contribute a fix or feature |

## Reporting a security vulnerability

**Do not post security issues in public GitHub issues.**

Use GitHub's [private vulnerability reporting](https://github.com/jayanti-prajapati/model-meter/security/advisories/new) feature, or contact the maintainers privately through the channels configured in the repository's Security tab.

See [SECURITY.md](SECURITY.md) for scope priorities and the full disclosure policy.

## Filing a good bug report

When opening an issue, include:
- Model Meter version or git commit
- Deployment method (Docker Compose, Railway, local dev)
- Steps to reproduce
- Expected vs. actual behavior
- Relevant logs (redact secrets and customer data)

The [bug report template](.github/ISSUE_TEMPLATE/bug_report.md) will prompt you for this.
