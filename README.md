# Maton CLI

![GitHub release (latest by date)](https://img.shields.io/github/v/release/maton-ai/cli)
![Build Status](https://github.com/maton-ai/cli/actions/workflows/deployment.yml/badge.svg)

## Installation

**MacOS/Linux:**
```bash
curl -fsSL https://maton.ai/install.sh | bash
```

**Windows:**
```powershell
irm https://maton.ai/install.ps1 | iex
```

**NPM:**
```bash
npm install -g @maton/cli
```

**Homebrew:**
```bash
brew install maton-ai/cli/maton
```

<details>
<summary>You can also go to the <a href="https://github.com/maton-ai/cli/releases/latest">latest GitHub Release</a> and download the appropriate binary for your platform.</summary>

Releases include archives named `maton_<version>_<os>_<arch>.{zip,tar.gz}` for
darwin, linux, and windows on amd64/arm64 (plus 386/armv6 on linux). Each archive
extracts to a directory containing `bin/maton`.

</details>

## Usage

Installing the CLI provides access to the `maton` command.

```sh-session
maton [command]

# Run `--help` for detailed information about CLI commands
maton [command] help
```

## Documentation

For a full reference, see the [CLI manual site](https://cli.maton.ai/manual/maton).

## Feedback

Got feedback for us? Please don't hesitate to email us at [support@maton.ai](mailto:support@maton.ai) or join our [Discord channel](https://discord.com/invite/dBfFAcefs2).
