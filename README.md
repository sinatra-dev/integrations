# sinatra.dev for Claude

Hand coding tasks to [sinatra.dev](https://www.sinatra.dev) without leaving Claude. Sinatra is a hosted coding agent: each task runs in its own sandbox on Sinatra's side, works on a GitHub repository connected to your Sinatra workspace, and opens a draft pull request for your team to review and merge. Sinatra never merges or deploys.

This plugin adds two things to Claude Code (and to Claude apps that load plugins):

- **Sinatra's remote MCP server** at `https://api.sinatra.dev/v1/mcp`, so Claude can start, follow, reply to and stop tasks.
- **A skill** (`sinatra-tasks`) that teaches Claude how to brief a task well, how to wait for answers without wasting calls, and when to ask you before acting.

## What you need

- A sinatra.dev account and a workspace with the Sinatra GitHub App installed on at least one repository. Sign up at [app.sinatra.dev](https://app.sinatra.dev).
- Model access connected to that workspace, such as an API key or a subscription you already have. See [Models](https://docs.sinatra.dev/docs/configuration/models).

## Install

From this repository's marketplace:

```sh
claude plugin marketplace add sinatra-dev/integrations
claude plugin install sinatra-dev@sinatra-dev
```

Then run `/mcp` in Claude Code, pick `sinatra`, and sign in.

## Sign in

When you sign in (from `/mcp` in Claude Code, or **Connect** in the Claude app), Claude opens Sinatra's sign-in page in your browser over OAuth. Sign in to Sinatra if you aren't already, check that the page names the app you're connecting, and click **Approve**. There's no token to create or paste.

The connection acts as you, in every Sinatra workspace you belong to, with your role in each. It's listed under **Connected apps** on the [CLI & API](https://app.sinatra.dev/tokens) page, where **Disconnect** ends its access immediately. Removing the plugin alone doesn't revoke it.

## Tools

| Tool | What it does |
| --- | --- |
| `whoami` | Your Sinatra user and the workspaces you belong to. |
| `list_tasks` | A workspace's tasks, most recently active first, with status and pull request. |
| `get_task` | One task's status, title, repository and pull request. |
| `start_task` | Starts a task from a prompt, on a repository you name or one the agent picks. It may open a pull request. |
| `get_messages` | A task's conversation: your messages, the agent's answers and questions, and Sinatra's notices. |
| `follow_up` | Replies on a task you started from Claude, the CLI or the API. |
| `stop_task` | Stops a task's current turn. |

`whoami`, `list_tasks`, `get_task` and `get_messages` only read. `start_task` and `follow_up` start work that uses your workspace's plan and your model access, and can push a branch and open a pull request. `stop_task` stops work in progress, so Claude asks before calling it.

## Try it

- "Which of my Sinatra tasks have an open pull request?"
- "Ask Sinatra to fix the failing date-parsing test in acme/web, and tell me when there's a PR."
- "What did Sinatra ask on my last task? Tell it to keep the three-attempt limit."

## What this plugin sends and runs

The plugin contains no scripts, hooks or local servers. It only points Claude at `https://api.sinatra.dev/v1/mcp`. When Claude calls a tool, the tool's arguments (for example your prompt or reply) go to Sinatra and are stored as that task's conversation, which members of the workspace can read in the dashboard. Don't put secrets in prompts.

## Links

- Docs: [CLI, MCP and API](https://docs.sinatra.dev/docs/cli-and-api#mcp-server)
- Privacy policy: [sinatra.dev/privacy](https://www.sinatra.dev/privacy)
- Terms: [sinatra.dev/terms](https://www.sinatra.dev/terms)
- Security reports: [sinatra.dev/security](https://www.sinatra.dev/security)
- Support: [contact@sinatra.dev](mailto:contact@sinatra.dev)

## License

MIT. See [LICENSE](LICENSE). The license covers the files in this repository. The Sinatra service itself is covered by the [terms](https://www.sinatra.dev/terms).
