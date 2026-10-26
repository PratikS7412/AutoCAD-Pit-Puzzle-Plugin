What This Is

`RR-Pit-Puzzle-Structure` is an AutoCAD 2027 plugin that draws a pit
structure: a pit boundary/frame with internal pieces fitted inside it (the
"puzzle" is fitting known-size pieces into a variable-size pit). The
earlier placeholder boundary-rectangle geometry builder was removed by user
request ("I have different plans"); real geometry creation has now begun
again from scratch, feature by feature (see §8), starting with the Plan
View Border's vertical grid centerlines. Real piece-fitting/arrangement is
still future work. The command is also reachable via a **"RR Pit Puzzle" Ribbon tab**
(`PitPuzzleRibbon.cs`, 2026-09-24) — one panel, one button, same logo as the input dialog — created
once at plugin load (`PluginEntry.Initialize`), not just by typing the command name.
