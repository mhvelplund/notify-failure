# notify-failure

A GitHub Action that sends a short, phone-readable [ntfy.sh](https://ntfy.sh) push notification
when a workflow run fails.

The notification contains:

- **Title:** `<title-prefix>: <name of the failed step>`
- **Body:** the first useful error annotation from the failed job (or its exit code), plus the run
  number, triggering event and short commit SHA
- **Tap action:** opens the failed run on GitHub (plus an "Open run" button)

It is sent with high priority and a 🚨 tag. If the topic is empty, the action logs a warning and
exits successfully, so a missing secret never breaks your workflow.

## Setup

1. Choose a long, hard-to-guess topic name. Topics on ntfy.sh are public, so anyone who knows the
   name can read and post to it.
2. Subscribe to that topic in the ntfy [Android/iOS app](https://docs.ntfy.sh/subscribe/phone/) or
   the [web app](https://docs.ntfy.sh/subscribe/web/).
3. Save the topic name as a repository **secret** named `NTFY_TOPIC`. Use a secret, not a variable,
   so GitHub masks the topic in the logs.

## Usage

Add a job that depends on the jobs you want to watch and runs only when they fail:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - run: ./build.sh

  notify-failure:
    needs: build
    if: failure()
    runs-on: ubuntu-latest
    timeout-minutes: 5
    permissions:
      actions: read # list this run's jobs and steps
      checks: read # read the failed job's error annotations
    steps:
      - uses: mhvelplund/notify-failure@main
        with:
          ntfy-topic: ${{ secrets.NTFY_TOPIC }}
          title-prefix: Build failed
```

## Inputs

| Input          | Required | Default               | Description                                                                 |
| -------------- | -------- | --------------------- | --------------------------------------------------------------------------- |
| `ntfy-topic`   | yes      |                       | The ntfy.sh topic to publish to. Pass it from a secret.                     |
| `title-prefix` | no       | `Workflow failed`     | Start of the notification title. The failed step's name follows the colon.  |
| `github-token` | no       | `${{ github.token }}` | Token used to look up the failed job, step and annotations. It needs `actions: read` and `checks: read`. |

## ntfy documentation

- [Publishing messages](https://docs.ntfy.sh/publish/): how messages are sent, including the
  [title](https://docs.ntfy.sh/publish/#message-title),
  [priority](https://docs.ntfy.sh/publish/#message-priority),
  [tags & emojis](https://docs.ntfy.sh/publish/#tags-emojis),
  [click action](https://docs.ntfy.sh/publish/#click-action) and
  [action buttons](https://docs.ntfy.sh/publish/#action-buttons) that this action uses
- [Subscribing from your phone](https://docs.ntfy.sh/subscribe/phone/)
- [FAQ](https://docs.ntfy.sh/faq/): topic privacy and other common questions

## License

[MIT](LICENSE)
