---
name: list-building-orchestrator
description: >
  T2 Email List Building Team vezető orchestrátora. Létrehozza a lead magnetet
  (PDF / mini-course / survey), felállítja a GHL landing page-et és automationt,
  visszaadja az opt-in URL-t T1-nek, és átadja a campaign_brief-et T3-nak.
  Trigger phrases: "lead magnet", "opt-in", "landing page", "funnel setup",
  "list building", "feliratkozási oldal", "lead magnet kész legyen",
  "állítsuk fel a funnelt", "GHL setup", "email lista", "új lead magnet",
  "mini kurzus lead magnet", "survey lead magnet", "PDF lead magnet".
---

# T2 List Building Orchestrator

**Role:** Email List Building Team pipeline vezető.
**Alapelv:** Egyetlen output számít igazán: egy élő opt-in URL, ami átmegy T1-nek és T3-nak.
**Token-stratégia:** Egyetlen state fájlt olvas. Az Avatar Cardot és a Style Guide-ot
csak a copy és design lépések töltik be — semmilyen más lépésnél nem szükséges.
**Brand kánon:** BRAND-GUIDE.md, gép-értékek tokens.css (git@github.com:YogiZoli/design-system.git,
helyi: C:/Users/gdp/projects/design-system). Ez felülírja a Drive-ról betöltött style guide-ot,
ha eltér. v2 gyors-értékek: Open Sans font (base 18px), primary CTA green `#0E8542` fehér
szöveggel, secondary azure `#1663C6`, brand azure `#1E7CE8`, háttér `#F6F8FC`, surface `#FFFFFF`,
body `#1E2733`, border `#E0E6F0`, PDF H1 `#144F9F`. Minden gomb tömör szín, fehér szöveg, min 4.5:1.
**Jelenlegi fázis:** PDF lead magnet (Phase 1). Mini-course és Survey = Phase 2/3.

---

## PIPELINE MAP

```
START
  │
  ▼
[00] Olvasd: _ListBuilding_State.md + playlist_brief.json
  │
  ▼
[01] LM Strategist — Mit és miért?        ← GATE 1: Concept jóváhagyás
  │  (típus: pdf / mini_course / survey)
  │  (téma, cím, fő ígéret, célszemély)
  ▼
[02] LM Copy — Tartalom megírása          ← GATE 2: Outline jóváhagyás
  │  PDF: fejezetek + szöveg outline
  │  Mini-course: modul struktúra + script vázlat [FUTURE]
  │  Survey: kérdések + bucket logika [FUTURE]
  ▼
[03] LM Design — Canva brief + cover      ← GATE 3: Design brief jóváhagyás
  │  PDF cover + belső oldalak vázlata
  │  Mini-course: landing page design brief [FUTURE]
  │  Survey: opt-in oldal design brief [FUTURE]
  ▼
[04] GHL Setup — Landing page + automation ← GATE 4: Live URL ellenőrzés
  │  Landing page létrehozása GHL-ben
  │  Lead magnet feltöltés / linkelés
  │  Automation workflow + tagelés
  │  Delivery email
  ▼
[05] URL Output — Handoff T1-nek + T3-nak ← GATE 5: Confirm és zárás
  │  optin_registry.json írás (T1 olvassa)
  │  campaign_brief.json írás (T3 olvassa)
  │  _CMO_State.md frissítés
  ▼
KÉSZ — az opt-in rendszer él
```

---

## STEP 0 — STARTUP: State + Input olvasás

### 0a. T2 state fájl
```
search_files(query: "title = '_ListBuilding_State'")
→ read_file_content(fileId)
```

Ha nem létezik: hozd létre (sablon lent), folytass.

### 0b. Input olvasás — playlist_brief.json
```
search_files(query: "title = 'playlist_brief' and parents in ['[T1_channel_folder_ID]']")
→ read_file_content(fileId)
```

**Elvárt mezők:**
```
playlist_id        → egyedi kulcs, amit minden output fájlban használunk
playlist_topic     → plain language téma
primary_keyword    → T1 SEO agent outputja
avatar_pain_point  → 2-3 mondat max
script_summary     → 3 mondat max (NEM a teljes script)
video_count        → tervezett videószám
```

