---
tags: [Technology, Productivity, Personal]
---

# Grab Screenshots from a Microsoft Teams Recording, Right in Your Browser

## Why I needed this

My son's school runs classes on Microsoft Teams, and the recordings are where the slides and screens his teachers share end up. He wanted those as images to revise from, without scrubbing through an hour of video and pressing Win+Shift+S on every slide.

The fix turned out to be one script pasted into the browser console while the recording plays. There's nothing to install and nothing is uploaded anywhere. It checks a frame every second and downloads a JPEG whenever the screen changes enough.

## How it works

Once a second, the script shrinks the current frame to a 32×18 thumbnail, which is 576 brightness values. If those differ from the last saved screenshot by at least `differenceThreshold` (12, on a 0–255 scale), it saves the full-resolution frame. Files are named like `video-frame-0007-00-14-32-416.jpg`: the seventh shot, taken at 00:14:32 in the recording.

## Running it

1. Open the recording in Edge or Chrome and press play.
2. Press F12 and open the **Console** tab. The first time, you may need to type `allow pasting`.
3. Paste the script below and press Enter.
4. Let the recording play to the end with the tab visible, and choose **Allow** when the browser asks about multiple downloads. Playing at 1.5× or 2× works too, but very short screens may be missed.
5. Run `videoGrabber.stop()` when it's done. `videoGrabber.status()` shows progress at any time.

If it reports no video found, the player is inside a frame. Pick it from the context dropdown at the top-left of the Console (it says "top") and paste again.

## The script

Copy all of it into the console in one paste.

