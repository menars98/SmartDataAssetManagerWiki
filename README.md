# Smart Data Asset Manager — Plugin Documentation
**Version:** 1.0 | **Author:** Menars | **Engine:** Unreal Engine 5.4+

---

## Overview

Smart Data Asset Manager is an editor plugin that gives you a dedicated, all-in-one panel for browsing, editing, validating, and managing Data Assets in your Unreal Engine project. Instead of hunting through the Content Browser, you get a focused tool with powerful filtering, bulk editing, and a built-in validation system.

---

## Installation

1. Copy the `SmartDataAssetManager` folder into your project's `Plugins/` directory.
2. Open your project — Unreal Engine will prompt you to build the plugin.
3. After building, the plugin is accessible via:
   - **Tools → SmartDataAssetManager** in the top menu bar
   - The **toolbar button** in the Level Editor

> **Requirement:** The `AssetManagerEditor` plugin must be enabled (it is enabled automatically as a dependency).

---

## Interface Overview

The panel is divided into three sections:

```
┌──────────────────────────────────────────────────────────────────────────────────────┐
│  TOP BAR                                                                             │
│  [💾] [🔍] [🔄] [⊞]  Folder: [All Folders ▾] [Clear]  Filter Class: [All Data ▾]  [+ Create New Asset] │
├──────────────────────────────────────┬───────────────────────────────────────────────┤
│                                      │                                               │
│  ASSET LIST                          │  DETAILS / PROPERTY EDITOR                    │
│  Scrollable asset picker             │  Edit properties directly.                    │
│  with multi-select support           │  Multi-select bulk editing supported.         │
│                                      │                                               │
├──────────────────────────────────────┴───────────────────────────────────────────────┤
│  [✓] Check Empty References  [ ] Check Empty Arrays  [ ] Strict Mode (Required Only) │
│  [Clear Selection]  [Validate Rules]                                                 │
└──────────────────────────────────────────────────────────────────────────────────────┘
```

---

## Features

### 1. Asset Browser
- Displays all Data Assets in your project using UE's native Asset Picker
- Supports **List** and **Tile (Grid)** view modes
- Multi-select supported — select multiple assets to bulk-edit them in the Details panel

### 2. Class Filter
- Dropdown list of all `UDataAsset` subclasses in the project
- Default selection: **All Data Assets** — shows every Data Asset regardless of type
- Blueprint Data Asset classes are included automatically
- Skeleton (`SKEL_`) and reinstanced (`REINST_`) classes are filtered out

### 3. Folder Filter
- Pick any folder from your Content Browser hierarchy
- The asset list narrows to show only assets in the selected folder and its subfolders
- **Clear** button to remove the folder filter instantly

### 4. Details / Property Editor
- Standard UE Details View — edit properties directly without opening each asset
- **Multi-select editing:** Select multiple assets to edit shared properties simultaneously
- Fields with different values across selected assets show **"Multiple Values"** — editing the field applies the new value to all selected assets at once

### 5. Asset Actions (Right-Click Context Menu)
Right-click any asset (or selection) to access:

