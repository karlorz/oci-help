# Repository Guidelines

## Project Structure & Module Organization
This repository is a small Go CLI for managing Oracle Cloud instances. The main application logic lives in `main.go`, so keep related helpers close to existing functions unless a new package is clearly justified. Runtime configuration is read from `oci-help.ini`. User-facing documentation lives in `README.md`, and screenshots referenced there are stored in `doc/`. Release automation is defined in `.github/workflows/release.yml`. Build artifacts belong in `build/`, which is ignored.

## Build, Test, and Development Commands
- `go run . -c ./oci-help.ini`: run the CLI locally with an explicit config file.
- `go test ./...`: run the current verification baseline. There are no committed test files today, so this is mainly a compile/smoke check.
- `make build`: build a local binary in `build/oci-help` with release ldflags.
- `make all`: cross-compile the most common release targets.
- `make release`: package release zips for the platforms listed in `Makefile`.
- `gofmt -w main.go`: apply canonical Go formatting before submitting changes.

## Coding Style & Naming Conventions
Follow standard Go style and let `gofmt` control whitespace; Go uses tabs for indentation. Keep exported identifiers in `CamelCase` and unexported locals in `camelCase`. Reuse the existing INI-backed struct tags and configuration field names unless you are making a deliberate config migration. Prefer small, focused functions over adding more branching to already large menu handlers.

## Testing Guidelines
Add tests as `*_test.go` files beside the code they cover. Favor table-driven tests for parsing, menu selection, and OCI request-building logic. Before opening a PR, run `go test ./...` and, for behavioral changes, a manual CLI smoke test such as `go run . -c ./oci-help.ini` against a non-production account or sanitized config.

## Commit & Pull Request Guidelines
Recent history uses short, one-line commit subjects, often in Chinese, for example `新增实例管理功能` or `修复未知错误导致的终止`. Keep the subject concise and action-oriented. PRs should include the purpose of the change, any config or OCI behavior impact, linked issues when applicable, and screenshots when updating `README.md` or `doc/`.

## Security & Configuration Tips
Do not commit real OCI credentials, Telegram tokens, private keys, or `.pem` files. Treat `oci-help.ini` as local-only configuration and sanitize any example values before sharing logs or screenshots.

## VPS Production Usage
The `ca01` host now runs `oci-help` as two separate `systemd` services so `SG01` and `CA01` retry independently. The legacy `oci-help` service is deprecated and should remain disabled. Use these commands for routine operations:

- Check status: `ssh ca01 'systemctl status oci-help-sg01 oci-help-ca01 --no-pager -l'`
- Start both services: `ssh ca01 'systemctl start oci-help-sg01 oci-help-ca01'`
- Restart both services: `ssh ca01 'systemctl restart oci-help-sg01 oci-help-ca01'`
- Stop both services: `ssh ca01 'systemctl stop oci-help-sg01 oci-help-ca01'`

To see logs:

- Follow both stdout logs: `ssh ca01 'tail -f /var/log/oci-help-sg01.log /var/log/oci-help-ca01.log'`
- Follow both stderr logs: `ssh ca01 'tail -f /var/log/oci-help-sg01.err.log /var/log/oci-help-ca01.err.log'`
- Show recent stdout: `ssh ca01 'tail -n 100 /var/log/oci-help-sg01.log /var/log/oci-help-ca01.log'`
- Show recent journal entries: `ssh ca01 'journalctl -u oci-help-sg01 -u oci-help-ca01 -n 100 --no-pager'`

To inspect what is configured:

- Show service files: `ssh ca01 'systemctl cat oci-help-sg01 oci-help-ca01'`
- Show batch wrappers: `ssh ca01 'sed -n "1,120p" /root/bin/oci-help-batch-sg01 && printf "\n=====\n" && sed -n "1,120p" /root/bin/oci-help-batch-ca01'`
- Show production configs: `ssh ca01 'sed -n "1,220p" /root/.config/oci-help/oci-help-sg01.ini && printf "\n=====\n" && sed -n "1,220p" /root/.config/oci-help/oci-help-ca01.ini'`
