# NyaTerm fork notes

## 2026-11-06 prepare serialized Windows component validation

Run the branch Base/Component workflow on Linux and Windows with serialized
GPUI tests. Linux bootstrap is conditional; Base invocations enable
`test-support` because its observation tests use that module. The complete
Base run already includes those regressions, so remove the duplicate step.

Workflow inspection and `git diff --check` passed. This is a validation
configuration change, not a claim of Windows results: Actions is disabled
for the fork and Windows automation/manual acceptance are pending.

## 2026-11-06 keep number-input accessibility focus on the editor

Give each number-input frame an entity-keyed independent focus scope while
its inner editor retains actual editing focus. The outer SpinButton keeps
numeric values, step actions, and action bubbling, delegates accessibility
Focus to the enabled editor, and contributes no Tab stop. Unhandled frame and
step-button pointer presses delegate focus after child selection handlers,
preventing GPUI's default frame focus from stealing the editor's focus.
Public component interfaces and persisted values remain compatible.

The new Component regressions activate the platform accessibility client and
draw the real tree. Before the fix, selecting a number reproduced
`set_focus called more than once in a single frame` in GPUI's a11y builder.
Afterward, click/double-click/drag selection, blur/re-focus, multiple inputs,
Tab navigation, keyboard/button stepping, and disabled accessibility Focus
pass with one focused TextInput and preserved SpinButton semantics.

Linux validation through the writable-source consumer workspace: Base library
1,554 passed, Component library 755 passed, and both Base external tests passed.
Existing Base tests cover step-button press focus and numeric bounds.
Touched-file rustfmt and `git diff --check` passed. Windows automated and real
settings/titlebar/native RDP acceptance remain unverified: no execution
endpoint is available and fork Actions is disabled.

## 2026-11-06 align GPUI accessibility and native hit-testing fixes

Pin all six Zed workspace dependencies and lockfile sources to
`75edab6ed56d8657ac555e1fdc082b75f5efbbe9` on `nyakang/zed:nyaterm`.
That revision exposes test-platform accessibility activation/actions and
publishes native window-control hit tests without borrowing App. Keep the
consumer and Kit on the same GPUI types; no package versions or data formats
change.

Validation: `cargo +stable metadata --locked --format-version 1` passed.
Manifest/lock inspection confirms all six aliases and the sole Zed lock source
use the published full revision. Integration tests exercise the writable fork
sources through a narrow consumer-style workspace, excluding unrelated Zed
root patches.

## 2026-11-05 align native overlay modal ordering

Pin all six Zed dependencies to `6cda7bc3f0e0ab64be33627851f56f8d908fe844`
on `nyakang/zed:nyaterm`. This keeps Kit and NyaTerm on one GPUI source while
GPUI keeps owned modal dialogs above the native child overlay popups and stops
the overlay input popup from accepting input for a disabled owner. No Kit APIs
or component behavior change.

Validation: manifest/lock consistency (`cargo metadata --locked`) and
whitespace checks passed. Zed Windows library tests passed 61 tests, zero
failures, one ignored with test-support in serialized mode. The consuming
NyaTerm workspace performs the final build and real RDP acceptance.

## 2026-11-03 fixed upstream synchronization

Rebase onto upstream `4cbbc0190c03a4a5e875785b45a00174609ffa84`
(GPUI Kit 0.9.0), retaining the concern-separated menu appearance, row height,
segmented tab layout, viewport scrollbar hover, macro crate paths and native
overlay patches. Preserve upstream checkable-menu state, safe notification
insets and dialog exit animations. All six Zed-derived dependencies use
`a755e55acb90229c687e64e6a84877fb70d21619`, identical to NyaTerm's direct GPUI.
No runtime patch was removed; upstream's 0.9 manifests and lockfile replace
the old version metadata.

A regression exposed the native modal wrapper blocking the application while
a closed dialog faded out. Keep the visual overlay passive once no open dialog
remains; keep modal input while any dialog is open. The existing
`closing_dialog_does_not_block_input_beneath_it` test failed before this fix
and passes afterwards. Adapt four upstream Hitbox fixtures to GPUI's overlay
input field.

Windows serialized validation passed: Base 1554 unit tests and two external
consumer tests, Component 756 tests, and facade integration export compile
checks. The three-crate check against the published Zed revision,
`bun script/check-gpui-pin.ts`, and workspace formatting also passed.
NyaTerm workspace check passed with local sources; published-source
validation is recorded in the consuming synchronization report. Real native
RDP interaction, Linux and macOS desktop behavior were not verified locally.


## 2026-11-03 align native child cursor ownership

Pin all six Zed dependencies to `597ca983eb6facf1a8528a56204aeeb7c857eaf6`
on `nyakang/zed:nyaterm`. This keeps Kit and NyaTerm on one GPUI source while
GPUI preserves native child HWND cursor ownership for WM_SETCURSOR and
asynchronous cursor changes. No Kit APIs or component behavior change.

