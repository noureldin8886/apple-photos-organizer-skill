---
name: "apple-photos-organizer"
description: "Analyze and organize a Mac Apple Photos library: read-only stats, cleanup recommendations, folder structure, trip albums from GPS/dates, family-trip split by People names, and date fixes via the Photos app."
---

# Apple Photos Organizer

Use this when someone asks to analyze, clean up or organize their Apple Photos library on a Mac. Reply in the user's language. Nothing is deleted without explicit approval.

## Golden rules
- **Read the database, change things through the app.** Analyze a *copy* of `Photos.sqlite`; never write to the library files (it corrupts the library). All changes go through the Photos app UI.
- **Recommend first, act second.** Present findings and a phased plan; wait for the go-ahead.
- **No deletion of photos** without explicit approval per batch. Deleting/merging *albums* never deletes photos — say so.
- **Don't identify people from faces.** Only use names the user has set in People (read from the database). Never guess who a named or unnamed person is: ask the user which People names are their family. If names are too sparse, ask them which trips/albums qualify instead.
- Changes sync to all iCloud devices; Smart Albums are Mac-only. Tell the user.
- Keep the Photos window open and not minimized while working.

## 1. Access & read-only analysis
1. Request access to `~/Pictures` (the library is `Photos Library.photoslibrary`).
2. Copy `database/Photos.sqlite` (+ `-wal`) to a scratch dir and query with Python `sqlite3`. Delete the copy afterwards.
3. Useful queries (dates are seconds since 2001-01-01; add 978307200 for Unix time):
   - `ZASSET` (filter `ZTRASHEDSTATE=0`): counts by `ZKIND` (0 photo, 1 video), `ZUNIFORMTYPEIDENTIFIER`, year, `ZHIDDEN`, `ZFAVORITE`, `ZISDETECTEDSCREENSHOT`, GPS present (`ZLATITUDE` valid and not 0,0), `ZDURATION`.
   - Join `ZADDITIONALASSETATTRIBUTES` (`ZASSET`) for `ZORIGINALFILESIZE`, `ZORIGINALFILENAME`, `ZIMPORTEDBYBUNDLEIDENTIFIER` → size by type/year/source, largest videos, exact duplicates (same name+size).
   - Rapid sequences: photos within ≤3 s of the previous one.
   - `ZGENERICALBUM` (`ZKIND` 2 = album, 4000 = folder, 1507 = smart album; `ZPARENTFOLDER`), membership via `Z_<n>ASSETS` (column names vary by version — inspect with `pragma table_info`).
   - `ZPERSON` / `ZDETECTEDFACE` (`ZPERSONFORFACE`, `ZASSETFORFACE`): how many faces are detected, clustered (`ZPERSONFORFACE` not null) and named. `ZADDITIONALASSETATTRIBUTES.ZFACEANALYSISVERSION` > 0 = face analysis done for that asset. A `ZPERSON` row with `ZFACECOUNT=0` and a `ZMERGETARGETPERSON` is an old merged record, hidden in the app: never call it a duplicate to merge.
   - Library age: if the database is only weeks old (new Mac / new library), face analysis is usually still catching up — say so before blaming recognition quality.
   - Suspicious dates: assets dated year 2000 or earlier (camera clock unset, common with DJI/GoPro).
4. Report: totals, size breakdown, what eats space (video/RAW), unorganized share, empty/duplicate albums, unnamed faces, bad dates, big events without albums.

## 2. Recommended phased plan
1. **Album cleanup** — delete empty albums, merge duplicate app albums (WhatsApp, Instagram…). No photo deletion.
2. **Structure** — top-level folders, e.g. `Family`, `Travel`, `Photography` (DJI/GoPro/Lightroom), `Documents & Apps`. Smart Albums: `RAW Photos` (Photo is RAW), `No Location` (Photo is not tagged with GPS). Photos has no file-size/duration condition — say so.
3. **Data fixes** — wrong dates (see §4).
4. **Space** — list big videos / old RAW / burst sequences for the user to decide. Deletions go to Recently Deleted (30 days).

