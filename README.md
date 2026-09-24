# Snapmaker Orca with working headless CLI

Linux AppImage builds of [Snapmaker Orca](https://github.com/Snapmaker/OrcaSlicer) with two changes: the command-line slicer works without a GUI, and G-code generation for Klipper printers no longer recompiles regular expressions on every line.

Released Snapmaker Orca (2.3.6, 2.4.0) cannot slice from the command line:

- The version check compares project files against a legacy constant (`01.10.01.50`), so it rejects every project, including ones saved by Snapmaker Orca itself (`-24`).
- Past that check, the CLI crashes on a null GUI application object in `PartPlate::generate_plate_name_texture()` and `expand_plate_extruders()`.

[`patches/0001-pr839-cli-slice-fixes.patch`](patches/0001-pr839-cli-slice-fixes.patch) is upstream pull request [Snapmaker/OrcaSlicer#839](https://github.com/Snapmaker/OrcaSlicer/pull/839) by Kuzuri, applied unchanged to tag `v2.4.0`.

[`patches/0002-orca14166-static-regex.patch`](patches/0002-orca14166-static-regex.patch) backports upstream OrcaSlicer commit [`5ed8f5ef`](https://github.com/OrcaSlicer/OrcaSlicer/commit/5ed8f5ef258898a4006677bab8a3f2e412adedec) ([OrcaSlicer/OrcaSlicer#14166](https://github.com/OrcaSlicer/OrcaSlicer/pull/14166)) by Grant Harkness. Snapmaker v2.4.0 forked before it landed. `GCodeProcessor::process_SET_VELOCITY_LIMIT()` compiled three `std::regex` objects for every `SET_VELOCITY_LIMIT` line, and U1 G-code has one on roughly 5% of lines. On a four-colour mixed-filament U1 project, perf attributed 69% of G-code generation to that function. The patterns are now `static const`, exactly as upstream; the hunk for `process_SET_PRESSURE_ADVANCE()`, which v2.4.0 lacks, is omitted. The generated G-code is unchanged.

Nothing else is modified. Once Snapmaker releases a version containing both fixes, use the official build instead.

## Build

The [workflow](.github/workflows/build.yml) checks out `Snapmaker/OrcaSlicer` at `v2.4.0`, applies `patches/*.patch`, and runs Snapmaker's own `build_linux.sh` (`-ur`, `-dr`, `-isr`) on `ubuntu-24.04`. It then publishes the AppImage and `SHA256SUMS` as a release. Each release's notes name the exact upstream commit and workflow run.

It exists for a private cloud slicing service that runs the CLI headless under `xvfb-run` as an unprivileged user. Note that the AppImage's `libexec/snapmaker-orca-env` launcher ships as mode `0754` upstream; extract the AppImage and `chmod -R a+rX` it before running it as a different user.

## License

Snapmaker Orca is licensed under the GNU Affero General Public License v3.0 ([LICENSE](LICENSE)). The corresponding source for every release is the named upstream commit plus the patches in this repository.
