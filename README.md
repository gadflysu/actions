# gadflysu/actions

[![CI](https://github.com/gadflysu/actions/actions/workflows/ci.yml/badge.svg)](https://github.com/gadflysu/actions/actions/workflows/ci.yml)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Handy GitHub Actions for your CI/CD workflows.

## Actions

| Action | Description |
|--------|-------------|
| [notify-telegram](notify-telegram/) | Send CI status notifications to Telegram |

## Usage

### notify-telegram

Send CI pass/fail notifications to Telegram. Add a `notify` job that runs after all other jobs:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: make build

  notify:
    runs-on: ubuntu-latest
    needs: [build]
    if: always()
    steps:
      - uses: gadflysu/actions/notify-telegram@main
        with:
          token: ${{ secrets.TELEGRAM_BOT_TOKEN }}
          chat_id: ${{ secrets.TELEGRAM_CHAT_ID }}
          status: ${{ contains(needs.*.result, 'failure') && 'failure' || 'success' }}
```

See [notify-telegram/README.md](notify-telegram/README.md) for full documentation.
