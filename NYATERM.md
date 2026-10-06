# NyaTerm fork notes

This branch carries [NyaTerm](https://github.com/nyakang/nyaterm)'s local changes
to `gpui-kit` on top of an unmodified upstream base.

- Fork: <https://github.com/nyakang/gpui-kit>
- Upstream: <https://github.com/longbridge/gpui-kit>
- Base revision: `ec6b87f5d9d04a38e9c9bbcf6e389a45a570afa4` (upstream `main`, `gpui-kit` 0.7.0, 2026-10-01)
- Branch: `nyaterm`

NyaTerm uses this crate through the stable `nyaterm-ui` facade, so these are the
changes that could not be made on the NyaTerm side.

## Patches

1. `fix(tab_bar): let segmented tabs fill the bar` — flex sizing set on each
   `Tab` is propagated to its layout wrapper, so segmented tabs using
   `Tab::flex_1()` divide the bar evenly instead of hugging their labels, while
   `Tab::flex_none()` keeps natural width for scrollable tab strips. Other
   variants preserve their requested flex behavior.
2. `feat(scrollbar): reveal a hover scrollbar from anywhere in the viewport` —
   `ScrollbarMode::Hover` revealed the bar only from inside the track bounds, so
   a hidden bar had to be aimed at blind. Adds
   `ScrollbarStateInner::hovered_viewport`, fed by `HitboxId::is_hovered` so an
   overlay does not reveal the bar behind it, and a `reveal_hover` predicate.
   Thumb styling still keys off track hover alone; `Scrolling` and `Always` are
   unchanged.
3. `feat(menu): support configurable popup appearance` — adds an opt-in
   `PopupMenuAppearance` for row, typography, icon-slot, spacing, separator,
   radius and disabled-state metrics. The default keeps the upstream rendering
   unchanged, while nested submenus inherit the nearest parent appearance
   unless they explicitly override it. NyaTerm uses this at its `nyaterm-ui`
   boundary to align ordinary component menus with its richer tab context menu.
4. `build(deps): use the NyaTerm GPUI fork` — upstream 0.7.0 uses the
   `gpui-pre 0.3.7` package set. This branch instead points every Zed-derived
   workspace dependency at `nyakang/zed:nyaterm` revision `4ba16e3a20`, so
   NyaTerm keeps its dynamic-texture and hidden-cursor APIs without linking two
   incompatible GPUI copies. `script/check-gpui-pin.ts` validates either the
   exact published snapshot set or the coherent NyaTerm fork revision.

## Not carried here

- `fix(theme): re-project scrollbar theme on every palette change` — dropped when
  this series was rebased from `b1e78a51` onto `0bfcb640`. Upstream fixed the same
  bug independently in `222cf964` ("theme: Follow the global radius setting
  everywhere"), and its `Theme::sync_base` is a superset: where the NyaTerm patch
  rebuilt only `gpui_base::Theme::scrollbar`, `sync_base` replaces the whole Base
  projection, and upstream added a regression test for it
  (`base_projection_carries_a_square_radius_to_the_scrollbar`). Consumers that
  called `Theme::sync_scrollbar_theme(cx)` call `Theme::sync_base(cx)` instead.
- `fix(dialog): make the backdrop event wrapper cover the viewport` — upstream
  `df1d07b2` fixed the same collapsed-wrapper bug with
  `.absolute().inset_0()` and added `the_backdrop_fills_the_host`. The merge
  keeps that implementation and drops NyaTerm's older `.size_full()` hunk.

## Merge notes

The 0.6.5 merge conflicted in `Cargo.toml`, `Cargo.lock`, and
`crates/component/src/tab/tab_bar.rs`. The manifest and regenerated lockfile
keep upstream's 0.6.5 workspace, features, tests, and QuickJS migration while
replacing the published `gpui-pre` set with the single NyaTerm Zed revision.
The tab-bar resolution keeps upstream's direct-child indexing and flex-basis
fix while retaining equal-width segmented tabs. The scrollbar viewport-hover,
popup-menu appearance, and macro crate-path fallback patches merged cleanly.

## Validation

Validated on Windows 11 against `nyakang/zed:nyaterm` revision
`4ba16e3a20`:

```sh
cargo test -p gpui-base
cargo test -p gpui-base reveal
cargo test -p gpui-component menu::popup_menu --lib
cargo check -p gpui-component -p gpui-kit
cargo clippy -p gpui-component --all-targets
bun script/check-gpui-pin.ts
```

The 2026-09-22 pin refresh moves all six Zed-derived dependencies together to
`952fab9804`; no gpui-kit source patch changed and upstream `main` was already
contained in this branch. The same-day cross-window drag refresh moves those
six dependencies together to `4ba16e3a20`; it only consumes the new GPUI
platform hook and does not change gpui-kit source.

## 2026-10-01 upstream merge

Merged upstream GPUI Kit 0.7.0. Manifest conflicts keep the upstream workspace
version and the six coherent NyaTerm Zed dependencies at `0544bd292a52fed9718e1ba9739c4ac82c41d223`.
The lockfile was resolved from the previous fork lock, retaining unrelated
registry versions. The menu conflict adopts upstream element hover so keyboard
navigation clears the previous highlight, while retaining custom disabled
opacity and popup appearance. Segmented-tab sizing and viewport scrollbar
reveal remain present.

A separate compatibility patch passes `IndentAdjustment::SameIndent` at all
three Base line-wrapper calls, preserving the previous indentation behavior.
The branch workflow installs Bun and runs the manifest coherence validator
instead of matching a hard-coded old revision.

Windows validation passed:

```sh
cargo check -p gpui-component -p gpui-kit
cargo test -p gpui-base                           # 1252 tests passed
cargo test -p gpui-component menu::popup_menu --lib # 4 tests passed
cargo test -p gpui-component --lib                # 578 tests passed
cargo clippy -p gpui-component --all-targets
bun script/check-gpui-pin.ts
```

The six Zed-derived packages also resolve to exactly one fork revision in the
lockfile. Formatting checks pass for all three changed Rust files.
`cargo fmt --all -- --check` reports existing upstream differences in
`crates/component/src/form/tests.rs` and `crates/component/src/styled.rs` under
both Rust 1.97.1 and 1.98.1. Both files are byte-identical to the upstream
baseline; this merge leaves those unrelated formatting differences untouched.


## 2026-10-06 coherent deferred-drag GPUI pin

Move every Zed workspace dependency from
0544bd292a52fed9718e1ba9739c4ac82c41d223 to
164d61d92987889e8c3dedf21ae63d9240a4306e on nyakang/zed branch nyaterm.
NyaTerm consumes the same revision directly, preventing duplicate GPUI source
identities and incompatible Entity/Window types. No Kit/Base behavior or public
API changes are introduced. The new Zed revision passed Windows GPUI checks,
four core virtual-drag tests and nine mock native file-export tests. NyaTerm
integration was checked with local source overrides before pin publication;
final pinned workspace checks are recorded in NyaTerm's drag-export report.
