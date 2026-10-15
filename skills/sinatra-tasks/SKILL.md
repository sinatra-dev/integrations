---
name: sinatra-tasks
description: How to hand a coding task to sinatra.dev and follow it with the sinatra MCP tools (whoami, start_task, get_messages, follow_up, get_task, list_tasks, stop_task). Use when the user asks Sinatra to do something in a repository, asks what a Sinatra task said or whether it opened a pull request, wants to reply to a Sinatra task, or wants to stop one.
---

# Working with sinatra.dev tasks

Sinatra is a hosted coding agent. A task runs in Sinatra's own sandbox, not on this machine, and its code changes come back as a draft pull request on GitHub. Nothing you do here edits local files. A person reviews and merges the pull request; Sinatra never merges or deploys.

Only call these tools when the user asked for Sinatra. Don't start a task on your own initiative.

## 1. Find the workspace

Call `whoami` first. Every other tool needs a `workspace_id` from it.

- One workspace: use it.
- Several: ask the user which one, by name. Don't guess.

## 2. Start a task

Call `start_task` with:

- `prompt`: write it the way you'd brief a teammate. Include the goal, the files or area you already know about, how to check the result (a test command, an expected output), and anything that's out of scope. A specific prompt goes straight to a pull request; a vague one gets a question back.
- `repo`: `owner/name` of a repository connected to the workspace. If you're inside a git checkout, read the GitHub `origin` remote and pass it. Omit `repo` only when the user doesn't know; the agent then picks one. If the call returns "That repository isn't connected", tell the user to connect it in the Sinatra dashboard or name another repository.
- `title`: optional, a short line for the dashboard.
- `idempotency_key`: always pass a fresh unique value, such as a UUID. If the call fails or times out, retry with the same key, so the retry can't start a second task.

Don't put secrets (tokens, passwords, keys) in a prompt or reply. The conversation is stored with the task, and every member of the workspace can read it.

## 3. Follow the task

There is no streaming. The task's conversation holds one answer per turn, not progress updates.

- `get_messages` returns the conversation oldest first. Keep the `id` of the last message you read and pass it as `after` next time, so you only read what's new.
- `get_task` returns the status, the repository and the pull request URL once one opens. `queued: true` means the workspace's plan is running as many tasks as it allows; the task starts on its own.
- A turn is finished when an `agent` message arrives (`kind: response` for an answer, `kind: clarify` for a question), or a `platform` message reports an error or a stop.

A turn usually takes several minutes. Don't poll in a tight loop: every MCP call shares one budget of 120 requests a minute. Unless the user asked you to wait, tell them the task started, give them the task id, and offer to check back. If they want you to wait, check every 30 seconds or so and stop after about 15 minutes with a note on how to check later.

When the agent asks a question (`clarify`), show it to the user word for word and ask how to answer. Don't answer on the user's behalf unless they already told you the answer in this conversation.

When a pull request opens, give the user its URL. Say it's a draft for them to review, not that the change is merged or done.

## 4. Reply

`follow_up` sends a message on a task, for example the answer to a question or a change of plan. Pass `idempotency_key` here too. Two things to know:

- It works only on tasks started from the CLI, MCP or API. A task started from Linear, GitHub or Slack returns "That task wasn't started with start_task"; tell the user to reply on that ticket, issue or thread instead.
- A message sent while the agent is working doesn't change the current turn. It runs as the next turn.

## 5. Stop

`stop_task` stops the task's current turn. Ask the user to confirm first, and say which task (title and id) you're about to stop. The sandbox keeps its files, and the task can take a reply later.

## Errors worth explaining

- "You already have 5 CLI, MCP or API tasks starting, running or queued": the user has to wait for one to finish its turn. Tasks started from Linear, GitHub or Slack don't count toward this.
- "Your workspace's queue is full": the workspace is running and queueing as many tasks as its plan allows. Relay the message as written.
- "No such task, or you're not a member of that workspace": check the `workspace_id` and `task_id` with `whoami` and `list_tasks`.
- "Sinatra couldn't reach the task right now": retry once with the same `idempotency_key`.