## 3. Trip albums
1. Find the home location = densest 0.5° GPS cell.
2. Days with photos >150 km from home; merge into trips if gaps ≤2 days; ignore trips with <15 items. Use `ZMOMENT.ZTITLE` for place names.
3. Show the user a trip list grouped by destination (abroad vs. domestic) and ask how to split (one album per trip is usually right; ask about long stays and recurring work trips).
4. Name albums `YYYY-MM Place` and put them in the `Travel` folder.
5. Albums are date-based: everything shot on those days goes in (including home-airport shots). Mention it.
6. **Received photos get the save date.** WhatsApp/Messages photos lose their original date and GPS, so photos the user received and saved during a trip land in that trip's album (and family trips the user wasn't on don't show up as trips at all). `ZIMPORTEDBYBUNDLEIDENTIFIER` tells camera (`com.apple.camera`) from received (`net.whatsapp.WhatsApp`, `com.apple.MobileSMS`, `com.apple.sharingd`) for newer photos; older ones are often null. Confirm the user appears (their People name) in each trip album before treating it as their trip.

## 3b. Splitting trips by who was there (e.g. "Travel with Family")
1. Prerequisite: the people are named in People and face analysis is mostly done (check the counts above). If not, give the People tips (§6), offer a check-in in a few days, and stop.
2. Ask the user which People names count (e.g. themselves, spouse, children). Use their own names for people in replies.
3. Per trip album, count family-member photos **only if** the photo has GPS within ~200 km of the album's median location, **or** the user is in the same photo. This drops photos received from home during the trip.
4. Put the result in a review spreadsheet (Sheets artifact if available): album, proposed status (Move / Your call / Keep), who appears, on-site family photos, photos with the user, current folder, note. Flag ambiguous rows: couple-only trips, and trips where family appears a lot but the user is in almost none of the photos.
5. Let the user set the status column, re-read the sheet, move only what they marked, then update the sheet to "Moved".

## 4. Fixing wrong dates (DJI etc.)
- DJI Mimo filenames embed a 13-digit epoch-ms = **transfer time, not capture time**. Group files by that, propose the transfer day as an approximate date, keep relative order, and get approval first.
- Apply with Image › Adjust Date and Time on the whole group (first item's time shifts the rest).

## 5. Driving the Photos app (computer use)
- Background `app_*` tools work for menus (`File › New Folder`, `New Smart Album…`) and renaming via the sidebar text field. **Right-click menus and drag-and-drop need full-screen control** — ask once, explain why, release when done.
- **Album from a date range** (reliable pattern): keep one temp Smart Album `Trip dates (temp)` with *Date Captured is in the range*; for each trip: Edit Smart Album → set dates (date fields are day/month/year segments; click the end field twice to focus) → check the match count → open it → ⌘A in the grid → File › New Album with Selection → confirm the "apply to N photos" dialog when it appears (shown for larger selections) → ⌘A + type name + Return → drag the new album onto the target folder in the sidebar. Delete the temp Smart Album at the end.
- Moving an album: the most reliable way is to open the source folder, scroll the sidebar so the target folder is visible, and drag each album **tile from the grid** onto the target folder in the sidebar (one at a time; ⌘-click multi-select doesn't drag). Moving from the end of the grid backwards keeps earlier tile positions stable. Dropping onto the open folder's grid does nothing. Sidebar-to-sidebar drag also works when both are visible; if not, right-click › *Move Album Out Of "…"* first, then drag at the top level. Background (`app_*`) drags don't register: use full-screen control.
- Creating a folder: select *All Albums* first so the folder lands at the top level, then File › New Folder and type the name into the sidebar field.
- Each computer_batch has a ~50 s total wait limit; split long sequences.
- Merging albums: select all in the source → drag onto the target album → confirm → delete the source album.
- Deleting an album: right-click › Delete Album → confirm (photos stay).
- Watch for: a click landing on a photo opens Edit mode (exit with Done, no changes); an accidental duplicate album (delete it); a rename field left open (Escape). Check the access grant (it expires after ~30 min idle).
- **Verify from the database** at the end: list albums with parent folder and counts; confirm the trashed count didn't grow.

## 6. People / face recognition tips (advice only)
- Unrecognized old photos usually mean too few confirmed samples or appearance changes.
- Steps: in People open the person → Review/Confirm Additional Photos; tag a few old photos from each period via (i); check the unnamed people at the bottom of People and merge only after the user confirms they're the same person; let the Mac analyze overnight while charging with Photos closed, for several nights on a large or new library.
- Offer a read-only re-check in a few days (scheduled follow-up if available) and report detected vs clustered vs named faces, and photos linked to the user. Recognition can jump from a few hundred to tens of thousands of named faces once analysis finishes.

## Wrap-up
Summarize what changed (albums created/moved/deleted, dates fixed), what was left undone and why, and offer the next phase.