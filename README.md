# Snapmaker Orca with working headless CLI

Linux AppImage builds of [Snapmaker Orca](https://github.com/Snapmaker/OrcaSlicer) with one change: the command-line slicer works without a GUI.

Released Snapmaker Orca (2.3.6, 2.4.0) cannot slice from the command line:

- The version check compares project files against a legacy constant (`01.10.01.50`), so it rejects every project, including ones saved by Snapmaker Orca itself (`-24`).
- Past that check, the CLI crashes on a null GUI application object in `PartPlate::generate_plate_name_texture()` and `expand_plate_extruders()`.

[`patches/0001-pr839-cli-slice-fixes.patch`](patches/0001-pr839-cli-slice-fixes.patch) is upstream pull request [Snapmaker/OrcaSlicer#839](https://github.com/Snapmaker/OrcaSlicer/pull/839) by Kuzuri, applied unchanged to tag `v2.4.0`. Nothing else is modified. Once Snapmaker releases a version containing that fix, use the official build instead.

## Build

The [workflow](.github/workflows/build.yml) checks out `Snapmaker/OrcaSlicer` at `v2.4.0`, applies `patches/*.patch`, and runs Snapmaker's own `build_linux.sh` (`-ur`, `-dr`, `-isr`) on `ubuntu-24.04`. It then publishes the AppImage and `SHA256SUMS` as a release. Each release's notes name the exact upstream commit and workflow run.

It exists for a private cloud slicing service that runs the CLI headless under `xvfb-run` as an unprivileged user. Note that the AppImage's `libexec/snapmaker-orca-env` launcher ships as mode `0754` upstream; extract the AppImage and `chmod -R a+rX` it before running it as a different user.

## License

Snapmaker Orca is licensed under the GNU Affero General Public License v3.0 ([LICENSE](LICENSE)). The corresponding source for every release is the named upstream commit plus the patches in this repository.
