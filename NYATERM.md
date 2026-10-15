# NyaTerm fork notes

This branch carries [NyaTerm](https://github.com/nyakang/nyaterm)'s local changes
to `gpui-kit` on top of an unmodified upstream base.

- Fork: <https://github.com/nyakang/gpui-kit>
- Upstream: <https://github.com/longbridge/gpui-kit>
- Base revision: `369f7a8f5a6602da980f008cf726467588ad517f` (upstream `main`, 2026-10-13)
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


## 2026-10-07 native drag input follow-up

Align all Zed sources with d7efcc2708a0f95418a0484318fb180b22adb5bf,
which adds scoped source-window input handoff for the native file-drag STA.
No Kit/Base behavior changes. Zed checks and its 10 native export / 4 core
virtual-drag tests pass; NyaTerm performs pinned integration validation.


## 2026-10-07 cross-platform deferred export contracts

Align every Zed dependency with 80dc747786822116ba51deae57b3ec2fa307b775
on nyakang/zed branch nyaterm. This adds validated virtual directory trees,
Finder file promises and Linux Xdnd local URI copies; Kit/Base behavior remains
unchanged. Zed's Windows virtual suite (6 tests) and native export suite (11 tests)
pass. Zed macOS/Linux native compile runs in branch CI; NyaTerm validates pinned
application integration separately.


## 2026-10-07 native export ABI validation follow-up

Move all Zed dependencies coherently to 1322967fc37535ca307c539181fc16ad3d5334e9.
This includes the native-compiled AppKit delegate ABI correction, a guarded Xdnd
completion and a macOS runtime dispatch regression. The corrected preceding Zed
revision passed native macOS, Linux and Windows checks. Pin coherence passes;
final branch CI runs the added dispatch test. There is no Kit/Base source change
or unrelated registry-version refresh.


## 2026-10-07 bounded native drag delivery

Align every Zed revision with d24d3775264eac79ef73334925c3f1b97fa09b4b.
Consumes optional typed drag lifecycle, Windows one-shot completion/idle
accounting and an application-wide three-operation macOS promise queue.
Windows native export suite: 13 passed. macOS branch CI and interactive Finder
validation are tracked in the Zed patch notes. Pin checker passes; no component
behavior or registry dependency versions changed.


## 2026-10-07 final enumeration failure pin

Move every Zed revision to 400d43cd62b6499c77a1400d6f83116d9845e187.
Windows enumeration failures remain failed during synchronous consumer release;
13 native tests passed and preceding three-platform CI succeeded. Pin checker
passes. No Kit source or registry-version changes.

## 2026-10-13 upstream rebase

Upstream `main` is fixed at `369f7a8f5a6602da980f008cf726467588ad517f`.
Previous published NyaTerm head: `c4f044e6a140a8320ed90192fd0508e91da20855`.
The previous branch is retained locally as `backup/nyaterm-20261013`.

The effective old-base-to-fork tree (including merge-only resolutions) was
reconstructed byte-for-byte as separate concern commits before rebasing.
Old upstream merge history was flattened; no functional net change was discarded
except the explicitly documented upstream replacements below. The concern
commits and subsequent compatibility fixes remain separate.

| Concern | Reconstructed commit on old base |
| --- | --- |
| coherent GPUI dependencies | `b080aee1147b77483acf550057a5c63efc5b47ac` |
| GPUI indentation compatibility | `f435f9052abb04c9e819841165bb1d05c2980e6c` |
| viewport scrollbar reveal | `bb52b5f0a74fa5f0d5a695dfdee277df9f45e63f` |
| macro crate resolution | `319a62012c14285eece2ba38062f4e0f09379f4d` |
| popup menu appearance | `c0ca3f5e982efed4fc4d4b0f90b5e11e27cc9d1b` |
| segmented tab sizing | `5b567f88c8f244ef34dc38011a93a6baa04690cd` |
| fork validation | `86fe33cebaf4a0ac7b4cb8998f9a2fbac9c7900d` |
| fork documentation | `bb34bffc376356fbdd05a2e504f1fef967dca624` |

Replay commits before this documentation update:

- `29a441769caa64b8bff7ec6fb97d3ee74f65afaa` build(fork): preserve coherent GPUI dependencies
- `acce3f08c4f977ec66d59b60ad6963d47dcf1b3a` feat(fork): preserve viewport scrollbar reveal
- `1d08096a6d1790341f83cb58549f8e39677bbe50` feat(fork): preserve popup menu appearance
- `136915c44bed99f7583253056208848db95620e0` feat(fork): preserve segmented tab sizing
- `1e66814b16218d123fa051d3a846550ae17dcf54` ci(fork): preserve fork validation
- `758dc695a952954d7a5720191005887189b03df3` docs(fork): preserve fork documentation
- `4c41f6214893a70686cfa18d46d5e3ea04b0813f` fix(macros): resolve the active native GPUI facade
- `e3e65c1d009341c4619e1cdc96407f9a5787cd67` build(deps): pin every native GPUI package to the rebased fork
- `a6b06b911958543550807671a1061825ce5f68e7` ci(kit): validate full component and native facade regressions

GPUI Kit is now 0.7.1. All six default Zed-derived dependencies point at
nyakang/zed revision 9d03436aa8185c972e957f36e95044ac524b2cf4; the consumer must
use the same revision. Optional upstream GPUI Fast packages remain available
but NyaTerm does not activate them. IntoPlot's crate-path lookup retains the
direct GPUI fallback before the inactive optional Fast dependency. Export tests
explicitly import the facade's gpui namespace for native Zed Action derive imports.
The default macro path remains the same native GPUI used by NyaTerm.

Equal-width/natural-width tab layout, viewport hover scrollbar reveal and custom
popup menu appearance are retained. The old SameIndent compatibility patch is
replaced by upstream 87d10ae5e1299d1be18670a37570c5402f952e34 (#3370), which adapts
the same calls. Upstream 3a39e9dfc97d8b12cf3c116708ada36bee02220b also improves
measured Unicode wrapping; the Base tests validate those paths rather than
replaying the older wrapper over them. Previous theme/backdrop replacements
remain documented in the historical sections.

Windows checks against the published Zed pin passed; Base tests (1333 plus one
integration), Component library (664), facade export compile test and all-target
clippy for Base/Component/Kit passed. Bun pin coherence check passed. Whole-workspace
fmt is checked separately; older upstream format failures are historical records.
CI now runs the full Component suite and facade export compile check.


Validation is on Windows; native macOS/Linux checks are delegated to the
branch workflow. Interactive GUI/file-drop and real-server authentication
checks were not performed by this update. Their outcomes must not be inferred
from scripted peers or compile checks.

## 2026-10-13 list-owned context menu compatibility

Upstream now registers the right-click bubble listener before child painting.
Rows that consume that bubble can prevent a list-owned menu from opening.
Add opt-in `ContextMenu::capture_right_click(true)` for such consumers, leaving
the upstream default unchanged. Capture does not stop child dispatch; the
existing deferred menu builder still observes the row target set during that
dispatch. NyaTerm enables this option in its context-menu wrapper.

Windows Component library tests: 665 passed, including the new consumed-bubble
regression proving keyboard action dispatch and dismissal focus restoration.
Component all-target Clippy and workspace formatting checks passed. The existing
NyaTerm list-menu/dialog regression is also used to validate this integration.

## 2026-10-13 component notification layering

Defer the component root's notification layer at `POPUP_PRIORITY + 1`, matching
the existing ShellRoot toast priority. Ordinary deferred content, including
NyaTerm's horizontal and vertical panel resize handles, previously painted over
notifications and could receive pointer input through them. Notifications now
paint and occlude above those handles and popups; tooltip priority remains higher.
Sheet offsets, notification positioning, transitions, and dismissal are retained.

Windows validation: all 105 NyaTerm UI tests passed using temporary local Cargo
patches, including a new production-root regression that asserts actual quad draw
order, notification click handling without resize activation, and uncovered
horizontal/vertical handle input. The draw-order assertion fails against the
previous revision. Fork workspace formatting and `git diff --check` passed.
No data, persistence, or protocol contracts change.

## 2026-10-14 Windows drag safety dependency

Pin all six native Zed dependencies to
`c776ed2a4bd732e977a1249807616e6c18207afe`, which adds safe HWND lookup and
deferred pointer cancellation. Consumers must use the same GPUI revision: mixing
the old and new pins creates incompatible GPUI entity and element types.
No Component or Base behavior changes are included in this dependency update.

Windows validation: `cargo check -p gpui-kit` and
`bun script/check-gpui-pin.ts` passed. Cargo.lock changes only the Zed source
revisions. Native drag lifecycle tests are recorded in the Zed fork's NYATERM.md;
interactive external-app and mixed-DPI tests remain pending.

The final pin is `09a27ca822b2a6906f90e55515e104cc04176f19`, adding the
late-button-up cancellation race fix. All six requirements and Cargo.lock move
together; Windows Kit check and Bun pin coherence passed again.


## 2026-10-15 test scheduler shutdown dependency

Pin all six Zed dependencies to `d26c15f6688e593800a2ef74e4aab6dfac7a28e6`,
which removes a lock inversion between timed GPUI test shutdown and real worker
completions. Native consumers must use the same revision. No Base or Component
behavior changes are included.

Windows validation: `cargo check -p gpui-kit --locked` and
`bun script/check-gpui-pin.ts` passed. Cargo.lock changes only Zed source
revisions. The Zed fork records the scheduler regression tests in NYATERM.md;
macOS CI validation remains pending.
