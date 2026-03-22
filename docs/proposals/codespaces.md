# Codespaces + Copilot Workshop for datawrapper-mcp

## Overview

Create a GitHub repo that, when opened as a Codespace, drops attendees into a VS Code environment where **GitHub Copilot is already wired up to your Datawrapper MCP server on Fly.io**. No local installs, no tokens, no terminal commands. They open the Codespace, switch Copilot to Agent mode, and start making charts.

**What attendees need:** A GitHub account with Copilot access (Free, Pro, or organizational).

**What you provide:** A repo URL (or a "Open in Codespaces" button in the README).

---

## How It Works

Codespaces runs VS Code in the browser. VS Code has Copilot built in. VS Code supports MCP server configuration through the workspace file `.vscode/mcp.json`. Since your Datawrapper MCP server is deployed on Fly.io as a remote HTTP endpoint, no local runtime is needed inside the container — Copilot connects to it directly over the network.

---

## Step 1: Create the Workshop Repo

Create a new public GitHub repo (e.g., `datawrapper-mcp-workshop`). It needs just a handful of files:

```
datawrapper-mcp-workshop/
├── .devcontainer/
│   └── devcontainer.json
├── .vscode/
│   └── mcp.json
└── README.md
```

---

## Step 2: Configure the Dev Container

Create `.devcontainer/devcontainer.json`. This keeps things minimal — you don't need any special runtime since the MCP server is remote:

```json
{
  "name": "Datawrapper MCP Workshop",
  "image": "mcr.microsoft.com/devcontainers/base:ubuntu",
  "customizations": {
    "vscode": {
      "settings": {
        "chat.agent.enabled": true
      }
    }
  }
}
```

That's it for the container. The base Ubuntu image is lightweight and boots fast. Copilot comes pre-installed in Codespaces, and the MCP config is picked up from the workspace.

---

## Step 3: Configure the MCP Server

Create `.vscode/mcp.json` pointing at your Fly.io deployment:

```json
{
  "servers": {
    "datawrapper": {
      "type": "http",
      "url": "https://your-domain.fly.dev/mcp"
    }
  }
}
```

Replace the URL with your actual Fly.io app URL. When the Codespace opens, VS Code discovers this file and registers the MCP server with Copilot. Attendees will see a **Start** button above the server config — one click and it's connected.

---

## Step 4: Write the README

The README is the first thing attendees see. Make it actionable:

```markdown
# Datawrapper MCP Workshop

Create data visualizations with AI using Datawrapper and GitHub Copilot.

## Getting Started

1. Click the button below to open this repo in a Codespace:

   [![Open in GitHub Codespaces](https://github.com/codespaces/badge.svg)](https://codespaces.new/YOUR_USERNAME/datawrapper-mcp-workshop)

2. Once the Codespace loads, open **Copilot Chat** (click the Copilot icon in the title bar or press `Ctrl+Alt+I`).

3. Switch to **Agent** mode using the dropdown at the top of the chat panel.

4. Open `.vscode/mcp.json` and click the **Start** button that appears above the server configuration to connect to the Datawrapper MCP server.

5. Try a prompt!

## Sample Prompts

- "Create a bar chart comparing the population of the 5 largest US cities"
- "Make a line chart showing global CO2 emissions from 1960 to 2020"
- "Create a column chart of the top 10 programming languages by popularity"
- "Build a scatter plot comparing GDP per capita vs life expectancy for 20 countries"

## Troubleshooting

- **Copilot not available?** You need a GitHub account with Copilot access (Free, Pro, or organizational).
- **MCP server not connecting?** Open `.vscode/mcp.json`, click the Start button, and check the output panel for errors.
- **Tools not showing?** In Agent mode, click the tools icon (wrench) at the bottom of the chat to verify Datawrapper tools are listed and enabled.
```

---

## Step 5: Test End-to-End

Before the conference:

1. Push the repo to GitHub
2. Open it in a fresh Codespace (use an incognito browser to simulate an attendee)
3. Verify Copilot Chat loads, Agent mode is available, and the MCP server connects
4. Run a sample prompt and confirm a chart is created in your Datawrapper account
5. Check the Codespace boot time — it should be under 30 seconds with the base image

---

## Day-of Checklist

- [ ] Fly.io server is running (`fly status`)
- [ ] Test a chart creation from a fresh Codespace
- [ ] Have the repo URL and QR code ready to display
- [ ] Keep the Datawrapper dashboard open to show charts appearing live
- [ ] Have a backup plan: if Codespaces is slow, demo from your own machine

---

## Architecture

```
┌──────────────────────────────┐
│   Attendee's Browser         │
│  ┌────────────────────────┐  │
│  │  GitHub Codespace      │  │
│  │  (VS Code in browser)  │  │
│  │                        │  │
│  │  Copilot Chat ←──────────────→  Fly.io
│  │  (Agent mode)          │  │     datawrapper-mcp
│  │                        │  │     (streamable-http)
│  └────────────────────────┘  │          │
└──────────────────────────────┘          │
                                          ▼
                                    Datawrapper API
                                   (charts created here)
```

---

## Considerations

### Copilot Access

Attendees need Copilot access on their GitHub account. As of 2025, **Copilot Free** is available to all GitHub users and includes Agent mode, but may have usage limits. If your audience is developers at a tech conference, most will already have this. Worth mentioning in pre-event communications.

### Organization MCP Policy

If attendees are on Copilot Business or Enterprise through an organization, their org admin must enable the **"MCP servers in Copilot"** policy. This is disabled by default for managed plans. Individual (Free/Pro/Pro+) users are not affected by this policy. Call this out in your setup instructions for enterprise attendees.

### Shared Datawrapper Token

All charts land in your single Datawrapper account since the token lives on Fly.io. This is fine for a workshop — just plan to bulk-delete afterward. The free Datawrapper tier has API rate limits, so if you expect 50+ simultaneous users, consider upgrading to a paid plan for the day.

### Codespace Costs

Codespaces are free for up to 120 core-hours/month on personal accounts. A 2-core Codespace running for a 2-hour workshop uses 4 core-hours per attendee — well within limits. If attendees are on organization-billed Codespaces, the org pays.

---

## After the Conference

1. Bulk-delete charts from your Datawrapper dashboard
2. Optionally archive or make the repo private
3. Shut down the Fly.io server: `fly apps destroy your-app-name`
