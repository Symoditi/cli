# Set up the Symoditi CLI with an agent

Copy the block below into Claude Code, Cursor, Codex, or any coding agent with shell access. It
installs the [Symoditi CLI](README.md), gets you authenticated, and verifies the result. The only
thing it needs from you is your API key — the agent will ask, and tell you where to find it.

````text
You are setting up the Symoditi CLI on my machine and authenticating me with it. Work through
these steps in order in a shell. Stop and ask me whenever a step needs something only I can give
you, and report honestly if something fails instead of working around it.

1. Install the CLI:

   curl -fsSL https://cli.symoditi.com/install | sh

   macOS only for now (Apple Silicon and Intel are both supported). It installs into a
   user-writable directory — never run it with sudo.

2. Verify the binary is reachable:

   symoditi --version

   Expect output like "@symoditi/cli/0.1.1 darwin-arm64 node-v24.14.1". If the shell cannot find
   `symoditi`, the installer printed a PATH hint — add that directory to my shell profile, then
   re-check in a fresh shell.

3. Ask me for my Symoditi API key. Do not invent one, do not guess, and do not reuse a key found
   anywhere else on this machine. Tell me I can mint one in the Symoditi app under
   Settings -> API Keys: https://app.symoditi.com/settings/api-keys — then wait for my answer.

4. Save the key as a profile. Keep it out of shell history: have me export it as SYMODITI_KEY
   first (or read it from a file), then run:

   symoditi config login --name prod --url https://api.symoditi.com --key "$SYMODITI_KEY"

   The command verifies the key against the API before writing anything. If it fails, the key is
   wrong, expired, or lacks access — tell me, don't retry silently with a different key.

5. Smoke-test the setup:

   symoditi config doctor
   symoditi me

   `config doctor` should report permissions 600 and "Health check: OK". Both commands must
   succeed before you continue.

6. Show me what the key can reach:

   symoditi brand-analytics get-workspaces -o json

   Summarize the workspaces and the brands nested under them as a short list, with their IDs.

7. Report back: the installed version, the profile name, and the workspaces and brands I have
   access to. Never print my API key, not even partially.

Useful things to know while you work:

- Output is a table on a terminal and JSON when piped, so `symoditi <command> | jq ...` works;
  pass `-o json` when you want to be explicit.
- Every failure exits non-zero, so `set -e` and `if ! symoditi ...; then` behave correctly.
- The CLI is self-documenting: `symoditi --help` lists topics, `symoditi <topic> --help` lists
  its commands, and `symoditi <topic> <command> --help` lists every flag.
- SYMODITI_API_KEY and SYMODITI_API_URL work as environment variables if you ever need to run
  the CLI without a saved profile.

If I ask about my brand once setup is done, these two commands are the usual starting points
(workspace and brand IDs come from step 6):

   symoditi brand-analytics get-overview <workspaceId> <brandId> --dateFrom <YYYY-MM-DD> --dateTo <YYYY-MM-DD> -o json
   symoditi brand-analytics get-sentiment-drivers <workspaceId> <brandId> --view pool --limit 10 -o json

The first returns the day-by-day sentiment split for the brand. The second ranks the pages
driving negative sentiment, with how much of the negative pool each one accounts for.
````

Once the agent is done, [README.md](README.md) walks through the same commands in more detail,
plus the "what changed since last week" view.
