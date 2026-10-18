# Verification Evidence: Issue #739 (Toolbar Slice Icon Silhouette Simplification)

## Problem Statement
In `crates/ui-egui/src/icons.rs`, `Icon::Slice` was drawn as a 16x16 box with a full diagonal line across its center (`(2.5, 2.5)` to `(13.5, 13.5)` with `p.line(2.5, 13.5, 13.5, 2.5)`).
At the toolbar's rendered optical scale (16px), this excessive density and internal stroke caused the mark to render as a heavy visual blob compared to surrounding icons (Rectangle, Frame, Section, Pen, etc.), making it hard to identify at a glance without hovering (Issue #739).

## Solution
Redrew `Icon::Slice` with clean clean-room vector primitives on the 16x16 grid:
- A centered crop/slice boundary frame (`p.rect(4.5, 4.5, 12.5, 12.5, 0.0)`).
- Distinct extending crop marks (`p.line(2.0, 4.5, 4.5, 4.5)` and `p.line(12.5, 13.5, 15.0, 13.5)`).
- This produces an unmistakable, lighter crop/slice silhouette that matches the optical weight and stroke conventions of the rest of the toolbar.

## Automated Verification

### 1. Unit Tests
Command:
```powershell
$env:PATH = "C:\Program Files\CodeBlocks\MinGW\bin;$env:PATH"
$env:LIBRARY_PATH = "C:\Program Files\CodeBlocks\MinGW\x86_64-w64-mingw32\14.2.0;C:\Program Files\CodeBlocks\MinGW\lib;C:\Program Files\CodeBlocks\MinGW\x86_64-w64-mingw32\lib"
rustup run stable-x86_64-pc-windows-gnu cargo test -p figcraft-ui-egui --lib -- icons::tests
```
Output:
```
running 4 tests
test icons::tests::concave_polygons_triangulate ... ok
test icons::tests::slice_icon_paints_clear_silhouette ... ok
test icons::tests::gradient_icons_fade ... ok
test icons::tests::every_icon_paints_at_every_size ... ok

test result: ok. 4 passed; 0 failed; 0 ignored; 0 measured; 517 filtered out; finished in 0.01s
```

### 2. Code Formatting
Command:
```powershell
rustup run stable-x86_64-pc-windows-gnu cargo fmt --check
```
Exit code: 0 (clean).

### 3. Review Feedback Resolution
In response to automated review (CodeRabbit):
- Added rustdoc to `fn draw` in `crates/ui-egui/src/icons.rs`.
- Strengthened `slice_icon_paints_clear_silhouette` test to inspect `out.shapes`:
  - Asserts exact primitive count of 3 (`1` crop frame rectangle + `2` crop tick mark line segments).
  - Verifies that the previous box-and-diagonal design (which had only 2 shapes) fails this test.
