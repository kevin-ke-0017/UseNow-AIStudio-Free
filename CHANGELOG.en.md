<p align="right"><a href="CHANGELOG.md">🇨🇳 简体中文</a> · <b>🇬🇧 English</b></p>

# Changelog

Only user-visible changes are listed. Download: **[Releases](../../releases/latest)**

---

## V10.1 · 2026-09-22

**Stated plainly: one redemption code = 1 computer and 1 phone at a time**

The download page, the notes inside the zip and Settings → Licence used to describe the same
thing two different ways: one said "1 computer and 1 phone at a time", the other said "up to 3
computers and up to 3 phones". **The second one turned the re-bind allowance into a
simultaneous-device count — promising two extra computers and two extra phones in writing.**

All four now say the same thing, matching what is actually sold:

| | |
|---|---|
| In use at once | **1 computer + 1 phone** (tallied separately; using both costs no re-binds) |
| Free re-binds | **2** per side |
| Does NOT count | reinstalling the app, clearing its data, swapping a network card |
| Counts as one re-bind | OS reinstall / factory reset / a different device |

When the re-binds are used up, contact the seller, who can sign a fresh activation code by hand.

> No functional change in this release — it only makes the wording unambiguous.

---

## V10.0 · 2026-09-22

**Fixes the sentence that was still wrong on the v9.9.0 download page and inside the zip**

V9.9 corrected the misleading "works offline forever" wording inside the app, **but the
download page and the `ReadMe-使用说明.txt` inside the zip were missed** — so the v9.9.0
release page carried the correction and the wrong claim side by side. The program itself is
unchanged and had no defect; this release exists because the text a buyer reads said the
opposite of the truth.

All four places (release body in both languages, in-zip ReadMe in both languages) now match
the app:

> Only the activation step needs the internet — afterwards the **licence is never re-verified
> online** and never asks again. That is about the **licence**: chat, images and video all call
> an AI API, so **using the AI features still needs a connection**. Offline the app still opens,
> and the local knowledge base, history and export all work.

> The release gate gained a rule that blocks "offline forever" style claims, right next to the
> one that blocks "free forever". **The miss happened precisely because that check relied on
> somebody remembering.**

---

## V9.9 · 2026-09-22

**The settings panel is renamed, plus three places where raw tags leaked into the UI**

- **“⚙️ API Settings” is now just “⚙️ Settings”.** It has long held more than API
  configuration — licence, personalisation, interface language, general options and the
  diagnostics log all live on that page, and the old name made them hard to find.
  Every in-app pointer to it (setup wizard, timeout hint, auth-failure hint and so on)
  was updated to match. This also fixes an older oversight: the title was never wired
  into the translation table, so **it showed in Chinese even in English mode**.
- **`<b>` `</b>` `<br>` no longer show up as literal angle brackets** in three
  explanatory paragraphs: Settings → General → the auto re-queue note,
  Settings → Licence → the re-bind note, and Settings → Diagnostics log.
  Those three were routed through the “fill as plain text” translation channel, so the
  bold tags were escaped and printed. Harmless to function, but it looked unfinished.

> A static assertion now covers this (`regress.js`): any string containing formatting
> tags that is wired to the wrong channel fails the suite immediately, instead of being
> spotted in a screenshot.

**Video progress: 1% used to render as 100%**

Polling was re-checked line by line against the official Agnes Video 2.5 docs. Three fixes:

- **`1%` progress showed as `100%`.** The docs state that `progress` is an **integer
  from 0 to 100**, but the app carried a "if it's ≤1, multiply by 100" rule meant for
  servers that report a 0–1 fraction. When the server honestly returned `progress: 1`,
  the bar filled instantly — and staring at a 100% bar for several minutes just looks
  like the app has frozen. Only a true fraction (strictly between 0 and 1) is now
  treated as a ratio; integers are read as 0–100.
- **A clip counts as finished only when `status` is `completed`.** The docs are explicit:
  go by `status` and `metadata.url`. The app used to declare success as soon as it found
  any video URL, which can be an intermediate asset. It now keeps waiting while the
  server says queued or in progress — but does not wait forever: after 3 consecutive
  polls (≥15s) of "URL present, status unchanged" it accepts the URL rather than
  discarding a clip that is actually playable.
- **Polling now backs off when it hits the rate limit** (5s → 10 → 20 → 40 → 60s cap,
  straight back to 5s once responses are normal) — exactly what the docs recommend for
  `429`. Before, it kept asking every 5 seconds, which both kept tripping the limit and
  ate the per-minute budget that submitting needs: one quota, two paths competing.
- The 20-minute polling cap is now measured in **wall-clock time** instead of attempts.
  It used to be "240 attempts × 5s", and once back-off exists those are not the same thing.

