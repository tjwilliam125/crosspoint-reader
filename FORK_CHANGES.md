# Fork Changes

Changes on branch `feat/toc-title-wrap` of this fork, on top of upstream
`develop`. Built and tested on an Xteink X4 Classic (`pio run -e x4c`).

The X4 is button-driven, so most of these changes cut down on button presses.

## Reader

- **Wrapping chapter titles.** In Select Chapter, long chapter titles wrap to up
  to 3 lines (`TOC_TITLE_MAX_LINES`) instead of being cut to one line with an
  ellipsis. Selection and paging still work with rows of different heights.
- **Go Home moved up.** In the reader menu, Go Home sits just below Select
  Chapter.
- **Go To Articles.** A reader menu entry between Select Chapter and Go Home
  opens the `/Articles` folder in the file browser. It only appears when
  `/Articles` exists on the SD card.

## Deleting books

- **File browser.** Long-pressing a file or folder opens its actions menu on
  **Delete** instead of Open, and the Delete? dialog opens on **Confirm**.
- **Library view.** The row actions menu opens on **Delete**, and the Delete?
  dialog opens on **Confirm**.

A delete now takes long-press, Select, Select. Back still cancels at either
step. Other confirmation dialogs still start on Cancel: `ConfirmationActivity`
takes an opt-in `defaultToConfirm` flag, and only the two delete dialogs set it.

## Wi-Fi

- **Saved networks after a failed auto-connect.** The network list used to show
  only "Add hidden network..." after a failed auto-connect, because the failed
  join kept retrying and blocked the scan. Auto-reconnect is now turned off
  before scanning, and a failed scan is retried up to twice.
- **Saved networks the scan didn't see.** Saved networks that are hidden or out
  of range are listed after the scanned ones, marked "--", so they can be
  joined without retyping the SSID. Auto-connect still only tries visible
  networks.

## Web upload

- **Multi-file uploads.** Each pick or drop adds to a visible queue, with
  per-file remove and "Clear all". Before, each pick replaced the last one, so
  on Android only the last file was uploaded.
- **Folder rows.** Tapping anywhere in a folder row opens the folder, not just
  the name link.
- **Google Drive files.** If an upload fails before sending a byte, it is
  retried once using an in-memory copy of the file. Android Chrome fails that
  way with files picked from Google Drive.

## Dictionary setup (no code change)

The built-in dictionary works with the Wiktionary English-English StarDict
dictionary from <https://xxyzz.github.io/wiktionary_stardict/> (`en-en.tar.zst`,
unpacks with Windows 11 `tar -xf`).

- **Use the plain `.dict`, not `.dict.dz`.** The firmware refuses compressed
  `.dict.dz` files with more than 8,192 chunks (`MAX_CHUNK_COUNT` in
  `src/util/DictZip.cpp`, which keeps the chunk table under 32 KB of memory).
  This dictionary has 17,634 chunks, so lookups fail with "couldn't decompress
  definition". Unpack `en-en.dict.dz` to a plain `en-en.dict` (about 1.03 GB,
  under FAT32's 4 GB file limit). The firmware uses the plain `.dict` when both
  are present, but there's no reason to keep the `.dz` on the card.
- **Files on the SD card.** `/dictionaries/en-en/` holds `en-en.dict`,
  `en-en.idx`, `en-en.syn` and `en-en.ifo`. The `res/` folder and `en-en.css`
  in the download aren't used.
- **Settings.** Choose **en-en** in Settings → Reader → Dictionary. For the
  quickest lookups, set Settings → Controls → **Long-press Menu** to
  "Dictionary". Then hold Select on a reading page, move to the word, and press
  Select twice.
- **First lookup.** The first lookup builds `.qidx` and `.sidx` index files next
  to the dictionary, which takes a while.

## Building on Windows

- **Run pio from PowerShell, not Git Bash.** In Git Bash the toolchain install
  fails.
- **Set `PYTHONUTF8=1`** when redirecting pio output to a file.
- **Use `core.autocrlf false`** for this clone, or the SdFat/JPEGDEC patch step
  fails.
- **Flash** with `pio run -e x4c -t upload --upload-port COM5`. Keep the X4
  awake, because the upload rebuilds first.
