# Paper remote MCP for Rescheme

## Overview

This plugin declares the `paper-rescheme` remote MCP server.
The server URL is `https://mcp.paper.design`.
The server uses `tokenScope: "user"`.

## Account separation

Devin stores OAuth tokens per MCP server name.
The name `paper-rescheme` keeps your Rescheme Paper login separate from other Paper accounts.

## Connect

1. Open the Devin MCP settings.
2. Find `paper-rescheme`.
3. Select Connect.
4. Sign in with your Rescheme Paper account.
5. Confirm access to the Rescheme Paper team before you edit design files.

## Data notice

Each member connects their own Rescheme Paper account.
Paper receives session context that you send to the MCP server.
Use Paper only for design work that may use that context.

## Local sessions

Local Devin Desktop sessions can also use the local Paper Desktop MCP server.
The local server uses the Paper account signed in to Paper Desktop.
