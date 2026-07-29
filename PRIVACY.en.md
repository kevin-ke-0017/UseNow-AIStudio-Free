<p align="right"><a href="PRIVACY.md">🇨🇳 简体中文</a> · <b>🇬🇧 English</b></p>

# Privacy Notice

> UseNow AI Studio
> Last updated: 2026-07-28

## The short version

**There is no account system. The app collects nothing about you. There is no analytics, telemetry or tracking code of any kind. All your content stays on your own computer.**

---

## 1. What the app connects to

Only two things, and both are required for AI to work at all:

| Destination | When | What is sent |
| --- | --- | --- |
| **The AI provider you configured** (Agnes AI, DeepSeek, OpenAI, Gemini, Claude, Qwen, Zhipu, …) | When you hit Send / Generate | Your input for that request (prompt, images, any knowledge-base passages you enabled) and your API key |
| **The web-search provider you configured** (Firecrawl, Tavily, Serper, Brave — off by default) | When you enable "Web" and ask a question | Your search terms |

Nothing else. No server of mine, no analytics platform, no update beacon.

> The desktop build bundles every third-party front-end library (PDF parsing, syntax highlighting, math rendering) inside the executable, so normal use requires no CDN access.

## 2. Where your data lives

All of it, on your machine, under `%APPDATA%\UseNowAIStudio`:

| Data | Stored in |
| --- | --- |
| Settings, API key, chat history, skills, prompt libraries, projects | localStorage |
| Generated images, knowledge-base documents | IndexedDB |
| Generated videos | only the provider's **temporary URL** (it expires — download what you want to keep) |

Your **API key is stored in plain text locally** and used only to call the provider you chose. Treat your computer as you would with any other saved password.

## 3. You stay in control

- **Export** — ⚙️ Settings → "Backup & Restore" → "📤 Export all data" packages everything into one file you can take with you
- **Delete** — remove the `%APPDATA%\UseNowAIStudio` folder and every trace is gone
- **Uninstall** — delete the .exe; no registry entries, no background services

## 4. Third-party providers

Your input is sent to **the provider you chose**, so how that data is handled is governed by **their** privacy policy, not by this app. Please read their terms before use.

If you work with **sensitive or confidential material**, verify that your chosen provider's data policy meets your requirements, or use a self-hosted model endpoint.

## 5. Children

This software is not directed at children under 14 and does not knowingly collect personal information from anyone.

## 6. Changes

Any change to this notice will be reflected in the "Last updated" date above.

---

📧 Privacy questions: **kdm164747031@gmail.com**
