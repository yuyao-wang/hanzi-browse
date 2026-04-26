---
name: play-console-review-reply
description: Reply to unanswered Google Play Store reviews from Play Console. Fetches reviews, categorizes them (bug / feature request / praise / complaint / question), drafts appropriate responses, and posts them one by one. Requires the hanzi browser automation MCP server and Chrome extension with Play Console open.
category: marketing
---

# Play Console Review Reply

You read unanswered Google Play Store reviews and post AI-drafted replies directly in Play Console via the user's Chrome browser.

## Tool Selection Rule

- **Use Hanzi only for Play Console interaction** — there is no public API for reading or replying to reviews without Google API credentials. The browser is genuinely needed.
- **Use your own reasoning** for categorizing reviews and drafting replies — do not open a browser for that.

## Before Starting — Preflight Check

Try calling `browser_status`. If the tool doesn't exist or returns an error:

> **Hanzi isn't set up yet.** This skill needs the hanzi browser extension running in Chrome.
>
> 1. Install from the Chrome Web Store: https://chromewebstore.google.com/detail/hanzi-browse/iklpkemlmbhemkiojndpbhoakgikpmcd
> 2. The extension will walk you through setup (~1 minute)
> 3. Then come back and run this again

Also confirm the user has **Play Console open and logged in** in Chrome before proceeding.

---

## What You Need From the User

1. **App name or package name** — e.g. "Zen Snake" or `com.example.snake`
2. **Reply tone** — professional, friendly, casual (default: friendly)
3. **Max reviews to reply to** — default 10
4. **Any context** — known bugs, upcoming features, things to mention or avoid

---

## Step 1: Fetch Unanswered Reviews

Start a browser session to navigate Play Console and extract reviews:

```
Task: Go to https://play.google.com/console and find the app "{app_name}".
Navigate to Ratings and reviews > Reviews.
Filter to show only reviews with no reply.
Extract up to {max} reviews. For each, return:
- reviewer name
- star rating (1-5)
- review text
- date posted
Return as a JSON array.
```

If the agent can't find the app by name, try searching by package name instead.

If Play Console asks to select a Google account, pick the one that has access to the app.

---

## Step 2: Categorize and Draft Replies

For each review, do this yourself (no browser needed):

**Categorize** the review as one of:
- `bug` — reports a crash, error, or broken feature
- `feature_request` — asks for something new
- `praise` — positive feedback
- `complaint` — negative experience, not a specific bug
- `question` — asks how something works

**Draft a reply** that:
- Acknowledges their specific experience (reference their exact words when possible)
- Is under 350 characters (Play Console limit)
- Matches the requested tone
- For bugs: thanks them for reporting, confirms you're looking into it, invites them to email support for a faster fix
- For feature requests: thanks them, says you've noted it for consideration
- For praise: thank them warmly, invite them to share with friends
- For complaints: apologize, offer a concrete next step (email, update coming, etc.)
- For questions: answer directly and clearly

Show the user a table before posting:

| # | Reviewer | Stars | Category | Draft reply | Chars |
|---|----------|-------|----------|-------------|-------|
| 1 | ... | ⭐⭐ | bug | ... | 210 |

Ask: **"Post all, skip some, or edit any replies before posting?"**

---

## Step 3: Post Approved Replies

Post one at a time. For each approved reply:

```
Task: In Play Console, find the review by {reviewer_name} that says "{first_20_chars_of_review}".
Click Reply. Type the following response exactly:
"{reply_text}"
Submit the reply.
```

Wait for confirmation before moving to the next review. If a post fails, report it and ask whether to retry or skip.

Log each posted reply:
```bash
mkdir -p ~/.hanzi-browse && echo "{app_name}|{reviewer_name}|$(date +%Y-%m-%d)" >> ~/.hanzi-browse/play-console-replied.txt
```

---

## Safety Rules

- **Never post a reply without user approval**
- Max 20 replies per session to avoid triggering Play Console rate limits
- Wait at least 3 seconds between posts
- If Play Console shows a CAPTCHA or error, stop and tell the user
- Never fabricate review details — only reply to reviews the browser actually found
- If the reply exceeds 350 characters, shorten it before posting

---

## When Done

Summarize:
- App name and total reviews found
- Replies posted / skipped / failed
- Categories breakdown (e.g. "3 bugs, 2 praise, 1 question")
- Any patterns worth flagging (e.g. "4 users mentioned the same crash on Android 14")
