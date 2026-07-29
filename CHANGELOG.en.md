<p align="right"><a href="CHANGELOG.md">🇨🇳 简体中文</a> · <b>🇬🇧 English</b></p>

# Changelog

Only user-visible changes are listed. Download: **[Releases](../../releases/latest)**

---

## V6.3 · 2026-07-28

**Fixed**
- English UI completeness: the footer's "Built by / License / Open-source notices" were still Chinese; they now follow the UI language.
- "Export all data" no longer fails with `Invalid string length` once you have a sizeable image library. Backups are now packed as a ZIP with images/videos stored as real binary files — **no size ceiling**, about 25% smaller, and both packing and restoring show progress.
- Import accepts both the new `.zip` backups and older `.json` ones.
- Export/import failures now explain the actual cause (size limit, disk space, memory).

## V6.2 · 2026-07-27

**Added**
- **Native Claude API support.** Paste `https://api.anthropic.com/v1` with an `sk-ant-` key and it just works — chat, streaming, thinking, vision and connection tests included.
- Claude presets updated to Claude Opus 5 / Sonnet 5 / Haiku 4.5.

## V6.1 · 2026-07-24

- Prompt character counters now show a plain count instead of implying a limit that doesn't exist.

## V6.0 · 2026-07-24

**Regenerate no longer discards the previous answer** (headline change)
- After "Regenerate", both answers are kept and a `‹ 1/2 ›` switcher lets you compare them. Works for text and images.
- Each version remembers which model produced it.
- Fixed: switcher only appearing after reopening the conversation; switcher vanishing after going back one version; regenerate re-pasting an entire attached file.
- Fixed: a result landing in the wrong conversation when you switched away while it was generating.

## V5.x · 2026-07-24

- History: project dropdown filter, favourites, and a much higher retention limit.
- Skills: "Add manually" moved next to "Import Skill"; search added to both the skill list and the quick-enable panel.
- Fixed: the History shortcut doing nothing outside the chat page.
- Video progress no longer sits at 0% for the whole wait.
- Uploaded images now reappear when a conversation is reopened.
- Exports show the attachment name instead of dumping the whole file body.

## V4.x · 2026-07-23

- Redesigned **Guide** page: three-step setup plus progressive disclosure, instead of a wall of text.
- Seven portrait presets added to the image prompt library.
- Fixed: long prompts were cut off in the popup and couldn't be scrolled.
- Every failure message now carries a plain-language explanation with the specific server-reported reason.
- DevTools and the context menu are disabled in packaged builds.
- Added a formal EULA with a first-run consent gate, plus a third-party open-source notices page.
- New product icon / logo.

## V3.x · 2026-07-22 ~ 07-23

- Fixed image-to-video producing a single frozen frame (black video).
- Larger video history capacity.
- First public Windows portable build.

---

<sub>Earlier versions were internal development builds and were never published.</sub>
