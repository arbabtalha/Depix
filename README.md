# Depix

**A little care. A clearer memory.**

Depix is a free, private image-restoration workspace. Repair small scratches, reduce blur and noise, correct faded colors, enlarge images, and read printed text. Images are processed on your device, not uploaded to an image-processing service.

**Live demo:** https://arbabtalha.github.io/Depix/

**Repository:** https://github.com/arbabtalha/Depix

## Files

- `index.html`: the entire application, including HTML, CSS, JavaScript, inline SVGs, a generated sample illustration, and an inline processing worker.
- `README.md`: usage, research, architecture, and limitations.
- `LICENSE`: MIT license for Depix.

No build step, account, paid key, installation, backend, or separate asset files are required. Only optional OCR downloads third-party code and language data.

## Features and usage

### 1. Open an image

1. Click **Choose image** or the upload icon in the editor toolbar.
2. Alternatively, drag an image into the preview or paste an image with Ctrl/Cmd + V. Paste an actual image, not a file path or URL.
3. Supported types: JPG/JPEG, PNG, WebP, and BMP. HEIC, animated GIF, PDF, and SVG are not supported.
4. Large inputs require your permission to resize the working copy. Your original file is untouched.
5. **Try a sample** opens a deliberately faded, noisy landscape illustration with a scratch and readable text. It is not a photograph or a claimed restoration result.

### 2. Deblur and sharpen

1. Open **Enhance**.
2. Choose **Out of focus**, **Motion blur**, or **Sharpen only**.
3. Adjust **Strength**. For motion blur, adjust **Motion angle** to match the direction of the streaks.
4. Click **Deblur & sharpen**. Start gently, compare, and undo if halos or extra noise appear.

Out-of-focus and motion modes use bounded, iterative Richardson–Lucy-style luminance deconvolution with a disk or line blur kernel, followed by light sharpening. Sharpen-only uses an unsharp mask. This is real classical processing, not a neural restoration model. Estimated kernels cannot match every real blur.

### 3. Reduce noise

1. Adjust **Noise reduction** in Enhance.
2. Click **Reduce noise**.
3. Compare small textures before accepting the result.

A local bilateral filter mixes neighboring colors according to color difference and distance. Strong settings can remove fine texture along with noise.

### 4. Enlarge 2× or 4×

1. Prefer to repair and refine at the original working size first.
2. Click **Upscale 2×** or **Upscale 4×**. Width and height each increase by that factor.
3. If the result exceeds the output limit, choose **Resize for safe 4×**. Confirm the reduction, then click 4×. That factor applies to the newly resized working image, not the original upload.

Enlargement uses alpha-aware bicubic interpolation. It creates a smoother, larger image; it does **not** reconstruct missing semantic detail.

### 5. Repair scratches, stains, and small damaged spots

1. Open **Repair**, choose **Paint**, and set the brush size in image pixels.
2. Draw over a small damaged area. The lime overlay is the mask, not an image edit.
3. **Erase mask** removes parts of your selection; **Clear mask** removes all of it.
4. Click **Fill damage**. The original unmasked pixels remain unchanged.
5. Repeat in small passes. Masks larger than 20% of the image are rejected.

The fill propagates neighboring colors inward, then relaxes the filled area to soften seams. It is classical diffusion inpainting, not object generation, texture synthesis, or OpenCV's Telea algorithm. It works best on narrow scratches and relatively smooth backgrounds.

**Keyboard alternative:** choose Paint/Erase, focus the image with Tab, move the cursor with arrow keys, and press Space to paint/erase. Shift + arrows move faster. **Keyboard & precise selection** also offers a rectangular mask using image-pixel coordinates.

### 6. Restore an old photo

1. In Enhance, adjust **Restoration strength** under **Bring back the color**.
2. Click **Restore old photo**.
3. Compare faces and highlights before downloading.

The tool uses per-channel percentile stretching to reduce fading and color casts. It is **not black-and-white photo colorization**, and it cannot know historical colors. Strong settings can clip colors; use undo and lower strength.

### 7. Clean and read blurred text

1. Open **Read text**.
2. Click **Select text area** and drag a rectangle, or enter Left/Top/Width/Height and click **Set area**.
3. **Whole image** clears the selection. Coordinates are relative to the current working image.
4. Choose **Grayscale + contrast** or **Adaptive black & white**, then **Enhance text area** to edit those pixels. Text cleanup can be undone.
5. Select English, Spanish, French, or German. Click **Read text with OCR**.
6. OCR separately prepares a grayscale, contrast-enhanced, enlarged crop without changing your image. First use downloads the engine and selected language model.
7. Review the detected text and confidence, then click **Copy detected text**. If clipboard permission is blocked, the text is selected for manual copying.

