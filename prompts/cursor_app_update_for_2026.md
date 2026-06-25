# App update for 2026
_Exported on 6/24/2026 at 19:26:13 PDT from Cursor (3.8.11)_

---

**User**

Let's update this app for 2026. 

- move the current index.html to a new file named ai-engineer-worlds-fair-san-francisco-2025.html and link to it from the new app
- the new app at index.html will be similar to the current app but have the following changes:

  - The sessions JSON feed for 2026 is now https://www.ai.engineer/worldsfair/2026/sessions.json. The 2025 app should still continue to use it's old JSON endpoint. 
  - Upgrade the model to gpt-5.4 for the 2026 app.



---

**Cursor**

I'll read the current app and locate where the sessions feed and model are configured so we can split 2025 vs 2026.


Checking the 2026 sessions JSON format and whether the data-loading logic needs changes.


The 2026 feed uses a different schema (no Sessionize IDs), so I'll preserve the 2025 page as-is and adapt the new app's data loading for the 2026 format.


The app is updated for 2026. Here’s what changed:

### 2025 archive
- Current app moved to `ai-engineer-worlds-fair-san-francisco-2025.html`
- Still uses the Sessionize endpoint (`https://sessionize.com/api/v2/w3hd2z8a/view/All`) and `gpt-4o-mini`
- YouTube links and all 2025 behavior are unchanged

### New `index.html` (2026)
- **Sessions feed:** `https://www.ai.engineer/worldsfair/2026/sessions.json`
- **Model:** `gpt-5.4`
- **Link to 2025:** footer line points to the archived 2025 page
- **Data loading:** adapted for the 2026 JSON shape (`day`, `time`, `speakers` as names, etc.) — generates stable session IDs from title/day/time/room and maps conference days to June 29–July 2, 2026
- **localStorage:** uses `csp_sessions_data_2026` and `csp_watched_sessions_2026` so 2025 and 2026 data don’t collide
- **YouTube links:** cleared for now (2026 recordings aren’t available yet); Watch buttons will appear once links are added

The 2026 feed has 543 sessions and loads correctly with the new parser.

---

**User**

For @index.html:275  will sessionids map the same way as before?

---

**Cursor**

Checking how session IDs are generated in the 2026 app versus the 2025 app and how they tie into the YouTube link map.


No — the **lookup mechanism** is the same, but the **IDs are not**.

### Same as before
The `session-ids-to-youtube-links` block still works the same way: tab-separated lines of `sessionId` + YouTube URL, parsed into `csp_youtubeLinks`, then looked up with `csp_youtubeLinks.get(s.id)` for Watch buttons and ChatGPT response links.

### Different from before
In **2025**, session IDs came from Sessionize (e.g. `935459`, `933607`). Those were the keys in the YouTube map.

In **2026**, the JSON feed has no session ID field, so IDs are **generated** by hashing `title|day|time|room`:

```419:427:index.html
    function csp_sessionId(session) {
      const key = `${session.title}|${session.day}|${session.time}|${session.room}`;
      let hash = 0;
      for (let i = 0; i < key.length; i++) {
        hash = ((hash << 5) - hash) + key.charCodeAt(i);
        hash |= 0;
      }
      return String(Math.abs(hash));
    }
```

So a 2026 session might get an ID like `586234684`, not a Sessionize-style `935459`. **You can't reuse the 2025 YouTube link list** in the 2026 app — the IDs won't match.

### When you add 2026 YouTube links
Use the **hash IDs** from the 2026 app, e.g.:

```
586234684	https://www.youtube.com/watch?v=...
383038620	https://www.youtube.com/watch?v=...
```

You can find them via the track filter debug logging (`session.id` is already logged in `csp_filterByTrack`), or by inspecting `csp_sessions` in the browser console after load.

If you'd like, I can add a small helper (e.g. a "Copy session IDs" button or a script to dump `id → title` for building that mapping file).