```javascript
(async () => {
  const CONFIG = {
    captureEverySeconds: 1, // Check one frame every second
    differenceThreshold: 12, // Higher = fewer screenshots
    imageType: "image/jpeg",
    imageQuality: 0.92,
    filePrefix: "video-frame"
  };

  if (window.videoGrabber?.running) {
    console.warn("Video grabber is already running.");
    return;
  }

  const videos = [...document.querySelectorAll("video")];

  if (!videos.length) {
    console.error("No HTML5 video element was found on this page.");
    return;
  }

  // Prefer the currently playing video, otherwise use the largest video.
  const video =
    videos.find(v => !v.paused && !v.ended && v.readyState >= 2) ||
    videos
      .filter(v => v.readyState >= 2)
      .sort(
        (a, b) =>
          b.getBoundingClientRect().width * b.getBoundingClientRect().height -
          a.getBoundingClientRect().width * a.getBoundingClientRect().height
      )[0];

  if (!video) {
    console.error("A usable video element was not found.");
    return;
  }

  const canvas = document.createElement("canvas");
  const context = canvas.getContext("2d", { willReadFrequently: true });

  // Small comparison canvas makes duplicate detection faster.
  const compareCanvas = document.createElement("canvas");
  compareCanvas.width = 32;
  compareCanvas.height = 18;
  const compareContext = compareCanvas.getContext("2d", { willReadFrequently: true });

  let previousSignature = null;
  let savedCount = 0;
  let checkedCount = 0;
  let timer = null;

  function formatTime(seconds) {
    const totalMilliseconds = Math.floor(seconds * 1000);
    const hours = Math.floor(totalMilliseconds / 3600000);
    const minutes = Math.floor((totalMilliseconds % 3600000) / 60000);
    const secs = Math.floor((totalMilliseconds % 60000) / 1000);
    const milliseconds = totalMilliseconds % 1000;

    return [
      String(hours).padStart(2, "0"),
      String(minutes).padStart(2, "0"),
      String(secs).padStart(2, "0"),
      String(milliseconds).padStart(3, "0")
    ].join("-");
  }

  function getFrameSignature() {
    compareContext.drawImage(video, 0, 0, compareCanvas.width, compareCanvas.height);
    const pixels = compareContext.getImageData(0, 0, compareCanvas.width, compareCanvas.height).data;
    const signature = new Uint8Array(compareCanvas.width * compareCanvas.height);

    for (let source = 0, target = 0; source < pixels.length; source += 4) {
      // Perceived brightness approximation.
      signature[target++] = Math.round(
        pixels[source] * 0.299 +
        pixels[source + 1] * 0.587 +
        pixels[source + 2] * 0.114
      );
    }

    return signature;
  }

  function calculateDifference(current, previous) {
    if (!previous) {
      return Infinity;
    }

    let totalDifference = 0;
    for (let i = 0; i < current.length; i++) {
      totalDifference += Math.abs(current[i] - previous[i]);
    }

    return totalDifference / current.length;
  }

  function downloadCanvas(filename) {
    return new Promise((resolve, reject) => {
      canvas.toBlob(
        blob => {
          if (!blob) {
            reject(new Error("The browser could not create the image."));
            return;
          }

          const url = URL.createObjectURL(blob);
          const link = document.createElement("a");
          link.href = url;
          link.download = filename;
          link.style.display = "none";

          document.body.appendChild(link);
          link.click();
          link.remove();

          setTimeout(() => URL.revokeObjectURL(url), 1000);
          resolve();
        },
        CONFIG.imageType,
        CONFIG.imageQuality
      );
    });
  }

  async function captureFrame() {
    if (
      !window.videoGrabber.running ||
      video.paused ||
      video.ended ||
      video.readyState < 2
    ) {
      return;
    }

    checkedCount++;

    try {
      const currentSignature = getFrameSignature();
      const difference = calculateDifference(currentSignature, previousSignature);

      if (difference < CONFIG.differenceThreshold) {
        console.debug(`Skipped ${video.currentTime.toFixed(2)}s, difference ${difference.toFixed(2)}`);
        return;
      }

      canvas.width = video.videoWidth;
      canvas.height = video.videoHeight;
      context.drawImage(video, 0, 0, canvas.width, canvas.height);

      savedCount++;

      const extension = CONFIG.imageType === "image/png" ? "png" : "jpg";
      const filename =
        `${CONFIG.filePrefix}-` +
        `${String(savedCount).padStart(4, "0")}-` +
        `${formatTime(video.currentTime)}.${extension}`;

      await downloadCanvas(filename);

      previousSignature = currentSignature;
      console.log(`Saved ${filename}, difference ${difference.toFixed(2)}`);
    } catch (error) {
      console.error("Frame capture failed:", error);

      if (error.name === "SecurityError" || String(error).includes("tainted")) {
        console.error("The site prevents the video from being copied to a canvas.");
        window.videoGrabber.stop();
      }
    }
  }

  window.videoGrabber = {
    running: true,

    stop() {
      this.running = false;
      if (timer) {
        clearInterval(timer);
      }
      console.log(`Video grabber stopped. Checked ${checkedCount} frames and saved ${savedCount}.`);
    },

    status() {
      console.table({
        running: this.running,
        checkedFrames: checkedCount,
        savedFrames: savedCount,
        videoTime: video.currentTime,
        threshold: CONFIG.differenceThreshold,
        intervalSeconds: CONFIG.captureEverySeconds
      });
    }
  };

  timer = setInterval(captureFrame, CONFIG.captureEverySeconds * 1000);

  console.log("Video grabber started.");
  console.log(
    `Sampling every ${CONFIG.captureEverySeconds} second(s), ` +
    `with uniqueness threshold ${CONFIG.differenceThreshold}.`
  );
  console.log("Run videoGrabber.stop() to stop.");
  console.log("Run videoGrabber.status() to view progress.");

  await captureFrame();
})();
```

## Tuning and limitations

The defaults struggle with slides that share a template and with screens that are still moving. I ran it on a 72-second synthetic Teams-style recording with eight known slides. At the default threshold it saved 2 of the 8. At threshold 2 it found 5, but 8 of its 13 files were blurry, half-scrolled or blank.

- **Lower `differenceThreshold` to 3–4 for text slides.** Raise it if you get near-duplicates. Set `imageType` to `"image/png"` for crisper text.
- **It saves the first frame that differs**, so it can catch a slide while it's still blurry, fading in or mid-scroll.
- **It compares only with the last saved shot**, so going back to an earlier slide saves it again.
- **It runs in real time**, and browsers slow timers in hidden tabs, so keep the tab visible.
- **Some players block it.** If the console says the site prevents copying the video to a canvas, download the recording and play the file locally.

## Wrapping up

With one paste, an hour-long class becomes a folder of timestamped screenshots for my son to revise from. For cleaner output, the next step would be to wait for the screen to hold still before saving, and to compare against every saved shot, not just the last.