OCR cannot reliably read severely blurred text, handwriting, missing letters, or all page layouts. The reported confidence is an engine estimate, not a guarantee. Always check names, dates, amounts, and numbers against the original. Do not use enhanced text as evidence.

If OCR fails to load, Depix explains the failure, provides **Retry last action**, and keeps the classic text-enhancement tool available. It does not fake OCR output or substitute guessed text.

### 8. Compare and manage edits

- Drag the comparison slider or the image in comparison mode. The left portion shows the original working image; the right shows the restored image.
- Use the slider's arrow keys for a non-drag alternative.
- **+ / −** change zoom; **Fit** fits the image. Scroll inside a zoomed preview to reach its edges.
- **Undo / Redo** restore pixel snapshots, including image dimensions. Ctrl/Cmd + Z undoes; Ctrl/Cmd + Shift + Z or Ctrl/Cmd + Y redoes.
- **Reset** returns to the original working copy, and can itself be undone.
- A committed edit clears the repair mask, text selection, and stale OCR output.
- Use **Cancel** during processing to stop a worker without committing partial results. Canceling a download ignores its encoder result; the browser's encoder itself may finish in the background.
- The sun button switches between dark and light themes. Help and privacy dialogs support Escape.

### 9. Download

1. Select **PNG** for lossless output with transparency, or **JPG** for a smaller file.
2. Click **Download**.
3. Allow downloads for the site if your browser blocks them.

Downloads use the full current working resolution without a watermark. JPG uses quality 0.94 and a white background for transparent pixels. Original EXIF metadata is not retained. Browser-side encoding errors show a retry action; Depix cannot confirm where your browser ultimately saves a downloaded file.

## Run locally

Double-click `index.html` in a current browser. The classic tools need no network. Some browsers restrict local-file workers or clipboard operations. If that happens, use a local static server for development:

```sh
cd Depix
python3 -m http.server 8080
```

Open http://localhost:8080. This server is optional for local testing; visitors to GitHub Pages do not have to run a server. Internet access is required for first-use OCR.

## Architecture, libraries, models, and APIs

| Component                           | Implementation                                                                      | License                                                 |
| ----------------------------------- | ----------------------------------------------------------------------------------- | ------------------------------------------------------- |
| UI, image decoding, preview, export | Native browser DOM and Canvas 2D                                                    | Browser APIs                                            |
| Restoration                         | Original JavaScript algorithms in an inline Blob Web Worker                         | MIT, this repository                                    |
| OCR wrapper                         | Tesseract.js **7.0.0**, loaded on demand from jsDelivr                              | Apache-2.0                                              |
| OCR engine                          | tesseract.js-core **7.0.0**, JavaScript/WebAssembly builds, selected by the wrapper | Apache-2.0; upstream dependencies retain their licenses |
| OCR language weights                | `@tesseract.js-data/{eng,spa,fra,deu}` **1.0.0**, `4.0.0_best_int` trained data     | MIT packaging; Apache-2.0 upstream trained data         |
| Fonts and icons                     | System sans, Georgia for display emphasis; inline original SVG icons                | No external font or icon dependency                     |

There are **no image-processing APIs, paid keys, cloud inference calls, accounts, analytics, or generative photo-restoration models**. Tesseract OCR uses learned language-recognition weights. Depix does not load OpenCV, TensorFlow, UpscalerJS, or ONNX Runtime; those were researched alternatives, not shipped dependencies.

The OCR CDN paths are explicitly versioned. Third-party packages remain under their own licenses, not Depix's MIT license. See [Tesseract.js](https://github.com/naptha/tesseract.js), [Tesseract.js-core](https://github.com/naptha/tesseract.js-core), and [language data](https://github.com/naptha/tessdata).

## Privacy, security, and terms

- Use only images you own or have permission to edit.
- Your image pixels and detected text remain in browser memory. Closing or reloading the page discards your work. Download before leaving.
- OCR code and language files are fetched from **cdn.jsdelivr.net**. That provider receives normal request metadata, such as your IP address, but not your image. The OCR engine may cache language data in IndexedDB.
- Classic tools work without OCR or its CDN. No restoration API receives your images.
- File type, size, successful decoding, dimensions, selection bounds, and output limits are checked locally. No uploaded file is executed as HTML or SVG.
- OCR text and filenames are inserted as text, never as executable HTML.
- Browser extensions and compromised devices remain outside the app's privacy guarantees. Hosting/CDN providers control their HTTP headers and access logs.
- Provided as-is, without warranty. Keep originals and verify results. Do not use restoration or OCR as proof that an image originally contained particular detail.

