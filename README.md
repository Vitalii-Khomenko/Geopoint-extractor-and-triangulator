# Geopoint Tools

Combined documentation for the Geopoint Extractor and Geopoint Triangulator — two client-side HTML/JavaScript tools for extracting Leica `.lqp` measurements and computing coordinates from angles and distances.

## Table of contents
- Overview
- Programs
  - Geopoint Extractor
  - Geopoint Triangulator
- File formats
- Quick usage
- Mobile & security notes
- Diagnostics & troubleshooting
- Developer notes
- License

## Overview
This repository contains two related browser tools:
- Geopoint Extractor — parses Leica `.lqp` files and exports measurement and known-coordinate text files.
- Geopoint Triangulator — reads extractor outputs (File1/File2) and computes point coordinates using Gauss–Newton, with auto-detection of angle conventions, RANSAC, and diagnostics.

Both tools run entirely in the browser (no network calls) and are implemented as single-file HTML + JavaScript.

## Programs

### Geopoint Extractor
Purpose:
- Parse Leica `.lqp` measurement protocols and export two text files:
  - `file1.txt` — measurement list (Point-ID, Hz, V, Distance, Ref.H)
  - `file2.txt` — known coordinates (Point-ID, East, North, Height)

Key features:
- Extracts Ref.H (prism height) from measurement lines.
- Mobile-friendly UI, drag & drop and file picker, file-size limit (default 5 MB).
- Safe downloads (temporary blob URLs) and robust parsing with error handling.

Usage summary:
1. Open `geopoint_extractor.html` in the browser (desktop or mobile).
2. Select / drop a `.lqp` file and click Parse.
3. Inspect preview and Download File1/File2.

Notes:
- File1 contains prism heights (Ref.H). Coordinate heights are kept in File2 only.
- Exported files start with a comment header line.

### Geopoint Triangulator
Purpose:
- Compute E,N,H for measured points from File1 (measurements) and File2 (knowns).
- Solve for instrument X0,Y0,Z0 and orientation theta using Gauss–Newton.

Key features:
- Supports grads (400 gon) and degrees (360).
- Supports vertical conventions (zenith vs inclination).
- Uses per-measurement Ref.H in vertical computations (prism-center = known H + Ref.H).
- Preserves File2 coordinates verbatim for points present in File2.
- Auto-detect orientation (flip/swap × quarter-turn offsets), RANSAC leave-one-out, fine theta grid-refinement.
- Diagnostics: per-ref dE,dN,dH and RMS in mm.
- Mobile-friendly and secure (no network).

Usage summary:
1. Generate File1 and File2 with the Extractor.
2. Open `geopoint_triangulator.html`.
3. Load File1 and File2, select ≥3 reference points (present in both files).
4. Optionally run Auto-detect orientation, then Solve.
5. Review diagnostics and download `computed_coords.txt`.

Quality controls:
- Solver reports errors on singular geometry, fewer than 3 refs, or parsing failures.
- RANSAC and fine-refinement reduce outliers and orientation bias.

## File formats (expected)

### File1 (measurements)
- First line: comment (starts with `#`)
- Columns: Point-ID  Hz  V  Distance  Ref.H
- Example:
```
# File1 - Point-ID Hz V Distance Ref.H
200370016 336.9697 95.7813 40.779 0.240
```

### File2 (known coordinates)
- First line: comment (optional)
- Columns: Point-ID  East  North  Height
- Example:
```
# File2 - Point-ID East North Height
200370016 3513459.157 5405284.422 245.257
```

Notes:
- `Ref.H` is optional in File1; when present it is treated as additive prism height (prism center = known H + Ref.H).
- The triangulator expects consistent separators (spaces/tabs) and numeric values.

## Quick troubleshooting
- "Need at least 3 reference points": choose at least 3 refs present in both files and well-distributed.
- Large RMS / systematic vertical bias: ensure Ref.H sign is correct and units match; enable diagnostics to inspect dH.
- Singular normal matrix: selected references are nearly colinear — add more references or change selection.
- Blob download error: update to latest tool (temporary blob URL flow implemented) or re-open page.

## Mobile and security notes
- Client-only: no network requests, no external libraries.
- Input is parsed as text; no dynamic code execution (no eval).
- UI optimized for touch (Samsung A55 & similar): large buttons, stacked layout, drag/drop, file-size guard.
- Temporary blob URLs are revoked after download to avoid stale blob issues.
- For hosted deployments, apply a strict Content-Security-Policy and HTTPS.

## Developer notes
- Main files:
  - `geopoint_extractor.html`
  - `geopoint_triangulator.html`
- Triangulator algorithm:
  - Gauss–Newton on parameters [X0, Y0, Z0, theta].
  - Optional parameters available: RANSAC inlier selection, fine theta search, Z-bias/scale extensions (can be added).
- Tests:
  - Consider adding small `.lqp` snippets and unit tests for parser edge cases.
- Conventions:
  - 400 gon → radians: multiply by PI/200.
  - 360 deg → radians: multiply by PI/180.

## License and privacy
- The code in this repository is provided as-is for private use. There are no external dependencies. You are free to adapt it for your workflows.

## Contact / next steps
If you want any of the improvements above (toggle for `Ref.H` sign, streaming parse, bias/scale parameters or unit tests), tell me which one and I will implement it next.
