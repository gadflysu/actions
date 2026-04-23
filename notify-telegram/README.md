# notify-telegram

Send CI status notifications to Telegram. Supports success and failure states, and automatically extracts error lines from failed job logs.

## Inputs

| Name | Required | Default | Description |
|------|----------|---------|-------------|
| `token` | Yes | — | Telegram bot token |
| `chat_id` | Yes | — | Telegram chat ID to send the message to |
| `status` | Yes | — | CI status: `success` or `failure` |
| `gh_token` | No | `${{ github.token }}` | GitHub token for fetching failed job logs |

## Usage

Add the following to your workflow. Call this action in a dedicated job that runs after all other jobs complete.

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build
        run: make build

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

## Advanced Usage

### Multiple jobs

When your workflow has multiple jobs, list all of them in `needs` so the notify job waits for everyone:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: make build

  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - run: make test

  notify:
    runs-on: ubuntu-latest
    needs: [build, test]
    if: always()
    steps:
      - uses: gadflysu/actions/notify-telegram@main
        with:
          token: ${{ secrets.TELEGRAM_BOT_TOKEN }}
          chat_id: ${{ secrets.TELEGRAM_CHAT_ID }}
          status: ${{ contains(needs.*.result, 'failure') && 'failure' || 'success' }}
```

### Notify only on failure

Skip the notification on success to reduce noise:

```yaml
  notify:
    runs-on: ubuntu-latest
    needs: [build, test]
    if: ${{ failure() }}
    steps:
      - uses: gadflysu/actions/notify-telegram@main
        with:
          token: ${{ secrets.TELEGRAM_BOT_TOKEN }}
          chat_id: ${{ secrets.TELEGRAM_CHAT_ID }}
          status: failure
```

### Notify only on specific branches

Limit notifications to pushes on `main` or `master`:

```yaml
  notify:
    runs-on: ubuntu-latest
    needs: [build]
    if: always() && github.ref == 'refs/heads/main'
    steps:
      - uses: gadflysu/actions/notify-telegram@main
        with:
          token: ${{ secrets.TELEGRAM_BOT_TOKEN }}
          chat_id: ${{ secrets.TELEGRAM_CHAT_ID }}
          status: ${{ contains(needs.*.result, 'failure') && 'failure' || 'success' }}
```

### Matrix builds

For matrix builds, use `needs.*.result` as usual — the notify job will reflect the combined result of all matrix legs:

```yaml
jobs:
  test:
    strategy:
      matrix:
        go: ['1.22', '1.23']
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-go@v5
        with:
          go-version: ${{ matrix.go }}
      - run: go test ./...

  notify:
    runs-on: ubuntu-latest
    needs: [test]
    if: always()
    steps:
      - uses: gadflysu/actions/notify-telegram@main
        with:
          token: ${{ secrets.TELEGRAM_BOT_TOKEN }}
          chat_id: ${{ secrets.TELEGRAM_CHAT_ID }}
          status: ${{ contains(needs.*.result, 'failure') && 'failure' || 'success' }}
```

## Message Preview

**On success:**
```
✅ CI passed
gadflysu/my-repo @ main
fix: correct typo in config #42

runs/12345678
```

**On failure:**
```
❌ CI failed
gadflysu/my-repo @ main
feat: add new feature #43

<expandable error log with up to 15 matching lines>

runs/12345679
```

The error log block is collapsible in Telegram and contains lines matching `error`, `warning`, `fatal`, `failed`, or `exception` from the failed job output.

## Setup

1. **Create a Telegram bot** — Talk to [@BotFather](https://t.me/BotFather), send `/newbot`, and copy the bot token.

2. **Get your chat ID** — Add the bot to a group or send it a direct message, then visit:
   ```
   https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates
   ```
   Find `"chat":{"id":...}` in the response.

3. **Add GitHub Secrets** — In your repo go to **Settings → Secrets and variables → Actions** and add:
   - `TELEGRAM_BOT_TOKEN` — the token from step 1
   - `TELEGRAM_CHAT_ID` — the chat ID from step 2
