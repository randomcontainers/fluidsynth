# fluidsynth

Container images with [FluidSynth](https://www.fluidsynth.org/), a software synthesizer that plays MIDI files with the instruments of a SoundFont. `fluidsynth` renders a MIDI file to a WAV, FLAC or Ogg Vorbis file, or writes raw samples to stdout for another program. It needs a SoundFont, which is not in the image. The default image also contains FFmpeg, for MP3, Opus, AAC and other formats. The images are rebuilt when FluidSynth publishes a release and when the base image changes, for `linux/amd64` and `linux/arm64`.

This is an unofficial build, not affiliated with or endorsed by the FluidSynth project. Report problems with the image in this repository and problems with FluidSynth itself [upstream](https://github.com/FluidSynth/fluidsynth/issues).

## Quick start

Render a MIDI file with a SoundFont, both in the current directory:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" \
  ghcr.io/randomcontainers/fluidsynth -ni -F song.wav FluidR3_GM.sf2 song.mid
```

`-F` renders the MIDI file as fast as the CPU allows instead of in real time and writes the audio to `song.wav`, as 16-bit stereo at 44.1 kHz. The audio runs on for about 2 seconds after the last note ends, so that reverb can fade out. The file type follows the extension, so `-F song.flac` writes FLAC. For Ogg Vorbis, pass `-T oga`: a name ending in `.ogg` alone gives a WAV file. `-r` sets the sample rate, `-O` the sample format (`s24`, `s32`, `float` and others), and `-g` the gain, which is 0.2 by default; raise it when the result is too quiet:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" \
  ghcr.io/randomcontainers/fluidsynth -ni -r 48000 -g 0.6 -F song.flac FluidR3_GM.sf2 song.mid
```

Several MIDI files are played one after another into the same output file. `-R 0` and `-C 0` turn reverb and chorus off, and `-R` with `fdn`, `free` or `lex` picks another reverb than the default `dat`. `-o name=value` changes any of the [settings](https://www.fluidsynth.org/api/fluidsettings.html), and `-o help` lists them with their ranges and defaults. Upstream's [user manual](https://www.fluidsynth.org/wiki/UserManual/) describes the other options.

`-F -` writes the audio to stdout. Add `-q`, or fluidsynth also prints its version banner and a few status lines into the audio. Use raw samples on stdout (`-T raw`, with `-E little` for the byte order), since a WAV header cannot be completed on a pipe. In the default image, FFmpeg encodes them:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" --entrypoint sh \
  ghcr.io/randomcontainers/fluidsynth -c \
  'fluidsynth -q -ni -r 44100 -T raw -E little -F - FluidR3_GM.sf2 song.mid | ffmpeg -loglevel error -f s16le -ar 44100 -ac 2 -i - -b:a 192k song.mp3'
```

## SoundFonts

The images contain no SoundFont. FluidSynth's [SoundFont page](https://www.fluidsynth.org/wiki/SoundFont/) lists two General MIDI sets that work well with it, both with all 128 instruments and a drum kit: [GeneralUser GS](https://www.schristiancollins.com/generaluser.php) by S. Christian Collins, and FluidR3_GM, which Debian and Ubuntu package as `fluid-soundfont-gm`. Check the terms of any SoundFont you use. SoundFont 3 files (`.sf3`, with compressed samples) and DLS files (`.dls`) are given the same way.

When no SoundFont is named, fluidsynth loads `/usr/local/share/soundfonts/default.sf2`. Mount one there, read-only:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD:/work" \
  -v "$HOME/soundfonts/FluidR3_GM.sf2:/usr/local/share/soundfonts/default.sf2:ro" \
  ghcr.io/randomcontainers/fluidsynth -ni -F song.wav song.mid
```

## What is in the image

| Area | Details |
|---|---|
| Program | `fluidsynth`, and the library `libfluidsynth.so.3` in `/usr/local/lib` |
| Instruments | SoundFont 2, SoundFont 3 and DLS files |
| Input | Standard MIDI files |
| Output | WAV, FLAC, Ogg Vorbis, raw samples and the other formats of libsndfile (`-T help` lists them), as 8 to 32-bit integer or floating point samples, at 8 to 96 kHz |
| Audio driver | `file` only: the image cannot play to a sound card |
| Default image | FFmpeg (`ffmpeg` and `ffprobe`), for MP3, Opus, AAC and other formats |

Not included: SoundFonts, sound card and MIDI input drivers (ALSA, PulseAudio, PipeWire, JACK and the others), line editing in the interactive shell, LADSPA effects, the output limiter and the `smith` reverb engine (both need Signalsmith Audio's libraries), the headers, pkg-config and CMake files of the library, and the man page. The commit, the gcem version and the build options are in `/usr/local/share/randomcontainers/fluidsynth/buildinfo`.

## Default or slim

Use the default image (`latest`) for general work. It adds FFmpeg, so a MIDI file becomes an MP3, Opus or AAC file in one container (see [Quick start](#quick-start)). `slim` has FluidSynth and the libraries it needs, without FFmpeg. Use it to build your own image, or when WAV, FLAC or Ogg Vorbis is all you need.

The default image includes FFmpeg, which is licensed under the GNU General Public License (GPL-3.0-or-later). Use `slim` if your policy excludes GPL. The default image is also published as `ghcr.io/randomcontainers/fluidsynth-ffmpeg`, built in the [fluidsynth-ffmpeg](https://github.com/randomcontainers/fluidsynth-ffmpeg) repository with the same contents and a different digest.

## Tags

`<version>` is a FluidSynth release such as `2.6.1`. `<minor>` and `<major>` are its shorter forms, `2.6` and `2`, and follow the newest release in that series.

| Default (with FFmpeg) | Slim | Base |
|---|---|---|
| `latest`, `<version>`, `<minor>`, `<major>` | `slim`, `<version>-slim`, `<minor>-slim`, `<major>-slim` | Ubuntu |
| `ubuntu`, `<version>-ubuntu`, `<minor>-ubuntu`, `<major>-ubuntu` | `slim-ubuntu`, `<version>-slim-ubuntu`, `<minor>-slim-ubuntu`, `<major>-slim-ubuntu` | Ubuntu |
| `<version>-ubuntu26.04` | `<version>-slim-ubuntu26.04` | Ubuntu 26.04 |
| `alpine`, `<version>-alpine`, `<minor>-alpine`, `<major>-alpine` | `slim-alpine`, `<version>-slim-alpine`, `<minor>-slim-alpine`, `<major>-slim-alpine` | Alpine |
| `<version>-alpine3.24` | `<version>-slim-alpine3.24` | Alpine 3.24 |

The images are currently built on Ubuntu 26.04 and Alpine 3.24. Tags without a distro version move to the next distro release when the project does; tags ending in `ubuntu26.04` or `alpine3.24` stay on that release and are no longer rebuilt once the project moves to the next one. Every tag of the current FluidSynth version, including the exact version, is rebuilt in place (see [Updates](#updates)), so pin a digest when you need the same bytes every time.

## Platforms

`linux/amd64` and `linux/arm64`, for both Ubuntu and Alpine. Both are compiled natively on GitHub-hosted runners, without emulation.

## Files and permissions

The working directory is `/work`. The image runs as UID 1000, and any other UID works too: `HOME` is then `/`, and caches go to `/cache`, which anyone can write to. How to get output files owned by you depends on how you run containers:

| Runtime | Flag |
|---|---|
| Docker on Linux (rootful), GitHub Actions | `--user "$(id -u):$(id -g)"` |
| Rootless Podman | `--userns=keep-id` |
| Rootless Docker | `--user 0:0` (root in the container is your user on the host) |
| Docker Desktop on macOS or Windows | none, file ownership is mapped for you |

## Untrusted files

FluidSynth parses MIDI, SoundFont and DLS files. Release 2.5.6 fixed six [security advisories](https://github.com/FluidSynth/fluidsynth/security/advisories) in its MIDI player, its SoundFont and DLS loaders and its command shell, and these images are built from the newest release. libsndfile and the other libraries from Ubuntu or Alpine get their fixes through the rebuilds described in [Updates](#updates). Rendering does not use the network. For files from unknown sources, mount the input read-only, write to a separate directory and take away what the container does not need:

```sh
docker run --rm --user "$(id -u):$(id -g)" -v "$PWD/uploads:/in:ro" -v "$PWD/out:/work" \
  --network none --read-only --cap-drop ALL --security-opt no-new-privileges --memory 1g \
  ghcr.io/randomcontainers/fluidsynth -ni -F upload.wav /in/FluidR3_GM.sf2 /in/upload.mid
```

## Extending the slim image

Use a `slim` tag as the base for your own image. It has no FFmpeg, so FFmpeg updates do not rebuild it. The packages FluidSynth needs are listed in `/usr/local/share/randomcontainers/fluidsynth/runtime-deps`. This image adds Ubuntu's FluidR3_GM SoundFont as the default one:

```dockerfile
FROM ghcr.io/randomcontainers/fluidsynth:slim-ubuntu@sha256:...
USER root
RUN apt-get update \
 && apt-get install -y --no-install-recommends fluid-soundfont-gm \
 && rm -rf /var/lib/apt/lists/* \
 && mkdir -p /usr/local/share/soundfonts \
 && ln -s /usr/share/sounds/sf2/FluidR3_GM.sf2 /usr/local/share/soundfonts/default.sf2
USER 1000:1000
```

On Alpine, `apk add --no-cache soundfont-timgm` installs the smaller TimGM6mb set as `/usr/share/soundfonts/TimGM6mb.sf2`. The entrypoint is `["tini", "--", "fluidsynth"]`; set your own `ENTRYPOINT` if your image runs something else. To pick up new FluidSynth releases and base image fixes, let Dependabot or Renovate update the digest in your `FROM` line.

## Verifying

Each image has a build provenance attestation from this repository's GitHub Actions run, signed by the shared build workflow in `randomcontainers/ci`:

```sh
gh attestation verify oci://ghcr.io/randomcontainers/fluidsynth:latest \
  --repo randomcontainers/fluidsynth --signer-repo randomcontainers/ci
```

Images from `ghcr.io/randomcontainers/fluidsynth-ffmpeg` are built in that repository, so verify them with `--repo randomcontainers/fluidsynth-ffmpeg` and the same `--signer-repo`.

Each platform image also carries an SPDX SBOM that lists every distro package with its version:

```sh
docker buildx imagetools inspect ghcr.io/randomcontainers/fluidsynth:latest --format '{{ json .SBOM }}'
```

FluidSynth attaches prebuilt binaries to its releases but no source archive, and does not sign its tags, so each version is pinned to the commit of its tag. The build clones the `v<version>` tag of [FluidSynth/fluidsynth](https://github.com/FluidSynth/fluidsynth) and stops unless the tag points to the commit recorded in `package.yml`, so a tag that is moved after the version was picked up fails the build. FluidSynth takes gcem from a git submodule; the build downloads GitHub's archive of the gcem release instead, checks it against the SHA-256 in `package.yml`, and stops unless the commit stored in the archive is the one FluidSynth records for the submodule. The build also runs FluidSynth's unit tests.

## Updates

The project checks the releases of [FluidSynth/fluidsynth](https://github.com/FluidSynth/fluidsynth/releases) every 15 minutes. A release is picked up once it is 24 hours old. The new version and the commit of its tag are then committed to `package.yml` and the images are rebuilt. Only the newest release is built; tags of older versions stay as they were last built, and a release of an older series, such as a 2.5.x after 2.6.0, is not built.

The images of the current version are also rebuilt when the Ubuntu or Alpine base image changes, the default ones when a new FFmpeg image is published, and all of them at least every 7 days, so distro security fixes reach the current tags. gcem is pinned by hand in `package.yml`. When a FluidSynth release records a new gcem commit, its build stops at the gcem check. Until the gcem version and SHA-256 in `package.yml` are updated, no image is built, and the existing tags get no rebuilds either.

## Building

```sh
docker build -f Dockerfile.ubuntu --target slim \
  --build-arg VERSION=<version> \
  --build-arg SOURCE_COMMIT=<commit from package.yml> \
  --build-arg GCEM_VERSION=<gcem version from package.yml> \
  --build-arg GCEM_SHA256=<gcem sha256 from package.yml> \
  -t fluidsynth:local .
```

Use `Dockerfile.alpine` for the Alpine image. `--build-arg JOBS=<n>` limits the number of parallel compile jobs. The default image is generated from the `combos` entry in `package.yml` by [randomcontainers/ci](https://github.com/randomcontainers/ci).

## Licenses

FluidSynth is licensed under the GNU Lesser General Public License, version 2.1 or later (LGPL-2.1-or-later). Its Freeverb reverb is in the public domain, and `fluid_dls.cpp` contains a string formatting helper under CC0-1.0. FluidSynth computes its lookup tables at build time with gcem, a header-only C++ library under the Apache License 2.0. `/usr/local/share/randomcontainers/fluidsynth/licenses/` holds FluidSynth's `LICENSE`, gcem's license and `NOTICE.txt` (`LICENSE.gcem`, `NOTICE.gcem`), and `NOTICES`, with the Freeverb and CC0-1.0 notices cut from the source files. Links to the source are in `/usr/local/share/randomcontainers/fluidsynth/source`. The image's license label is `LGPL-2.1-or-later AND Apache-2.0 AND CC0-1.0 AND LicenseRef-Public-Domain`, where the `LicenseRef-` entry stands for Freeverb.

Upstream links readline (GPL-3.0-or-later) by default for line editing in the interactive shell. These images are built without it.

SoundFonts are not part of the images and come with their own licenses.

The corresponding source for each image:

- FluidSynth: every version has a GitHub release in this repository, named `v<version>`, with `fluidsynth-<version>.tar.gz`, a `git archive` of the commit that was compiled, and the gcem archive the build used, named after the gcem release (`v<gcem version>.tar.gz`). `/usr/local/share/randomcontainers/fluidsynth/source` lists that release, the commit on GitHub and the gcem archive.
- Build scripts: this repository at the commit in the image's `org.opencontainers.image.revision` label. The Dockerfiles hold every build option.
- Ubuntu packages: the source packages on [Launchpad](https://launchpad.net/ubuntu) for the versions listed in the SBOM. `apt-get source <package>=<version>` fetches a version that is still in the Ubuntu archive.
- Alpine packages: Alpine has no source packages. For the versions listed in the SBOM, the source is the APKBUILD and patches in [aports](https://gitlab.alpinelinux.org/alpine/aports/-/tree/3.24-stable), branch `3.24-stable`, and the archives on [distfiles.alpinelinux.org](https://distfiles.alpinelinux.org/distfiles/v3.24/).

The default image adds FFmpeg, licensed under GPL-3.0-or-later; its license files and corresponding source are described in the [ffmpeg](https://github.com/randomcontainers/ffmpeg#licenses) repository. libsndfile, libgomp, libstdc++ and the other libraries from Ubuntu or Alpine keep their own licenses. The SBOM lists them.

SoundFont is a registered trademark of Creative Technology Ltd. The files in this repository are available under the MIT license, see [LICENSE](LICENSE).

## Requesting a tool

To suggest another tool, use the [Request a tool](https://github.com/randomcontainers/.github/issues/new?template=tool-request.yml) form.
