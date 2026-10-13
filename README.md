# morning-recap

A daily AI morning briefing that lands on your phone before you start work — yesterday's progress, today's calendar, one clear priority, emails that matter, local news. One short message, zero fluff.

Runs as a scheduled [Hermes Agent](https://github.com/NousResearch/hermes-agent) cron job. No code — the whole thing is a well-built prompt delivered to Telegram (or any Hermes channel).

```
Yesterday recap (12/10)
- Shipped landing v2, fixed billing webhook, signed coaching prospect
- 10:00 call with Marc — prepare the audit deck
- Priority: finish the Stripe integration, it's the only thing blocking revenue
- Emails: invoice from Scaleway (pay by 15/10), Anna waiting for your quote since Monday
- Lisbon: storm warning this afternoon, move the coworking session
→ Don't end the day without the webhook deployed.
```

## Why

You wake up, and instead of opening 5 apps to reconstruct where you left off, one message tells you: what you did, what's scheduled, what to attack first, and who is waiting on you. The "one clear priority" rule is the point — a briefing that lists 5 options is just procrastination delivered to your phone.

## Setup

### 1. Fill in the prompt

Copy [`PROMPT.md`](PROMPT.md) and replace the placeholders:

| Placeholder | Example |
|---|---|
| `{{USER_NAME}}` / `{{USER_CONTEXT}}` | `Alex — solo founder, based in Lisbon, UTC+1` |
| `{{LANGUAGE}}` | `English` |
| `{{OPENING_LINE}}` | `Yesterday recap (DD/MM)` |
| `{{CALENDAR}}` | `my Google Calendar (primary)` |
| `{{PRIORITY_PROJECTS}}` | `my SaaS (billing v2), the newsletter, 2 coaching clients` |
| `{{PAUSED_PROJECTS}}` | `the crypto bot and the mobile game` |
| `{{CITY}}` | `Lisbon` |

The more specific `{{PRIORITY_PROJECTS}}` is — with names, deadlines, what "done" means — the sharper the daily priority gets. Update it when your focus shifts.

### 2. Create the cron job

```bash
hermes cron add \
  --name "Morning recap" \
  --cron "0 9 * * *" \
  --prompt "$(cat PROMPT.md)" \
  --deliver telegram
```

Adjust the cron expression to your wake-up time. Check `hermes cron add --help` for your version's exact flags (delivery targets, model pinning).

### 3. Two settings that make or break it

- **Pin a cheap model on the job.** An unpinned job silently SKIPS when your global default model changes ("skipped to prevent unintended spend") — and an unpinned job can also inherit an expensive model. Pin something fast and cheap; this prompt doesn't need a flagship:
  ```bash
  hermes cron edit <job_id> --model <your-cheap-model> --provider <your-provider>
  ```
- **Turn off the response wrapper** so the message starts with your recap instead of a `Cronjob Response:` header:
  ```bash
  hermes config set cron.wrap_response false
  ```

## How it works

- The job runs in a **fresh session** every morning — no memory of yesterday's chat. That's why the prompt is fully self-contained, and why it must *search* your recent sessions/emails instead of assuming context.
- It runs **unattended**: no clarification questions possible. The prompt explicitly tells the agent to make do and note failures in one line.
- The opening-line rule (`Yesterday recap (DD/MM)`, no greeting) is enforced in the prompt because with the wrapper off, the model's output IS the message.

## Troubleshooting

- **Nothing arrived**: read the job's last transcript in `~/.hermes/profiles/<profile>/cron/output/<job_id>/` — it states the exact error (provider stall, delivery failure, skipped-for-spend).
- **It stopped after you changed models**: that's the unpinned-job skip. Re-pin the model (see above).
- **Computer was asleep**: on a laptop, cron jobs only fire while the machine is awake. A missed morning is silent — check the output dir before assuming it ran.
- **Testing**: `hermes cron run <job_id>` sends a REAL message to your chat. Fine once, annoying twice.

## License

MIT
