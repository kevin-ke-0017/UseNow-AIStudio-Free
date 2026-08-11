<p align="right"><a href="CHANGELOG.md">🇨🇳 简体中文</a> · <b>🇬🇧 English</b></p>

# Changelog

Only user-visible changes are listed. Download: **[Releases](../../releases/latest)**

---

## V7.1 · 2026-08-11

**Fixed**
- **A reference image rejected by content moderation used to be misdiagnosed.**
  The service puts the reason in `code` (`content_policy_violation`) and the prose in
  `message`; the app only read the message, so "content rejected" was classified as
  "some parameters may be unsupported — try a different aspect/duration", pointing you
  the wrong way entirely. The `code` is now read too, and the hint says plainly that the
  **reference image** is the usual cause and that aspect/duration/resolution are irrelevant.
- **No more waiting 20 minutes for nothing when the service stalls a task.** Sometimes it
  neither rejects nor starts — it just hangs. After three minutes still queued, the app now
  names the two likely causes (backend queue / stuck in moderation) and offers a
  **Stop waiting** button; stopping keeps the `video_id` so a task that finishes later can
  still be retrieved. The 20-minute timeout screen carries the same note and the ID.
- A failure detected while polling used to dump raw JSON only. It now gives the same
  plain-language hint as a failure at submit time, and notes the reference image is kept.

## V7.0 · 2026-08-10

**Fixed**
- After switching to English, the result line under the video parameters
  (size / duration / frames / frame rate) and the empty state of the video
  gallery stayed in Chinese. Both are assembled in JS without translation
  markers, so the language sweep never reached them; they are now recomputed
  along with everything else.

## V6.9 · 2026-08-10

> V6.6–V6.8 were interim builds that were never released on their own; their changes are folded into this entry.

**Fixed**
- After generating one image-to-video clip, pressing "Generate" again claimed a first
  frame was missing. The reference image was cleared on submit and never restored on
  success, so a picture chosen from the gallery was lost. It is no longer cleared.
- **Choosing an aspect now actually applies.** The output ratio is driven by the reference
  image — a portrait photo with 16:9 selected still produced a portrait clip. The reference
  image is now fitted to the aspect you picked before it is sent.
- **Reference images are no longer cropped.** When the ratios differ a lot (say a 3:4
  photo with 16:9 selected) the whole image is kept, centred and scaled, with the sides
  filled by a blurred enlargement of the same picture — nothing cropped, stretched, or
  letterboxed in flat black.
- **Thin blurred edges with a matching aspect are fixed.** Preset sizes snap to multiples
  of 64 and are not exactly the nominal ratio (720p 3:4 is really 832×1088). That 2%
  gap used to be padded, leaving a few pixels of blur on each side. Close ratios now fill.
- **1080p at 10 seconds no longer fails** with `num_frames exceeds max frames for the
  resolved resolution and ratio`. Beyond the documented 441-frame cap, the service also
  limits frames per resolution: 297 frames pass at 720p and are rejected at 1080p. The
  real duration limit for the current combination is now shown under the duration field,
  and a rejection is remembered and retried shorter instead of surfacing as an error.
- The "raw response" shown on failure used to be the previous successful response, which
  was actively misleading while debugging. It is now the current one.

**Improved**
- Duration now rounds **up**. `num_frames` must satisfy `8n+1`, so clips come out a touch
  longer than requested rather than shorter. Default frame rate is now 30.
- All "calibration" wording is gone. The real frame rate is measured silently and stored
  per requested rate, so switching rates never contaminates the maths — you don't need to
  know the mechanism exists.
- Once a reference image is chosen, the aspect field explains up front what will happen
  ("Reference image is 3:4: kept whole and padded with a blurred backdrop — pick 3:4 to
  fill the frame") instead of leaving it to be discovered after generating.
- The video parameter block was rebuilt: duration / aspect / resolution / frame rate sit
  in one even row, with a single result line beneath —
  `1280×704 · 10.2 s · 305 frames · 30 fps · up to 14.2 s` — plus a note only when a
  limit is hit or the reference ratio differs.

## V6.5 · 2026-08-09

**Fixed**
- Video card → "Prompt → Edit again" no longer jumps to the Image page. The popup was
  shared with image generation and had the image textarea hard-coded.
- **Videos coming out shorter than selected is fixed.** No frames were lost — the frame
  rate differed: the service always encodes at 30 fps while the app computed frames at 24 fps,
  so a "10 s" clip ran 8 s. The app now **measures the real frame rate** from your first
  video and uses it for every calculation after that.

- **Video resolution mismatch is fixed.** The service only emits fixed presets
  (480p / 720p / 1080p × 16:9 / 9:16 / 1:1 / 4:3 / 3:4). The app used to send sizes like
  1152×768 that sit off-preset and got remapped. Aspect is now picked as
  ratio + resolution tier, so the request lands on a real preset.
- Added the `num_frames ≤ 441` cap that the API requires.

**New**
- Custom video duration instead of only 3/5/10/15, with a live readout of the frame count,
  expected runtime, and a notice when the 441-frame cap is reached.
- Added 4:3 and 3:4 aspects, and the exact pixel size is shown before you generate.
- After generation the app shows both the **API-reported** `size` / `seconds` and the
  **player-measured** resolution, duration and frame rate, quoting the service's own
  `size_mapping.message` when they differ from your request.

## V6.4 · 2026-08-01

**New**
- **Releases now ship a zip**: `UseNowAIStudio-x.y.z-win64.zip` (app + bilingual quick-start + SHA256) instead of a bare `.exe`. Unzip and double-click — SmartScreen usually stays quiet on first run.
- **Agnes AI China endpoint**: the default base URL is now `https://apihub.agnes-ai.cn/v1`, so users in mainland China no longer need a proxy.
- One-click **Endpoint** switch in API Settings and the setup wizard: **China (.cn) / Global (.com)**. Same account, same key — both sites work.
- Guide updated with both sign-up sites: `agnes-ai.cn` for mainland China, `agnes-ai.com` elsewhere.

> Existing users keep whatever URL they saved. To switch, open ⚙️ API Settings and click the endpoint you want.

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
