# github-workflows

Reusable GitHub action workflows for .NET projects.

## Available Workflows

- [dotnet-format](#dotnet-format) - Check or auto-fix code formatting using `dotnet format`
- [dotnet-test-coverage](#dotnet-test-coverage) - Run tests with code coverage reporting
- [dotnet-coverage](#dotnet-coverage) - .NET code coverage workflow

> **Using Private Repositories?** See the [Using with Private Repositories](#using-with-private-repositories) section for instructions on passing tokens.

---

## Using with Private Repositories

When using these workflows with private repositories, you need to provide a Personal Access Token (PAT) that has permissions to checkout the repository. The default `GITHUB_TOKEN` doesn't have sufficient permissions to access private repositories in workflow calls.

### Creating a Personal Access Token

1. Go to **GitHub Settings** → **Developer settings** → **Personal access tokens** → **Tokens (classic)** or **Fine-grained tokens**
2. Click **Generate new token**

**For Classic tokens:**
- Select the `repo` scope (Full control of private repositories)

**For Fine-grained tokens:**
- Select the target repository
- Under **Repository permissions**, set **Contents** to **Read-only** (or **Read and write** if using `fix` mode in dotnet-format)

### Storing the Token as a Repository Secret

1. Navigate to your repository's **Settings** → **Secrets and variables** → **Actions**
2. Click **New repository secret**
3. Name it (e.g., `PAT_TOKEN`) and paste your token value
4. Click **Add secret**

### Passing the Token to Workflows

Pass the token using the `secrets` parameter with `CHECKOUT_TOKEN`:

```yaml
jobs:
  example:
    uses: ardalis/github-workflows/.github/workflows/dotnet-format.yml@main
    with:
      solution-or-project: './MyProject.sln'
    secrets:
      CHECKOUT_TOKEN: ${{ secrets.PAT_TOKEN }}
```

> **Note**: The `CHECKOUT_TOKEN` secret name is used by the workflow, but your repository secret can be named anything (e.g., `PAT_TOKEN`, `PRIVATE_REPO_TOKEN`).

---

## dotnet-format

Runs `dotnet format` on your solution or project to enforce consistent code style.

### Features

- **Check mode**: Fails if formatting changes are needed (useful for CI)
- **Fix mode**: Automatically formats code and commits changes to PR branches
- Optionally format only files changed in a pull request
- Configurable git user for auto-commit

### Usage

```yaml
name: Format Check

on:
  pull_request:
    branches: [main]

jobs:
  format:
    uses: ardalis/github-workflows/.github/workflows/dotnet-format.yml@main
    with:
      solution-or-project: './src/MyProject.sln'
      mode: 'check'
```

### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `solution-or-project` | Path to .sln or .csproj to format (relative to repo root) | Yes | |
| `dotnet-version` | .NET SDK version to use | No | `8.x` |
| `mode` | `check` to fail if changes needed; `fix` to auto-commit & push | No | `check` |
| `only-changed-files` | If true, format only files changed in the PR (when possible) | No | `true` |
| `git-user-name` | Git user.name used in 'fix' mode | No | `dotnet-format-bot` |
| `git-user-email` | Git user.email used in 'fix' mode | No | `dotnet-format-bot@users.noreply.github.com` |
| `commit-message` | Commit message used in 'fix' mode | No | `chore: apply dotnet format` |

### Secrets

| Name | Description | Required |
|------|-------------|----------|
| `CHECKOUT_TOKEN` | Token for private repository checkout | No |
| `github-token` | Optional PAT (maintained for backward compatibility; prefer CHECKOUT_TOKEN) | No |

### Examples

#### Check formatting on pull requests

```yaml
jobs:
  format-check:
    uses: ardalis/github-workflows/.github/workflows/dotnet-format.yml@main
    with:
      solution-or-project: './MyProject.sln'
      mode: 'check'
```

#### Auto-fix formatting and commit changes

```yaml
jobs:
  format-fix:
    uses: ardalis/github-workflows/.github/workflows/dotnet-format.yml@main
    with:
      solution-or-project: './MyProject.sln'
      mode: 'fix'
      commit-message: 'style: auto-format code'
```

#### Using with private repositories

```yaml
jobs:
  format-check:
    uses: ardalis/github-workflows/.github/workflows/dotnet-format.yml@main
    with:
      solution-or-project: './MyProject.sln'
      mode: 'check'
    secrets:
      CHECKOUT_TOKEN: ${{ secrets.PAT_TOKEN }}
```

---

## dotnet-test-coverage

Runs .NET tests with code coverage collection and generates a coverage summary report.

### Features

- Runs tests with XPlat Code Coverage
- Generates markdown coverage summary
- Uploads coverage results as artifacts
- Outputs coverage percentage for use in other jobs
- Saves PR number for downstream workflows

### Usage

```yaml
name: Test with Coverage

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  test:
    uses: ardalis/github-workflows/.github/workflows/dotnet-test-coverage.yml@main
    with:
      solution-path: './src'
      configuration: 'Release'
```

### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `dotnet-version` | .NET SDK version to use | No | `8.x` |
| `solution-path` | Path to solution file, project file, or directory containing them | No | `.` |
| `configuration` | Build configuration | No | `Release` |

### Secrets

| Name | Description | Required |
|------|-------------|----------|
| `CHECKOUT_TOKEN` | Token for private repository checkout | No |

### Outputs

| Name | Description |
|------|-------------|
| `coverage-percentage` | Overall code coverage percentage |

### Artifacts

This workflow uploads the following artifacts:

- `code-coverage-results` - Markdown file with coverage summary
- `pr-number` - Text file containing the PR number (for pull request events)

### Example with coverage output

```yaml
jobs:
  test:
    uses: ardalis/github-workflows/.github/workflows/dotnet-test-coverage.yml@main
    with:
      solution-path: '.'
  
  report:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - name: Coverage was
        run: echo "Coverage: ${{ needs.test.outputs.coverage-percentage }}%"
```

### Using with private repositories

```yaml
jobs:
  test:
    uses: ardalis/github-workflows/.github/workflows/dotnet-test-coverage.yml@main
    with:
      solution-path: '.'
    secrets:
      CHECKOUT_TOKEN: ${{ secrets.PAT_TOKEN }}
```

---

## dotnet-coverage

Basic .NET code coverage workflow configuration.

### Usage

```yaml
name: Code Coverage

on:
  push:
    branches: [main]

jobs:
  coverage:
    uses: ardalis/github-workflows/.github/workflows/dotnet-coverage.yml@main
    with:
      dotnet-version: '8.x'
      build-configuration: 'Release'
```

### Inputs

| Name | Description | Required | Default |
|------|-------------|----------|---------|
| `dotnet-version` | .NET SDK version to use | No | `8.x` |
| `build-configuration` | Build configuration | No | `Release` |

### Secrets

| Name | Description | Required |
|------|-------------|----------|
| `CHECKOUT_TOKEN` | Token for private repository checkout | No |

### Using with private repositories

```yaml
jobs:
  coverage:
    uses: ardalis/github-workflows/.github/workflows/dotnet-coverage.yml@main
    with:
      dotnet-version: '8.x'
      build-configuration: 'Release'
    secrets:
      CHECKOUT_TOKEN: ${{ secrets.PAT_TOKEN }}
```

---

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the terms of the license included in the repository.