> The interval stays at **5 seconds** rather than the 1–2s the docs suggest: that would be
> 30–60 requests a minute, and the free tier limits requests per minute — which is exactly
> what caused the whole round of rate limiting fixed in V9.8.

**Two inaccurate explanations, corrected**

- "Once activated it **works offline forever**" reads as "this app works without a
  connection". What is actually true: the **licence** is verified once and never
  re-checked online — but chat, images and video all call an AI API, so **using the AI
  features still needs a connection**. Offline the app still opens, and the local
  knowledge base, history and export all work. The line on the activation page was
  corrected too.
- "2.5 reference images **must be public URLs**" — **not true**; local uploads do work.
  Both routes are now described: a public URL is what the docs specify and is the most
  reliable (gallery images carry one); a local file is sent as embedded data, which the
  docs do not cover and the provider could tighten. The failure hint no longer claims
  that route is impossible — it just says which route this attempt used.

---

## V9.8 · 2026-09-22

**Video queueing: no longer cut short by its own retries, and a full queue no longer looks like a fault**

What used to happen — it looked exactly like a bug in the app:

1. Submit a video → the server answers **`503 video queue is full`** (the backend runs one
   job at a time, so this is common at peak)
2. The app re-queued every 8 seconds → 3 requests inside 25 seconds
3. The 3rd request tripped the **rate limit** (free tiers count requests per minute)
4. The app treated the rate limit as a **final failure** → a screen full of red

**So the "queue for up to 20 minutes" design never actually happened** — the real ceiling was
"however many probes your quota allows", i.e. 3 probes in 25 seconds. That red screen was not a
queueing failure; the app was cutting its own queueing short.

This release:

- **A rate limit hit while queueing no longer ends the wait** — it means "slow down", not
  "no slot for you". The 20-minute budget works for the first time: success now depends on
  patience rather than luck.
- **New setting: ⚙️ Settings → "Auto re-queue when the video queue is full"**
  On by default, **30s** interval (minimum 15s), **20 minutes** maximum wait — all adjustable.
  Why adjustable: the safe interval depends on **your API tier**, which the app cannot see.
  Free and paid tiers differ a lot, so hard-coding a number would just be guessing for you.
  **Switch it off** and a full queue is simply reported, with a "🔄 Re-queue" button for you.
- **The queueing screen is now neutral**: "re-queueing automatically · waiting 3m20s ·
  attempt 5" instead of red failure styling — queueing is not an error, it is just "not yet".
- **Removed the forced 60-second wait** after hitting a rate limit.
- **The failure message shrank from 304 to 121 characters** by default; the diagnostics (the
  first error verbatim, the raw response, the step-by-step advice) all moved into the
  collapsed "show raw response" section.

> ⚠ **The single most useful habit**: **do not click "Generate" repeatedly yourself.**
> Each re-queue is one API request, and several submissions in quick succession trip the
> provider's rate limit — making a slot harder to get, not easier. Let auto re-queue do it,
> or raise the interval.

---

## V9.7 · 2026-09-22

**Spelled out how many devices one redemption code covers**

- The activation page only said "each code covers 2 free re-binds" and never mentioned that
  computer and phone are **tallied separately**. Now it says so:

  | | First activation | Free re-binds | Total |
  |---|---|---|---|
  | 💻 Computer | 1 | 2 | **3** |
  | 📱 Phone | 1 | 2 | **3** |

  **One code runs on your computer and your phone at the same time, costing no re-binds.**
  After three computers, the phone's three are still untouched.
- It also spells out what does *not* count as a re-bind: reinstalling the app, clearing its
  data or swapping a network card (the same device simply gets its original licence
  re-issued). Only an OS reinstall, factory reset or a different device consumes one.
- Corrected "free forever for personal use" in the licensing section — this has been **paid,
  per-device software since V8.7**. Whatever you earn with it is still entirely yours.

---

## V9.6 · 2026-09-21

**A failed video submit now tells you what the *first* error was**

- A "server busy" response makes the app retry. The catch: retrying is itself what trips
  rate limits — so **what you end up seeing is "rate limited", while the error that
  actually caused the retry left no trace at all**. Diagnosis dead-ends there: you only
  ever see the consequence of the retry, never its cause.
- The failure message now adds **how many requests that attempt really sent** and **the
  first error verbatim**. The diagnostics log gained `busyErr` / `busyStatus`.
- If that first line does not look like "server busy", it *is* the real cause — send it
  to the developer.

---

## V9.5 · 2026-09-21

**Fix: video generation gave up after 10 minutes, not the intended 20**

- The poll counter was **incremented twice per poll**, and the timeout check reads that
  counter. So the code said "240 polls × 5s = 20 minutes" while it actually gave up at
  poll 120 — **10 minutes**. A job that would have finished at 12 minutes was abandoned
  and the screen said "⏱ Timeout"; all you saw was another failure. It is now really 20.