Ha a fájl nem létezik: kérdezd meg a CMO-t / Zoltant hogy adja meg a témát kézzel,
és hozd létre a playlist_brief.json-t az elhangzottak alapján.

### 0c. Dashboard Artifact létrehozása
Induláskor azonnal `mcp__cowork__create_artifact` (HTML sablon lent).
Lépések: mind `pending`. Frissítsd minden gate után.

**`_ListBuilding_State.md` formátum:**
```markdown
# List Building State
Last Updated: [YYYY-MM-DD]

## Active Lead Magnets
- [playlist_id] | Type: [pdf/mini_course/survey] | Status: [BUILDING/LIVE]
  Title: [lead magnet cím]
  Opt-in URL: [URL vagy PENDING]
  GHL Tag: [tag neve]
  GHL Workflow ID: [ID vagy PENDING]

## Completed
- [playlist_id] | [cím] | URL: [url] | Created: [dátum]
```

---

## STEP 1 — LM Strategist: Concept meghatározás

**Cél:** Eldönteni mit csinálunk, és miért fogja az avatar letölteni.

**Input (playlist_brief.json-ből):**
- `playlist_topic` → mihez kapcsolódjon a lead magnet
- `avatar_pain_point` → mi az azonnali probléma amit megold
- `primary_keyword` → SEO kontextus (milyen szavakat keres az avatar)

**A Strategist eldönti:**

```
1. Lead magnet TÍPUS (jelenleg csak PDF aktív):
   ✅ PDF              → Phase 1 — most
   🔜 Mini-course      → Phase 2 — jövőben
   🔜 Survey (ASK)     → Phase 2 — jövőben

2. Lead magnet CÍM — 3 variáció:
   Formátum: "[Szám] [Eredmény] [Időkeret/Módszer]"
   Pl: "7 AI Agent amit ma bevezethetsz a vállalkozásodba"

3. Fő ígéret (1 mondat):
   Mit kap az avatar, amit a letöltés után azonnal alkalmazni tud?

4. Tartalom struktúra vázlat:
   PDF:          3-7 fejezet, minden fejezethez 1 mondat összefoglaló
   Mini-course:  3-6 modul, minden modul 3-5 perces videó [FUTURE]
   Survey:       5-7 kérdés + 3 bucket definíció [FUTURE]

5. GHL tag neve:
   Formátum: lm-[playlist-slug]
   Pl: lm-ai-agents-2026
```

**Avatar Card betöltés — csak itt, csak ha kell:**
```
search_files(query: "title contains 'AvatarCard' and parents in ['[00_Foundation folder ID]']")
→ read_file_content(fileId)
```
Ha az avatar_pain_point a playlist_brief-ben elegendő kontextus: NE töltsd be az Avatar Cardot.

**Output Zoltannak (GATE 1):**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⏸ GATE 1 — Lead Magnet Concept
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Típus: PDF (Phase 1)

Cím opciók:
  A) [cím variáció 1]
  B) [cím variáció 2]
  C) [cím variáció 3]

Fő ígéret: [1 mondat]

Tartalom vázlat:
  1. [fejezet]
  2. [fejezet]
  3. [fejezet]
  ...

GHL tag: lm-[slug]

→ Válassz: A / B / C vagy "cím: [saját verzió]"
→ Módosítás: "fejezetek: [változtatás]"
→ Jóváhagyás: "ok" / "mehet"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Várd meg a jóváhagyást. NE lépj tovább.

---

## STEP 2 — LM Copy: Tartalom megírása

**Input:** Jóváhagyott concept (Gate 1 output)

### PDF Lead Magnet (Phase 1 — aktív)

**Betöltés (copy írás előtt):**
```
search_files(query: "title contains 'StyleGuide' and parents in ['[00_Foundation folder ID]']")
→ read_file_content(fileId)
```
Csak a tone of voice és brand voice részt olvasd — ne töltsd be az egész fájlt ha nem szükséges.

**A copy agent megírja:**

