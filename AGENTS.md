# AGENTS.md — Coding Agent Operating Manual

> Give this file to your coding agent and check that it is using these instructions. Keep the stack, commands, and ground rules current.

## 1. Project Overview
- **What this repo builds:** Springboard, a DJ set builder. Users keep several sets, add songs by pasting a list or uploading a playlist screenshot, and the AI checks each song's BPM, Camelot key and genre online, suggests next songs, and explains every transition.
- **Primary user & core workflow:** Beginner/intermediate DJs. See the living PRD in `docs/index.html`.
- **Tech stack:** Plain HTML/CSS/JS (no build step) + Python 3 standard-library server + OpenRouter API + PostHog analytics. Hosted on Render (free web service).
- **Data:** no database. Sets are saved in each visitor's own browser (localStorage). The server stores nothing.

## 2. Project Structure
```
docs/index.html      Product Page (PRD, build log). Served at /.
docs/app/index.html  The app (Springboard). Single file: styles, data and logic. Served at /app/.
docs/analytics.js    PostHog setup, shared by both pages. Holds the public phc_ key.
server.py            Serves docs/ and forwards POST /api/ai to OpenRouter, with rate limits.
requirements.txt     Empty on purpose: no third-party packages.
SECURITY.md          Security audit results.
.env                 Your private OpenRouter key, local only (never committed). Copy from .env.example.
```
- AI request flow: app → `POST /api/ai {prompt, image?, web?}` → `server.py` (rate limit, size checks) → OpenRouter → JSON list → app.
- `GET /api/health` tells the app whether the AI is available.
- **Two copies of index.html exist.** `docs/index.html` is the Product Page; `docs/app/index.html` is the app. Check line 6 (`<title>`) before replacing either.

## 3. Commands
- **Install dependencies:** none (Python 3 standard library only).
- **Set up locally:** `cp .env.example .env`, then put your OpenRouter key in `.env`.
- **Run locally:** `python3 server.py`, then open http://localhost:8000/app/ (Product Page at http://localhost:8000/).
- **Audit dependencies:** `pip install pip-audit && pip-audit -r requirements.txt`
- **Run deterministic tests:** none yet. Manual check: create a set, paste two songs, confirm they get "verified" tags, click "Suggest songs", reorder, open "Your set".
- **Rate-limit check (live site, browser console):** see SECURITY.md.
- **Run model evals:** not yet (Build 4).

## 3b. Shipping (deploy to Render)
1. Commit and push to `main` on GitHub.
2. First time only: on render.com, **New → Web Service**, connect the `building-with-ai-f26/add437-springboard` repo, and set:
   - Runtime: Python 3 · Build command: `pip install -r requirements.txt` · Start command: `python3 server.py` · Instance: Free
   - Environment variables: `OPENROUTER_API_KEY` (secret, from OpenRouter) and `OPENROUTER_MODEL=openai/gpt-4o-mini`
3. Render redeploys automatically on every push to `main`. Watch the **Events/Logs** tab until it says "Live".
4. Verify the live site: open `/app/` on the Render URL, run the manual check above on a phone, confirm PostHog shows events and a session replay.
5. Roll back if broken: Render → **Events** → pick the last good deploy → **Rollback**.
6. Never put the OpenRouter key in any file in the repo; it lives only in Render's Environment settings and your local `.env`.

## 3c. Ask me first
Stop and ask the human before:
- Deleting files, data or branches, or changing how data is stored (adding a database, changing localStorage keys).
- Anything involving keys, secrets, environment variables, permissions, or GitHub/Render/OpenRouter account settings.
- Anything that costs money: changing the AI model, raising rate or spending limits, upgrading a Render plan.
- Replacing `docs/index.html` or `docs/app/index.html` wholesale.

## 4. Non-Negotiable Engineering Rules
1. **Plan before coding:** Before touching multiple files, propose a brief plan and wait for approval.
2. **Small batches:** Make small, focused changes that can be reviewed and explained by a human in a single Pull Request.
3. **Explainability:** Avoid unnecessary abstractions or dependencies. Help the human builder understand the main components, what changed, why it matters, what could break, and how it was checked.
4. **Verify changes:** Run relevant checks. For a reproducible logic bug, use a failing test to demonstrate the fix when practical. Record manual checks and known limitations; follow the selected Build 4 evaluation track.
5. **Zero secrets in code:** Never hardcode API keys, tokens, or credentials. Always read from environment variables (`.env`) and update `.env.example` when new variables are needed. The PostHog `phc_` key is public and may be committed.
6. **Stop when looping:** If a fix fails twice in a row, stop, summarize what failed, and ask the human before trying a third approach.
7. **Check the destination path before replacing a file (real mistake):** The app once overwrote `docs/index.html`, replacing the Product Page with the app. Before replacing any `index.html`, confirm the folder and the `<title>` on line 6.
8. **Don't retry blocked pushes (real mistake):** A coding agent hit repeated GitHub 403 errors while pushing. When GitHub refuses access, stop and ask the human to fix permissions instead of retrying.
