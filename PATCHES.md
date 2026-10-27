# isuttell/godot

Godot with the patches Isaac's projects need. Each `<version>-patched` branch starts at upstream's `<version>-stable` tag and carries one commit per patch, plus this file.

## Patches

| Commit | Why | Upstream |
|---|---|---|
| macOS: Size the embedded OpenGL view when it's created | Agents' Godot runs render offscreen in embedded mode, which left Compatibility projects (pandaboom, spaceminer) at 0x0 | Not sent. Upstream treats `--embedded` as editor-only ([#124020](https://github.com/godotengine/godot/pull/124020)), where the editor's resize hides the bug, and closed an AI-written fix for standalone embedded runs as spam ([#123721](https://github.com/godotengine/godot/pull/123721)) |

## Build

```sh
python3 misc/scripts/install_accesskit.py
python3 misc/scripts/install_angle.py
BUILD_NAME=isuttell scons platform=macos arch=arm64 target=editor vulkan=no angle=yes -j16
```

This writes `bin/godot.macos.editor.arm64`, which `~/.local/bin/godot` runs for every agent run. Runs from a terminal use the official Godot.app, and the wrapper refuses agent runs when the two versions differ. Vulkan is off because the MoltenVK SDK isn't installed; Metal and OpenGL cover every project.

## Moving to a new Godot release

After Homebrew installs the new Godot:

```sh
git fetch upstream tag <version>-stable --no-tags
git checkout -b <version>-patched <version>-stable
git cherry-pick <each patch commit, minus any upstream has merged>
git checkout 4.7.2-patched -- PATCHES.md && git commit -m "Describe the patches"
```

Then build, push the branch to `origin`, and update the table.

### 4.8 needs one more patch

From 4.8, `Main::setup` in `main/main.cpp` aborts `--embedded` unless it has the editor's `--remote-debug`, `--wid` and `--editor-pid`, and a debug server connection ([#124020](https://github.com/godotengine/godot/pull/124020)). The wrapper runs `--embedded` with the local debugger (`-d`) and none of those, so every agent run would print `"--embedded" requires "--remote-debug <uri>"...` and quit. The 4.8 branch needs a patch that lets `--embedded` run with any attached debugger, the local one included, and still aborts without one, since the embedded display server can't start without a debugger.
