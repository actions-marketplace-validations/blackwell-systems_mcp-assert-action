# mcp-assert GitHub Action

Deterministic correctness testing for MCP servers in CI. One line to add assertion testing to any MCP server repo.

## Usage

```yaml
- uses: blackwell-systems/mcp-assert-action@v1
  with:
    suite: evals/
    server: "npx @modelcontextprotocol/server-filesystem ."
    fixture: test/fixtures/
```

## Inputs

| Input | Required | Default | Description |
|-------|----------|---------|-------------|
| `suite` | Yes | — | Directory containing assertion YAML files |
| `server` | No | — | Override server command for all assertions |
| `fixture` | No | — | Fixture directory (`{{fixture}}` substitution) |
| `threshold` | No | `100` | Minimum pass percentage |
| `timeout` | No | `30s` | Per-assertion timeout |
| `version` | No | `latest` | mcp-assert version to install |
| `baseline` | No | — | Baseline JSON for regression detection |
| `fail-on-regression` | No | `false` | Fail if previously-passing assertions regress |

## Outputs

| Output | Description |
|--------|-------------|
| `passed` | Number of assertions that passed |
| `failed` | Number of assertions that failed |
| `total` | Total number of assertions |

## Examples

### Basic — test a filesystem server

```yaml
name: Test MCP Server
on: [push, pull_request]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Install server
        run: npm install -g @modelcontextprotocol/server-filesystem

      - uses: blackwell-systems/mcp-assert-action@v1
        with:
          suite: evals/
          fixture: test/fixtures/
```

### With threshold and server override

```yaml
- uses: blackwell-systems/mcp-assert-action@v1
  with:
    suite: evals/
    server: "python -m my_mcp_server"
    threshold: 95
    timeout: 60s
```

### Regression detection

```yaml
- uses: blackwell-systems/mcp-assert-action@v1
  with:
    suite: evals/
    baseline: baseline.json
    fail-on-regression: true
```

## What it does

1. Downloads the `mcp-assert` binary from [GitHub Releases](https://github.com/blackwell-systems/mcp-assert/releases)
2. Runs `mcp-assert ci` with your configuration
3. Uploads JUnit XML results as an artifact
4. Uploads shields.io badge JSON as an artifact
5. Writes GitHub Step Summary automatically
6. Fails the step if pass rate is below threshold

## Writing assertions

Each assertion is a YAML file that calls one MCP tool and checks the result:

```yaml
# evals/read_file.yaml
name: read_file returns file contents
server:
  command: npx
  args: ["@modelcontextprotocol/server-filesystem", "{{fixture}}"]
assert:
  tool: read_file
  args:
    path: "{{fixture}}/hello.txt"
  expect:
    not_error: true
    contains: ["Hello, world!"]
timeout: 15s
```

- `server` — how to start your MCP server (binary + args)
- `assert.tool` — which MCP tool to call
- `assert.args` — arguments passed to the tool
- `assert.expect` — deterministic checks on the response (`contains`, `not_error`, `equals`, `is_error`, `json_path`, `min_results`, etc.)
- `{{fixture}}` — replaced with the `fixture` input path at runtime
- `setup` — optional list of tool calls to run before the assertion (for stateful tests)

See [mcp-assert documentation](https://github.com/blackwell-systems/mcp-assert) for the full assertion format reference and example suites.

## License

MIT