- **Half the requests while waiting.** Progress checks asked the new endpoint first and
  then fell back to the legacy one whenever no video URL had come back yet — but a job
  that is simply still running has no URL, so the fallback fired every 5 seconds for
  nothing. It now only runs when the new endpoint genuinely can't answer. This also
  lowers the chance of tripping a rate limit.
- `attempts` on the video rows of the diagnostics log used to be double the real poll
  count; it is now accurate.

> Both were found by a user questioning the `attempts` number in the diagnostics log —
> it was arithmetically impossible (125 polls × 5s needs 10m25s, but only 6m11s had passed).

---

## V9.4 · 2026-09-21

**Fix: the video retry loop turned "server busy" into "rate limited"**

- When a video submit hit "server busy", the app retried on a **fixed 8-second timer**.
  One click therefore fired 3 requests within 25 seconds — and free tiers limit exactly
  that: requests per minute. So the first request was merely *busy* and the third came
  back 429. **The app was manufacturing its own failure.**
- Retries now **back off**: 8s → 16s → 32s → 60s max. Same three attempts, one third of
  the request density.
- **After a rate limit, a 60-second cooldown** blocks Generate and tells you how many
  seconds are left. Previously the button was immediately clickable and said nothing, so
  clicking again just dug the limit deeper. The cooldown lives in memory only — reopening
  the app retries right away.

> To check whether this is what you are hitting: ⚙️ Settings → "🩺 API diagnostics log",
> and look at `tries` on the video rows. Anything above 1 means that click sent multiple
> requests.

---

## V9.3 · 2026-09-21

**Fix: a rate limit was reported as "the server is busy"**

- When video generation failed because the API account hit a rate limit (HTTP 429),
  the app said "⏳ The video backend handles one task at a time — wait 1–2 min and retry."
- **Those two call for opposite actions.** Busy clears if you wait; a rate limit
  **does not** — you have to slow down or upgrade with your API provider. Anyone
  reading "wait 1–2 min" will keep waiting for an error that never clears.
- It now says plainly that this is a rate limit rather than congestion, and gives three
  concrete steps. The provider's own message is still shown in full so you can forward it.
- Worth knowing: the image page sends **one request per image**, so a burst of image
  generation around a video job can easily exhaust your per-minute allowance. The hint
  says so too.
- Images and chat were always classified correctly; only the video path missed this.

---

## V9.2 · 2026-09-21

**Video requests now appear in the diagnostics log**

- The "🩺 API diagnostics log" only recorded **image** requests, while the description
  said "image/video". Video is the slowest path and the one most likely to stall — so
  when a video never arrived and you sent the log to support, it contained nothing
  about video at all.
- One video generation now records two entries, **kept apart**: how long the submit
  queued (and how many retries it took) versus how long the task itself ran. A stall
  in the first means the video service is busy and waiting is the answer; a stall in
  the second means the task itself went wrong.
- Task failures are logged, and so is "the server said it finished but gave no video
  URL" — the screen says *done* while you have nothing, and that state used to leave
  no trace at all.
- Cancelling yourself is not a failure and is not logged.
- Nothing new is collected: the log still has **no API key and no prompt text** (length
  only).

---

## V9.1 · 2026-09-21

**The diagnostics log now names where your machine code came from**

- On Windows the machine code has two sources. Normally it reads a system identifier that
  **survives reinstalling the app and swapping network cards**. If that read fails, it
  falls back to "computer name + CPU + RAM" — and **that one changes if you rename the PC
  or add a memory stick**, which means re-activating.
- Until now there was no way to tell which path you were on; you only found out when the
  app asked you to activate again. The header of ⚙️ Settings → "🩺 API diagnostics log"
  (the "version" line) now says so, which makes support a one-glance question.
- It is one extra line and collects nothing new: the log still contains **no API key and no
  prompt text**.

---

## V9.0 · 2026-09-21

**Fix: the machine code was missing from Settings**

- **⚙️ Settings → "🔑 Licence" showed a placeholder "…"** — no machine code, a copy
  button that did nothing, and a blank licence status. If you needed your machine code
  after changing computers, the only way was to delete the licence and let the activation
  page reappear. It works now.
- Cause: Settings is a modal, but the code that fills in the licence section was wired to
  an entry point no user can reach, so it never ran in the shipped product.

---

## V8.9 · 2026-09-21

**Overseas users can actually get in now**

- **First launch picks Chinese or English from your system language.** It used to always
  start in Chinese, and the language switch lives *inside* the app — so an overseas user
  faced a full page of Chinese licence text with no switch anywhere in reach.
- **Both the copyright page and the activation page now carry a 🌐 switch** in the
  top-right, so you can change language before entering the app. Once you pick one
  yourself, the system language never overrides it again.
