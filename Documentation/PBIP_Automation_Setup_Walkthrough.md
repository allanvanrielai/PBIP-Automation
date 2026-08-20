# PBIP Automation — Setup Walkthrough

Everything needed between "PBIP saved to disk" and "ready to submit the first Claude Code prompt."

---

## 0. Already done

- ✅ Created the `PBIP Automation` folder
- ✅ Saved the Power BI report into it as a `.pbip` (Power BI Project format, not `.pbix`)
- ✅ Created a `Documentation` subfolder for project notes

Everything below builds on top of that.

---

## 1. Set up `.gitignore` — before the first commit

Power BI Desktop writes local cache and settings files alongside a PBIP that should never be committed (they're machine-specific and can bloat the repo). Create the `.gitignore` **before** running `git init` / making the first commit, not after — cleaning tracked files out of history later is a hassle worth avoiding.

At minimum, exclude:
```
.pbi/
*.pbi.bak
~$*
.vscode/
```
(Check whether your specific PBIP has additional local cache folders — Power BI Desktop version and settings can vary what gets written locally.)

---

## 2. Initialise git in the folder

```
cd "PBIP Automation"
git init
git add .
git commit -m "Initial commit: PBIP export of Stride Runs report"
```

This is the point where you get real version history — every DAX/model change Claude Code makes from here becomes a reviewable diff.

---

## 3. (Optional) Connect to a GitHub remote

Not required to get started, but worth doing for backup and history preservation:

```
git remote add origin <your-repo-url>
git push -u origin main
```

Can be done now or any time later — doesn't need to happen before Claude Code's first session.

---

## 4. Verify the PBIP structure

Confirm the folder contains what a PBIP export should have:
- `<ReportName>.pbip`
- `<ReportName>.Report/` (folder — report layout as JSON)
- `<ReportName>.SemanticModel/` (folder — model as TMDL files)

If any of these are missing, the Save As from Power BI Desktop may not have completed cleanly — worth re-exporting before going further.

---

## 5. Confirm your local Power BI modeling MCP server

You'd previously identified a local modeling MCP server as the viable path (no Fabric/Premium capacity for a remote option). Before starting:
- Confirm which specific server/tool you're using, and that it's installed
- Confirm it connects to a local Analysis Services instance (the one Power BI Desktop spins up when a file is open) — not to the PBIP files on disk directly
- Check its documentation for one important behavioural detail we flagged earlier but haven't confirmed: **does it edit the live Desktop session directly, or does it write to the on-disk TMDL files while Desktop is closed?** This determines whether Power BI Desktop should be open or closed during a Claude Code session — get this backwards and the in-memory copy and on-disk files can drift out of sync.

---

## 6. Open the PBIP in Power BI Desktop and check the outstanding data question

Before writing any DAX, resolve the open question from earlier: **does the `TCX Data` table actually include the even-numbered (recovery/walk) laps at full per-second resolution, or only the odd "work" laps** (like the Excel pipeline did)?

Quick check: filter `TCX Data` by `Lap Number` = 2 (or any even number) in a table visual and see if rows come back.

This directly determines whether the recovery-HR measure is buildable as-is, or needs a Power Query change first — worth knowing before Claude Code starts, not discovering mid-session.

---

## 7. Register the MCP server with Claude Code specifically

Separate from Claude Desktop's config. Typically either:
- A project-level `.mcp.json` in the `PBIP Automation` folder, or
- `claude mcp add <server-name> ...` run from that folder

Confirm it shows up when you list Claude Code's connected MCP servers before starting the real session.

---

## 8. Capture the project brief in `Documentation`

Worth writing down (or having me draft) a short brief covering what's already been worked out, so Claude Code doesn't start from zero:
- The recovery-HR calculation logic (last HR of an interval minus first HR of the next)
- The TCX lap-numbering convention (odd laps = work intervals 1,3,5...21; even = recovery)
- The normalization approaches discussed (HRR60 vs. average decay rate) and which one to build
- The outcome of the Step 6 check (whether even laps are loaded)
- The goal: replace/retire the Excel "Recovery" sheet once this is validated

Save this as a `.md` file in `Documentation` — happy to draft it if useful.

---

## 9. Ready to go

At this point you should have:
- [ ] `.gitignore` in place, git initialised, first commit made
- [ ] (Optional) GitHub remote connected
- [ ] PBIP structure verified
- [ ] Local MCP server confirmed installed, and its live-vs-file editing behaviour understood
- [ ] Power BI Desktop open with the PBIP loaded, and the even-lap data question resolved
- [ ] MCP server registered with Claude Code (not just Desktop)
- [ ] Project brief written into `Documentation`

Once those are checked off, open the `PBIP Automation` folder in VS Code, launch Claude Code, and submit the first prompt — referencing the brief in `Documentation` rather than re-explaining the context from scratch.
