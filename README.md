# NCreate edition manifests

This repository is the publication point for the official NCreate Minimal, Standard, and Ultra editions. It contains the [version 1 JSON schema](schema/official-edition-v1.schema.json), a release builder, checks, and an automated publishing workflow. **No official edition has been released yet.** Files under `tests/fixtures/` are synthetic test data and are never published as packs.

The launcher reads `https://raw.githubusercontent.com/Yozekkk/ncreate-manifests/main/channels/{stable|beta}/{minimal|standard|ultra}.json`. The six channel JSON files are intentionally absent until a real pack is published. HTTP 404 means that edition is not available. An empty or sample manifest would incorrectly advertise an installable edition.

## Manifest and release format

Each published channel JSON is a complete v1 manifest. `schemaVersion`, edition `id`, pack `version`, Minecraft and loader versions, Java version, memory limits, `releaseChannel`, and `files` are required. Each file has a relative `path`, HTTPS `url`, byte `size`, and SHA-256 digest; `required` and `updatePolicy` control installation. The launcher applies its own validation and protects user data such as saves and screenshots. The publisher also rejects protected paths, executable file extensions, symlinks, duplicate paths, remote launch arguments, oversized packs, and non-HTTPS URLs.

The schema describes the wire format; it does **not** prove that files are licensed, compatible, malware-free, or an official finished pack. Review content and redistribution rights before adding it. A SHA-256 digest detects changes to a specific file; it does not establish who supplied that file. The published channel pointer and release assets remain under control of this repository's maintainers.

## Publishing a real pack

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

If a workflow fails before the channel commit, the channel remains on its prior version. A draft or published release may need maintainer inspection before retrying; do not edit an already published version in place. A failed push after the release is published leaves that release available by tag but undiscoverable to the launcher until the channel is advanced.

See [the launcher update documentation](https://github.com/Yozekkk/ncreate-launcher/blob/main/docs/UPDATES.md) for the client behavior and channel URLs.
