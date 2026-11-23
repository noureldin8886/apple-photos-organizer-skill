# Apple Photos Organizer — a Claude skill

A [Claude](https://claude.ai) skill that analyzes a Mac's Apple Photos library and organizes it with you, step by step, through the Photos app.

## What it does

- **Read-only analysis** of your library: totals, what takes up space (videos, RAW), unorganized photos, empty or duplicate albums, wrong dates, face-recognition progress.
- **A phased plan** you approve before anything changes.
- **Album cleanup and structure**: deletes empty albums, merges duplicates (WhatsApp, Instagram…), builds folders like `Travel`, `Family`, `Photography`, `Documents & Apps`, plus useful Smart Albums.
- **Trip albums from GPS and dates**: finds every trip away from home and creates one album per trip (`YYYY-MM Place`).
- **Family trips**: splits trips by who was there, using the names *you* set in People, with a review spreadsheet before anything moves.
- **Date fixes** for drone/action-camera files with a wrong clock (e.g. DJI files dated 2000).
- **Face-recognition tips** and a progress check for new or large libraries.

## Safety rules built in

- Never deletes photos without your explicit approval.
- Never writes to the library files directly; all changes go through the Photos app.
- Never identifies people from their faces; it only uses names you have set in People.
- Recommends first, acts second.

## Requirements

- A Mac with the Apple Photos app.
- The Claude desktop app, linked to the conversation, with **computer use** turned on.
- Access to your `~/Pictures` folder (Claude asks for it).
- Changes sync to your other devices through iCloud Photos. Smart Albums stay on the Mac.

## Install

**Claude.ai / desktop app:** download this repo, zip the `apple-photos-organizer` folder, then upload it in Claude under **Customize › Skills** (or Settings › Capabilities › Skills).

**Claude Code:** copy the `apple-photos-organizer` folder into `~/.claude/skills/`.

## Use

Ask Claude something like:

> "Analyze my Photos library and give me recommendations to organize it."

## Feedback

Tried it? Please open an [issue](../../issues) with what worked, what didn't, and your macOS version. Suggestions and pull requests are welcome.

## License

MIT — see [LICENSE](LICENSE).
