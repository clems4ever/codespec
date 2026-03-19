# CodeSpec

CodeSpec keeps your Go codebase's documentation honest. It detects functions whose documentation has fallen out of sync with the code and blocks commits until documentation is brought up to date — with AI agents doing the heavy lifting for you.

## How it works

1. CodeSpec tracks cryptographic hashes of each function's documentation and implementation.
2. When code or documentation changes, CodeSpec flags the function as **outdated**.
3. AI agents (Documenter and Tester) review the flagged functions, update the docs and tests, and confirm the changes with CodeSpec.
4. `@testcase` declarations are checked the same way — the Tester agent ensures every declared test case has a matching test implementation.

## Installation

Download the latest binary for your platform from the [Releases](../../releases) page and place it somewhere on your `PATH`:

```bash
# Example for Linux amd64
curl -L https://github.com/clems4ever/codespec/releases/latest/download/codespec_linux_amd64.tar.gz | tar xz
sudo mv codespec /usr/local/bin/
```

## Getting Started

### 1. Initialize your project

CodeSpec supports **VS Code** and **Claude** as AI backends.

```bash
# For VS Code + GitHub Copilot
codespec init vscode

# For Claude
codespec init claude
```

This sets up the MCP server configuration and the built-in **Documenter** and **Tester** agents for your AI backend.

### 2. Install the git pre-commit hook (optional)

To automatically block commits when documentation is outdated, install the pre-commit hook:

```bash
codespec install commit-hook
```

This copies the hook script to `scripts/commit_hook.sh` and registers it in `.git/hooks/pre-commit`.

### 2. Write documented functions

Use the structured docstring format that CodeSpec and its agents understand:

```go
// CreateUser creates a new user and returns a pointer to the record.
//
// @arg ctx Context for request-scoped values and cancellation.
// @arg req A UserRequest containing validated email and raw password.
// @return *User The newly persisted user object, including the assigned UUID.
// @error ErrDuplicateEmail if the email address is already taken.
//
// @testcase TestCreateUser tests successful user creation.
// @testcase TestCreateDuplicateUser expects ErrDuplicateEmail on a duplicate email.
func CreateUser(ctx context.Context, req UserRequest) (*User, error) { ... }
```

Supported tags: `@arg`, `@return`, `@error`, `@testcase`.

### 3. Let AI keep docs up to date

When you modify a function, the pre-commit hook will block the commit and tell you which functions need attention. Simply ask the **Documenter** agent to review the file — it will update the documentation and confirm the new state with CodeSpec so the next commit goes through.

The **Tester** agent works the same way for `@testcase` declarations: it ensures every declared test case has a corresponding test implementation.

## CLI Reference

| Command | Description |
|---|---|
| `codespec init vscode [path]` | Initialize a VS Code project |
| `codespec init claude [path]` | Initialize a Claude project |
| `codespec install commit-hook [path]` | Install the git pre-commit hook |
| `codespec check` | Check documentation for recently changed files |
| `codespec check --all` | Check documentation across the entire codebase |
| `codespec check --exit` | Exit with code 2 when issues are found (used in hooks) |
| `codespec confirm function --file=<file> --function=<name> [--receiver=<type>]` | Confirm a single function's documentation is up to date |
| `codespec confirm multiple --input='<json>'` | Confirm multiple functions' documentation from a JSON array |
| `codespec confirm all` | Confirm **all** currently outdated functions (see warning below) |
| `codespec statistics documentation` | Report documentation coverage statistics |
| `codespec statistics tests` | Report test declaration coverage (functions with at least one `@testcase`) |
| `codespec statistics outdated` | List functions with missing or outdated documentation |
| `codespec statistics outdated --all` | Include files with no codespec entry (all functions treated as outdated) |
| `codespec statistics untested` | List functions without any `@testcase` declaration |
| `codespec statistics undocumented` | List functions without a documentation comment |
| `codespec mcp` | Start the MCP server (used by AI agents) |

> **Warning:** `codespec confirm all` should only be used after a thorough manual review of all code and documentation. It marks every currently flagged function as up to date. Do not use it as a shortcut to bypass documentation review.

## Requirements

- Go project
- Git repository
- VS Code with GitHub Copilot **or** Claude (for AI-assisted documentation updates)