# CLAUDE.md

Krowi's Tutorials (`Krowi_Tutorials`) is a World of Warcraft library in Lua 5.1: a paged help window with an image and text per page, page navigation, a glow that can point at another frame, and per-page "viewed" marks kept in the consumer's saved variables. Addons embed it as a `Libs/Krowi_Tutorials` git submodule. The workspace-wide rules (tooling traps, the game loop, taint, git, release safety) load from `../../CLAUDE.md` and `Krowi_ClaudeKit/WowAddons.md`; this file adds only what is specific to this repo.

There is no build, lint or test step: the game, through a consumer, is the test harness. The `krowi` plugin (enabled in `.claude/settings.json`) syntax-checks every edited `.lua` with Lua 5.1 and guards the known tooling traps. The repo is public, but `.claude/settings.json` points at the private kit by absolute path, so the plugin loads only on Krowi's machine; that is intended.

## Clients and loading

- One TOC, `Krowi_Tutorials.toc`: `## Interface: 120000, 110207, 50503, 20505, 11508` (Retail 12.0.0, 11.2.7, which no longer exists, Mists 5.5.3, Anniversary 2.5.5, Era 1.15.8). It counts only for a standalone install, the README badges and the upload game versions; an embedded copy loads through the consumer's XML.
- The TOC loads `Krowi_Tutorials.xml`, which loads the only code file, `Krowi_Tutorials.lua`. No XML templates.
- Needs `KROWI_LIBMAN` and LibStub from Krowi_Util loaded first (the consumer's `Libs/Files.xml` includes Krowi_Util earlier); the TOC declares no dependency.
- One code path for every client: `ButtonFrameTemplate` with `PortraitContainer`, `TitleText` and `Inset` exists in every client's FrameXML (`Blizzard_SharedXML/Mainline/` and `Classic/SharedUIPanelTemplates.xml`). The only branch is the ElvUI skin, applied when ElvUI is loaded with its Blizzard "tutorials" skin on (`Krowi_Tutorials.lua:280-301`).

## Versioning

- One registration: `Krowi_Tutorials.lua:9` `KROWI_LIBMAN:NewLibrary('Krowi_Tutorials_3', 1, { SetCurrent = true })`, which calls `LibStub:NewLibrary('Krowi_Tutorials_3', 1)` (`Krowi_Util/LibMan.lua:22`). Any change to `Krowi_Tutorials.lua` needs that minor bumped; it has been 1 since 3.1.
- Tutorial frames use the library table as their metatable (`lib.__index = lib`, `:416`, `:423`), so an upgrade's methods reach frames an older copy built. File locals (`default`, `numFrames`, `locked`) start over in the newer copy, and `numFrames` starting over reuses the global names `Krowi_Tutorial<n>`.
- `_3` is the major. 3.1 renamed `Krowi_Tutorials-3.0` to `Krowi_Tutorials_3`, which LibStub treats as a separate library.

## Consumers

Only Krowi_AchievementFilter (`Libs/Krowi_Tutorials`, at 3.1); no third-party copy is installed on Krowi's clients. A change ships by releasing the library and bumping that submodule. Its use, through `addon.TutorialsLib = KROWI_LIBMAN:GetLibrary('Krowi_Tutorials_3')` (`Libs/Libs.lua:12`):

- `Tutorials.lua:390-394`: `New("FeaturesTutorial", KrowiAF_SavedData)`, `SetFrameTitle`, `SetPages`, `SetImageMargin(10)`, `SetTextMargin{10, 0, 10, 20}`, with pages that set `Image`, `ImageSize`, `SubTitle`, `Text` and `OnShow`. It stores the frame on the shared library table (`tutorials.FeaturesTutorial`).
- `.Pages`, `ShowTutorial(i)` and `HasUnviewedPages()` from the filter button's Help menu (`Gui/WindowFrames/FilterButton/FilterButtonMixin.lua:203`, `:234-235`) and the options (`Options/General.lua:51-60`, `:307`).

## Architecture and API

- `lib:New(key, savedVariables, icon, font)` (`:417`) builds `Krowi_Tutorial<n>` from Blizzard's `ButtonFrameTemplate` on `UIParent`: `DIALOG` strata, clamped, movable, closed by the game-menu key, paged by the mouse wheel, with Prev/Next buttons, a page counter and a `Shine` glow (`BackdropTemplate` with a looping alpha animation). It creates `savedVariables[key].PageViewed` and returns the frame, whose methods come from the library table.
- Settings: `SetFrameTitle`, `SetFrameWidth`, `SetPages`, `AddPage`, `SetImageMargin`, `SetImageSize`, `SetTextMargin`, `SetTextSize`; a margin is a number, `{l, t, r, b}` or named keys, a size `{w, h}` or named keys. `SetCloseButtonHook(func)` replaces the `OnHide` script; `Reset()` clears the viewed marks and hides.
- `ShowTutorial(index, minPage, maxPage)` shows a page (default the first unviewed one); `HasUnviewedPages()`. `UpdateFrame` (`:60-253`) takes each setting from the page, then the frame, then `default` (`:14-45`), lays out image and text vertically or `HORIZONTAL`, runs the page's `OnShow` and `OnHide`, puts the glow on `page.Shine` with the `Shine*` offsets, and marks the page viewed.
- No events and no saved variables of its own.

## Taint-sensitive areas

The windows are the library's own, built from Blizzard templates. Run the `krowi:taint-reviewer` agent after changing any of these:

- The close button's `OnKeyDown` calls `SetPropagateKeyboardInput` on every key press while a tutorial is shown (`:389-400`). The API docs flag that function `HasRestrictions` on every client (live `Blizzard_APIDocumentationGenerated/SimpleFrameAPIDocumentation.lua:1424`).
- `frame.Shine:SetParent(page.Shine)` (`:177`) reparents the glow onto whatever frame a page names, which can be a Blizzard frame; Krowi_AchievementFilter sets no `Shine`.
- `page.RelativeFrame` anchors the window to any frame (`:103`).
- `setmetatable(frame, setmetatable(lib, getmetatable(frame)))` (`:423`) swaps the frame's widget metatable for the library table, chained to the original.
- The ElvUI skin runs ElvUI's `StripTextures`, `CreateBackdrop` and skin handlers on the window (`:280-301`).

## Localization

No locale files. The buttons use Blizzard's `PREVIOUS` and `NEXT` (`:446-447`); the default title is the English `'Tutorial'` (`:24`), which consumers override with `SetFrameTitle`.

## Code style

Match the file you are editing:

- Tabs, except the copyright blocks, `MakeMovable` (`:370-387`) and `:110`, which use spaces.
- No semicolons; single quotes.
- `Krowi_Tutorials.lua` opens with the four-line `--[[ Copyright (c) 2023 Krowi ... ]]` block, a blank line, `---@diagnostic disable: undefined-global` and `---@diagnostic disable: redefined-local`, a blank line, then the `NewLibrary` call and its early return. Sections are `-- [[ INTERNAL API ]] --` (file-local helpers) and `-- [[ EXTERNAL API ]] --` (the `lib:` methods). The XML and the TOC carry the same notice as comments.
- CRLF on disk (LF in git, `core.autocrlf=true`), no BOM, no final newline. No `.editorconfig` and no `.vscode/settings.json`.

## Testing in game

Test through Krowi_AchievementFilter, not with `Deploy.ps1` from this repo: that installs a second copy (`AddOns\Krowi_Tutorials`) that loads after Krowi_AchievementFilter and counts only with a higher minor.

1. Commit the change on this repo's branch.
2. In a Krowi_AchievementFilter worktree (`New-Worktree.ps1`), point the submodule at it: `git -C Libs\Krowi_Tutorials fetch "E:\World of Warcraft Addon Development\Krowi_Libraries\Krowi_Tutorials" <branch>`, then `git -C Libs\Krowi_Tutorials checkout FETCH_HEAD`.
3. Deploy that worktree (Retail, then `-Client Classic`), ask Krowi to `/reload`, and run `Read-GameErrors.ps1` from it. Exercise: open the tutorial from the filter button's Help menu, page with the buttons and the mouse wheel, close it with Escape, and check after a `/reload` that viewed pages stay marked.
4. Put the submodule pointer back before anything is committed there; the bump goes into Krowi_AchievementFilter's own PR after the release.

Krowi_AchievementFilter loads on Retail, Mists and Titan only; no Krowi addon loads this library on Era or Anniversary, which the TOC claims.

## Git and release

- `main` is the only branch and the release branch: the `Release 3.0.1` and `Release 3.1` commits and tags are on it. Branch from `main` and open PRs against it.
- Release only with the plugin's `krowi:release` skill, from `E:\World of Warcraft Addon Development\Krowi_Libraries\Krowi_Tutorials` (the library is in the addon manager's `LIBRARY_NAMES`). Every GitHub release so far is a full release: `--stability stable`.
- The addon manager writes `README.md` (badges from the TOC's `## Interface:` plus `_Packaging/Description.md`) and `_Packaging/ReleaseNotes.md` (for a library only the newest changelog entry). Edit `_Packaging/Description.md` and `_Packaging/Changelog.md`.
- After a release, bump `Libs/Krowi_Tutorials` in Krowi_AchievementFilter.

## Known debt (verified 2026-10-17)

- A tutorial window open when combat starts misbehaves on every client: Escape runs `self:Click()` from the insecure `OnKeyDown` (`:393`), `UIPanelCloseButton_OnClick` reaches `HideUIPanel`, which refuses an insecure caller in combat, so the window stays open with "Interface action failed because of an AddOn."; every key press calls the restricted `SetPropagateKeyboardInput` (`:394`, `:398`) and logs a blocked action against the embedding addon; and if no key was pressed since the window opened, it swallows keybinds until it is closed with the mouse. Confirmed in source by the baseline review (2026-10-17), not reproduced in game. Smallest fix: guard both calls with `InCombatLockdown()` and hide the parent instead of clicking; cleaner: drop the handler and add the frame to `UISpecialFrames`.
- `SetPages` and `AddPage` read `page.ImageSize[1]` before normalising it (`FixImageTexCoord`, `:350-359`, and `FixTextSize`, `:361-368`, called at `:525-526` and `:541-542`): a page without `ImageTexCoord` needs a positional `ImageSize`, and so does any page with `Layout` and no `TextSize`, or the call errors. The description calls every page field optional.
- `UpdateFrame` calls the shown page's `OnHide` right after its `OnShow` (`:94-99`); the page that was left is never told.
- `New`'s `icon` argument only switches the portrait on: any value shows `Interface\TutorialFrame\UI-HELP-PORTRAIT` (`:426-431`), while the description calls it an image path. `font` is a font file passed to `SetFont(font, 12)`, not a font object name.
- `_Packaging/Description.md`, and so `README.md`, uses `LibStub("Krowi_Tutorials")` (the name is `Krowi_Tutorials_3`), names LibStub as the only requirement (Krowi_Util's LibMan is needed), shows `SetImageSize(width, height)` and `SetTextSize(width, height)` (both take one table), gives `SetFrameWidth` a default of 532 (the code's is 50), and omits `HasUnviewedPages`.
- Dead code: the commented-out edit box and button blocks, `frame.Images` (`:436`, never read), and the `frame:SetTitle` fallback (`:109-111`; every client's `ButtonFrameTemplate` has `TitleText`).
- `## Interface:` lags the live builds (Retail 120105, Mists 50504, Anniversary 20506, Era 11509) and still lists 110207.
- The TOC declares no dependency on Krowi_Util: a standalone install loads before it (alphabetical order) and errors on `KROWI_LIBMAN` unless an addon loaded earlier embeds Krowi_Util.