```
1. OPT-IN OLDAL COPY (rövid):
   Headline: [erős, benefit-driven]
   Subheadline: [pontosítja az ígéretet]
   Bullet points (3-5): mit kap az avatar konkrétan
   CTA gomb szöveg: [cselekvésre hívó]
   Privacy note: [rövid, bizalomépítő]

2. PDF TARTALOM (teljes):
   Minden fejezethez:
   - Fejezet cím
   - Bevezető mondat (hook)
   - Főbb pontok (3-5 bullet vagy folyószöveg)
   - Actionable takeaway / következő lépés

3. DELIVERY EMAIL:
   Tárgy: [2 variáció]
   Body: köszönet + link + mit várjon ezután (1 mondatos előzetes a következő emailről)

4. THANK YOU PAGE COPY (rövid):
   Headline: megerősítés
   1-2 mondat: mi jön ezután
   Opcionális upsell hint: [ha van, vagy SKIP]
```

**Drive-ra mentés:**
```
create_file(
  name: "LM_Copy_[playlist_id]_[YYYY-MM-DD]",
  content: [teljes copy],
  parents: ["[09_Lead_Magnets folder ID]"]
)
```

**Output Zoltannak (GATE 2):**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⏸ GATE 2 — Copy jóváhagyás
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Az opt-in oldal copy és a PDF teljes tartalma Drive-on van.
A delivery email 2 tárgysoros variációval kész.

→ "ok" / "jóváhagyom" = mehetünk a designra
→ "módosítsd: [rész] → [változtatás]"
→ "újra a [fejezet/szekció]"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

### Mini-Course Lead Magnet — PHASE 2 (jövőben)

Trigger: ha Gate 1-ben `mini_course` lett választva.

```
[PHASE 2 — NEM AKTÍV MÉG]

A mini-course-producer agent:
- Modul struktúra megtervezése (3-6 modul × 3-5 perc)
- Minden modulhoz Ownership Method script (Hook, Outcome, Testimonial)
- HeyGen production brief minden modulhoz (9:16 vagy 16:9)
- Progress bar landing page design brief
  (tetején progress bar, email-gated, minden modul után "következő" gomb)
- Email-gated hozzáférés GHL-ben:
  Opt-in → kapja a linket → landing page-en progress bar
  Minden modul befejezése után: GHL tag frissítés (lm-mc-[modul-slug])
- Befejezési email: "Gratulálok, elvégezted a kurzust" + következő lépés (CTA)

Aktiválás: Zoltan kéri → Phase 2 sprint indítása
```

---

### Survey Lead Magnet — PHASE 2 (jövőben)

Trigger: ha Gate 1-ben `survey` lett választva.

```
[PHASE 2 — NEM AKTÍV MÉG]

A survey-builder agent (Ryan Levesque ASK Method):
- 5-7 kérdés megfogalmazása (szegmentációs logikával)
- 3 bucket definíció (melyik válasz → melyik avatarba kerül)
- GHL survey oldal + conditional logic beállítása
- Tag map:
  bucket-A: [tag neve] → [email szekvencia neve]
  bucket-B: [tag neve] → [email szekvencia neve]
  bucket-C: [tag neve] → [email szekvencia neve]
- Minden buckethez külön thank you page copy
- campaign_brief.json-be kerül: az összes bucket tag + workflow ID

Aktiválás: Zoltan kéri → Phase 2 sprint indítása
Az ask_bucket → T2 visszacsatolás: GHL natívan kezeli,
nem kell külön integráció — T3 olvassa a GHL tag performance-ot.
```

---

## STEP 3 — LM Design: Canva brief + design anyagok

**Input:** Jóváhagyott copy (Gate 2 output)

**A design agent létrehozza:**

```
1. PDF COVER BRIEF (Canva-ba másolható):
   Méret: A4 (210×297mm) vagy 8.5×11in
   Cím: [jóváhagyott cím]
   Szerző: Zoltan Gero
   Vizuális irány: brand v2 (BRAND-GUIDE.md): PDF cover háttér `#0E2038` vagy `#FFFFFF`,
     H1 `#144F9F`, CTA blokk green `#0E8542` fehér szöveggel, azure `#1663C6` vonalak + [1 fő vizuális elem]
   Betűtípus: Open Sans (H1 700, body 400), lásd tokens.css
   Hangulatszavak: [3 jelző]

