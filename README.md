# mdbook

Container images with [mdBook](https://rust-lang.github.io/mdBook/), which turns a directory of Markdown files into a book: a static website with a table of contents, full-text search and syntax highlighting, and a single printable page. `mdbook serve` previews the book and rebuilds it whenever a file changes. The images build mdBook from the source of each mdBook release with a pinned Rust release. They are built on Ubuntu and Alpine for `linux/amd64` and `linux/arm64`, and rebuilt when mdBook publishes a release and when the base image changes.

This is an unofficial build, not affiliated with or endorsed by the mdBook project. Report problems with the image in this repository and problems with mdBook itself in the [mdBook issue tracker](https://github.com/rust-lang/mdBook/issues).

## Quick start

Build the book in the current directory:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" ghcr.io/randomcontainers/mdbook
```

Without arguments the image runs `mdbook build`. It reads `book.toml` and `src/SUMMARY.md` in `/work` and writes the HTML to `book/`. Chapters that `SUMMARY.md` links to but that do not exist yet are created with a heading. Arguments replace the `build` command: `build --dest-dir public` writes the book to `public/` instead, and `--help` lists the other commands.

Start a new book in the current directory:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" ghcr.io/randomcontainers/mdbook init --title "My book" --ignore git
```

This writes `book.toml`, `src/SUMMARY.md`, a first chapter and a `.gitignore` that lists `book`. Without `--title` and `--ignore`, `mdbook init` asks for them on standard input; add `-it` to the `docker run` options to answer.

Preview the book at http://localhost:3000:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" -p 127.0.0.1:3000:3000 ghcr.io/randomcontainers/mdbook serve -n 0.0.0.0
```

`mdbook serve` builds the book, serves it and rebuilds it when a file in `src/` or `book.toml` changes, and the open page reloads itself. It listens on `localhost` unless told otherwise, which a published port does not reach, so `-n 0.0.0.0` is needed in a container. `-p 127.0.0.1:3000:3000` keeps the port on your own machine. It checks the files for changes once a second, which also works for bind mounts on Docker Desktop. Stop it with Ctrl-C. `mdbook watch` rebuilds the same way without serving.

## Preprocessors and renderers

`book.toml` can name preprocessors, which change the chapters before they are rendered, and renderers, which write other output formats. mdbook runs each as a command (`[preprocessor.mermaid]` runs `mdbook-mermaid`, `[output.epub]` runs `mdbook-epub`). The image has none of them, and no Rust toolchain to install them with `cargo install`, so a book that uses one fails to build unless the preprocessor or renderer is marked `optional = true`. Add the ones you need in your own image (see [Extending the slim image](#extending-the-slim-image)). They must support mdBook 0.5: 0.5.0 changed the protocol, and plugins written for 0.4 no longer work.

The built-in `html` and `markdown` renderers and the `links` and `index` preprocessors need nothing else. A renderer can also be a script, since the image has `sh`. mdbook runs a renderer in its output directory, `book/`, or `book/<name>/` when the book has more than one output, and writes the book and its configuration to its standard input as JSON. A program named with a path, such as `command = "./render.sh"`, is found relative to the book's root, so an executable script next to `book.toml` that starts with `#!/bin/sh` works. Paths in the arguments are relative to the output directory. Preprocessors run in the book's root.

`mdbook test` compiles the Rust code blocks of a book with `rustdoc`, which is not in the image.

## What is in the image

- `mdbook` in `/usr/local/bin`, built from the release tag with the default features (`watch`, `serve` and `search`) and with link-time optimization, as upstream's release builds are. It is linked against the distro's C library and libgcc_s: glibc on Ubuntu, musl on Alpine.
- mdBook's HTML theme, compiled into the binary: templates, CSS and JavaScript, the Open Sans and Source Code Pro fonts, highlight.js 10.1.1, the Ace editor 1.4.4 for editable code blocks, elasticlunr.js and mark.js for search, clipboard.js, and the Font Awesome Free icons. mdbook copies them into the books it builds.

Copying `/usr/local/bin/mdbook` into another image with `COPY --from` works only on a base with the same C library, at the same or a newer version, and libgcc_s. For a single file that runs on any Linux system, use the statically linked builds that mdBook attaches to each [release](https://github.com/rust-lang/mdBook/releases), `mdbook-v<version>-x86_64-unknown-linux-musl.tar.gz` and `mdbook-v<version>-aarch64-unknown-linux-musl.tar.gz`.

The bundled highlight.js 10.1.1 has two published advisories, [CVE-2020-26237](https://github.com/advisories/GHSA-vfrc-7r7c-w9mx) (fixed in 10.1.2) and [GHSA-7wwv-vh3v-89cq](https://github.com/advisories/GHSA-7wwv-vh3v-89cq) (fixed in 10.4.1). Both concern highlighting text from untrusted sources. The file is inside the mdbook binary, so image scanners do not report it, and it ends up in every book that uses the default theme. Upstream tracks replacing it in [rust-lang/mdBook#2467](https://github.com/rust-lang/mdBook/issues/2467).

`/usr/local/share/randomcontainers/mdbook/buildinfo` has the commit, the Rust release with the URL and SHA-256 of each archive it was installed from, the output of `rustc -vV`, the cargo-auditable version, the version mdbook reports, and every crate the binary was built from, with its version, whether it is linked in or used only during the build, and its license.

## Default or slim

mdBook's default image adds no other tools, so `latest` and `slim` are the same image, with the contents listed above. Use `latest` to run it and the `slim` tags as a base for your own image.

## Tags

`<version>` is an mdBook release such as `0.5.4`. `<minor>` and `<major>` are its shorter forms, `0.5` and `0`, and follow the newest release in that series. mdBook is below 1.0, and a release that raises the minor version can change the configuration and break plugins, as 0.5.0 did, so pin `<minor>` or an exact version rather than `<major>` or `latest`. Each row lists the default tag and its `slim` twin, which point to the same image.

| Tags | Base |
|---|---|
| `latest`, `slim` | Ubuntu |
| `<version>`, `<version>-slim` | Ubuntu |
| `<minor>`, `<minor>-slim`, `<major>`, `<major>-slim` | Ubuntu |
| `ubuntu`, `slim-ubuntu` | Ubuntu |
| `<version>-ubuntu`, `<version>-slim-ubuntu` | Ubuntu |
| `<minor>-ubuntu`, `<minor>-slim-ubuntu`, `<major>-ubuntu`, `<major>-slim-ubuntu` | Ubuntu |
| `<version>-ubuntu26.04`, `<version>-slim-ubuntu26.04` | Ubuntu 26.04 |
| `alpine`, `slim-alpine` | Alpine |
| `<version>-alpine`, `<version>-slim-alpine` | Alpine |
| `<minor>-alpine`, `<minor>-slim-alpine`, `<major>-alpine`, `<major>-slim-alpine` | Alpine |
| `<version>-alpine3.24`, `<version>-slim-alpine3.24` | Alpine 3.24 |

The images are currently built on Ubuntu 26.04 and Alpine 3.24. Tags without a distro version move to the next distro release when the project does; tags ending in `ubuntu26.04` or `alpine3.24` stay on that release and are no longer rebuilt once the project moves to the next one. Every tag of the current mdBook version, including the exact version, is rebuilt in place (see [Updates](#updates)), so pin a digest when you need the same bytes every time.

## Platforms

`linux/amd64` and `linux/arm64`, for both Ubuntu and Alpine. Each image contains mdbook built for its own architecture and is built natively on GitHub-hosted runners, without emulation.

## Files and permissions

The working directory is `/work`. The image runs as UID 1000, and any other UID works too: `HOME` is then `/`, and caches go to `/cache`, which anyone can write to. mdbook writes the book into `book/` next to `book.toml`, or into the directory that `--dest-dir` or `build.build-dir` names, and `mdbook init` writes into `/work`. Run it as your own user, so that it can write there and the files it creates belong to you:

| Runtime | Flag |
|---|---|
| Docker on Linux (rootful), GitHub Actions | `--user "$(id -u):$(id -g)"` |
| Rootless Podman | `--userns=keep-id` |
| Rootless Docker | `--user 0:0` (root in the container is your user on the host) |
| Docker Desktop on macOS or Windows | none, file ownership is mapped for you |

## Extending the slim image

Use a `slim` tag as the base for your own image. `slim`, `slim-ubuntu` and `slim-alpine` move to each new mdBook release and are rebuilt when the base image changes. This adds the [mdbook-mermaid](https://github.com/badboy/mdbook-mermaid) preprocessor from its statically linked release build for `linux/amd64`, checked against the SHA-256 that GitHub lists for the release asset:

```dockerfile
FROM ghcr.io/randomcontainers/mdbook:slim-ubuntu@sha256:...
USER root
ADD --checksum=sha256:<sha256> https://github.com/badboy/mdbook-mermaid/releases/download/v<version>/mdbook-mermaid-v<version>-x86_64-unknown-linux-musl.tar.gz /tmp/mdbook-mermaid.tar.gz
RUN tar -xzf /tmp/mdbook-mermaid.tar.gz --no-same-owner -C /usr/local/bin mdbook-mermaid \
 && rm /tmp/mdbook-mermaid.tar.gz
USER 1000:1000
```

Use the `aarch64-unknown-linux-musl` archive for `linux/arm64`, and `slim-alpine` for the same on Alpine. The entrypoint is `["tini", "--", "mdbook"]` and the command is `["build"]`; set your own `ENTRYPOINT` if your image runs a script. To pick up new mdBook releases and base image fixes, let Dependabot or Renovate update the digest in your `FROM` line.

Everything the image adds is under `/usr/local`. `/usr/local/share/randomcontainers/mdbook/` holds the version, the source URLs, the build details, the license files and `runtime-deps`, the list of distro packages mdbook needs at run time. It is empty on Ubuntu, and on Alpine it names musl and libgcc_s by their `so:` names.

## Verifying

Each image has a build provenance attestation from this repository's GitHub Actions run, signed by the shared build workflow in `randomcontainers/ci`:

```sh
gh attestation verify oci://ghcr.io/randomcontainers/mdbook:latest \
  --repo randomcontainers/mdbook --signer-repo randomcontainers/ci
```

Each platform image also carries an SPDX SBOM that lists every distro package with its version:

```sh
docker buildx imagetools inspect ghcr.io/randomcontainers/mdbook:latest --format '{{ json .SBOM }}'
```

mdbook is built with [cargo-auditable](https://github.com/rust-secure-code/cargo-auditable), which stores the list of crates compiled into it in the binary's `.dep-v0` section. Syft, Grype and Trivy read that list, so SBOM tools and image scanners see those crates. The Rust standard library is not in the list, and scanners do not report its advisories; `buildinfo` names the Rust release that built mdbook, and [Updates](#updates) says when it moves. Scanners do not see the JavaScript of the theme either (see [What is in the image](#what-is-in-the-image)). Follow mdBook's [security advisories](https://github.com/rust-lang/mdBook/security) as well.

mdBook publishes no source tarball, so each version is pinned to the commit of its tag. The build clones the `v<version>` tag of [rust-lang/mdBook](https://github.com/rust-lang/mdBook) and stops unless the tag points to the commit recorded in `package.yml`. cargo downloads the crates in the tag's `Cargo.lock` and checks each against the checksum there, and the build that follows has no network access. It also stops when upstream's `.cargo/config.toml`, which applies to the build, differs from the SHA-256 pinned in `cargo-config.sha256`, and when the theme gains or loses a file or a theme file with parts from another project differs from the SHA-256 pinned in [licenses/front-end/FILES](licenses/front-end/FILES).

Rust comes from static.rust-lang.org. The release manifest named in `package.yml` is checked against the SHA-256 recorded there and against its signature, made with the Rust release key (fingerprint `108F 6620 5EAE B0AA A8DD 5E1C 85AB 96E6 FA1B E5FE`, kept in [keys/rust.gpg](keys/rust.gpg)). Each archive installed from it is checked against the SHA-256 that the manifest lists. cargo-auditable, which runs only during the build, is compiled from its crates.io crate, checked against the SHA-256 in `package.yml`, with the crates listed in its own `Cargo.lock`.

## Updates

The project checks the releases of [rust-lang/mdBook](https://github.com/rust-lang/mdBook/releases) every 15 minutes. A release is picked up once it is 24 hours old; pre-releases are skipped. The new version and the commit of its tag are then committed to `package.yml` and the images are rebuilt. Only the newest release is built; tags of older versions stay as they were last built.

The Rust release that builds mdbook follows Rust's stable channel. Each new stable release is committed to `package.yml` once it is a day old, and the images are rebuilt with it. cargo-auditable is pinned by hand. An mdBook release that needs a newer Rust than the pinned one does not build until the pin moves. A release also waits for a person when it adds a crate under a license that the image's label does not cover, adds a crate without a license file, changes the version of the Font Awesome crate, adds or removes a theme file, changes a theme file that comes from another project or contains parts of one, or changes upstream's `.cargo/config.toml`.

The images of the current version are also rebuilt when the Ubuntu or Alpine base image changes and at least every 7 days, so distro security fixes reach the current tags. While a release waits for a person, the images of the current version are not rebuilt either: the new version is already in `package.yml`, and every rebuild builds it.

## Building

```sh
docker build -f Dockerfile.ubuntu --target slim \
  --build-arg VERSION=<version> \
  --build-arg SOURCE_COMMIT=<commit from package.yml> \
  --build-arg RUST_TOOLCHAIN_VERSION=<rust-toolchain version from package.yml> \
  --build-arg RUST_TOOLCHAIN_SHA256=<rust-toolchain sha256 from package.yml> \
  --build-arg CARGO_AUDITABLE_VERSION=<cargo-auditable version from package.yml> \
  --build-arg CARGO_AUDITABLE_SHA256=<cargo-auditable sha256 from package.yml> \
  -t mdbook:local .
```

Use `Dockerfile.alpine`, with the same arguments, for the Alpine image. The build downloads 115 to 135 MB of Rust archives on Ubuntu and 165 to 170 MB on Alpine, and the crates in `Cargo.lock`. `--build-arg JOBS=<n>` limits the number of crates cargo compiles in parallel.

## Licenses

mdBook is available under the [Mozilla Public License 2.0](https://github.com/rust-lang/mdBook/blob/main/LICENSE) (MPL-2.0). The binary also contains the Rust crates mdBook depends on. Most of them are under `MIT OR Apache-2.0` or MIT, and the others offer a choice of licenses that includes MIT, with these exceptions: num-modular, num-order and sync_wrapper are under Apache-2.0 and ryu under `Apache-2.0 OR BSL-1.0`; ego-tree, inotify and inotify-sys are under ISC; notify is under CC0-1.0; matchit includes code under BSD-3-Clause; unicode-ident includes Unicode data under Unicode-3.0; and font-awesome-as-a-crate is under MIT and embeds the Font Awesome Free icons. The Rust standard library and the crates it uses are linked in too. They are under `MIT OR Apache-2.0` or another choice that includes MIT, except compiler-builtins, which is under `MIT AND Apache-2.0 WITH LLVM-exception`.

The icons in mdBook's theme are from [Font Awesome Free](https://fontawesome.com) by Fonticons, Inc., licensed under the [Creative Commons Attribution 4.0 International License](https://creativecommons.org/licenses/by/4.0/) (CC-BY-4.0), except the copy button's icon, which is from GitHub's [Octicons](https://github.com/primer/octicons) (MIT). mdBook's favicon is a modified version of an image by Dave Gandy, the author of Font Awesome, also under CC BY 4.0. The rest of the theme includes the Open Sans fonts (Apache-2.0), the Source Code Pro font by Adobe (OFL-1.1), highlight.js and the Ace editor (BSD-3-Clause), clipboard.js, elasticlunr.js and mark.js (MIT), and highlighting color schemes based on Tomorrow, Ayu and Base16 Atelier Dune (MIT). Books built with the default theme contain these files and icons, under the same licenses.

The image's license label is `MPL-2.0 AND MIT AND Apache-2.0 AND BSD-3-Clause AND ISC AND CC0-1.0 AND Unicode-3.0 AND CC-BY-4.0 AND OFL-1.1 AND Apache-2.0 WITH LLVM-exception`, with MIT chosen wherever a crate offers it. The build stops when a crate linked into mdbook has a license that this label does not cover.

The license files are in `/usr/local/share/randomcontainers/mdbook/licenses/`: mdBook's `LICENSE` from the release tag, and the copy in each of its crates under `workspace/`; `crates/<crate>-<version>/` with the license files of every crate that the binary records, including those used only while it is built, and `crates/CRATES.tsv`, which lists each of them with its license; `rust/` with the license files of rust-lang/rust, the standard library and the crates it vendors, from the `rust-src` archive of the Rust release that built mdbook (all of them except LLVM's libunwind, which the dynamically linked binary does not contain); and `front-end/` with the licenses of the theme files that come from other projects or contain parts of them, and `FILES`, which names the license of each theme file. font-awesome-as-a-crate has the Font Awesome Free license only in a subdirectory, where the build does not look, and lacks the MIT license of its own code, which is in the docs.rs repository it comes from. This repository keeps both, with the CC BY 4.0 text, in [licenses](licenses), together with the license texts of the theme files that carry a short notice or none, and the build copies them into the image. The font licenses come from the release tag, and Ace's license from the header of `ace.js`. The Ubuntu and Alpine packages in the image keep their own licenses.

MPL-2.0 requires whoever distributes a binary to make its source available to its recipients. The source code for each image:

- mdBook: every version has a GitHub release in this repository, named `v<version>`, with `mdBook-<version>.tar.gz`, an archive of the commit its tag points to, together with the Rust release manifest and its signature and the cargo-auditable crate. The build applies no patches. `/usr/local/share/randomcontainers/mdbook/source` names the commit, the Rust release manifest, the `rust-src` archive with the source of the Rust standard library, the cargo-auditable crate and the release. The crates are on [crates.io](https://crates.io/) at the versions in that commit's `Cargo.lock`, and `buildinfo` lists those mdbook was built from.
- Build scripts: this repository at the commit in the image's `org.opencontainers.image.revision` label.
- Ubuntu packages: the source packages on [Launchpad](https://launchpad.net/ubuntu) for the versions listed in the SBOM. `apt-get source <package>=<version>` fetches a version that is still in the Ubuntu archive.
- Alpine packages: Alpine has no source packages. For the versions listed in the SBOM, the source is the APKBUILD and patches in [aports](https://gitlab.alpinelinux.org/alpine/aports/-/tree/3.24-stable), branch `3.24-stable`, and the archives on [distfiles.alpinelinux.org](https://distfiles.alpinelinux.org/distfiles/v3.24/).

The files in this repository are available under the MIT license, see [LICENSE](LICENSE), except the license texts in `licenses/`, which keep their own terms.

## Requesting a tool

To suggest another tool, use the [Request a tool](https://github.com/randomcontainers/.github/issues/new?template=tool-request.yml) form.