- The intro paragraph on the copyright page was hard-coded Chinese and did not follow the
  language switch. It is now bilingual.
- **Switching language no longer wipes the activation error message** — it is re-rendered
  in the new language. That code in brackets is the only clue for diagnosing a failure.

---

## V8.8 · 2026-09-20

**New default text model · 21:9 for video**

- **Chat now defaults to `agnes-3.0-flash`** (free, latest generation, still reads images).
  The previous `agnes-2.5-flash` is still in the ⚙️ Settings dropdown.
- **New "Ultrawide 21:9" video ratio** (1680×720). Only `agnes-video-2.5-flash` has it;
  switching to `agnes-video-v2.0` hides it and falls back to 16:9.
- **`agnes-video-2.5-flash` is labelled "free for now"**, matching the provider's wording.
- **"🎞️ Multi-image / audio" is now described per model**: on 2.5 it runs in reference
  mode with **up to 5** reference images (one is enough) and 3 audio clips; on V2.0 it is
  keyframe interpolation and still takes 2–3 images only. Both used to say "2–3 key frames".
- **The "how to pick several images at once" hint now matches your device**: hold Ctrl on
  a computer, long-press in the system picker on a phone. It used to say "hold Ctrl
  (⌘ on Mac)" — a phone has neither key, and this product has no Mac build.

---

## V8.7 · 2026-09-12

**Important: this version requires an activation code**

Copies of this product were being resold on a second-hand marketplace, so this version adds
per-device licence activation.

- **On first launch the app shows this device's machine code** (like `W-7K2M-4XQ8-VD3N-PB9F`;
  `W`=computer, `A`=phone). Two ways to activate: enter the **redemption code** from the seller
  for one-click automatic activation, or send your machine code to the seller and paste back the
  **activation code** they give you (use this one when you have no network).
- **Once activated it works offline forever** and never asks again — only that one step needs a
  connection.
- **The machine code contains no hardware details**, no API key and no prompts — it is just a hash
  of a device identifier, so it is safe to send. It is always available under ⚙️ Settings → Licence.
- **Reinstalling the app, swapping a network card or plugging in a USB drive will not void it.**
  Reinstalling the OS, a factory reset or a new device will, and needs a new code. Each redemption
  code covers **2 free re-binds**, counted separately for computer and phone — using both does
  **not** consume them.
- Failures carry a **code in brackets**; send it along with your machine code: `E1` incomplete copy ·
  `E2` invalid · `E3` issued for a different device · `E4` expired · `E5` cannot verify on this
  machine (please use the official build).

**Upgrading from an earlier version** (this version does not inherit an older licence):

1. In the **old version**, open Settings → 💾 Backup & Restore and click "📤 Export all data";
   keep the .zip
2. Uninstall / delete the old version
3. Install this version and activate it
4. In the same place in the new version, click "📥 Import & restore" and pick that .zip

Don't reorder those steps — deleting the old version first means the history and favourites are
gone for good.

## V8.6 · 2026-09-05

**Changed**
- **"🎬 Video" under a gallery image now feeds both modes at once**: it becomes the first frame for
  image→video *and* joins the reference list for multi-image / audio. It used to do only the
  former, so using the image in multi-image mode meant uploading it again — which 2.5 cannot even
  accept, since it only takes public URLs. Images already in the list are not duplicated, and
  nothing is pushed out once the model's per-generation cap is reached.
- **A mismatch between a reference image and the selected aspect ratio is now called out.** The
  clip is rendered at the ratio you pick, so a reference in another ratio gets cropped or
  stretched. The controls now spell it out ("reference 16:9 · selected 9:16") and confirm when they
  match. Covers image→video, multi-image reference and V2.0 keyframes.
- **"Custom…" is removed from all three model dropdowns** (text / image / video). The official
  models are fixed and free, and the custom entry mostly invited typos. **This supersedes the V8.5
  note about picking "Custom…".** Anyone who had typed another provider's model keeps it: the saved
  value appears in the dropdown as "(current setting)" and still works — it is never silently
  rewritten.

**Fixed**
- A regression of our own: after switching to a 2.5 model, the reference-ratio note disappeared
  entirely.
- A message-formatting bug: when the same placeholder appeared twice in one sentence, only the
  first was substituted and the second showed raw markers like `%a` to the user. A sweep of the
  whole app found the same fault in the image rate-limit hint; both are fixed and a static check
  now guards against the pattern.

## V8.5 · 2026-09-05

**Added / Changed**
- **The default chat model is now `agnes-2.5-flash` (free).** The provider has **deprecated**
  `agnes-2.0-flash`; 2.5 Flash upgrades coding, tool calling, multi-turn consistency and image
  understanding while staying fully API-compatible.
