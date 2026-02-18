# ARS Image Archive — Development Plan

## Current State (Feb 18, 2026)

Single-page archive app (`index.html`) with 21 research entries. Grid/list views, detail modal with notes, sources panel with Chicago citations, HQ page with abstract + rich-text notes, upload form. All data client-side with localStorage persistence.

### Recently Completed
- **Connections**: Bidirectional entry linking via "Link +" dropdown in detail view, stored in localStorage
- **Sort button active state**: Orange (#FF5F00) instead of black
- **Entry count**: Total shown next to Type sort button
- **Sources formatting**: Proper Chicago citations with hanging indent, line breaks before URLs, clickable author names (orange on hover → navigate to detail)
- **Cursor**: Restored to browser default

---

## Planned Features

### 1. Google Research Assistant (New Page View)

A chat-based research interface powered by the Google Gemini API, accessible as a new panel within the app.

**UI/UX:**
- New "Research" nav item in the header bar
- Full-panel view (same pattern as HQ, Sources, Upload)
- Chat interface: user messages styled in orange, API responses in grey
- Times New Roman typography, bordered cells, consistent with the archive aesthetic
- Conversation history persisted in localStorage

**Architecture:**
- Client-side only — API key stored in localStorage via a settings input at the top of the panel
- Uses Google Generative Language API (`generativelanguage.googleapis.com`)
- Model: Gemini Pro (free tier available)
- Context injection: can pass current archive entry data as system context for research-aware responses

**Implementation Steps:**
1. Add "Research" button to header nav
2. Create `researchPanel` full-panel with panel-bar, settings row (API key input), chat history area, input row
3. Build message rendering (orange = user, grey = API) with timestamps
4. Implement `sendMessage()` — constructs prompt, calls Gemini API via fetch, streams/renders response
5. Add context mode: toggle to include current archive data in prompts ("Research this entry further")
6. localStorage persistence for chat history and API key
7. Error handling for rate limits, invalid keys, network failures

**API Integration Detail:**
```
POST https://generativelanguage.googleapis.com/v1beta/models/gemini-pro:generateContent
Headers: Content-Type: application/json
Query: ?key={API_KEY}
Body: { contents: [{ parts: [{ text: "..." }] }] }
```

**Stretch:**
- Google Custom Search API integration for finding real source URLs
- Export chat as notes to an entry
- "Ask about this entry" button in detail view that pre-populates context

---

### 2. Data & Architecture Improvements

**Backend migration (optional future):**
- Current: everything in a single HTML file with inline JS/CSS and hardcoded data
- Consider: splitting into `style.css`, `app.js`, `data.json` for maintainability
- Consider: simple JSON server or Firebase for shared/persistent data beyond localStorage

**Export/Import:**
- Export archive data + notes + connections as JSON backup
- Import from JSON to restore state

---

### 3. UI Refinements (Backlog)

- [ ] Mobile-responsive connection dropdown (currently positioned absolute, may overflow on small screens)
- [ ] Search within connection dropdown for large archives
- [ ] Bulk connection management view
- [ ] Note search/filter across all entries
- [ ] Keyboard shortcuts reference (beyond existing Esc/Arrow)
- [ ] Print-friendly stylesheet for Sources panel
- [ ] Image lazy loading with intersection observer (replace onload approach)

---

## Technical Notes

- All state in `localStorage` under keys: `arsArchiveImages`, `arsUserEntries`, `arsArchiveNotes`, `arsHqNotes`, `arsArchiveConnections`
- Connections are bidirectional: linking A→B automatically creates B→A
- Chicago citations use hanging indent via `text-indent: -2em; padding-left: 2em`
- Sort state tracked in `currentSort`, view state in `currentCols`
- Entry IDs 1–21 are base entries; user-uploaded entries get `max(id)+1`
