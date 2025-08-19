Geopoint Triangulator — compute coordinates from angles + distances
=================================================================

Overview
--------
This is a client-side browser tool (plain HTML + JavaScript) that computes 2D/3D coordinates for measured points from horizontal angles, vertical angles and slope distances. It is designed to work with the File1/File2 text files produced by the Geopoint Extractor (or other similar exports).

Key capabilities
----------------
- Solve for instrument position (X0, Y0, Z0) and orientation (theta) using Gauss–Newton nonlinear least-squares.
- Support horizontal units in grads (400 gon) and degrees (360) — grads are handled correctly.
- Support vertical-angle conventions: "zenith" (0 = up, horizon = 100 gon / 90°) and "inclination" (0 = horizontal).
- Preserve known coordinates from File2 verbatim (points present in File2 are not re-computed).
- Use per-measurement prism height (Ref.H) from File1 in vertical calculations; output point height = prism center - prismHeight.
- Automatic orientation detection: tests flip/swap conventions and quarter-turn theta offsets and chooses the best variant.
- RANSAC-style leave-one-out inlier selection to remove a single outlier reference automatically.
- Fine theta grid refinement around the best offset (0.1 gon/deg step by default).
- Diagnostics and residual reporting (per-reference dE,dN,dH and RMS in mm).
- Mobile-friendly UI (touch-optimized controls and safe download logic).

Files and formats
-----------------
- File1 (measurements) format expected by the triangulator:
  - First line is treated as a comment and can start with `#`.
  - Following lines: Point-ID  Hz  V  Distance  Ref.H
  - Example (tab/space separated):
    # for File 1 - Point-ID     Hz           V       Distance    Ref.H
    200370016    336.9697     95.7813   40.779  0.240

  - Notes: `Ref.H` (prism height) is optional. If present it is applied as an additive height to the known H in File2 for prism-center comparisons.

- File2 (known coordinates) format expected:
  - First line may be a comment (starting with `#`).
  - Following lines: Point-ID  East  North  Height
  - Example:
    # for File 2 - Point-ID       East          North        Height
    200370016    3513459.157 5405284.422  245.257

- Standpunkt/instrument station parsing:
  - If the source `.lqp` contains a Standpunkt(Koordinaten) block, the triangulator will attempt to parse it and can use it to test conventions (station-preserving checks). When solving, the instrument station is estimated from references unless you use a station-check path.

How it works (contract)
-----------------------
- Inputs: File1 text, File2 text, selected reference IDs (at least 3 references), angle unit (400 or 360), vertical type (zenith/inclination).
- Output: computed coordinates for measured points (E,N,H) with known points preserved verbatim; downloadable `computed_coords.txt` with numeric values (3 decimals).
- Error modes: returns an error if geometry is degenerate (singular normal matrix), if fewer than 3 references selected, or if parsing fails.
- Success criteria: instrument X/Y matching parsed station when available; low RMS residuals (mm-level) across references.

Algorithm and implementation details
------------------------------------
- Uses Gauss–Newton to solve for parameters p=[X0,Y0,Z0,theta]. The Jacobian is assembled per reference (3 residuals: dE,dN,dH).
- The code uses a small Gaussian elimination solver for the 4x4 normal system. A failure to invert JTJ returns an explicit error.
- Orientation auto-detect tries combinations of {flipHz, swapXY} and quarter-turn theta offsets (0/90/180/270 deg or 0/100/200/300 gon), picks the best RMS, runs leave-one-out RANSAC to remove a single outlier, then performs a fine theta grid search.

Mobile and UI notes
-------------------
- The UI is optimized for touch: larger buttons, stacked controls on narrow screens, and temporary object-URL downloads (created per click and revoked) to support repeated downloads and avoid blob:null errors on mobile browsers.
- A file-size limit (default 5 MB) is enforced on mobile to avoid OOM/crashes; if you have larger `.lqp` data, run the tool on a desktop browser or increase the limit in `geopoint_triangulator.html`.

Security and safety
-------------------
- Client-only: the application runs entirely in the browser, no network calls are made and nothing is uploaded.
- No dynamic code execution (`eval`) is used.
- All file content is treated as plain text and validated numerically; outputs are generated via safe DOM/text APIs (no HTML injection).
- Blob URLs are created temporarily and revoked shortly after triggering a download to avoid stale blob issues.

Testing and verification
------------------------
- Use the Geopoint Extractor to produce `file1.txt` and `file2.txt`, then load them in `geopoint_triangulator.html`.
- Recommended flow:
  1. Load File1 and File2.
  2. Select at least 3 references in the table (check the boxes next to point IDs present in both files).
  3. Optionally run "Auto-detect orientation" to let the tool pick the best orientation.
  4. Run Solve. Review per-reference residuals and RMS in the log area.
  5. Download `computed_coords.txt` and inspect values.

Troubleshooting
---------------
- "Need at least 3 reference points": select three or more reference points that appear in both files.
- Singular normal matrix (bad geometry): references are nearly colinear or insufficiently distributed — pick different references or add more known points.
- Systematic vertical bias after solving: ensure `Ref.H` in File1 has the correct sign and units; the triangulator treats `Ref.H` as an additive prism height (prism center = known H + Ref.H). If your workflow uses opposite sign, invert the sign in File1 or ask to add a toggle.

Possible improvements
---------------------
- Implement streaming parsing for very large `.lqp` files (>5 MB) to reduce peak memory usage on mobile devices.
- Add estimation of additional parameters (height bias, distance scale / ppm) in the GN model if systematic biases persist.
- Provide a small suite of unit tests and example `.lqp` snippets for regression testing.

Developer notes
---------------
- Key file: `geopoint_triangulator.html` (client-only). See code comments for numerical details.
- If you change horizontal unit handling, verify conversion constants used (PI/200 for 400-gon, PI/180 for degrees).

License and privacy
-------------------
- The code in this repository is provided as-is for private use. There are no external dependencies. You are free to adapt it for your workflows.

Contact / next steps
--------------------
If you want any of the improvements above (toggle for `Ref.H` sign, streaming parse, bias/scale parameters or unit tests), tell me which one and I will implement it next.