- **All three model fields are now dropdowns labelled "(free)"**: text, image and video model.
  The official models are fixed and free, so the **"Fetch models" button was removed** — it
  required a working key first and was an extra hurdle for new users. To use another provider's
  model, pick "Custom…" in the dropdown and type it as before.
- **The video model dropdown offers both generations directly**: `agnes-video-2.5-flash` and
  `agnes-video-v2.0`, switchable from Settings or from the Video page, kept in sync.
- **The in-app guide is updated** (both languages): the three video modes, switching models on the
  page, the differences between the two generations, reference audio, the API diagnostics log, the
  adjustable image timeout and prompt extraction.

**Fixed**
- An internal start-up error (video-model constants were read before they were defined). It never
  surfaced as a visible error — it just silently interrupted initialisation, which is the hardest
  kind to notice.

## V8.4 · 2026-09-05

**Added**
- **Agnes Image 2.5 Flash is now the default** (the provider's latest generation, currently free).
  It surpasses 2.1 Flash across the board while keeping identical parameters, size tiers and
  pricing, so it is purely a model-name change. The image-model field now has a dropdown for
  switching between generations. **Existing saved settings are untouched** — to upgrade, pick
  `agnes-image-2.5-flash` in ⚙️ Settings → Image model.
- **Quick model switcher on the Video page.** No need to open Settings: pick the model at the top
  of the page and the controls immediately follow that model's rules (resolution, duration range,
  frame rate, reference-image count, audio input).
- **Reference audio for Agnes Video 2.5** (up to 3 clips). Switch to 2.5 and use the
  "Multi-image / audio" mode to get the audio uploader; refer to clips as `<Audio 1>` in the prompt.
- **The mobile menu button now says "Menu"** next to the ☰ icon and pulses once on first launch.
  Users reported seeing only the icon and **assuming the app had just the chat page**.

**Changed**
- **"Keyframe" is merged into "Image→Video (first/last)".** The two entry points sent
  **exactly the same request** — in both model generations — differing only in how many images you
  supply. Now: a first frame alone animates it; adding a last frame makes it a first→last
  transition. Four tabs become three.
- The multi-image count hint now follows the model: Agnes Video 2.5 allows **5**, V2.0 still 2–3.
  Previously it still said "2–3" after switching to 2.5.

## V8.3 · 2026-09-05

**Added**
- **Support for Agnes Video 2.5 / 2.5 Flash** (currently free at the provider). Set the video
  model to `agnes-video-2.5-flash` in ⚙️ Settings and the UI adapts to that generation's rules:
  resolution locked to 720P, duration limited to 4–12s, the frame-rate control hidden (it has no
  such parameter), and the spec line showing the real output size from the official table
  (e.g. 9:16 → 720×1280).
  > **The model ID has no letter "v"**: the new generation is `agnes-video-2.5-flash`; only the
  > old V2.0 is `agnes-video-v2.0`. Getting it wrong returns "no available channel", and that
  > error now tells you it is probably a typo.