| Action | Description |
|--------|-------------|
| **Find in Content Browser** | Highlights the selected asset(s) in the Content Browser |
| **Duplicate** | Creates a copy of each selected asset in the same folder |
| **Delete** | Deletes selected assets (with UE's standard reference check) |
| **Reference Viewer** | Opens UE's Reference Viewer to inspect asset dependencies |

### 6. Create New Asset
- The **+ Create New Asset** button opens UE's asset creation dialog
- The newly created asset automatically opens in the Details panel for immediate editing
- The class used for creation is pre-set to the currently selected **Filter Class**

### 7. Save All
- Saves all dirty (modified) packages in the project
- Equivalent to **File → Save All** or `Ctrl+S`

### 8. Validation System
The built-in validator scans selected Data Asset properties using **Unreal's Reflection System** — no manual configuration required.

#### How to Validate
1. Select one or more assets in the asset list
2. Configure the validation options using the checkboxes (see below)
3. Click **Validate Rules**

#### Validation Options

| Checkbox | Default | Description |
|----------|---------|-------------|
| **Check Empty References** | ✅ On | Flags any `UObject*`, `TSubclassOf`, `TSoftObjectPtr`, or `TSoftClassPtr` property that is `null` / `None` |
| **Check Empty Arrays** | ❌ Off | Flags arrays with zero elements |
| **Strict Mode (Required Only)** | ❌ Off | Only validates properties marked with `meta=(Required)` in C++ |

#### What Gets Validated
When **Check Empty References** is enabled, the following property types are checked:

- `TSubclassOf<>` / `UClass*` — Missing class reference
- `TSoftClassPtr<>` — Missing soft class reference
- `TSoftObjectPtr<>` — Missing soft object reference
- `UObject*` and subclasses — Missing hard object reference (Texture, Mesh, Sound, DataAsset, etc.)
- `FName` — Empty name (`None`)
- `FString` — Empty string
- `TArray<>` containing any of the above — Null elements inside arrays

#### Validation Results
- A popup shows up to **20 errors** with the asset name and property name for each issue
- If there are more than 20 errors, the popup directs you to the **Output Log** for the full list
- All errors are also logged to the Output Log under `LogSmartDataAssetManager Warning` for easy filtering
- A green **SUCCESS** popup confirms when no issues are found

#### Strict Mode (C++ Only)
To use Strict Mode, mark required properties in your C++ Data Asset class:

```cpp
UCLASS()
class UMyDataAsset : public UDataAsset
{
    GENERATED_BODY()
public:
    // This field will be checked in Strict Mode
    UPROPERTY(EditAnywhere, meta=(Required))
    UTexture2D* MainTexture;

    // This field will be IGNORED in Strict Mode
    UPROPERTY(EditAnywhere)
    UTexture2D* OptionalTexture;
};
```

> **Note:** `meta=(Required)` only works in C++. Blueprint-only Data Asset classes should use standard mode (Strict Mode off).

---

## Keyboard Shortcuts

| Shortcut | Action |
|----------|--------|
| `Ctrl + S` | Save All |
| `Ctrl + B` | Find Selection in Content Browser |
| `Ctrl + V` | Validate Rules |
| `Ctrl + R` | Clear Selection |
| `Ctrl + D` | Duplicate Selected Assets |

---

## Frequently Asked Questions

**Q: Why does the Class Filter show engine classes like "AutomationViewSettings"?**
A: This was a known issue in early versions. Version 1.0 filters out abstract, deprecated, skeleton, and reinstanced classes. Only valid, user-accessible Data Asset classes are listed.

**Q: Can I validate Blueprint Data Assets?**
A: Yes. Blueprint Data Assets are fully supported in standard validation mode. Strict Mode (`meta=(Required)`) requires C++ base classes — Blueprint variables cannot be marked as Required.

**Q: Why does "Multiple Values" appear in the Details panel?**
A: This is correct UE behavior. When multiple assets are selected and their property values differ, "Multiple Values" is displayed. Typing a new value will apply it to all selected assets simultaneously.

**Q: What happens if I delete an asset that is referenced elsewhere?**
A: The plugin uses UE's standard `ObjectTools::DeleteObjects`, which checks for references and prompts you before deleting — same behavior as deleting from the Content Browser.

**Q: Can I use this plugin in a packaged game?**
A: No. This is an Editor-only plugin (`"Type": "Editor"`) and has no impact on packaged builds.

---

## Technical Details

| Property | Value |
|----------|-------|
| Plugin Type | Editor Only |
| Engine Version | UE 5.4+ |
| Module | SmartDataAssetManager |
| Loading Phase | PostEngineInit |
| C++ Modules Used | Slate, SlateCore, PropertyEditor, ContentBrowser, AssetTools, AssetRegistry, AssetManagerEditor, UnrealEd |

---

## Changelog

### v1.0
- Initial release
- Asset browser with class and folder filtering
- Multi-select bulk editing via Details View
- Right-click context menu (Find, Duplicate, Delete, Reference Viewer)
- Reflection-based validation system with configurable options
- Keyboard shortcuts
- List / Tile view toggle
- Create New Asset with class pre-selection

---

*Created by Menars — Smart Data Asset Manager v1.0*
