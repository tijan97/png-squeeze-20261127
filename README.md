# png-squeeze — smallest lossless PNG for screenshots

A Node 22 program (`main.ts`, no dependencies) that re-encodes each
1280x720 RGBA screenshot in `/input/<id>/image.png` losslessly as the
smallest PNG it can, writing `/output/<id>/image.png`.

How it works:

- Decodes the stored input PNG with Node's built-in zlib and un-filters
  its rows back to raw RGBA pixels.
- Analyses each image (opaque? grey? how many distinct colours?) and
  picks the smallest legal colour type: palette (type 3) when there are
  at most 256 colours, grey (type 0) or grey+alpha (type 4) when every
  pixel is grey, RGB (type 2) or RGBA (type 6) otherwise — with a tRNS
  key when the image has a single fully-transparent value.
- Screens filter choices per form: fixed filters 0–4, the minimum-sum
  heuristic, the fewest-distinct-pairs heuristic, and per-row brute
  force weighed by zlib at level 4 with a 32 KiB dictionary context
  (pre-ranked by signed sum, top four weighed). Each candidate is
  weighed by zlib at level 6; the best few are re-weighed at level 9
  with both the default and filtered strategies at memLevel 8 (and
  memLevel 9 for the winner), and the smallest is written.
- Writes a minimal legal PNG: IHDR, optional PLTE/tRNS, a single IDAT
  zlib stream, IEND, with correct CRCs. Bit depth 8, never interlaced.

The program is deterministic: two runs write byte-identical outputs.
It uses only Node built-ins and reads nothing outside `/input` and its
own directory. One run answers all 8 instances of a set in about 33
seconds, inside the 60-second wall budget.

On the kit's practice set it scores about 705,000 of 1,000,000, against
make.fun's fixed-paeth house baseline at about 420,000.