Validation: manifest/lock consistency and whitespace checks passed. Zed Windows
library tests passed 58 tests, zero failures, one ignored with test-support in
serialized mode. The consuming NyaTerm workspace performs the final build;
real RDP cursor and DPI validation remains pending because desktop automation
is unavailable in this environment.

## 2026-11-02 align native input thread observation

Pin all six Zed dependencies to `fa96acbd51d347ba7a4fff2053036a1559ef9387`
for GPUI menu outside-click notification across private ActiveX input threads.
No Kit APIs change. Manifest/lock consistency and whitespace checks passed;
Zed Windows library tests passed 57 tests, zero failures, one ignored.
The consuming NyaTerm workspace performs final build and real input validation.

## 2026-11-02 align native overlay pointer routing

Pin all six Zed dependencies to `e33b65da712adc702e1c8816066a2aefca619a18`.
This aligns the pointer-transparent visual and regioned input overlay fix
with the consuming application without changing Kit APIs.

Validation: manifest/lock revision consistency and whitespace checks passed.
Zed Windows tests passed 57 tests, zero failures, one ignored.
The consuming NyaTerm workspace checks the shared GPUI graph and real RDP input.

## 2026-11-02 align native child owner redirection

Pin all six Zed dependencies to `8ecf76c3abf4f3c564b14ce9de5181c9b42244b5`,
which adds the window creation opt-in required by legacy HWND child painting.
The earlier target-order change alone did not pass NyaTerm's real RDP check.
This commit only aligns dependency revisions; it changes no Kit component API.

Validation: manifest and lockfile revision diff and whitespace checks passed.
Zed Windows library tests passed 56 tests with one ignored in serialized mode.
The consuming NyaTerm workspace validates the dependency graph and actual RDP
display after pinning this commit.

## 2026-11-02 align native child composition fix

Pin the Zed workspace dependencies to `4d23d1497004fc8d93228e40b661e970995ee764`
on the `nyaterm` branch. This keeps Kit and NyaTerm on one GPUI source while the
owner DirectComposition scene moves below native child HWNDs. No component or
Base behavior is changed here; the renderer fix and Windows regression live in
the Zed fork.

Validation: `cargo check -p gpui-kit -j 1` passed on Windows x64; manifest diff
and whitespace checks passed. Zed Windows library tests passed 55 tests with one
ignored. The consuming NyaTerm workspace checks and real RDP display verification
follow after pinning this dependency-only commit.

