<div align="center">
  <h1>NCreate edition manifests</h1>
  <p>Schema, validation, and publication tooling for proposed Minimal, Standard, and Ultra editions.</p>
  <p>
    <a href="https://github.com/Yozekkk/ncreate-manifests/actions/workflows/check.yml"><img src="https://github.com/Yozekkk/ncreate-manifests/actions/workflows/check.yml/badge.svg" alt="Validate manifests"></a>
    <img src="https://img.shields.io/badge/Node.js-22-339933" alt="Node.js 22 in CI">
    <a href="LICENSE"><img src="https://img.shields.io/badge/license-GPLv3-555" alt="GPLv3 license"></a>
  </p>
</div>

This repository contains the publication pipeline for proposed NCreate Minimal, Standard, and Ultra editions: a [version 1 JSON schema](schema/official-edition-v1.schema.json), release builder, validation scripts, tests, and GitHub Actions. **No edition from this repository has been released or activated.** Files under `tests/fixtures/` are synthetic test data and are never published as packs.

The six potential `channels/{stable|beta}/{minimal|standard|ultra}.json` pointers are absent until reviewed packs are published. HTTP 404 means that an edition is unavailable; a sample manifest would incorrectly advertise an installable edition. **The current [NCreate Launcher](https://github.com/Yozekkk/ncreate-launcher) does not consume these pointers.** Its published NCreate Server edition currently comes from [ncreate-pack](https://github.com/Yozekkk/ncreate-pack). Before activating this pipeline for users, maintainers must integrate its channels into the launcher and test that integration.

## Repository map

```text
schema/               JSON Schema for the v1 wire format
packs/                Location for reviewed edition source directories
channels/             Stable and beta pointers after publication
scripts/              Build, validate, verify release, activate channel
tests/                Publisher tests and synthetic pack fixtures
.github/workflows/     Validation and manual publication workflows
```

The repository uses Node.js scripts, npm, AJV, and JSON Schema. There is no desktop application or development server here.

## Manifest and release format

Each published channel JSON is a complete v1 manifest. `schemaVersion`, edition `id`, pack `version`, Minecraft and loader versions, Java version, memory limits, `releaseChannel`, and `files` are required. Each file has a relative `path`, HTTPS `url`, byte `size`, and SHA-256 digest; `required` and `updatePolicy` control installation. The launcher applies its own validation and protects user data such as saves and screenshots. The publisher also rejects protected paths, executable file extensions, symlinks, duplicate paths, remote launch arguments, oversized packs, and non-HTTPS URLs.

The schema describes the wire format; it does **not** prove that files are licensed, compatible, malware-free, or an official finished pack. Review content and redistribution rights before adding it. A SHA-256 digest detects changes to a specific file; it does not establish who supplied that file. The published channel pointer and release assets remain under control of this repository's maintainers.

## Local validation

CI runs on Node.js 22. Install the locked dependencies and run the same checks locally:

```bash
git clone https://github.com/Yozekkk/ncreate-manifests.git
cd ncreate-manifests
npm ci --ignore-scripts
npm run check
```

`npm run check` runs the Node.js test suite and validates any published channel files. `npm test` runs only the tests. With no channel JSON files, validation reports zero published manifests; that is the expected current state.

## Publishing a reviewed pack

Only maintainers should add a reviewed pack under `packs/<edition>/<version>/`:

```text
packs/standard/1.0.0/
  edition.json
  files/
    mods/example.jar
    config/example.toml
  file-policies.json       # optional
```

`edition.json` contains manifest fields except `files` and `releaseChannel`, which the builder generates. It must include a meaningful `changelog`. The optional `file-policies.json` maps relative file paths to `required` and/or `updatePolicy` (`managed_only` or `preserve`). All files in `files/` become release assets. Do not add a channel JSON by hand.

1. Review the pack contents, licenses, Minecraft/loader compatibility, and release notes. Commit the reviewed pack source to `main`.
2. Run `npm ci --ignore-scripts` and `npm run check` locally. The `Validate manifests` workflow repeats this for pushes and pull requests.
3. Start `Publish official pack` from GitHub Actions on `main`, selecting the edition, channel, and exact version directory.
4. The workflow builds the manifest and release assets, calculates each payload's SHA-256 and `SHA256SUMS.txt`, creates a draft GitHub Release, uploads the assets, and compares the remote asset names, byte sizes, and GitHub SHA-256 digests with the local files. Only after that check does it publish the release and advance one channel JSON on `main`. A channel version cannot move backwards or be replaced in place.

The versioned download URLs have the form `https://github.com/Yozekkk/ncreate-manifests/releases/download/pack-<edition>-<channel>-v<version>/file-<sha256-of-relative-path>.<extension>`. The manifest stores the SHA-256 of the **file contents**, not the digest in the asset name. `SHA256SUMS.txt` also includes the generated manifest and can be checked after downloading all release assets with `sha256sum -c SHA256SUMS.txt`.

If a workflow fails before the channel commit, the channel remains on its prior version. A draft or published release may need maintainer inspection before retrying; do not edit an already published version in place. A failed push after the release is published leaves that release available by tag but undiscoverable through the channel pointer until it is advanced.

The workflow in [.github/workflows/publish-pack.yml](.github/workflows/publish-pack.yml) implements this sequence. [Launcher update documentation](https://github.com/Yozekkk/ncreate-launcher/blob/main/docs/UPDATES.md) describes the **currently integrated** NCreate Server channel in `ncreate-pack`, not this inactive edition pipeline.

## License

The repository is distributed under [GPLv3](LICENSE). The fixture files are tests, not a downloadable Minecraft edition.