## Honest limitations and resource limits

Restoration cannot recover detail that is truly lost. Generative AI tools may add details that were not in the original; Depix intentionally does not use generative photo models. Classical methods can still introduce halos, smooth texture, or fill damage inaccurately.

- File size: **25 MiB** maximum.
- Decoded dimensions: **60 megapixels** maximum. Browsers decode before this check; very large compressed images can still exhaust device memory during decoding.
- Working input: approximately **1.5 megapixels**, with a **2048-pixel** longest side. Larger inputs are resized only with permission. The original comparison uses this working copy, not a full-resolution discarded upload.
- Output: **6 megapixels**. A 4× enlargement uses 16× as many pixels, so its input must be at most 375,000 pixels.
- Undo/redo snapshots: **48 MiB** combined. Oldest entries are dropped when necessary. This is a history budget, not a limit on the browser's total memory use.
- A 6-megapixel RGBA buffer is about 24 MB; previews, overlays, history, worker arrays, and OCR allocate additional memory. Phones can still run out of resources. Choose smaller inputs and close other tabs.
- Compute-heavy restoration runs in a worker. Progress reflects completed rows/iterations, not an ETA. Main-thread decoding, Canvas drawing, mask previews, crop preparation, and encoding can still briefly pause the UI.
- OCR crops are capped at approximately 2 megapixels. OCR engine downloads and memory use are additional to pixel buffers.
- Dark/light appearance and smaller layouts work from 320 pixels wide. Touch brush size is in image pixels, so zoom in for precision.
- Browser support target: current Chromium, Firefox, and Safari with Canvas, Web Workers, WebAssembly, ResizeObserver, and native dialogs. Only Chromium was exercised in the automated test session; real-device Safari/Firefox and slow-phone performance remain unverified.
- Severe motion blur, complex stains, large tears, missing faces, high-quality texture reconstruction, and historical colorization are outside the capabilities of these classic methods.
- No batch processing, RAW support, persistent projects, or magic face reconstruction.

## Research notes (October 13, 2026)

Research was completed before implementation. Product claims below come from the linked documentation/reviews, not an independent ranking or firsthand benchmark.