2. PDF BELSŐ OLDALAK SABLON:
   Fejléc: logó + oldalszám
   Fejezet nyitó oldal: [design brief]
   Tartalom oldal: [layout brief]
   Lábléc: [cta / web]

3. OPT-IN LANDING PAGE HERO SZEKCIÓ BRIEF:
   Headline vizuálisan kiemelve
   Lead magnet mockup (cover kép)
   CTA gomb: green `#0E8542` tömör, fehér szöveg, min 4.5:1, [méret, szöveg]
   Background: `#F6F8FC` (brand v2, BRAND-GUIDE.md)

4. THANK YOU PAGE BRIEF:
   Egyszerű, megnyugtató design
   "Nézz a postaládádba" üzenet vizualitása
```

**Style Guide betöltés (ha még nem töltődött be Step 2-ben):**
Elsődleges forrás: tokens.css + BRAND-GUIDE.md (git@github.com:YogiZoli/design-system.git).
A Drive style guide csak kiegészítés, ütközésnél a repo kánon nyer. Csak a color palette és
typography részt — max 30 sor.

**Output Zoltannak (GATE 3):**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⏸ GATE 3 — Design brief jóváhagyás
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
PDF cover brief és opt-in oldal design brief kész.
Drive-ra mentve: LM_Design_[playlist_id]

→ Nyisd meg Canva-ban → készítsd el a cover-t
→ Mikor kész: "design ok" / "canva kész"
→ Módosítás: "változtasd a [rész]-t"
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

Várd meg a jóváhagyást. A GHL setup-ot NE indítsd el a design jóváhagyása előtt.

---

## STEP 4 — GHL Setup: Landing page + Automation

**Input:** Jóváhagyott copy + design (Gate 3 output)

**A GHL setup agent elvégzi az alábbi lépéseket GHL MCP-n keresztül:**

### 4a. Landing Page létrehozása
```
ghl_create_page vagy mcp__gohighlevel__create_... (releváns GHL tool)

Tartalma:
- Opt-in oldal copy (Step 2-ből)
- Lead magnet cover image (ha Canva-ból megvan)
- Opt-in form: Keresztnév + Email cím
- CTA gomb
- Privacy note
- Thank you page URL beállítása (következő lépés)
```

### 4b. Lead Magnet feltöltés
```
PDF esetén:
- PDF feltöltés GHL Media Library-ba
- Delivery email-be a letöltési link beágyazása

Mini-course esetén [FUTURE]:
- Privát landing page URL beállítása a delivery email-be
- Progress bar oldal linkelése

Survey esetén [FUTURE]:
- Survey URL beágyazása a thank you page-be
```

### 4c. GHL Automation Workflow
```
Trigger: Form submission (opt-in)

Lépések:
1. Tag hozzáadása: lm-[playlist-slug]  (minden típusnál)
2. Tag hozzáadása: source-youtube      (traffic forrás)
3. Delivery email küldése (azonnal)
4. Wait: 1 óra
5. Belépés a Campaign szekvenciába (T3 workflow ID — ha már él)
   Ha T3 még nincs: wait tag alkalmazása → campaign-pending-[playlist-slug]

Survey esetén [FUTURE]:
1. Form submission → survey oldal megjelenítése
2. Survey válaszok → conditional logic → bucket tag alkalmazása
3. Bucket-specifikus delivery email
4. Bucket-specifikus campaign belépési pont
```

### 4d. URL ellenőrzés
Miután a GHL page él:
- Nyisd meg a page URL-t (GHL MCP-vel lekérdezve)
- Ellenőrizd: látszik-e a form, működik-e a submit

**Output Zoltannak (GATE 4):**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
⏸ GATE 4 — Landing Page live ellenőrzés
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
GHL landing page létrehozva.
Automation workflow aktív.
Tag: lm-[slug]

Opt-in URL: [URL]

→ Nyisd meg a böngészőben, ellenőrizd
→ "ok" / "él" = mehetünk
→ "javítsd: [mit]" = módosítás
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## STEP 5 — URL Output: Handoff T1-nek és T3-nak

**Ez a lépés automatikus Gate 4 jóváhagyása után. Nincs külön gate.**

### 5a. optin_registry.json írása (T1-nek)
```
Fájl neve: optin_registry
Helye: [Channel Drive folder] → kereshető playlist_id alapján

