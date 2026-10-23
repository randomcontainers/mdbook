# Extra license files

The build copies these files into the image, next to the license files that mdBook's tag and the crates ship.

`crates/<crate>-<version>/` holds texts that a crate lacks, and `scripts/cargo-licenses.py` adds them to the crate's own files in the image. The build stops when the binary records a crate without any license file, and when it records another version of a crate that has a directory here, so that the text is checked again before the directory is renamed. A directory named without a version, `crates/<crate>/`, is kept across versions instead; it needs an `EXPRESSION` file with the crate's license expression, and the build stops when the crate's expression changes.

`front-end/` holds the license texts of the files in mdBook's HTML theme that come from other projects or contain parts of them. [front-end/FILES](front-end/FILES) lists every file of the theme, pins those files by SHA-256 and names the directories with their licenses. Two of those directories are filled during the build: `fonts`, with `OPEN-SANS-LICENSE.txt` and `SOURCE-CODE-PRO-LICENSE.txt` from the tag, and `ace`, with the license at the top of `ace.js`. The build stops when FILES names a directory without a license file, or when a directory here is not named in FILES.

| Files | Source |
|---|---|
| `crates/font-awesome-as-a-crate-0.3.1/LICENSE.txt` | `fontawesome-free-6.2.0-desktop/LICENSE.txt` inside the crate, the Font Awesome Free license: CC BY 4.0 for the icons, which the crate embeds, OFL-1.1 for the fonts and MIT for the code, which it does not |
| `crates/font-awesome-as-a-crate-0.3.1/LICENSE` | `LICENSE` of [rust-lang/docs.rs](https://github.com/rust-lang/docs.rs), where the crate is developed, unchanged since commit b6e9041f; the crate's own code is MIT |
| `crates/font-awesome-as-a-crate-0.3.1/CC-BY-4.0` | the legal code of [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/legalcode.txt) |
| `front-end/highlight.js/LICENSE` | `LICENSE` of [highlightjs/highlight.js](https://github.com/highlightjs/highlight.js) at tag 10.1.1, the version of `js/highlight.js` |
| `front-end/clipboard.js/LICENSE` | `LICENSE` of [zenorocha/clipboard.js](https://github.com/zenorocha/clipboard.js), added in commit ce79f170; tag v2.0.4, the version of `js/clipboard.min.js`, says "Licensed MIT" in its header and README but has no license file |
| `front-end/elasticlunr/LICENSE` | `LICENSE` of [weixsong/elasticlunr.js](https://github.com/weixsong/elasticlunr.js) at tag v0.9.5, the version of `searcher/elasticlunr.min.js` |
| `front-end/mark.js/LICENSE` | `LICENSE` of [julmot/mark.js](https://github.com/julmot/mark.js) at tag 8.11.1, the version of `searcher/mark.min.js` |
| `front-end/tomorrow-theme/LICENSE.md` | `LICENSE.md` of [chriskempson/tomorrow-theme](https://github.com/chriskempson/tomorrow-theme) at commit ccf6666d; `css/tomorrow-night.css` and Ace's `theme-tomorrow_night.js` are based on this theme |
| `front-end/color-themes-for-highlightjs/LICENSE.md` | `LICENSE.md` of [jmblog/color-themes-for-highlightjs](https://github.com/jmblog/color-themes-for-highlightjs) at commit 7ff6ccfa, which `css/tomorrow-night.css` names as its source |
| `front-end/ayu/LICENSE` | `LICENSE` of [dempfi/ayu](https://github.com/dempfi/ayu) at commit 140aa8fc; `css/ayu-highlight.css` is based on this theme |
| `front-end/base16/LICENSE.md` | `LICENSE.md` of [chriskempson/base16](https://github.com/chriskempson/base16) at commit 5950a567; `css/highlight.css` names its color scheme as a source |
| `front-end/atelier-schemes/LICENSE` | the MIT license that the README of [atelierbram/syntax-highlighting](https://github.com/atelierbram/syntax-highlighting) links to, <https://atelierbram.mit-license.org/license.txt>, as served on 2026-10-23, with the year in the copyright line set to 2013, as the README gives it ("Copyright (c) 2013 Bram de Haan"); the site fills in the year of the request. `css/highlight.css` is loosely based on his Atelier Dune Light scheme |
| `front-end/octicons/LICENSE.txt` | `LICENSE.txt` of [primer/octicons](https://github.com/primer/octicons) at tag v2.0.0, which `css/chrome.css` names as the source of the copy button's icon: MIT for all files other than the font |
| `front-end/octicons/MIT` | the MIT license text from [choosealicense.com](https://choosealicense.com/licenses/mit/), which Octicons' `LICENSE.txt` links to, with the copyright line of that file |
| `front-end/font-awesome/CC-BY-4.0` | the legal code of [CC BY 4.0](https://creativecommons.org/licenses/by/4.0/legalcode.txt), the license that `images/favicon.svg` names for the image it is based on |
| `front-end/font-awesome/NOTICE` | the attribution for `images/favicon.svg` and `images/favicon.png`, with the copyright comment at the end of `favicon.svg` |
