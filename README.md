# build-tools

Reusable GitHub Actions workflows for Maven Central publishing across `markpollack` repositories.

## Workflows

| Workflow | Purpose | Trigger |
|----------|---------|---------|
| `ci-build.yml` | Build and test | Push/PR |
| `publish-snapshot.yml` | Deploy SNAPSHOT to Maven Central | Push to main |
| `maven-central-release.yml` | Release to Maven Central with GPG signing | Manual dispatch |

## Quick Start

Add these workflow files to your repository's `.github/workflows/`:

### CI Build (`ci.yml`)

```yaml
name: CI Build

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    uses: markpollack/build-tools/.github/workflows/ci-build.yml@main
    with:
      java-version: '17'
```

### Publish Snapshot (`publish-snapshot.yml`)

```yaml
name: Publish Snapshot

on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  publish:
    uses: markpollack/build-tools/.github/workflows/publish-snapshot.yml@main
    secrets:
      MAVEN_USERNAME: ${{ secrets.MAVEN_USERNAME }}
      MAVEN_PASSWORD: ${{ secrets.MAVEN_PASSWORD }}
```

### Release (`release.yml`)

```yaml
name: Release

on:
  workflow_dispatch:
    inputs:
      version:
        description: 'Release version (e.g., 0.1.0)'
        required: true
        type: string

jobs:
  release:
    uses: markpollack/build-tools/.github/workflows/maven-central-release.yml@main
    with:
      version: ${{ inputs.version }}
    secrets:
      MAVEN_USERNAME: ${{ secrets.MAVEN_USERNAME }}
      MAVEN_PASSWORD: ${{ secrets.MAVEN_PASSWORD }}
      GPG_SECRET_KEY: ${{ secrets.GPG_SECRET_KEY }}
      GPG_PASSPHRASE: ${{ secrets.GPG_PASSPHRASE }}
```

## Repository Secrets

Each consuming repository needs these secrets (Settings > Secrets > Actions):

| Secret | Source |
|--------|--------|
| `MAVEN_USERNAME` | Sonatype Central Portal user token (central.sonatype.com/account) |
| `MAVEN_PASSWORD` | Sonatype Central Portal user token password |
| `GPG_SECRET_KEY` | ASCII-armored GPG private key (`gpg --export-secret-keys --armor <KEY_ID>`) |
| `GPG_PASSPHRASE` | GPG key passphrase |

## POM Requirements

Consuming projects must have:

1. **`<distributionManagement>`** for snapshot repository
2. **`<scm>`** section
3. **`release` profile** with `central-publishing-maven-plugin`, `maven-gpg-plugin`, `maven-source-plugin`, `maven-javadoc-plugin`

See [agent-journal](https://github.com/markpollack/agent-journal) for a reference POM configuration.