Tartalom (append — ne írj felül meglévő bejegyzéseket):
{
  "playlist_id": "[playlist_id]",
  "opt_in_url": "[GHL landing page URL]",
  "lead_magnet_type": "pdf",
  "lead_magnet_title": "[cím]",
  "ghl_entry_tag": "lm-[slug]",
  "landing_page_ghl_id": "[GHL page ID]",
  "confirmed_date": "[YYYY-MM-DD]"
}
```

T1 video-production-orchestrator ezt olvassa, és automatikusan beilleszti
a YouTube videó leírásokba. Nincs szükség manuális koordinációra.

### 5b. campaign_brief.json írása (T3-nak)
```
Fájl neve: campaign_brief_[playlist_id]
Helye: [Shared Drive — T3 is eléri]

Tartalom:
{
  "playlist_id": "[playlist_id]",
  "lead_magnet_title": "[cím]",
  "lead_magnet_topic": "[téma 1-2 mondatban]",
  "lead_magnet_type": "pdf",
  "ghl_entry_tag": "lm-[slug]",
  "entry_workflow_id": "[GHL workflow ID]",
  "avatar_segment": "[melyik avatar szegmens — playlist_brief-ből]",
  "recommended_campaign": "survey_then_plf",
  "ask_bucket": null
}
```

`ask_bucket` null marad amíg a Survey Lead Magnet Phase 2-ben el nem készül.

### 5c. _CMO_State.md frissítése
```
Active Lead Magnets szekcióban:
[playlist_id] | Type: pdf | Status: LIVE
  Title: [cím]
  Opt-in URL: [URL]
  GHL Tag: lm-[slug]
  GHL Workflow ID: [ID]

Next Recommended Action:
"T3 Campaign Team megkapja a campaign_brief-et — survey + PLF launch indítható."
```

**Zárás üzenet:**
```
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ EMAIL LIST BUILDING KÉSZ
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Lead magnet: [cím] (PDF)
Opt-in URL: [URL] ← automatikusan bekerül a YouTube leírásokba

A YouTube csapat megkapja az URL-t.
A kampány csapat megkapja a campaign brief-et.

Következő lépés a rendszerben:
→ [N] Kampány indítása (T3 Campaign Orchestrator)
→ [V] Következő YouTube videó (T1 Video Production)
→ [K] Kész, semmi más most
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## DASHBOARD ARTIFACT (HTML)

```html
<!DOCTYPE html><html><head><meta charset="UTF-8"><style>
body{font-family:-apple-system,BlinkMacSystemFont,'Segoe UI',sans-serif;
background:#0a0a0a;color:#e0e0e0;padding:24px;max-width:500px;margin:0 auto}
h2{color:#fff;font-size:17px;margin-bottom:4px}
.sub{color:#555;font-size:12px;margin-bottom:18px}
.step{display:flex;align-items:flex-start;gap:12px;padding:10px 14px;
border-radius:8px;margin-bottom:6px;background:#161616}
.step.done{background:#0a2015;border-left:3px solid #22c55e}
.step.active{background:#12122a;border-left:3px solid #818cf8}
.step.waiting{background:#1f1500;border-left:3px solid #f59e0b}
.step.pending{opacity:.35;border-left:3px solid #333}
.step.skipped{opacity:.5;border-left:3px solid #444;font-style:italic}
.icon{font-size:17px;width:22px;flex-shrink:0;padding-top:1px}
.info{flex:1}
.label{font-size:13px;font-weight:500;margin-bottom:2px}
.detail{font-size:11px;color:#666}
.step.done .detail{color:#22c55e}
.step.active .detail{color:#818cf8}
.step.waiting .detail{color:#f59e0b}
.gate{font-size:10px;color:#333;margin-left:34px;margin-bottom:7px;margin-top:-2px}
.url-box{background:#0d1f14;border:1px solid #1a3d24;border-radius:8px;
padding:10px 14px;margin-top:14px}
.url-label{font-size:10px;color:#555;text-transform:uppercase;
letter-spacing:.08em;margin-bottom:4px}
.url-val{font-size:12px;color:#22c55e;word-break:break-all}
</style></head><body>
<h2>📧 List Building Pipeline</h2>
<div class="sub"><!-- PLAYLIST_TOPIC --> · <!-- DATE --></div>

<div class="step <!-- S01 -->">
  <span class="icon">🧠</span>
  <div class="info">
    <div class="label">01 — Lead Magnet Concept</div>
    <div class="detail"><!-- S01D --></div>
  </div>
</div>
<div class="gate">▸ Gate 1: Cím + struktúra jóváhagyás</div>

<div class="step <!-- S02 -->">
  <span class="icon">✍️</span>
  <div class="info">
    <div class="label">02 — Copy megírása</div>
    <div class="detail"><!-- S02D --></div>
  </div>
</div>
<div class="gate">▸ Gate 2: Copy jóváhagyás</div>

<div class="step <!-- S03 -->">
  <span class="icon">🎨</span>
  <div class="info">
    <div class="label">03 — Design Brief (Canva)</div>
    <div class="detail"><!-- S03D --></div>
  </div>
</div>
<div class="gate">▸ Gate 3: Design jóváhagyás + Canva elkészítés</div>

<div class="step <!-- S04 -->">
  <span class="icon">⚙️</span>
  <div class="info">
    <div class="label">04 — GHL Landing Page + Automation</div>
    <div class="detail"><!-- S04D --></div>
  </div>
</div>
<div class="gate">▸ Gate 4: Élő URL ellenőrzés</div>

<div class="step <!-- S05 -->">
  <span class="icon">🔗</span>
  <div class="info">
    <div class="label">05 — Handoff T1 + T3</div>
    <div class="detail"><!-- S05D --></div>
  </div>
</div>

<!-- URL BOX — csak ha él -->
<!-- URL_BOX_PLACEHOLDER -->

</body></html>
```

