# agxmart-baseline

agxmart-baseline records which tools an MCP server exposes, as names and description hashes, and reports when that list changes, for example a tool description changing after you approved it.

It has no runtime dependencies and needs Node 20 or newer.

## Install

```sh
npx github:EshaRana17/agxmart-baseline --help
```

## Usage

```sh
agxmart-baseline fetch <type>/<slug> [--out file] [--force]
agxmart-baseline probe [--out file] [--force] [--timeout ms] [--name n] [--repo r] [--commit c] -- <command...>
agxmart-baseline diff <old> <new> [--json]
agxmart-baseline to-policy <baseline> [--out file] [--force]
```

- `fetch` downloads a baseline from the AGXMart API. Set `AGXMART_API` to change the base URL.
- `probe` starts the local stdio server you name, lists its tools, writes a baseline with source `local-probe`, and stops the server. It tries the `initialize` handshake first, then falls back to stateless requests carrying `_meta` (MCP revision 2026-07-28) if the server rejects it.
- `diff` reports added, removed and changed tools. Exit codes: 0 no differences, 1 differences, 2 error.
- `to-policy` writes a policy for [agxmart-guard](https://github.com/EshaRana17/agxmart-guard) with an allow list, expected hashes, `onToolListChange` set to `block`, and starting limits of 60 calls a minute, 5 identical calls and 1000 total. Adjust them for your workload.

Output files are not replaced unless you pass `--force`, so an approved baseline is not overwritten by accident.

Real output from a test run with a small fake server, where the second probe changes one description:

```text
$ agxmart-baseline probe --out approved.json -- node fake-server.cjs
wrote baseline with 3 tools to approved.json

$ agxmart-baseline probe --out today.json -- node fake-server.cjs
wrote baseline with 3 tools to today.json

$ agxmart-baseline diff approved.json today.json
changed tools (description hash differs):
  write_file: 591ce17f0daf -> d387787718d7
(exit code 1)

$ agxmart-baseline to-policy approved.json --out policy.json
wrote guard policy to policy.json
```

## Limits

- A baseline contains only what was observed at that moment. It is not a judgment that a server is well behaved.
- Tools are not executed during scans or probes, so behavior that appears only when a tool runs is not captured.
- `probe` runs the command you give it on your machine, with your permissions. Only probe servers you are willing to start.
- `probe` does not observe network hosts, so its `hosts` list is always empty.
- A server can return a different list at another time or to another client. Hashes cover the description text only, not input schemas or tool behavior.
- Baselines from the API depend on AGXMart hashing descriptions the same way, as SHA-256 of the UTF-8 description text.

## Tests

```sh
node --test
```

18 tests, all passing on Node 24.14.0 on Windows 10. The test server is in `test/fixtures`.

## License

MIT

Developed by [buildwithesha.web.app](https://buildwithesha.web.app/) for [AGXMart](https://agxmart.com)