This branch carries [NyaTerm](https://github.com/nyakang/nyaterm)'s local changes
to `gpui-kit` on top of an unmodified upstream base.

- Fork: <https://github.com/nyakang/gpui-kit>
- Upstream: <https://github.com/longbridge/gpui-kit>
- Base revision: `1676826d9d49b90f4d7b6808b69924f58796f6f1` (upstream `main`, 2026-10-20)
- Branch: `nyaterm`

NyaTerm uses this crate through the stable `nyaterm-ui` facade, so these are the
changes that could not be made on the NyaTerm side.

## 2026-11-02 native child overlay presentation

Require GPUI revision `be61ba9ce2b8bccc4a5cabd8c1ad135a87a115d6` and annotate
window-owned dialog/sheet layers as modal, tooltip as passive, and touch-selection
and fallback-menu layers as regional. The component entities and focus handles
remain in the original window; deferred drawing inherits the presentation scope.
This allows the same widgets to remain visible above system native child HWNDs.

Validation: Windows x64 consuming NyaTerm workspace check passed with both
editable forks; GPUI's five native-overlay tests passed. Real RDP interaction,
IME and ARM64 hardware acceptance are separately tracked by NyaTerm.

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

## 2026-10-17 dynamic texture allocation dependency

Pin Zed 304f112a4bb6ecf48b7276caa645b49563a137d6 to consume fallible dynamic
texture staging and avoid multiple GPUI type identities in NyaTerm. No Kit
implementation or public API changes. Validation is performed by the consuming
NyaTerm workspace on Windows; GPUI texture tests passed in the Zed fork.

## 2026-10-17 native allocation failure pin

Align all GPUI dependencies and lockfile entries with Zed nyaterm revision
9e2fabb8dac907911852ca8b9f67ab74fdd8c6e4, which propagates native atlas
allocation errors. No gpui-kit source or persisted data changes. Validation:
the consuming NyaTerm workspace checks and tests exercise this exact pin.


## 2026-10-18 Windows modal close dependency

Align all six Zed dependencies and lockfile entries with nyaterm revision
77a6246ddff9cafa6a72380fee60feedaa89b7d2. This restores the Windows modal
owner before hiding a foreground dialog and avoids foreground stealing when
closing in the background. No Kit source, public API or persisted data changes.
Consumers must align their direct GPUI dependency with this revision.

Validation: `bun script/check-gpui-pin.ts` checks all six requirements. The
consuming NyaTerm workspace validates the dependency alignment on Windows;
the Zed fork records its passing native lifecycle tests in NYATERM.md.


## 2026-10-20 upstream 0.8.0 rebase

Rebase onto fixed upstream main `1676826d9d49b90f4d7b6808b69924f58796f6f1`.
Previous published head: `922e57752137a86e9edeb575607fb7e99bf7970b`;
retained locally as `backup/nyaterm-20261020` with original patches and range-diff.
All six native Zed dependencies now use
`c054331114f9abdc44e5d587cd23314db6ac5aff`, also consumed directly by NyaTerm.
Optional upstream GPUI Fast packages remain available but are not enabled by
NyaTerm. The lockfile preserves compatible registry versions, updates proptest
to 1.12.0 as required by the new Zed manifest, and resolves upstream WGPU 30.

Viewport-hover scrollbar reveal, popup appearance, opt-in right-click capture,
notification priority, macro crate-path resolution, and natural-width tab
regressions are retained. Conflict resolution keeps upstream menu callbacks,
new tab wheel/font tests, and root imports. No old runtime patch was removed.
The facade exports tests moved upstream into the consolidated `integration`
target; branch CI now selects `integration exports::`.

Upstream's disabled-control snapshot test referenced feature-gated test-support
without enabling that feature. Gate that test and its harness consistently;
the default suite and an explicit test-support run validate both configurations.
No runtime behavior or persisted data contract changes.


Windows validation passed on the rebased branch:

```sh
bun script/check-gpui-pin.ts
cargo check -p gpui-base -p gpui-component -p gpui-kit
cargo test -p gpui-base                         # 1435 unit + 2 external tests
cargo test -p gpui-base --features test-support # 1436 unit + 2 external tests
cargo test -p gpui-component --lib              # 702 tests
cargo test -p gpui-kit --test integration exports:: # facade compile checks
cargo clippy -p gpui-base -p gpui-component -p gpui-kit --all-targets
cargo fmt --all -- --check
```

NyaTerm's workspace check also passed with this checkout substituted for Kit;
final published-pin validation and CI outcomes are recorded in the consuming
project's `docs/reports/fork-sync-2026-10-20.md`. Headless tests do not validate
real desktop interaction; native Linux CI is dispatched after publication.

## 2026-10-29 align GPUI with Windows IME caret fix

Pin all six Zed workspace dependencies and their lockfile sources to
`8e5a53c0aa6ff991bdf2bcbb4e6049e2f4715658` on `nyakang/zed:nyaterm`.
NyaTerm consumes that revision directly for the Windows IME caret-only
composition notification fix; using the same revision avoids linking two
incompatible GPUI type sets. No component or public API changes.

Validation on Windows: `bun script/check-gpui-pin.ts` passed;
`cargo metadata --locked --offline --no-deps --format-version 1` passed.
The consuming NyaTerm workspace checks validate the complete integration.

## 2026-11-02 preserve scrollable popup menu rows

Prevent popup menu rows from flex-shrinking inside the height-limited item
container. NyaTerm's terminal theme submenu adds a "Follow UI theme" entry to
the appearance catalog; with 20 themes, that extra item enables automatic
scrolling and previously compressed the configured 28px rows. Rows now retain
their normal height and hitboxes while the existing scroll handle and keyboard
navigation reveal overflow. The fix applies to all popup item kinds without
changing public APIs, menu sizing defaults, or persisted data.

Windows validation:

- Added `scrollable_menu_preserves_rows_and_scrolls_to_keyboard_selection`.
  Before the fix it failed with a 22.5px row instead of 28px. After the fix it
  passes, including scroll-range and keyboard reveal checks for the final row.
- `cargo test -p gpui-component --lib menu:: --offline` passed (27 tests).
- `cargo fmt --all -- --check` passed.
- `bun script/check-gpui-pin.ts` passed.

The test commands used the consuming workspace's target directory for build
artifacts. NyaTerm validates the published revision in its own workspace.

### Overlay HWND lifetime dependency update

Pin GPUI to `be61ba9ce2b8bccc4a5cabd8c1ad135a87a115d6`, which retains the
overlay context through reentrant activation/input and invalidates destroyed
HWNDs. The dependency's hidden owner-destruction test passed on Windows.
Validation: `bun script/check-gpui-pin.ts`, Cargo metadata resolution, and
`git diff --check`; consuming NyaTerm checks cover integration.


### Logical dialog lifetime during exit animation

Expose `WindowExt::open_tracked_dialog` and a weak `DialogHandle` whose
`is_active()` becomes false synchronously on logical close, including
programmatic close and close-all. The component stack owns this state;
retaining the builder for its exit animation cannot keep the logical handle
active. NyaTerm uses it to reject a verification result from a replaced
security dialog without suppressing entrance or exit motion.

Windows serialized Component tests passed all 757 tests, including animated
close and closing an unrendered stack. Facade integration export compilation
and workspace formatting passed. NyaTerm all-target compilation and the three
focused sidebar, plugin cancellation and security replacement regressions
passed with this checkout. Final published-source workspace validation is
recorded in the consuming report.