**Known limits (the provider's, not ours)**
- **2.5 accepts reference images only as publicly reachable URLs.** A file picked from your own
  computer has no URL, and whether that path works at all is unverified. Workarounds: generate
  the image in the Image tab first (gallery images carry a URL) and pick it as the reference, or
  switch the video model back to `agnes-video-v2.0`, which accepts local uploads. The UI explains
  this when the related error occurs.
- **2.5 has no negative prompt** (it is absent from the official parameter table), so it is no
  longer sent for that model — sending it would only be silently ignored while looking effective.

**Other**
- The two generations' request formats are fully isolated: `agnes-video-v2.0` behaviour is unchanged.

## V8.2 · 2026-08-26

**Fixed**
- **Image generation could hang on "Generating…" forever with no error.** When the service
  is under load it may accept a request and then never respond, and image requests had
  **no timeout at all** — so the spinner ran indefinitely, the button never reset, and
  nothing was ever shown. There is now a 3-minute cap; on timeout you get a clear
  "the server never responded" message plus a one-line report (time, endpoint, model,
  symptom) you can forward straight to your API provider.
- **Errors did not show the HTTP status code.** Previously you got a bare message with no
  "500" to quote back to the provider. Every failure now carries the status, e.g.
  "HTTP 500: …". When the server returns a web page instead of data (common with gateway
  faults), you no longer see a meaningless "Unexpected token '<'" either.
- **Generating several images at once discarded the successful ones if any single one hung.**
  Successful images are now kept, and the failures are reported separately, e.g.
  "Only 3 succeeded, 1 failed: request timed out".

- **New "🩺 API diagnostics log" (in ⚙️ Settings).** Records the timing and outcome of the
  last 100 image/video requests. When generation is slow or never finishes, hit "Copy
  diagnostics" and send it to your API provider or the developer — it shows whether the wait
  is on the server or here. It stores only time, endpoint host, model, elapsed time and HTTP
  status — **never your API key or prompt text**.

- **New "Image timeout (seconds)" setting** (⚙️ Settings → General; default 180, range 30–900).
  When the provider is congested a request may legitimately queue for several minutes, and a
  hard-wired 3-minute cap would mean "congested = can never generate". Raise it and retry if you
  suspect it is just a long queue. (The separate "Timeout (seconds)" above applies to chat only.)

> All three are about how the app behaves when the service misbehaves. **It cannot fix an
> outage on the provider's side, but it can at least tell you what happened — and give you
> something concrete to report.**

## V8.1 · 2026-08-24

**Fixed**
- **On Windows, "➕ New folder" did nothing when clicked.** Reported by a customer.
  The desktop runtime (Electron) **does not implement** the browser's text-input dialog —
  calling it does not return empty, it throws, so nothing after that line in the click
  handler ever runs: no dialog, no error, no trace. Phones and browsers both support it,
  so only people running the exe hit this.
  > The same cause reached further than that one button: **image favourites, video
  > favourites, history favourites and knowledge-base subfolders — 5 "New folder" entry
  > points in total — were all dead on Windows.** They now share one in-app input dialog:
  > Enter confirms, Esc cancels, clicking outside cancels, and the Android back button
  > closes it.
- **On Android, folder and document names in the knowledge base were invisible.** The name
  is the only element in that row that can shrink — the caret, icon, item count and button
  row are all fixed width — so on a narrow screen it was squeezed to zero width. The
  buttons now wrap to a second line and the name gets its full width back. The image,
  video and history favourite folders share the same structure and were fixed too.
- **"Copy" sometimes did nothing.** The message and code-block copy buttons had no failure
  fallback, so a rejected clipboard write (window not focused, permission denied) failed
  silently. All copying now goes through one entry point that falls back automatically and
  always reports the result.

**Other**
- A full desktop sweep: every button on all seven pages was clicked inside the real desktop
  runtime to confirm there is no second "click does nothing". All three classes of problem
  are now covered by automated checks.

## V8.0 · 2026-08-20

**Fixed**
- **Multi-image video always failed with HTTP 400.** The server's own words:
  `This request contains multiple images, but mode was omitted` — sending several images
  requires declaring `mode=keyframes` alongside them, and that field was missing, so this
  workflow had never once succeeded since the day it was added. It is now sent.
  > This also corrects a deeper misunderstanding: Agnes supports exactly three shapes —
  > no image (text-to-video), **exactly one** image (image-to-video), and
  > **2–3 images with mode=keyframes** (keyframe video). There is no "multiple images
  > without a mode" form. The image count for multi-image video therefore drops from 8 to
  > the **3 the service actually allows**, and picking a fourth is flagged on screen rather
  > than failing at submit time.
- **Multi-image video could not take images one at a time** — each pick replaced the whole
  previous batch, so you had to Ctrl-select them all in one go. Same cause as the image
  reference field in the previous release, and the same fix: an accumulating list where
  every pick is appended, thumbnails are numbered and individually removable, and the screen
  explains how to add more.
- **Switching to English left the reference-image instructions in Chinese.** That line is
  assembled in JavaScript and carries no translation marker, so the language pass never saw
  it. Fixed here and in the sibling spots, with a regression assertion to keep it fixed.

**Also**
- Every interface screenshot on the product page was retaken (six per language). They were
  still showing V7.0 — the image page without **🔍 Image → prompt**, and the video page
  without the negative prompt and seed fields.

## V7.9 · 2026-08-16

**Fixed**
- **The “China direct” route pointed at the wrong address, so keys registered on the
  Chinese site were always rejected as invalid tokens.** It used `apihub.agnes-ai.cn`,
  while the Chinese site's own documentation gives `api.agnes-ai.cn`. This was a nasty one
  to diagnose: that host is alive and answers normally — it simply does not recognise keys
  created in the Chinese console, so the error looked like a mistyped key no matter how
  many times you pasted it again. The “China direct” button now fills in the correct address.
  > Existing saved settings are untouched. If you were stuck on “invalid token”, open
  > ⚙️ API settings and click the “China direct” button once to update it.

- **Only 3 of the 16 image size options were sizes the model actually produces.** The
  other 13 (16:9 at 2560×1440, 4K at 3840×2160, and so on) are not native output sizes,
  so the server silently remapped them to the nearest preset — what you picked and what
  you got were different, with nothing on screen to tell you. The menu is now rebuilt from
  the official native size table (1K / 2K / 3K / 4K across five aspect ratios, 20 entries),
  and each label states the real output size.
  > 7:4 is gone — the model never supported it — and a 3K tier has been added.

**Added**
- **Multi-image composition.** The reference slot is now a list rather than a single image:
  one image is image-to-image, **two or more is composition** — elements from each are
  merged into a single new image, e.g. "put this character and that product on one poster".
  - **Add them one at a time.** Every pick is appended instead of replacing what was there;
    hold Ctrl (⌘ on Mac) to grab several at once. The screen says so.
  - **The 🖼️ button on gallery images also adds one per click**, and mixes freely with
    local uploads — previously those two paths overwrote each other.
  - Each thumbnail carries an index and its own ✕ to remove just that one.
  - No count limit: **Agnes does not document a maximum**, so the app does not invent one.
  - The prompt enhancer switches to a composition-aware structure (role of each reference,
    target scene, how they relate).
- **🔍 Image → prompt.** Pick a reference image and get back an English prompt that would
  reproduce it, with **copy** and **use as prompt** buttons.
  > This uses the **chat model** from your settings (not the image model), so that model
  > must support images. If it doesn't, you get a plain instruction to switch to a
  > multimodal model rather than a raw API error.
- Choosing 3K / 4K with a count above 1 now warns up front: **the provider allows only one
  image per minute at those tiers**, so most of a batch will fail. That's a rate limit, not
  a bug in the app.

**Removed**
- **The negative prompt field for images is gone.** It is not in the image API's parameter
  list at all — sending it raised no error, it was simply ignored, which is worse than not
  having it: you believed it was doing something.
  > The video negative prompt **is** documented and stays.

**Improved**
- When a connection test fails with a token/key error, the route line now tells you which
  address to switch to instead of just echoing the server's message — **nine times out of
  ten this error is the wrong route, not a wrong key.**
- The chat `/draw` command follows the same native sizes and now understands the 2:3, 3:2
  and 21:9 aspect ratios.

## V7.8 · 2026-08-13

**Fixed (desktop and Android)**
- **The “last frame” in Image→Video never took effect.** The thumbnail appeared and the
  video generated fine, but that last frame was never sent to the model — the result was
  identical to supplying the first frame alone. First and last frames are now submitted
  the way the API documents first→last transitions, so the last frame really participates.
- **Keyframe animation could be submitted with only one image.** Keyframe animation
  interpolates *between* two frames; with only one it either errors out or degrades into
  Image→Video. It now clearly asks for both a first and a last frame.

**Added**
- **Negative prompt for video.** Describe what you don't want (blurry, watermark, …),
  the same way image generation already works. Leave it blank to skip.
- **Seed for video.** Reuse the same number to reproduce the same result, which makes it
  easy to tweak wording without changing the whole shot. Blank means random every time.

## V7.7 · 2026-08-12

**Added**
- **Android build.** Same application as the desktop version — identical UI, data format
  and API settings. Installed directly from an APK, no app store. Voice input and
  read-aloud are not available on mobile (the system WebView does not provide them).

**Fixed (mobile)**
- **Saving an image or video jumped to a browser.** Tapping a download link was treated
  as ordinary navigation and handed to an external browser, so you had to switch apps and
  wait for the file to load before you could save it. Downloads now go straight to the
  **system gallery** (other files to the Downloads folder), with a confirmation message.
- **The top bar sat under the status bar** — buttons were hard to hit and easily pulled
  down the notification shade; some panels overlapped it so far that their close button
  could not be tapped. Everything now clears the status bar and gesture area.
- **Scrolling the top bar to the right carried the menu button off screen.** It is pinned now.
- **The version number was invisible on phones** — the footer that carries it is hidden
  there (it covered the composer). It now appears at the bottom of the side menu.
- **The user guide still described voice features that do not exist on mobile.**
- Back button / back gesture now steps out one level at a time: side menu → dialog →
  chat page → press twice to exit. Previously it quit the app outright with a panel open.
- Buttons and dialogs enlarged for touch.

## V7.6 · 2026-08-12

**Fixed**
- **The "elapsed" figure was fake.** It was computed as *poll count × 5 s*, and `genVideo`
  never cancelled the previous polling chain — every submission started another one, each
  counting independently while writing to the same panel, producing self-contradicting
  readouts like "elapsed ~5.9 min · 3:27". It now uses real wall-clock time only, and a
  generation keeps exactly one polling chain (older ones carry a sequence number and retire).
- The long-wait notice fired too early. The threshold is now 2.5× the **measured typical
  duration for that resolution tier** rather than a fixed number of minutes, and the notice
  states what the tier usually takes.

**New**
- **The percentage is back, and this time it is grounded.** Real progress is used when the
  service reports it; otherwise the app estimates from the measured typical duration for
  that tier and labels it **est.** Past 1.5× the typical duration it **stops estimating** and
  falls back to the indeterminate bar — estimating is fine, pretending it is measured is not.
  Defaults come from real runs (480p ≈ 100 s, 720p/1080p ≈ 3 min) and each success is
  recorded, using the median of the last 8.
- The result line now shows **"~N min to produce"** before you press Generate.
- **Video records now show the full parameters**: mode (text/image/keyframe/multi-image),
  resolution, tier, aspect, duration, frame rate, start→end time — plus a **thumbnail of the
  reference image** for image-to-video. Older records simply omit what was never stored.
- **Locally uploaded reference images now have a remove (✕) button.** Previously only
  gallery-picked images had one. Fixed for all four slots (image-to-image, first/last frame,
  multi-image); in multi-image mode a single picture can be removed on its own.

## V7.5 · 2026-08-11

**Retracting the previous release's judgement**

Testing showed that **a perfectly normal reference image also sits at zero progress for
over two minutes and then succeeds.** So "long time at zero progress = stuck" is simply
false — the service reports progress 0 for normally queued tasks too. The
"⚠️ Likely stuck — almost certainly not slowness" claim added in V7.2/V7.4 was unfounded
and would talk you out of jobs that were going to succeed. All of it is withdrawn:

- Removed the "⚠️ Likely stuck" state and every assertion attached to it.
- The long-wait notice is now a **neutral statement**, and only appears after 5 minutes:
  it says plainly that a normal queue can take this long and waiting may still succeed,
  and that the other possibility is moderation — while stating that **the two are
  indistinguishable from here**. Whether to wait or stop is your call.
- Removed the reference-image memory. It treated "timed out" as evidence that an image
  was bad, and that evidence is now shown to be unreliable — a normal image that timed
  out in a queue would be flagged too.
- The stop-waiting and timeout screens no longer show `video_id` or "Query again".
  Querying again returns nothing, so both were noise. They now simply say the reference
  image and settings are kept and you can retry.

> Also verified: sending the same image to the chat endpoint returns a normal description,
> no rejection. Image moderation therefore happens only inside the video pipeline, and the
> client **cannot** find out ahead of time. Telling you before you submit is not possible —
> better to say so than to ship a guess.

## V7.4 · 2026-08-11

**New**
- **Reference images are remembered.** Confirmed by testing: when the same picture is sent
  inline, the service neither rejects it nor starts it — it just hangs until timeout. In other
  words the service will not tell you up front that it was rejected. So the app remembers
  instead: a reference image that was **explicitly rejected**, or that **never started
  processing and timed out**, is flagged the moment you pick it again, right under the
  parameters — no need to spend another few minutes hitting the same wall.
  Advisory only; you can still proceed.

**Improved**
- The indeterminate progress bar is now a smooth sweeping light instead of black stripes —
  the stripes looked poor and could be misread as "a few segments already done".
- Tighter escalation: likely causes and **Stop waiting** now appear at **2 minutes**, and the
  status becomes "⚠️ Likely stuck" at **5 minutes** of zero progress (was 3 / 8). Three or
  four minutes is already enough to tell.

## V7.3 · 2026-08-11

**Improved**
- The debug panel now reports how the reference image was sent. The same picture
  uploaded **from disk** versus picked **from the gallery** may not reach the service the
  same way: a gallery image that is a link and cannot be fetched locally is passed through
  as a link, while every other case is fetched, fitted and embedded. The service may not
  run the same moderation pipeline for a link as for an embedded image — which would
  explain why the same picture is sometimes rejected instantly and sometimes hangs
  until timeout. Failures, timeouts and stop-waiting screens all show which path was used.

## V7.2 · 2026-08-11

**Fixed**
- **The progress bar no longer invents progress.** It used to show
  `max(server progress, a curve estimated from elapsed time)` — added because a bar
  pinned at 0% looked broken. The cost: when the service hangs a task and reports
  progress 0 forever, the bar still crept to 17%, so you waited 20 minutes for nothing.
  **A fabricated progress bar is far worse than an empty one.** When the service reports
  no progress, the app now says "no progress reported" and shows an indeterminate
  striped bar — it never claims a completion percentage it does not have.
- **The stall notice no longer flickers.** It used to be appended after rendering, and the
  5-second poll rewrote the whole block, so it appeared and vanished as the status
  alternated between queued and running. It is now part of the render and stays put.

**Improved**
- Zero progress for a long time is now escalated in stages: at 3 minutes you get the likely
  causes and a **Stop waiting** button; **at 8 minutes with still zero progress the status
  becomes "⚠️ Likely stuck"**, stating plainly that this is almost certainly not slowness —
  the usual cause is a reference image that failed moderation, which the service sometimes
  hangs indefinitely instead of rejecting.
  (It says "likely stuck", not "failed": the service never confirmed a failure, and the
  wording does not draw that conclusion on its behalf.)

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
