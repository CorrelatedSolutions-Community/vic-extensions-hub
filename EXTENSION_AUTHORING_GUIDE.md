# Vic-3D / Vic-2D Extension Development Reference

A guide to authoring Vic-3D / Vic-2D extensions (`.zve` files) using the
`vicpyx` Python framework.

This document deliberately does **not** reproduce the `vicpyx` API. The module
ships fully documented (typed stub + readable source), and the product ships ~30
working extensions. Those are the authoritative, version-matched reference —
read them directly instead of trusting a transcribed copy that can silently go
stale. What this document keeps is the part that is *not* in the module: the
`config.json` schema, the extension-type model, the conventions, and the
packaging/installation workflow.

---

## Table of Contents

1. [Prerequisites — check your environment](#prerequisites)
2. [The shipped extensions — your read-only reference](#shipped-extensions)
3. [The vicpyx module — your API reference](#vicpyx-module)
4. [.zve File Format](#zve-file-format)
5. [config.json Reference](#configjson-reference)
6. [Extension Types & Base Classes](#extension-types--base-classes)
7. [Conventions & Gotchas](#conventions--gotchas)
8. [Which built-in extension to crib from](#which-built-in-extension-to-crib-from)
9. [Packaging & Installing an extension](#packaging--installing-an-extension)
10. [Authoring Workflow](#authoring-workflow)

---

## Prerequisites — check your environment

**Before authoring, running, or packaging anything, confirm the toolchain is
present.** Run the script below with the Python interpreter your Vic
installation uses — i.e. the one that has `vicpyx` installed. If more than one
Python is installed, pick the one whose `vicpyx` version matches your Vic
version.

> **If this check fails, STOP. Prompt the user to install the missing components
> (Python 3 and the `vicpyx` module) and to re-run the check.
> Do not attempt to author or package an extension until it passes.**

```python
# prereq_check.py — verify Python + vicpyx are available, and locate the module.
import sys

print(f"Python:  {sys.version.split()[0]}  ({sys.executable})")

try:
    import vicpyx
except ImportError:
    sys.exit(
        "ERROR: 'vicpyx' is not installed for this interpreter.\n"
        "Install Python 3 and the vicpyx module before continuing,"
        "then re-run this check."
    )

import os

module_dir = os.path.dirname(vicpyx.__file__)
print(f"vicpyx:  {getattr(vicpyx, '__version__', 'unknown')}")
print(f"location: {module_dir}")
print("  - API stub (compiled classes, enums, signatures + docstrings): vicpy.pyi")
print("  - framework source (base classes, mixins, output objects):     extensions/")
print("OK: prerequisites satisfied.")
```

The printed `location` is where you read the API (see
[The vicpyx module](#vicpyx-module)). Because the API can change between
versions, always read the stub/source from the install that matches the Vic
version you are targeting — not from memory.

---

## The shipped extensions — your read-only reference

Vic-3D / Vic-2D ship with a set of working extensions. Their unpacked sources
are the best, version-matched examples of every extension type, config option,
mixin, and output object.

```
Vic-3D:  C:\Program Files\Correlated Solutions\Vic-3D <version>\bin\extensions\
Vic-2D:  C:\Program Files\Correlated Solutions\Vic-2D <version>\bin\extensions\
```

Each `.zve` there is a ZIP archive — open it (read-only) to study its
`config.json`, Python script, and help file.

> **Read-only rule (non-negotiable):** Treat the install directory as
> reference *only*. **Never modify, add, delete, repackage, or overwrite
> anything inside the Vic installation directory** — including the shipped
> `.zve` files and the extensions folder. Author your extension in a separate
> working folder and install it through the application (see
> [Packaging & Installing](#packaging--installing-an-extension)).

---

## The vicpyx module — your API reference

Do not transcribe or memorize the `vicpyx` API. Read it from the installed
module (the `location` printed by the [prerequisite check](#prerequisites)).
It is fully self-documenting:

- **`vicpy.pyi`** — the compiled core, as a typed stub with complete
  signatures, default values, and `Args:`/`Returns:` docstrings. This covers
  the data classes (`VicData`, `VicDataSet`), geometry/transform classes
  (`Rotation`, `RigidTransformation`), calibration/project classes, the
  reader/writer classes, and every enum (`VicDataVariableType`,
  `CsDataSystemType`, `TensorType`, `AngleType`, …) with its full value list.
- **`extensions/*.py`** — plain, readable Python source (with docstrings) for
  the extension framework itself: base classes (`extension_base.py`), all
  mixins (`extension_mixins.py`), data/image sequences
  (`extension_sequences.py`), the output objects (`line_data.py`,
  `point_data.py`), the single-processor project object (`single_object.py`),
  inspector items (`inspector_item_set.py`), transformations
  (`transformation_object.py`), start points, and AOI objects.

| To learn… | Read |
|---|---|
| The dataset / AOI API (`get_values`, `set_values`, `add_variable`, `compute_strain`, `at_global_xy`, …) | `vicpy.pyi` → `VicDataSet`, `VicData` |
| Enum names and their full value lists | `vicpy.pyi` → the `Enum` classes |
| Base classes and the method you override | `extensions/extension_base.py` |
| Which mixins exist and what argument each adds | `extensions/extension_mixins.py` |
| Inspector-item dict shapes and their `read_*` parsers | `extensions/extension_mixins.py` |
| The `SingleProcessor` project object (`obj`) API | `extensions/single_object.py` |
| Output objects (`LineData`, `PointData`, their `*Set` containers) | `extensions/line_data.py`, `extensions/point_data.py` |
| Image / data sequence access | `extensions/extension_sequences.py` |
| Transformations | `extensions/transformation_object.py` + `RigidTransformation` in `vicpy.pyi` |
| Helper functions (e.g. `change_file_extension`, `parse_*_vector`) | `vicpy.pyi` and `extensions/extension_mixins.py` |

When you need an API that isn't obvious from a shipped example, read the stub or
the source — do not guess a method name or signature.

---

## .zve File Format

A `.zve` file is a **ZIP archive** (a `.zip` renamed to `.zve`). Minimum
required contents, all at the archive root:

```
config.json          ← extension metadata (see below)
<name>.py            ← Python script
help.html | help.md  ← help documentation (HTML or Markdown)
requirements.txt     ← pip dependencies (can be empty; list vicpyx with a
                        minimum version, e.g. vicpyx>=0.8.0)
```

Optional contents:

```
locale/<lang>/LC_MESSAGES/<name>.po, <name>.mo   ← translations
images/                                          ← assets referenced by the help file
```

Build it with the packaging script in
[Packaging & Installing](#packaging--installing-an-extension).

---

## config.json Reference

> **This schema is accurate as of the time of writing, but unlike the API it is
> not read live from the module — so it can drift as the application evolves.**
> The installed extensions, by contrast, always ship with configurations valid
> for your installed version. **If anything here conflicts with a shipped
> extension's `config.json`, trust the extension, not this guide.**

### Top-level fields

```json
{
  "name": "Display Name",
  "description": "Short description shown in the UI.",
  "type": "full-field",
  "menu-group": "Finite element analysis",
  "app-type": ["Vic-3D", "Vic-2D"],
  "script": "script_name.py",
  "required-variables": ["exx", "eyy", "exy"],
  "required-args": ["reference-data", "aoi-mask", "subset-size"],
  "preview-variable": "W",
  "x-variables": ["pos"],
  "y-variables": ["epsilon"],
  "help": "help.html",
  "argument-groups": [ ... ],
  "presets": [ ... ]
}
```

| Field | Required | Notes |
|---|---|---|
| `name` | Yes | Display name in UI |
| `description` | Yes | Tooltip/subtitle |
| `type` | Yes | One of: `full-field`, `full-field-export`, `point-data-generator`, `line-slice-generator`, `single-processor` |
| `app-type` | Yes | List containing `"Vic-3D"`, `"Vic-2D"`, and/or `"Common"` |
| `script` | Yes | Python file inside the .zve |
| `menu-group` | No | Submenu name (e.g. `"Finite element analysis"`, `"Forming limit curve"`) — groups extensions under a submenu |
| `required-variables` | No | List of variable names that must exist in datasets before the extension is offered |
| `required-args` | No | Built-in arg shortcuts: `"reference-data"`, `"aoi-mask"`, `"subset-size"` — these auto-wire to the matching mixin |
| `preview-variable` | No | Variable to display in preview pane (full-field only) |
| `x-variables` | No | For line generators — x-axis variable names |
| `y-variables` | No | Output variable names exposed to plotting |
| `help` | No | Path to help file inside .zve (typically `"help.html"`) |
| `auto-run` | No | `true` to start processing as soon as the dialog opens |
| `auto-accept` | No | `true` to close the dialog automatically after a successful run |
| `presets` | No | Named value sets offered to compatible args via a right-click menu — see [presets](#presets) |

### argument-groups

Each entry defines a UI section, optionally bound to a named tab.

```json
{
  "name": "Options",
  "ui-options": {
    "no-group": true,
    "tab": "Options",
    "exclusive": true
  },
  "args": [ ... ]
}
```

| `ui-options` field | Notes |
|---|---|
| `no-group: true` | Renders args directly without a box border |
| `tab: "Name"` | Places the group on a named tab; multiple groups can share a tab |
| `exclusive: true` | Args inside become mutually exclusive (radio-button behavior; only one can be set) |

### Argument types

| `type` | Maps to | Notes |
|---|---|---|
| `project-files` | `self.options.input` | File selector. See sub-options below |
| `variable-list` | `self.options.variable_list` (or named) | Variable picker; requires `VariableListMixin` |
| `inspector-items` | `self.options.inspector_line` / `_circle` / `_rect` | Inspector selector; requires matching mixin(s) |
| `input-file-select` | `self.options.<name>` (path str) | Single file picker with extension filter |
| `output-file-select` | `self.options.<name>` (path str) | Output path picker with extension filter |
| `aoi-mask` | `self.options.aoi_mask` (numpy 2D array) | Provides AOI binary mask; pairs with `AoiMaskMixin` |
| `subset-size` | `self.options.subset_size` (int) | Provides subset size from project; pairs with `SubsetSizeMixin` |
| `integer` | `int` | Supports `min`, `max`, `step`, `default` |
| `double` | `float` | Supports `min`, `max`, `digits`, `step`, `default`, `optional`, multi-element via list `ui-text` |
| `string` | `str` | Free text input. Supports `can-be-empty` |
| `checkbox` | flag (`store_true`) | `"default": true/false` |
| `combobox` | `str` | Dropdown; requires `options: [{value, ui-text}, ...]` |
| `global-constants` | `self.options.<name>` (value) | Dropdown of a project global-constants group; passes the selected constant's **value**. Requires `constant-group-id` |
| `argument-list` | repeated `--<name>` rows | Editable table of homogeneous rows (variable-length list). Requires an `element` spec |

### Common argument sub-options

These can appear on most argument entries.

| Field | Applies to | Notes |
|---|---|---|
| `name` | all | argparse destination — accessible as `self.options.<name with underscores>` |
| `ui-text` | all | UI label. May be a **list** of strings for multi-element doubles (e.g. `["Center x [mm]", "Center y [mm]"]`) |
| `default` | all | Default value |
| `store-default` | `checkbox`, `integer`, `double`, `combobox`, `string`, `input-file-select`, `output-file-select` | `true` to persist the user's last value across runs. When any arg sets it, a "Store default values" checkbox appears at the bottom of the dialog |
| `optional` | `integer`, `double`, `global-constants`, `argument-list` | If `true`, shows an enabling checkbox; the arg is only passed to the script when checked. Check at runtime with `hasattr(self.options, '<name>')` and `is not None` |
| `default-check-state` | args with `optional: true` | Initial checked state of the enabling checkbox (default `false`) |
| `can-be-empty` | `string`, `input-file-select` | If `false` (default), the extension can't start while the field is blank; `true` allows empty. For `input-file-select`, an empty selection means the argument is **omitted** from the script command line |
| `app-type` | all | Per-argument visibility — show only for Vic-3D, Vic-2D, or Common (overrides top-level when narrower) |
| `stretch` | all args + `argument-groups` ui-options | UI layout weight (int, default `0`). `9` is the canonical value to fill remaining vertical space |
| `ui-style` | `project-files`, `variable-list`, `inspector-items` | `"list"` for list-box, `"combobox"` for dropdown |

### `project-files` arg

```json
{
  "name": "input",
  "type": "project-files",
  "file-type": "data",
  "ui-style": "list",
  "single-selection": false,
  "preselection": true,
  "filter-size": 3,
  "filter-padding": true,
  "stretch": 9
}
```

| Field | Notes |
|---|---|
| `file-type` | `"data"` (data files) or `"images"` (image files). Default `"data"` |
| `ui-style` | `"list"` standard list box (default); `"combobox"` collapses to dropdown |
| `single-selection` | `true` allows only one file to be selected. Default `false` |
| `preselection` | Boolean (default `true`). When `true`, all files (or the first, if single-selection) are preselected; `false` preselects none. Not applicable with `combobox` |
| `filter-size` | Positive integer half-window of neighboring files loaded around the current one (default `1`; e.g. `3` → ±3). **This** is what makes `self.options.input` iterable/indexable over neighbors — there is no `file-sequence` key |
| `filter-padding` | Boolean (default `true`). Pads the neighbor window at sequence ends |

### `inspector-items` arg

```json
{
  "name": "inspector",
  "type": "inspector-items",
  "inspector-types": ["rect", "circle", "line"],
  "multiselection": true,
  "ui-style": "combobox",
  "ui-text": "Inspector Items"
}
```

| Field | Notes |
|---|---|
| `inspector-types` | Subset of `["point", "line", "circle", "rect", "polygon", "extensometer"]` to allow |
| `multiselection` | `true` to allow many; `false` for single-pick (use combobox UI) |
| `ui-style` | `"combobox"` renders a single-pick dropdown |

> **Wiring:** For the inspector mixins (`InspectorLineMixin`, etc.) to bind
> automatically, the arg `name` must be `"inspector"`. The host then passes the
> per-type arguments `--inspector-line`, `--inspector-rect`,
> `--inspector-circle`, `--inspector-point`, `--inspector-polygon`,
> `--inspector-extensometer` as applicable, which the mixins read into
> `self.options.inspector_line`, `self.options.inspector_rect`,
> `self.options.inspector_circle`, etc. (note `rect`, not `rectangle`). To use a
> different arg name, add the per-type arguments yourself with the matching
> `read_*` parser — see `extensions/extension_mixins.py`.

### `variable-list` arg

```json
{
  "name": "strain-variable",
  "type": "variable-list",
  "preselection": ["exx", "eyy", "exy"],
  "ui-style": "combobox",
  "multiselection": false,
  "variable-types": ["strain"],
  "ui-text": "Variable"
}
```

| Field | Notes |
|---|---|
| `preselection` | List of variable names selected by default |
| `ui-style` | `"combobox"` for single-pick dropdown; default is multi-select list |
| `multiselection` | `false` forces single-pick |
| `variable-types` | Filter visible variables by type tag, e.g. `["strain"]`, `["global-displacements"]` |

`preselection` defaults to `["U", "V", "W", "exx", "eyy", "exy"]`. Valid
`variable-types` tags are the kebab-case forms of the `VicDataVariableType`
enum: `pixel-coordinates`, `global-coordinates`, `pixel-displacements`,
`global-displacements`, `strain`, `correlation`, `disparity`,
`cylinder-coordinates`, `cylinder-displacements`, `pixel-velocities`,
`global-velocities`, `strain-rate`, `curvature`, `angle`, `angular-velocity`,
`global-accelerations`, `pixel-accelerations`, `stress`, `angular-acceleration`.
Read the enum in `vicpy.pyi` for the version-matched list.

> **Wiring:** For `VariableListMixin` to bind automatically, the arg `name`
> must be `"variable-list"`. To use a custom name (as in the example above), add
> the argument yourself with `type=VariableListMixin.read_variable_list`.

### `input-file-select` / `output-file-select` args

```json
{
  "name": "json-file",
  "type": "input-file-select",
  "ui-text": "JSON",
  "file-extensions": { "JSON": [".json"] },
  "can-be-empty": false,
  "store-default": true
}
```

```json
{
  "name": "output-file",
  "type": "output-file-select",
  "default": "result.csv",
  "ui-text": "Output file",
  "file-extensions": { "CSV": [".csv"] }
}
```

`file-extensions` is a dict of label → list of extensions; the picker shows them in its filter dropdown. On `input-file-select`, `can-be-empty: true` allows leaving the field blank, in which case the argument is omitted from the script command line (check with `hasattr(self.options, '<name>')`).

### `aoi-mask` and `subset-size` args

These wire to mixins automatically; no extra fields needed.

```json
{ "name": "aoi-mask", "type": "aoi-mask", "optional": true, "ui-text": "Restrict to AOI" }
{ "name": "subset-size", "type": "subset-size" }
```

### `integer` / `double` arg options

```json
{
  "name": "strain-window",
  "type": "integer",
  "min": 5, "max": 499, "default": 15, "step": 2,
  "ui-text": "Strain window size",
  "store-default": true
}
```

```json
{
  "name": "scale",
  "type": "double",
  "min": 1.0, "max": 1e5, "default": 1.0, "step": 1.0,
  "digits": 4,
  "ui-text": "Scale"
}
```

`digits` controls displayed decimal precision. `optional: true` on a double makes it skippable.

#### Vector arguments (`integer`, `double`, `string`)

For **`integer`, `double`, and `string`** args, making `ui-text` a **list** of
labels turns the arg into a fixed-length vector: one input widget is created per
label. `default` may be a single value (applied to every element) or a list
whose length matches `ui-text`. The host passes the whole vector to the script
as one **colon-separated** value (e.g. `--center-point 1.0:2.0`), so add a
matching argparse type that splits it (for doubles, the shipped
`parse_double_vector`). The parsed value is iterable.

```json
{
  "name": "center-point",
  "type": "double",
  "digits": 6,
  "optional": true,
  "ui-text": ["Center x [mm]", "Center y [mm]"]
}
```

### `combobox` arg

```json
{
  "name": "delimiter",
  "type": "combobox",
  "default": ";",
  "ui-text": "Delimiter",
  "store-default": true,
  "options": [
    {"value": ";", "ui-text": "Semicolon (;)"},
    {"value": ",", "ui-text": "Comma (,)"},
    {"value": "\\t", "ui-text": "Tab (\\t)"}
  ]
}
```

Tab and other escape sequences in `value` are passed as literal strings — convert at runtime:

```python
delim = '\t' if self.options.delimiter == r'\t' else self.options.delimiter
```

### `checkbox` arg

```json
{
  "name": "save-mask",
  "type": "checkbox",
  "default": true,
  "ui-text": "Save original mask",
  "store-default": true
}
```

Maps to `store_true` argparse — the attribute is `True` or `False`.

### `global-constants` arg

```json
{
  "name": "youngs-modulus",
  "type": "global-constants",
  "constant-group-id": "material",
  "default": "E",
  "ui-text": "Young's modulus"
}
```

Dropdown populated from a project global-constants group; the script receives
the selected constant's **value**, not its name. `constant-group-id` must name
an existing group with at least one entry. Supports `optional` /
`default-check-state`.

### `argument-list` arg

```json
{
  "name": "thresholds",
  "type": "argument-list",
  "ui-text": "Thresholds",
  "min-rows": 1,
  "max-rows": 0,
  "element": {
    "type": "double",
    "ui-text": ["Lower", "Upper"],
    "digits": 3
  }
}
```

An editable table of homogeneous rows for variable-length lists.

| Field | Notes |
|---|---|
| `min-rows` / `max-rows` | Row-count bounds; `max-rows: 0` means unlimited (default both `0`) |
| `optional` / `default-check-state` | Optional table with an enabling checkbox (`default-check-state` defaults to `true` here) |
| `default` | List of rows used to populate the table initially |
| `element` | Required spec for each row: `type` (`integer`/`double`/`string`), `ui-text` (str or list of column headers), `min`/`max`/`step`, `digits`, `can-be-empty`, `default` |

Each row is passed to the script as a separate `--<name>` argument whose value
is the row's cells joined by colons.

### `presets` (top-level)

`presets` is a **top-level** config property (a sibling of `argument-groups`,
not an argument type). It defines named value sets that the user can pour into a
compatible argument from a right-click menu, instead of typing values by hand.
**Presets are an input convenience only — they are never passed to the script on
their own.**

```json
"presets": [
  {
    "name": "Standard speckle sizes",
    "type": "integer",
    "values": { "Coarse": 7, "Medium": 5, "Fine": 3 }
  },
  {
    "name": "Crop regions",
    "type": "integer",
    "values": {
      "Top left":  [0, 0, 512, 512],
      "Centered":  [256, 256, 512, 512]
    }
  }
]
```

| Field | Notes |
|---|---|
| `name` | Unique display name for the preset (shown in the menu; may be translated) |
| `type` | Value type: `"integer"`, `"double"`, or `"string"` |
| `values` | Non-empty object mapping a menu label → a scalar **or** a flat list (a fixed-length vector). Every value in one preset must have the same number of elements, each matching `type` |

**Arity matching.** The number of elements in a preset's values is its *arity*.
A preset is offered for an argument only when both type and arity match:

- A **scalar** preset (arity 1) matches a single `integer`/`double`/`string` arg.
- An **arity-*N*** preset matches the *vector* form of those args (an
  `integer`/`double`/`string` whose `ui-text` is a list of *N* labels — see
  [Vector arguments](#vector-arguments-integer-double-string)).
- A preset also matches an `argument-list` arg whose element `type` and column
  count match its arity.

**UI.** Compatible args expose matching presets through a right-click **"From
presets"** submenu (grouped per preset when more than one matches); selecting a
key fills the arg with that value. For `argument-list`, right-clicking a cell
offers presets that match the row shape and fills the whole clicked row. The
same menu has an **"Edit presets…"** entry for adding/removing/editing keys and
values; those edits are saved to the extension's stored defaults
**independently of** the "Store default values" checkbox.

> **Reserved name:** `__presets__` is reserved for storing edited presets — do
> not use it as an argument `name`.

> This section is curated, not exhaustive. The host's config parser
> (`iris/vice/script_argument_basic.cpp`, `script_config.cpp`,
> `script_argument_list.cpp`) and the official product documentation are the
> authoritative `config.json` schema — consult them when in doubt.

---

## Extension Types & Base Classes

The config `type` selects how output is presented; the base class you inherit
selects how input is consumed. For the exact method to override and its
arguments, read `extensions/extension_base.py` and the matching example
extension.

| config `type` | Base class | Override | Crib from |
|---|---|---|---|
| `full-field` | `FullFieldProcessor` | `process_data(self, data)` | `displacement_magnitude`, `cylindrical_coordinates` |
| `full-field-export` | `FullFieldExporter` | `process_data(self, data)` | `export_csv`, `export_mat`, `export_vtk` |
| `point-data-generator` | `PointDataGenerator` | `process_data(self, data, point_data_set)` | `aoi_area`, `data_statistics` |
| `line-slice-generator` | `LineSliceGenerator` | `process_data(self, data, line_data)` | `extensometer_strip`, `histogram` |
| `single-processor` | `SingleProcessor` | `process_data(self, obj)` | `max_strain_inspector`, `noise_floor` |
| image-only | `VicExtension` | `process(self)` | `image_histogram` |

Notes:

- **`SingleProcessor`** does not receive `data`/`image` positionally in modern
  extensions — mix in `DataSequenceMixin` / `ImageSequenceMixin` and read
  `self.options.input` / `self.options.image`. For sequence iteration without
  shuffling, construct the instance, call `e.enable_shuffle_data(False)`, then
  `e.run()`.
- **`VicExtension`** is for image-only extensions and overrides `process(self)`
  rather than `process_data`. The config `type` is still one of the standard
  categories (it controls output presentation).
- **Mixins** are stacked alongside the base class, e.g.
  `class Extension(LineSliceGenerator, InspectorLineMixin, ReferenceDataMixin)`.
  Read `extensions/extension_mixins.py` for the full list, the argument each
  adds, and the `read_*` static parsers used for custom-named arguments.
- Every script ends with the standard entry point
  (`if __name__ == "__main__": Extension().run()`).

---

## Conventions & Gotchas

These are project conventions and pitfalls that the API docstrings won't tell
you:

1. **Never shadow `_`.** `_` is the gettext translation function — use it for
   every user-visible string. To discard an unused return value use `__`:

   ```python
   counts, __ = np.histogram(...)
   ```

2. **Always `.save()`.** `data.save()`, `point_data_set.save()`,
   `line_data.save()`, `obj.save()` — nothing persists otherwise. Note
   `obj.save()` (project-level: inspectors, transformations, AOIs) is distinct
   from `data.save()` (per-dataset variables).
3. **`sigma < 0` means an invalid / masked point.** Filter with `sigma >= 0`.
   If you overwrite `sigma`, back up the original first — the shipped
   extensions use a `__sigma_orig__` variable for this (see `restore_mask`).
4. **3D and 2D variable names differ.** Branch on
   `data.current_data_type()` (`CsDataSystemType.Type3d` / `Type2d`). 3D uses
   `X,Y,Z` / `U,V,W` and pixel `x,y`; 2D uses pixel `x_c,y_c` / `u_c,v_c`.
   `sigma`, `exx/eyy/exy`, `e1/e2` exist in both.
5. **`clear_search_trees()` before mutation.** If you ran interpolation queries
   (`at_global_xy`) and then modify values, call `clear_search_trees()` or
   later queries return stale results.
6. **`flush=True` on prints.** The shipped extensions pass `flush=True` so log
   output reaches the Vic console immediately.
7. **`set_translation` expects a Python list/tuple, not a numpy array.** Use
   `T.tolist()`.
8. **Inspector config vs Python names.** The config arg uses `"rect"`, but the
   Python attribute is `self.options.inspector_rect`; `line` and `circle`
   match directly. Hyphenated config arg names become underscores in Python
   (`my-arg` → `self.options.my_arg`).
9. **`get_values([...])` returns a numpy *structured array*, not a dict.**
   Index by field name (`vals['exx']`) works, but `'sigma' in vals` raises
   *"Cannot compare structured or void to non-void arrays."* Test membership
   with `vals.dtype.names` instead. Inspector-item coordinates are **pixel**
   coords — test region membership against the pixel `x`/`y` variables, not
   world coords.
10. **Don't use `reference-data` / `ReferenceDataMixin` in a `full-field`
    extension.** `--reference-data` expects a value, but the app launches
    full-field scripts *without* one, so argparse aborts at startup
    (*"argument --reference-data: expected one argument"*, once per file). It
    only works in point-data generators. In a full-field extension, read
    reference coordinates off the current dataset instead (in 2D, `x_c`/`y_c`
    are already reference coords).

---

## Which built-in extension to crib from

| If you want to… | Look at |
|---|---|
| Compute a new full-field variable | `displacement_magnitude`, `cylindrical_coordinates`, `polar_coordinates` |
| Use `compute_strain` | `compute_strain_fea` |
| Export to a file format | `export_csv` (text/CSV), `export_mat` (binary), `export_obj` / `export_ply` / `export_stl` / `export_vtk` (mesh) |
| Reduce a dataset to a scalar | `aoi_area`, `arc_length`, `data_statistics` |
| Produce a curve along an inspector | `extensometer_strip`, `histogram` |
| Iterate the whole sequence | `noise_floor`, `max_strain_inspector`, `flc_iso12004` |
| Modify the project (AOI, inspectors, transformation) | `align_rotary_motion`, `max_strain_inspector` |
| Import external data | `import_fea_json` |
| Combine DIC with FE results | `subtract_fe`, `subtract_fe_2d` |
| Mask manipulation | `erode_boundary`, `restore_mask`, `interpolate_data` |
| Time derivatives across files | `time_derivatives` |
| Inspector-based reference frame | `relative_displacement`, `rotation_angle` |
| Image processing | `image_histogram` (uses `VicExtension`) |

The exact set and names depend on your installed version — list the
[install directory](#shipped-extensions) to see what is actually present.

---

## Packaging & Installing an extension

### Packaging

Author your extension in its own working folder (containing `config.json`, the
`.py` script, the help file, `requirements.txt`, and any assets). Package it
with the script below — it validates that `config.json` is present, zips the
folder's **contents** (so `config.json` ends up at the archive root), and
produces a `.zve`.

> **Build the archive with Python's `zipfile` (the script below), not the Unix
> `zip` / Info-ZIP tool.** Info-ZIP writes Unix host & permission metadata that
> Vic's Qt zip reader rejects with *"load failed, could not open the extension
> file."* The application's **Extensions → Tools → Package new** is equally safe.

```python
# package_extension.py — bundle an extension working folder into a .zve
# Usage: py package_extension.py <extension_folder> [output.zve]
# A .zve is simply a .zip with a different extension.
import sys
import zipfile
from pathlib import Path

REQUIRED = ["config.json"]


def package(src_dir: str, output: str | None = None) -> Path:
    src = Path(src_dir).resolve()
    if not src.is_dir():
        sys.exit(f"ERROR: '{src}' is not a directory.")

    missing = [f for f in REQUIRED if not (src / f).is_file()]
    if missing:
        sys.exit(f"ERROR: missing required file(s) in {src}: {', '.join(missing)}")

    out = Path(output).resolve() if output else src.with_suffix(".zve")
    if out.suffix != ".zve":
        out = out.with_suffix(".zve")

    with zipfile.ZipFile(out, "w", zipfile.ZIP_DEFLATED) as zf:
        for path in sorted(src.rglob("*")):
            if path.is_file() and path != out:
                zf.write(path, path.relative_to(src))

    print(f"Created {out}")
    return out


if __name__ == "__main__":
    if len(sys.argv) < 2:
        sys.exit("Usage: py package_extension.py <extension_folder> [output.zve]")
    package(sys.argv[1], sys.argv[2] if len(sys.argv) > 2 else None)
```

### Installing

> **Never copy a `.zve` (or loose files) into the Vic installation's
> `extensions` folder.** That directory is read-only reference (see the
> [read-only rule](#shipped-extensions)).

Install the packaged extension through the application:

1. Open **Vic-3D** (or **Vic-2D**).
2. Go to **Extensions → Tools → Add new**.
3. Select your `.zve` file.

The application installs it into the per-user extension location and validates
the package, reporting any structural problems. To iterate, fix your working
folder, re-run the packaging script, and add the new `.zve` again.

---

## Authoring Workflow

1. **Check the environment.** Run the
   [prerequisite check](#prerequisites). If it fails, stop and ask the user to
   install what's missing.
2. **Identify the extension type** from the output shape (new variable per file
   → `full-field`; file on disk → `full-field-export`; scalar per file →
   `point-data-generator`; curve per file → `line-slice-generator`;
   sequence-wide action or project mutation → `single-processor`; pure image
   processing → `VicExtension`).
3. **Pick the closest shipped extension** (see
   [the crib table](#which-built-in-extension-to-crib-from)) and read its
   `config.json` and `.py` from the [install directory](#shipped-extensions) —
   without modifying it.
4. **Read the API you need** from the
   [vicpyx module](#vicpyx-module) (the `.pyi` stub and `extensions/*.py`).
   Don't guess method names or signatures.
5. **Write `config.json`** (name, description, type, app-type, script, help;
   `argument-groups` for your args; `required-args` if you need `aoi-mask`,
   `subset-size`, or `reference-data` injected). Mirror every config arg with a
   matching `parser.add_argument` in the script (hyphens become underscores).
6. **Write the Python script** in your own working folder: import
   `from vicpyx import *`, inherit the right base class + mixins, branch 3D/2D
   if needed, filter on `sigma >= 0`, build outputs, call `.save()`, and end
   with the standard entry point.
7. **Write the help file** (`help.html` or `help.md`) describing inputs,
   outputs, and the math.
8. **Package** with `package_extension.py`.
9. **Install** via **Extensions → Tools → Add new** and iterate (errors appear
   in the Vic console / extension log).

### Things to avoid

- Modifying anything in the Vic installation directory, including the shipped
  extensions, or copying your `.zve` into its `extensions` folder. The user should
  install via **Extensions → Tools → Add new** instead.
- Inventing API methods. If you need behavior you can't find, read the
  `.pyi` stub / `extensions/*.py` source rather than guessing.
- Using `_` as a throwaway variable (it's the translation function — use `__`).
- Forgetting `.save()`.
- Overwriting `sigma` without backing it up first.
- Assuming 3D variable names exist in 2D data, or vice versa.
- Adding `data`/`image` positional params to `SingleProcessor.process_data` —
  read them from `self.options`.
- Using `print` without `flush=True`.
