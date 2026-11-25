personal site built with Astro and deployed via Cloudflare workers

## CV

The CV served at `/cv.pdf` is generated from `cv/cv.html`. Edit the HTML, then run:

```bash
npm run cv
```

It prints the page to `public/cv.pdf` with headless Chrome, so it needs Google Chrome installed in `/Applications` (macOS). Commit both files.
