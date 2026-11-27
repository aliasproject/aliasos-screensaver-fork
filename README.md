# aliasos-screensaver

The [aliasOS](https://github.com/aliasproject/aliasos) screensaver: a native Wayland overlay on every display that animates the aliasOS mark with [ttfx](https://github.com/omacom/ttfx) effects. A key press, click, scroll or mouse movement dismisses it.

It replaces aliasOS's earlier screensaver, which opened a Ghostty window on each display and ran ttfx inside it. A layer surface needs none of the window workarounds that took: closing menus and pickers first, unpinning picture-in-picture windows, walking focus across displays, keyboard-only dismissal.

## In aliasOS

aliasOS installs it as the `aliasos-screensaver` package, at `/usr/lib/aliasos-screensaver/aliasos-screensaver`, and starts it through `aliasos-launch-screensaver`, both when the session goes idle and from the System menu. It:

- draws the text in `~/.config/aliasos/branding/screensaver.txt`, falling back to the bundled `assets/screensaver.txt` (the aliasOS mark)
- stays off while the `screensaver-off` toggle is set (`aliasos-toggle-enabled screensaver-off`), or while the session is locked (`aliasos-shell lock isLocked`)
- maps one layer surface per display under the namespace `org.aliasos.screensaver`
- runs one instance at a time, and on Hyprland hides the cursor while it is up and restores it when it exits
- appears and disappears at once, exiting on input, `SIGTERM`, `SIGINT`, `SIGHUP` or `SIGQUIT`

## How it draws

ttfx runs in process at its own 120 Hz clock. A frame is drawn only when cells changed, the 120 Hz deadline passed and the compositor has released the previous buffer, and then only the changed cells are redrawn and damaged. A fully opaque surface also tells the compositor it can skip blending the desktop underneath. [LEARNINGS.md](LEARNINGS.md) has the details.

## Options

```bash
aliasos-screensaver [force] [options]
```

- `force`: start even when the toggle is set or the session is locked
- `--input PATH`: draw this file instead of the branding text
- `--effect NAME`: pin one effect instead of picking at random; the names are ttfx's effects (`ttfx --help`)
- `--include-effects LIST`, `--exclude-effects LIST`: narrow the random pick
- `--seed N`: repeat the same effect and animation
- `--headless --frames N --cols N --rows N`: render without a display, for tests

`--help` lists the rest.

## Build and test

Building needs Rust, `pkg-config` and libxkbcommon. Running it needs a compositor with `wlr-layer-shell`, such as Hyprland, and a monospace font: JetBrains Mono Nerd Font, or the closest monospace font `fc-match` finds.

```bash
make                                  # build/rust/release/aliasos-screensaver
make check                            # the test suite; needs no compositor
make benchmark                        # render benchmark, checked byte-for-byte against a full redraw
make install PREFIX="$HOME/.local"    # as $PREFIX/bin/aliasos-screensaver, outside aliasOS's package
```

To release, bump the version in `Cargo.toml`, run `cargo update -p aliasos-screensaver` so `Cargo.lock` agrees, commit both, tag `v<version>`, and publish a GitHub release for the tag. aliasos-pkgs builds the package from the tag.

## Upstream

Forked from [omacom/omarchy-screensaver](https://github.com/omacom/omarchy-screensaver) 1.3.1, itself a fork of [tobi/omarchy-launch-screensaver](https://github.com/tobi/omarchy-launch-screensaver). The `upstream` remote tracks Omarchy's; aliasOS changes live on top of it.

## Credits

Written by [Tobi Lütke](https://github.com/tobi), with Omarchy's later changes. Built on [ttfx](https://github.com/omacom/ttfx), [Smithay Client Toolkit](https://github.com/Smithay/client-toolkit) and [fontdue](https://github.com/mooman219/fontdue).

MIT; see [LICENSE](LICENSE).
