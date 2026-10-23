# OmniTools in the Tofu gallery

OmniTools is a toolbox of more than 100 small everyday tools that run in the browser: image tools (resize, crop,
compress, convert, remove a background, read the text in an image), video and audio tools (trim, convert, turn a
video into a GIF), PDF tools (merge, split, compress, rotate, PDF to images), and tools for text, lists, JSON, CSV,
XML, numbers, dates and units. Files are processed on the visitor's own device and are not uploaded anywhere. There
is no account and no server side: the language, the theme, bookmarked tools and the "user type" filter are kept in
the browser's localStorage.

## How Tofu runs it

| | |
|---|---|
| Framework preset | `vite` |
| Package manager | npm, with the committed `package-lock.json` (lockfile version 3). Upstream's CI uses Node 20; verified here with Node 22.22.3 and npm 10.9.8 |
| Install | `npm ci` |
| Build | `npm run build` (runs `tsc && vite build`) |
| Output directory | `dist` (entry page `dist/index.html`) |
| Routes | client-side paths such as `/pdf/merge-pdf` and `/string/reverse`; `vercel.json` serves `index.html` for every path that is not a file, as upstream's `public/_redirects` does on Netlify and Cloudflare Pages |
| Environment variables | none |
| Database | none |

The build output is about 47 MB in 387 files. The largest file is the ONNX Runtime WebAssembly module the
background remover uses, `dist/assets/ort-wasm-simd-threaded.jsep-*.wasm`, at 22.8 MB.

`.env.example` names `LOCIZE_API_KEY`. Only the maintainers' translation scripts (`npm run i18n:pull`,
`npm run i18n:sync`) use it; the app never reads it and loads its translations from its own `/locales` folder.

## What reaches other sites

There are no analytics, tracking or advertising scripts, and nothing needs a key or an account.

- On every page: the header's GitHub star button is an iframe from `ghbtns.com`, which asks `api.github.com` for the
  star count, and the icons are fetched from the Iconify API (`api.iconify.design`).
- Only when a person uses one of these tools: the video and audio tools, rotating an image and resizing a GIF load
  FFmpeg from `unpkg.com`; compressing or password-protecting a PDF loads a Ghostscript WebAssembly file from
  `cdn-wasm.b-cdn.net`; background removal downloads its model from IMG.LY's `staticimgly.com`; and image to text
  (OCR) downloads Tesseract and its language data from jsDelivr. Without those sites, only that tool stops working.
- Links that open only when clicked: the upstream Discord, the "Help translate" invitation on Locize, and the
  upstream GitHub repository (the version label next to the logo links to its tree).

## What we changed, and why

The name, logo, favicon and every tool are unchanged. Every change is a commit on the `tofu` branch:

1. **`vite.config.ts` reads the commit hash safely.** It ran `git rev-parse --short HEAD` when it loaded and threw
   when there was no git history, which stopped the build. Tofu builds a copy from a source archive, which has no
   `.git` folder, so the unchanged build fails there. It now uses git when it is there, else the host's
   `VERCEL_GIT_COMMIT_SHA`, else `main`; a copy built from an archive shows "v0.7.0 (main)" next to the logo, and
   that label links to the upstream repository's `main` branch.
2. **`vercel.json` added.** It names the install, build and output above and rewrites every path that is not a file
   to `/index.html`, so a reload or a shared link to a tool opens that tool instead of a 404.
3. **`package-lock.json` locks `nerdamer-prime` to 1.2.5.** Upstream regenerated its lockfile on 2026-10-23, which
   moved `nerdamer-prime` (the algebra library behind the formula calculators, such as Ohm's law) from 1.2.5 to
   1.5.0. Version 1.5.0 renamed a type the calculator code uses, so `tsc` stopped the build; upstream's own CI build
   fails on the same commits. The lock entry is now exactly the one in the v0.7.0 release lockfile, and two packages
   only 1.5.0 needed (`big-integer`, `decimal.js`) are gone. `package.json` is unchanged. Drop this commit once
   upstream's build passes again.
4. **`TOFU.md`.** This file.

Verified from a copy of the `tofu` branch with no `.git` folder: `npm ci` and `npm run build` succeeded, and
`vite preview` answered 200 for `/`, `/pdf/merge-pdf`, `/locales/en/translation.json` and the 22.8 MB WebAssembly
file. In a headless browser the home page rendered with no errors, the reverse tool turned "hello" into "olleh", and
the Ohm's law calculator solved R = 5 from V = 5 and I = 1.

## Upstream and licence

- Upstream: <https://github.com/iib0011/omni-tools>, branch `main`, commit
  `60bc64f5a6974f3c802c5ec06cb3cf7f824485e1` (package version 0.7.0).
- Licence: MIT, Copyright (c) 2024 Ibrahima Gaye Coulibaly. See `LICENSE`, kept exactly as upstream ships it.