| Tool researched                                                                                                                                           | Features found                                                                                                   | Depix decision                                                                       |
| --------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| [Topaz sharpening documentation](https://docs.topazlabs.com/photo-ai/enhancements/sharpen) and [enhancer](https://www.topazlabs.com/tools/image-enhancer) | Motion/lens blur modes, strength, denoise, upscale, selective control                                            | Explicit blur types, adjustable strength, comparison                                 |
| [Upscale.media](https://www.upscale.media/tools/pic-enhancer)                                                                                             | Upload, 2×/4×/8× enlargement, preview, download                                                                  | 2×/4× controls, exact dimensions, no watermark                                       |
| [Photopea smart tools](https://www.photopea.com/learn/bt-smart)                                                                                           | Spot healing, healing brush, patch, scratch removal                                                              | Brush mask, erase/clear, manual control                                              |
| [MyHeritage, Remini, VanceAI, Fotor comparison](https://www.unite.ai/best-photo-restoration-tools/)                                                       | Historical portrait enhancement/color/repair; portrait enhancement; denoise/deblur/upscale; browser repair tools | Color-cast correction and local repair, without claiming generative face restoration |
| [Reported Remini complaints](https://www.toolsforhumans.ai/ai-tools/remini-ai)                                                                            | Watermarks, subscription friction, privacy, over-processing                                                      | No paid API, local pixels, undo, adjustable controls                                 |
| [Restoration workflow review](https://pict.ai/blog/best-photo-restoration-app/)                                                                           | Compare at 100%, preserve originals, avoid invented details                                                      | Zoom, comparison, explicit limitations                                               |

### Methods and browser constraints

- [OpenCV out-of-focus deblur](https://docs.opencv.org/4.x/de/d3c/tutorial_out_of_focus_deblur_filter.html) explains estimated point-spread functions and Wiener deconvolution. Depix uses a lighter spatial iterative method instead of shipping OpenCV/FFT machinery.
- [OpenCV inpainting](https://docs.opencv.org/3.1.0/d1/d0d/group__photo.html) describes filling marked areas from boundary neighborhoods. Depix implements a simpler diffusion fill suited to small scratches.
- [Tesseract quality guide](https://tesseract-ocr.github.io/tessdoc/ImproveQuality.html) recommends rescaling, binarization, and noise removal before OCR.
- [Tesseract.js releases](https://github.com/naptha/tesseract.js/releases) document browser OCR and memory improvements. Actual wrapper/core versions were checked against installed npm packages.
- [UpscalerJS](https://upscalerjs.com/) supports TensorFlow.js models and patch-based processing. [MAXIM deblur](https://upscalerjs.com/models/available/deblurring/maxim-deblurring/) lists MIT package licensing and Apache-2.0 upstream licensing. It was not included because reliable low-memory classical processing better fits this single-file baseline.
- [ONNX Runtime large-model guide](https://onnxruntime.ai/docs/tutorials/web/large-models.html) describes browser ArrayBuffer and 4 GB WASM constraints. [WebGPU guide](https://onnxruntime.ai/docs/tutorials/web/ep-webgpu.html) describes hardware and browser dependencies. Depix does not assume WebGPU is available.
- Maintenance/license checks used the GitHub repository API for `naptha/tesseract.js`, `opencv/opencv`, `tensorflow/tfjs`, `thekevinscott/UpscalerJS`, and `microsoft/onnxruntime`. All were non-archived with 2026 pushes at research time. Tesseract/OpenCV/TensorFlow list Apache-2.0; UpscalerJS/ONNX Runtime list MIT. That is an activity check, not a promise of future support or an audit of every model weight.

## Design system

The [Costume Webapp Kit](https://github.com/arbabtalha/Costume-instructions) agent rules, relevant build/design/quality instructions, and single-file starter were read before implementation. Its local UI/UX search was executed for a photo-editing workstation and mobile canvas interactions. The output recommended a Swiss/minimal product-demo layout, tokenized styling, visible focus, modest motion, mobile-first layout, and reduced-motion support.

The app adapts that direction to a **charcoal / paper / lime** workspace instead of the suggested violet palette. CSS primitive, semantic, and component tokens are embedded in `index.html`. Typography pairs system sans with Georgia for display emphasis. Numeric labels use tabular figures. The design record lives here to preserve the requested three-file repository. No remote component-catalog tool was available; no component installation is claimed.

## Verification

The implementation was tested in headless Chromium with Playwright, including actual pixel comparisons and actual Tesseract recognition. Test tooling and screenshots were kept outside the published three-file app.

**58 end-to-end checks passed** (48 primary checks plus 10 extra input/touch/language/cancellation checks). Passed checks covered:

- Empty states; both upload buttons; PNG/JPEG/WebP/BMP input; large-input resize permission; drag/drop; clipboard image paste.
- Wrong type, oversize file, corrupt image, oversized output, invalid region, empty/large mask errors.
- Out-of-focus and motion deconvolution, angle setting, sharpening, denoise, color correction, and undo restoring exact pixels.
- Exact 2×/4× dimensions, safe pre-resize, reset/undo/redo, keyboard shortcuts, zoom and comparison.
- Continuous brush strokes, touch brush, erase, keyboard brush, rectangle mask, clear mask, and fill leaving every unmasked pixel unchanged.
- Text rectangle selection, grayscale contrast, adaptive thresholding, whole-image selection, and real English/Spanish/French/German OCR returning **NORTH SHORE** on the sample crop.
- Clipboard copy; actual PNG/JPG binary downloads; forced encoding failure and retry; worker, OCR, and export cancellation.
- Help/privacy dialogs, Escape, keyboard tool tabs, theme toggle, reduced motion, and no uncaught browser errors.
- No horizontal page overflow at **320 / 375 / 768 / 1024 / 1440** pixels.
- axe-core automated accessibility scans in both themes: **zero violations** in the exercised editor state. This is not a complete WCAG certification.
- OCR dependency-load failure and successful classic enhancement afterward.

The sandbox cannot reach jsDelivr directly. For successful OCR tests, the exact pinned CDN requests were mapped to the matching npm-distributed files; recognition was real, not mocked. Failure tests blocked those requests. Public-CDN delivery from a visitor's browser remains a network-dependent step.

JavaScript syntax checks, ESLint recommended rules, HTML validation, and formatting checks are used for this no-build vanilla app. There is no framework compilation step. A full strict TypeScript conversion, real-phone performance benchmarks, and cross-browser certification are not claimed.

## Publish with GitHub Pages

The public site is intended to serve `index.html` from `main` / root:

1. Merge the implementation pull request into `main`.
2. Open **Settings → Pages**.
3. Choose **Deploy from a branch → main → / (root) → Save**.
4. Wait for the Pages deployment to finish.
5. Open https://arbabtalha.github.io/Depix/.

## License

Depix is licensed under **MIT**. See [LICENSE](LICENSE). Optional OCR dependencies retain their Apache-2.0 and upstream notices. Use the app only with images you own or have permission to edit.
