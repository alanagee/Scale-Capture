[README_1.md](https://github.com/user-attachments/files/32196495/README_1.md)
# Scale-Capture# Scale Capture

A small web app for your iPhone: photograph an area that has a known-size
square target in it, and get back a photo that's been automatically
perspective-corrected and scaled so it's dimensionally accurate — then hand
it straight to Mail as an attachment.

It's two pages plus this readme:

- **`index.html`** — the app itself (camera capture, detection, scaling, share-to-Mail)
- **`marker.html`** — a printable target-marker generator
- **`README.md`** — this file

No build step, no app store, no account. It's a static site — host it
anywhere and add it to your iPhone home screen.

## How it works

1. You tell it the real-world size of a square target (default 10×10").
2. You photograph a flat area with that target placed in the same plane
   (e.g. taped to a car panel), fully visible.
3. It automatically finds the black square in the photo (OpenCV.js running
   entirely in your browser — nothing is uploaded anywhere), works out the
   perspective distortion, and warps the *entire* photo so the target
   becomes a perfect, axis-aligned square of the correct size. Because the
   whole image shares the same flat plane as the target, this also
   straightens out the perspective skew across the rest of the photo — the
   result is a to-scale, flattened image you can measure directly off of.
4. You tap **Attach & Send**, which hands the image to iOS's share sheet.
   Pick **Mail** and it opens a new message with the photo attached, ready
   for you to address and send.

If auto-detection can't confidently find the square (bad lighting, target
not fully in frame, low contrast against the background), the app falls
back to letting you tap the four corners yourself — it never dead-ends.

## Hosting it (GitHub Pages)

Same pattern as your PPF tracker:

1. Create a new GitHub repo (or a folder in an existing one) and add
   `index.html` and `marker.html` to it.
2. In the repo's **Settings → Pages**, set the source to your main branch
   (root folder), save, and GitHub gives you a URL like
   `https://<you>.github.io/<repo>/`.
3. Open that URL in Safari on your iPhone.
4. Tap the Share icon → **Add to Home Screen**. It now behaves like a
   regular app icon — full screen, no Safari chrome.

No server, database, or backend involved — everything (including your
settings) lives in the browser on your phone.

## Making a target marker

Open `marker.html` (there's a link to it from the main app). It generates a
precise black square sized to your target setting, laid out across one or
more Letter-size sheets with trim lines and small alignment marks so you
can tape multiple sheets into one bigger square (needed for anything larger
than about 7×7", including the 10×10" default).

Printer scaling isn't always exact, so:

- Always print at **100% / Actual Size** — never "Fit to page."
- Check the small ruled line included on the first sheet with a real ruler
  — it should measure exactly 3.00". If it's off, your printer scaled the
  page; reprint at the corrected scale, or just adjust the target-size
  setting in the app to match what actually printed.
- Mount the finished square on something stiff (foam board, cardboard) so
  it stays flat.

If you'd rather not deal with printer accuracy at all, you can skip
printing entirely: cut a square from any stiff material and measure it
directly with a tape measure/ruler — as long as you enter its *actual*
measured size into the app's target-size field, detection and scaling work
exactly the same way.

**Marker design matters for reliable auto-detection.** The detector looks
for a solid black square surrounded by a lighter margin (that's what
`marker.html` prints). For best results:
- Good, even lighting with no glare on the target.
- The target fully inside the frame, with all 4 corners visible.
- Reasonable contrast between the target and whatever it's sitting on.
- The target lying flat, in the same plane as the area you're capturing —
  the "flatten the whole photo" math only holds if that's true.

## Settings

- **Target size** — the real-world side length of your square target, in
  inches. Change it any time; it's remembered on your phone between uses,
  and is shared with the marker generator page.
- **Output resolution (PPI, under Advanced)** — pixels per inch in the final
  image. 100 is a good default (a 10×10" target becomes a 1000×1000px
  square in the output). Raise it if you need finer measurement precision;
  lower it to keep file sizes down.
- **Show detection debug view (under Advanced)** — if auto-detect is
  struggling with your setup, turn this on to see the thresholded image the
  detector is working from, with the square it found (if any) outlined.
  Handy for figuring out whether it's a lighting, contrast, or framing
  problem.

## About the OpenCV.js dependency

The app loads OpenCV.js (the computer-vision library that does the actual
square-detection and image-warping) from `https://docs.opencv.org` at
runtime — that's the only external dependency, and it's only needed once
per device (the browser caches it after the first load). If you'd rather
not depend on that site staying up, you can self-host it instead:

```
curl -L -o opencv.js https://docs.opencv.org/4.x/opencv.js
```

Put the downloaded `opencv.js` file next to `index.html` in your repo, then
change the `<script src="...">` line near the bottom of `index.html` from
the `docs.opencv.org` URL to plain `"opencv.js"`. It's a large file
(several MB), so this does make your repo bigger, but it means the app has
no runtime dependency on an external site at all.

## Tuning detection

If auto-detect is unreliable in your actual working conditions, the
thresholds it uses are all named constants near the top of the `<script>`
block in `index.html` (search for `const CFG = {`) — things like how square
the four corners need to be, how much contrast is required between the
target and its margin, and so on. They're commented with what each one
controls.

## Privacy

Everything happens on-device — the photo is decoded, analyzed, and warped
entirely in your phone's browser. Nothing is uploaded to any server. The
photo only leaves your phone if and when you choose to send it as an email
attachment.