**Step class-ok:** `done` / `active` / `waiting` / `pending` / `skipped`

**URL box beillesztés Gate 4 után:**
```html
<div class="url-box">
  <div class="url-label">🔗 Opt-in URL (él)</div>
  <div class="url-val">[OPT_IN_URL]</div>
</div>
```

---

## RESUME LOGIC

Ha a pipeline félbemaradt (Drive scan alapján):
```
search_files(query: "title = '_ListBuilding_State'") → olvasd a státuszt

Folytatás az utolsó befejezetlen lépéstől → "igen"
Elölről → "újrakezdem"
Adott lépés → "újra [N]"
```

---

## GLOBÁLIS PARANCSOK

| Parancs | Hatás |
|---------|-------|
| `"skip"` | Aktuális lépés átugrása, Drive output megőrzése |
| `"újra"` | Aktuális lépés újrafuttatása |
| `"újra [N]"` | Adott lépés újrafuttatása (pl. "újra 02") |
| `"hol tartunk"` | Dashboard artifact frissítés |
| `"állj meg"` | Pipeline pause, state mentés |
| `"milyen lead magnet típusok vannak"` | Típusok listája + aktuális státusz |

---

## HIBAKEZELÉS

Ha GHL step hibázik:
1. Felszínre hozni plain language-ben: "A landing page létrehozása elakadt a GHL-ben."
2. Két opció: "Próbáljuk újra" / "Csináljuk manuálisan — add meg a page URL-t"
3. Ha manuális URL-t ad Zoltan: fogadd el, folytasd Step 5-tel
4. Soha ne hagyd el az optin_registry.json írást — ez az egyetlen kritikus output

---

## FÁZIS ÖSSZEFOGLALÓ

```
Phase 1 — MOST (aktív):
  ✅ PDF Lead Magnet
  ✅ GHL Landing Page + Automation
  ✅ Delivery Email
  ✅ Thank You Page
  ✅ optin_registry.json → T1
  ✅ campaign_brief.json → T3

Phase 2 — JÖVŐBEN:
  🔜 Survey Lead Magnet (Ryan Levesque ASK Method)
      → bucket logika, GHL conditional workflow, szegmentált email szekvenciák
  🔜 Video Mini-Course Lead Magnet
      → HeyGen production brief, progress bar landing page, modul-szintű tagelés

Phase 3 — JÖVŐBEN:
  🔜 ask_bucket → T2 feedback loop
      → GHL tag performance olvasás, következő lead magnet témajavaslat
```
