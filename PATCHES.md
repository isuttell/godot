# isuttell/godot

Godot with the patches Isaac's projects need. Each `<version>-patched` branch starts at upstream's release of that version and carries one commit per patch, plus this file. A stable release starts at its `<version>-stable` tag. Upstream tags prereleases only in [godot-builds](https://github.com/godotengine/godot-builds/releases), so a beta or release candidate starts at the commit its official build names in `--version`: `4.8.beta2.official.cafc17a19` is `cafc17a19`.

## Patches

| Commit | Branches | Why | Upstream |
|---|---|---|---|
| macOS: Size the embedded OpenGL view when it's created | 4.7.2 on | Agents' Godot runs render offscreen in embedded mode, which left Compatibility projects (pandaboom, spaceminer) at 0x0 | Not sent. Upstream treats `--embedded` as editor-only ([#124020](https://github.com/godotengine/godot/pull/124020)), where the editor's resize hides the bug, and closed an AI-written fix for standalone embedded runs as spam ([#123721](https://github.com/godotengine/godot/pull/123721)) |
| macOS: Let --embedded run under any attached debugger | 4.8-beta2 on | From 4.8, `Main::setup` in `main/main.cpp` aborts `--embedded` unless it has the editor's `--remote-debug`, `--wid` and `--editor-pid` ([#124020](https://github.com/godotengine/godot/pull/124020)). The wrapper attaches the local debugger with `-d` and passes none of them, so every agent run printed `"--embedded" requires "--remote-debug <uri>"...` and quit. The check now asks only for a debugger, which the embedded display server can't start without, and still aborts when it fails to connect | Not sent, for the same reason as the patch above |

## Versions on this Mac

Projects pin their Godot in `.godot-version`. `~/.local/bin/godot` finds the pin above `--path`, or else above the working directory, and runs that version. Runs from a terminal and `--headless` runs use the official app; agent runs use the patched build.

| Version | Official app | Patched build |
|---|---|---|
| Homebrew's, which also serves projects with no pin | `/Applications/Godot.app` | `~/src/godot/bin/godot.macos.editor.arm64` |
| Any other | `/Applications/Godot_v<version>.app` | `~/src/godot-<version>/bin/godot.macos.editor.arm64`, built in a worktree of this clone |

The wrapper refuses a pin whose app isn't installed, and an agent run whose patched build reports another version.

## Build

From the version's checkout:

```sh
python3 misc/scripts/install_accesskit.py
python3 misc/scripts/install_angle.py
BUILD_NAME=isuttell scons platform=macos arch=arm64 target=editor vulkan=no angle=yes -j16
```

A prerelease also needs the status its official build reports, as in `GODOT_VERSION_STATUS=beta2 BUILD_NAME=isuttell scons ...`. Without it the build reports `4.8.beta` instead of `4.8.beta2`, and the wrapper refuses it.

This writes `bin/godot.macos.editor.arm64`. Vulkan is off because the MoltenVK SDK isn't installed; Metal and OpenGL cover every project.

## Adding a Godot version

A project that pins a version Homebrew doesn't install, such as a beta, gets its own app and worktree. Download the macOS arm64 build from [godot-builds](https://github.com/godotengine/godot-builds/releases), check it against that release's `SHA512-SUMS.txt`, and move its `Godot.app` to `/Applications/Godot_v<version>.app`. Then:

```sh
git fetch upstream tag <version>-stable --no-tags  # or, for a prerelease, git fetch upstream --no-tags
git worktree add -b <version>-patched ~/src/godot-<version> <version>-stable  # or the prerelease's commit
cd ~/src/godot-<version>
git cherry-pick <each patch commit, minus any upstream has merged>
git checkout <the newest patched branch> -- PATCHES.md && git commit -m "Describe the patches"
```

Then build, push the branch to `origin`, and update the table on every branch.

When Homebrew upgrades Godot.app, move `~/src/godot` to the new version's branch the same way, in place instead of in a worktree, and rebuild. A project still pinned to the old version needs that version added as above first.
