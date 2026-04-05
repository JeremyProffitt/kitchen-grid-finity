# Claude Code Guidelines for Kitchen Gridfinity Project

## Project Overview

This repository contains OpenSCAD designs for Gridfinity-compatible drawer inserts for kitchen organization. The designs follow the Gridfinity standard (42mm grid pitch, stackable bases) and are intended for 3D printing, optimized for the Bambu Labs H2D printer.

### Gridfinity Specification Reference

Sources: [gridfinity.xyz/specification](https://gridfinity.xyz/specification/), [kennetek/gridfinity-rebuilt-openscad](https://github.com/kennetek/gridfinity-rebuilt-openscad), [ostat/gridfinity_extended_openscad](https://github.com/ostat/gridfinity_extended_openscad), [vector76/gridfinity_openscad](https://github.com/vector76/gridfinity_openscad)

#### Core Grid Dimensions

| Parameter | Value | Notes |
|-----------|-------|-------|
| Grid pitch | **42mm x 42mm** | Center-to-center spacing of each grid unit |
| Z pitch (height unit) | **7mm** | One Gridfinity height unit (U) |
| Bin outer size | **41.5mm** | `pitch - clearance` = 42 - 0.5 |
| Grid clearance | **0.5mm** | Gap between bin outer wall and grid cell |
| Corner radius | **3.75mm** | Outer corner rounding radius on bin body |
| Taper angle | **45 degrees** | All bevels are at 45 degrees |

#### Base Profile (Stepped Bottom of Bins)

The base profile is the 2D cross-section swept around rounded corners that creates the male stacking feature on the bottom of every bin. It is defined as a polyline:

```
              2.95
         |<--------->|
         |           |
   4.75  +           / --- bevel 2: 2.15mm at 45° (width: 2.15, height: 2.15)
    ^    |          /
    |    |         |  --- vertical riser: 1.8mm tall
    |    |         |
    |    |          \ --- bevel 1: 0.8mm at 45° (width: 0.8, height: 0.8)
    v    +----------+ --- z = 0 (bottom)
```

| Segment | Start [X,Z] | End [X,Z] | Description |
|---------|-------------|-----------|-------------|
| Bottom point | [0, 0] | [0.8, 0.8] | 45° bevel, 0.8mm |
| Vertical riser | [0.8, 0.8] | [0.8, 2.6] | Straight up, 1.8mm |
| Upper bevel | [0.8, 2.6] | [2.95, 4.75] | 45° bevel, 2.15mm |

**Derived values:**
- Total profile width (X): **2.95mm**
- Total profile height (Z): **4.75mm**
- Base bottom corner radius: `3.75 - 2.95 = 0.8mm`
- Total base height: **7mm** (profile 4.75mm + bridge 2.25mm tying units together)

#### Baseplate Profile (Female Socket)

The baseplate profile is nearly identical to the base profile but **0.1mm smaller on the first bevel**, creating the sliding fit clearance:

| Segment | Base (male) | Baseplate (female) | Clearance |
|---------|------------|-------------------|-----------|
| Bevel 1 | [0.8, 0.8] | [0.7, 0.7] | **0.1mm** |
| Riser top | [0.8, 2.6] | [0.7, 2.5] | 0.1mm |
| Bevel 2 top | [2.95, 4.75] | [2.85, 4.65] | 0.1mm |

Baseplate height: **4.75mm** (profile only), **5mm** minimum with clearance floor.

#### Stacking Lip (Top Edge of Bins)

The stacking lip is the interlocking feature at the top of bins that allows stacking:

```
    \        upper taper: 1.9mm at 45°
     |       vertical riser: 1.8mm
      \      lower taper: 0.7mm at 45°
       |     support wall: 1.2mm
      /      support taper (fills to wall)
     /
    /
```

| Segment | Start [X,Z] | End [X,Z] | Description |
|---------|-------------|-----------|-------------|
| Inner tip | [0, 0] | [0.7, 0.7] | 45° lower taper |
| Riser | [0.7, 0.7] | [0.7, 2.5] | Vertical, 1.8mm |
| Upper taper | [0.7, 2.5] | [2.6, 4.4] | 45° upper taper |

**Derived values:**
- Nominal lip size: **2.6mm deep x 4.4mm tall**
- Fillet radius at outer tip: **0.6mm** (replaces sharp point)
- Actual lip height with fillet: **~3.55mm** (no impact on stacking)
- Support wall height below lip: **1.2mm**

#### Magnet & Screw Holes

| Parameter | Value | Notes |
|-----------|-------|-------|
| Magnet diameter | **6mm** (hole: **6.5mm**) | 0.5mm clearance |
| Magnet thickness | **2mm** (hole depth: **2.4mm**) | +2 layer heights (0.2mm each) |
| Screw type | **M3** | |
| Screw hole diameter | **3.0mm** | |
| Screw hole depth | **6.0mm** | |
| Hole center from bin edge | **4.8mm** | From bottom outer edge of base |
| Hole center from cell center | **13.0mm** | `min(pitch/2 - 8, pitch/2 - 4 - magnet_od/2)` |
| Crush rib count | **8** | For press-fit magnet retention |
| Crush rib inner radius | **2.95mm** (5.9mm diameter) | |

#### Wall & Compartment Dimensions

| Parameter | Value | Notes |
|-----------|-------|-------|
| Minimum wall thickness | **0.95mm** | For bins < 6U tall |
| Medium wall thickness | **1.2mm** | For bins 6-12U tall |
| Heavy wall thickness | **1.6mm** | For bins >= 12U tall |
| Divider wall thickness | **1.2mm** | Between compartments |
| Internal fillet radius | **2.8mm** | Compartment corner rounding |
| Cup floor thickness | **0.7mm** | Minimum floor above base |

#### Weighted Baseplate Cutouts

| Parameter | Value | Notes |
|-----------|-------|-------|
| Bottom part height | **6.4mm** | Weighted section depth |
| Square cutout size | **21.4mm x 21.4mm** | Center weight pocket |
| Square cutout depth | **4.0mm** | |
| Rounded cutout width | **8.5mm** | |
| Rounded cutout length | **4.25mm** | |
| Rounded cutout depth | **2.0mm** | |

#### Height System

```
Total bin height = (num_z * 7mm) + stacking_lip_height

Standard heights:
  1U =  7mm +  lip =  ~10.55mm
  2U = 14mm +  lip =  ~17.55mm
  3U = 21mm +  lip =  ~24.55mm
  6U = 42mm +  lip =  ~45.55mm
  10U = 70mm + lip =  ~73.55mm (common kitchen drawer height)
```

---

## 1. README.md Maintenance

The `README.md` must always be kept up-to-date and include rendered images of each SCAD design.

### Required Images

For **each** `.scad` file in the project, the README must display two images:

1. **Front-angle view**: Camera positioned at 45° azimuth (horizontal rotation) and 45° elevation (vertical tilt) from the front face
2. **Rear-angle view**: Camera positioned at the opposite angle (225° azimuth, 45° elevation) - effectively the "flipped" view showing the back

### Image Specifications

- Format: PNG
- Resolution: 800x600 pixels minimum
- Background: Transparent or neutral gray
- Naming convention: `<scad_filename>_front.png` and `<scad_filename>_rear.png`

### Camera Settings

OpenSCAD camera parameter format: `--camera=translateX,translateY,translateZ,rotX,rotY,rotZ,distance`

**Important**: The camera must be centered on the model's bounding box center, not the origin. Use `--autocenter --viewall` flags to automatically frame the entire model.

Camera angles:
- **Front view**: `rotX=55, rotY=0, rotZ=45` (looking from front-right-above)
- **Rear view**: `rotX=55, rotY=0, rotZ=225` (looking from back-left-above)

The `--autocenter` flag centers the model at the origin before rendering, and `--viewall` adjusts the camera distance to fit the entire model in frame.

### README Structure

```markdown
## Designs

### [Design Name]

| Front View | Rear View |
|------------|-----------|
| ![Front](https://github.com/<owner>/<repo>/releases/latest/download/<design>_front.png) | ![Rear](https://github.com/<owner>/<repo>/releases/latest/download/<design>_rear.png) |

[Description of the design and its purpose]

**Grid Size**: [e.g., 2x3 units, 84mm x 126mm]

**Documentation**: [design.md](<design>.md)

**Download STL**: [design.stl](https://github.com/<owner>/<repo>/releases/latest/download/<design>.stl)
```

Note: Replace `<owner>/<repo>` with the actual GitHub repository path. Images and STL files are served from GitHub releases since generated files are not committed to the repository.

### GitHub Releases

Each release must include:
- All generated STL files
- All rendered PNG images (both views)
- The release pipeline should automatically generate these artifacts

---

## 2. OpenSCAD Coding Conventions

### Coordinate System Convention

All designs use the following coordinate system (document this at the top of each file):

```openscad
/**
 * Coordinate System:
 *   X = Width  (positive = right)
 *   Y = Depth  (positive = back/away from viewer)
 *   Z = Height (positive = up)
 *
 * Origin: Front-left corner of baseplate, at ground level
 *
 * Gridfinity Grid:
 *   Each grid unit = 42mm x 42mm
 *   Grid position [gx, gy] maps to world [gx*42, gy*42]
 */
```

### Dimension Variables

All measurements must be defined as named constants at the top of the file. **Never use hardcoded numbers in geometry.**

#### Gridfinity Standard Constants

Every file that uses Gridfinity dimensions must include the full standard constants. These are derived from the official specification and community implementations. **Do not modify these values.**

```openscad
// ===========================================
// GRIDFINITY STANDARD DIMENSIONS (do not modify)
// ===========================================

// --- Core Grid ---
grid_pitch_mm           = 42;       // Grid unit size (center-to-center)
grid_zpitch_mm          = 7;        // Height unit size (1U = 7mm)
grid_clearance_mm       = 0.5;      // Bin-to-baseplate clearance per side
grid_outer_size_mm      = 41.5;     // Bin outer size (pitch - clearance)
grid_taper_angle        = 45;       // All bevels at 45 degrees

// --- Corner Geometry ---
corner_radius_mm        = 3.75;     // Outer corner radius
base_bottom_radius_mm   = 0.8;     // Bottom corner radius (corner_radius - profile_width)

// --- Base Profile (male, stepped bottom of bins) ---
base_bevel1_mm          = 0.8;      // First bevel: 0.8mm at 45°
base_riser_mm           = 1.8;      // Vertical riser height
base_bevel2_mm          = 2.15;     // Second bevel: 2.15mm at 45°
base_profile_width_mm   = 2.95;     // Total profile X extent (0.8 + 0 + 2.15)
base_profile_height_mm  = 4.75;     // Total profile Z extent (0.8 + 1.8 + 2.15)
base_total_height_mm    = 7;        // Profile + bridge (4.75 + 2.25)
base_bridge_height_mm   = 2.25;     // Flat section tying base units together

// --- Baseplate Profile (female, 0.1mm smaller for clearance) ---
bp_bevel1_mm            = 0.7;      // First bevel: 0.7mm at 45°
bp_riser_mm             = 1.8;      // Vertical riser height
bp_bevel2_mm            = 2.15;     // Second bevel: 2.15mm at 45°
bp_profile_width_mm     = 2.85;     // Total profile X extent
bp_profile_height_mm    = 4.65;     // Total profile Z extent
bp_min_height_mm        = 5;        // Minimum baseplate height

// --- Stacking Lip (top edge of bins) ---
lip_lower_taper_mm      = 0.7;      // Lower 45° taper
lip_riser_mm            = 1.8;      // Vertical riser
lip_upper_taper_mm      = 1.9;      // Upper 45° taper
lip_depth_mm            = 2.6;      // Total lip depth (X)
lip_nominal_height_mm   = 4.4;      // Nominal height (Z)
lip_fillet_radius_mm    = 0.6;      // Fillet at outer tip
lip_actual_height_mm    = 3.55;     // Actual height with fillet
lip_support_height_mm   = 1.2;      // Support wall below lip

// --- Magnet & Screw Holes ---
magnet_diameter_mm      = 6;        // Physical magnet diameter
magnet_hole_diameter_mm = 6.5;      // Hole diameter (with clearance)
magnet_thickness_mm     = 2;        // Physical magnet thickness
magnet_hole_depth_mm    = 2.4;      // Hole depth (magnet + 2 layer heights)
screw_diameter_mm       = 3;        // M3 screw
screw_hole_depth_mm     = 6;        // Screw hole depth
hole_from_edge_mm       = 4.8;      // Hole center from base outer edge
hole_from_center_mm     = 13;       // Hole center from cell center

// --- Walls & Compartments ---
wall_thin_mm            = 0.95;     // Minimum wall thickness (bins < 6U)
wall_medium_mm          = 1.2;      // Medium wall thickness (bins 6-12U)
wall_thick_mm           = 1.6;      // Heavy wall thickness (bins >= 12U)
divider_thickness_mm    = 1.2;      // Internal divider wall thickness
internal_fillet_mm      = 2.8;      // Compartment corner fillet radius
cup_floor_thickness_mm  = 0.7;      // Minimum floor above base

// --- Pad Corner Position ---
pad_corner_position_mm  = 17;       // pitch/2 - 4 (MUST be 17 for compatibility)

// --- Manufacturing ---
layer_height_mm         = 0.2;      // Default FDM layer height
tolerance_mm            = 0.02;     // Cut overlap / anti-z-fighting
```

#### Naming Convention

Variables follow this pattern: `<scope>_<description>_mm`

- **Object-specific dimensions**: Prefix with the module/function name
  - `utensil_bin_width_mm` - width specific to utensil_bin module
  - `spice_rack_divider_thickness_mm` - divider thickness in spice_rack module

- **Shared/global dimensions**: Use descriptive prefix
  - `wall_thickness_mm` - common wall thickness used throughout
  - `drawer_height_mm` - target drawer interior height

- **Grid-unit dimensions**: Use `_gu` suffix for grid units
  - `bin_width_gu = 2` - bin is 2 grid units wide
  - `bin_depth_gu = 3` - bin is 3 grid units deep

- **Always use `_mm` suffix** for millimeter measurements

#### Example

```openscad
// ===========================================
// DIMENSIONS (all values in millimeters)
// ===========================================

// --- Global / Shared Dimensions ---
wall_thickness_mm     = 1.2;    // Common wall thickness (3 perimeters at 0.4mm)
drawer_height_mm      = 70;     // Target drawer interior height

// --- utensil_bin dimensions ---
utensil_bin_width_gu  = 2;      // Grid units wide (X)
utensil_bin_depth_gu  = 3;      // Grid units deep (Y)
utensil_bin_width_mm  = utensil_bin_width_gu * grid_pitch_mm - grid_clearance_mm;
utensil_bin_depth_mm  = utensil_bin_depth_gu * grid_pitch_mm - grid_clearance_mm;
utensil_bin_height_mm = 60;     // Z dimension (interior height)

// --- spice_rack dimensions ---
spice_rack_width_gu   = 3;      // Grid units wide (X)
spice_rack_depth_gu   = 2;      // Grid units deep (Y)
spice_rack_divider_count     = 4;
spice_rack_divider_thickness_mm = 1.2;
```

#### Usage in Modules

```openscad
// GOOD: All dimensions from variables, grid-aware
module utensil_bin() {
    difference() {
        cube([utensil_bin_width_mm, utensil_bin_depth_mm, utensil_bin_height_mm]);
        translate([wall_thickness_mm, wall_thickness_mm, base_height_mm])
            cube([utensil_bin_width_mm - 2*wall_thickness_mm,
                  utensil_bin_depth_mm - 2*wall_thickness_mm,
                  utensil_bin_height_mm]);
    }
}

// BAD: Hardcoded numbers
module utensil_bin() {
    cube([83.5, 125.5, 60]);  // NO! Use variables
}
```

### Modular Function Design

Every distinct object or component must be defined in its own module:

```openscad
// GOOD: Separate modules for each component
module gridfinity_base() { ... }
module bin_walls() { ... }
module stacking_lip() { ... }
module dividers() { ... }

// BAD: Everything in one monolithic block
difference() {
    cube([100, 100, 10]);
    // ... hundreds of lines ...
}
```

### Position and Alignment Documentation

Each module must include a comprehensive comment block:

```openscad
/**
 * Utensil Bin - Tall container for spatulas, spoons, etc.
 *
 * GRID SIZE: 2x3 units (84mm x 126mm footprint)
 *
 * POSITION:
 *   Origin: [0, 0, 0] (front-left-bottom corner)
 *
 * BOUNDING BOX:
 *   Min: [0, 0, 0]
 *   Max: [83.5, 125.5, 65]
 *   Size: [83.5, 125.5, 65]
 *
 * COMPONENTS:
 *   - gridfinity_base: [0, 0, 0] to [83.5, 125.5, 5]
 *   - bin_walls: [0, 0, 5] to [83.5, 125.5, 65]
 *   - stacking_lip: integrated into top edge of bin_walls
 *
 * CONNECTS TO:
 *   - Gridfinity baseplate: base snaps into standard baseplate grid
 */
module utensil_bin() {
    gridfinity_base(utensil_bin_width_gu, utensil_bin_depth_gu);
    bin_walls();
    stacking_lip();
}
```

### Gridfinity Base Module

All bins must use a shared Gridfinity base module for compatibility:

```openscad
/**
 * Standard Gridfinity Base
 *
 * Generates a Gridfinity-compatible base for the given grid dimensions.
 * Includes the stepped base profile for baseplate compatibility.
 *
 * Parameters:
 *   width_gu  - Width in grid units
 *   depth_gu  - Depth in grid units
 *   magnets   - Include magnet holes (default: false)
 */
module gridfinity_base(width_gu, depth_gu, magnets = false) {
    // ... standard base geometry ...
}
```

### Connection Interfaces

Define named attachment points that components share:

```openscad
// ===========================================
// CONNECTION INTERFACES
// ===========================================

// Top surface of gridfinity base where walls begin
function base_top_surface_z() = base_height_mm;

// Interior cavity origin (inside walls, above base)
function interior_origin(wall_t = wall_thickness_mm) = [
    wall_t,
    wall_t,
    base_height_mm
];
```

### Component Index Table

Include a master index of all components at the top of the file:

```openscad
/**
 * ===========================================
 * COMPONENT INDEX
 * ===========================================
 *
 * | Component        | Origin [X,Y,Z]  | Size [W,D,H]       | Grid Size | Attaches To       |
 * |------------------|-----------------|---------------------|-----------|-------------------|
 * | gridfinity_base  | [0, 0, 0]       | [83.5, 125.5, 5]   | 2x3       | baseplate         |
 * | bin_walls        | [0, 0, 5]       | [83.5, 125.5, 60]  | -         | gridfinity_base   |
 * | stacking_lip     | [0, 0, 60]      | [83.5, 125.5, 2.6] | -         | bin_walls top     |
 *
 */
```

### Debug and Visualization Modules

Every SCAD file must include these debug modules:

```openscad
// ===========================================
// DEBUG / VISUALIZATION
// ===========================================

// Render coordinate axes at origin (length = 50mm)
module debug_axes(length = 50) {
    color("red")   translate([0,0,0]) cylinder(h=length, r=1, $fn=16);  // Z
    color("green") rotate([0,90,0]) cylinder(h=length, r=1, $fn=16);    // X
    color("blue")  rotate([-90,0,0]) cylinder(h=length, r=1, $fn=16);   // Y
}

// Show bounding box for a component (transparent)
module debug_bounds(min_pt, max_pt) {
    color("yellow", 0.2)
        translate(min_pt)
            cube(max_pt - min_pt);
}

// Show grid overlay for Gridfinity alignment
module debug_grid(width_gu, depth_gu) {
    for (x = [0:width_gu])
        translate([x * grid_pitch_mm, 0, 0])
            color("red", 0.3) cube([0.5, depth_gu * grid_pitch_mm, 0.5]);
    for (y = [0:depth_gu])
        translate([0, y * grid_pitch_mm, 0])
            color("red", 0.3) cube([width_gu * grid_pitch_mm, 0.5, 0.5]);
}

// Color-coded assembly for visual debugging
module assembly_colored() {
    color("gray")   gridfinity_base(bin_width_gu, bin_depth_gu);
    color("blue")   bin_walls();
    color("green")  stacking_lip();
}

// Exploded view - separates components along Z axis
module assembly_exploded(separation = 30) {
    translate([0, 0, 0])              gridfinity_base(bin_width_gu, bin_depth_gu);
    translate([0, 0, separation])     bin_walls();
    translate([0, 0, separation * 2]) stacking_lip();
}
```

### Assembly Module

The main assembly module combines all components:

```openscad
/**
 * Main Assembly
 *
 * Combines all components in their final positions.
 * Use assembly_colored() or assembly_exploded() for debugging.
 */
module assembly() {
    gridfinity_base(bin_width_gu, bin_depth_gu);
    bin_walls();
    stacking_lip();
}

// Default render
assembly();
```

---

## 3. Documentation Files

### Per-Design Documentation

**IMPORTANT**: For each `.scad` file, create and maintain a corresponding `.md` file in the **same directory** as the `.scad` file.

| SCAD File | Documentation File |
|-----------|-------------------|
| `utensil_bin.scad` | `utensil_bin.md` |
| `spice_rack.scad` | `spice_rack.md` |
| `cutlery_tray.scad` | `cutlery_tray.md` |

### Documentation File Structure

Each documentation file must contain:

```markdown
# [Design Name]

## Overview

Brief description of what this insert is for and what kitchen items it holds.

## Grid Size

| Property | Value |
|----------|-------|
| Grid Units | 2x3 |
| Footprint | 84mm x 126mm |
| Height | 65mm |
| Target Drawer | Utensil drawer |

## Dimensions

| Parameter | Value | Description |
|-----------|-------|-------------|
| **Gridfinity Standard** | | |
| grid_pitch_mm | 42 | Grid unit size |
| grid_clearance_mm | 0.5 | Bin-to-baseplate clearance |
| **Bin Specific** | | |
| wall_thickness_mm | 1.2 | Wall thickness (3 perimeters) |
| bin_height_mm | 60 | Interior height |
| divider_count | 3 | Number of internal dividers |

## Component Diagram

### Top View (looking down, -Z)

```
        +Y (back)
           ^
           |
    +------+------+
    |  |   |   |  |
    |  | D | D |  |   D = divider
    |  |   |   |  |
    |  |   |   |  |
    +--+---+---+--+  --> +X (right)
   /
  Origin [0,0]
```

### Front View (looking from front, -Y)

```
        +Z (up)
           ^
    +------+------+------+
    |      |      |      |
    |      |      |      |  bin compartments
    |      |      |      |
    +------+------+------+
    |   gridfinity base  |
    +--------------------+----> +X (right)
```

## Components

### gridfinity_base

- **Purpose**: Standard Gridfinity base for baseplate compatibility
- **Position**: Origin [0, 0, 0]
- **Bounding Box**: [0,0,0] to [83.5, 125.5, 5]

### bin_walls

- **Purpose**: Outer walls of the container
- **Position**: Origin [0, 0, 5]
- **Attachment**: Sits on gridfinity_base top surface

[Continue for each component...]

## Print Settings

- Print orientation: Base facing down (no supports needed)
- Recommended infill: 15-20%
- Material: PLA or PETG
- Layer height: 0.2mm recommended
- Perimeters: 3 minimum for wall strength

## Changelog

| Date | Change |
|------|--------|
| YYYY-MM-DD | Initial design |
```

### Keeping Documentation in Sync

**CRITICAL**: Whenever a `.scad` file is modified:
1. Update the corresponding `<name>.md` file (same directory as the `.scad`)
2. Update the component index table in both files
3. Update ASCII diagrams if positions change
4. Update the README.md if the design's purpose or images change

---

## 4. Build Scripts and Generated Files

### Scripts

The repository must contain build scripts for both platforms:

- `build.sh` - Bash script for Linux/macOS
- `build.bat` - Batch script for Windows

Both scripts must:
1. Find all `.scad` files in the project
2. Generate STL files for each design
3. Generate PNG images (front and rear views) for each design
4. Output files to a `build/` directory

### Script Requirements

- Use OpenSCAD command-line interface
- Camera settings for images:
  - Use `--autocenter` to center the model at origin before rendering
  - Use `--viewall` to automatically fit the entire model in frame
  - Front view: `--camera=0,0,0,55,0,45,0` (rotX=55, rotY=0, rotZ=45)
  - Rear view: `--camera=0,0,0,55,0,225,0` (rotX=55, rotY=0, rotZ=225)
  - Note: With `--viewall`, the distance parameter (last value) is ignored
- Image size: `--imgsize=800,600`
- Output format: `--export-format=binstl` for STL files

### .gitignore

The following must be in `.gitignore`:

```
# Generated build artifacts
build/
*.stl
*.png

# Exception: Keep source images if any exist in assets/
!assets/*.png
```

Generated files should never be committed to the repository - they are produced by the build scripts and included only in GitHub releases.

---

## 5. Gridfinity OpenSCAD Patterns & Techniques

Reference implementations: [kennetek/gridfinity-rebuilt-openscad](https://github.com/kennetek/gridfinity-rebuilt-openscad) (MIT), [ostat/gridfinity_extended_openscad](https://github.com/ostat/gridfinity_extended_openscad) (GPL-3.0), [vector76/gridfinity_openscad](https://github.com/vector76/gridfinity_openscad) (MIT).

### 5.1 Base Profile Construction

The base profile is the heart of Gridfinity compatibility. Two approaches are used in the community:

#### Approach A: Sweep a 2D polygon around rounded rectangle (kennetek)

```openscad
// Define the base profile as a 2D polygon
BASE_PROFILE = [
    [0, 0],                                         // Bottom point
    [base_bevel1_mm, base_bevel1_mm],               // [0.8, 0.8]
    [base_bevel1_mm, base_bevel1_mm + base_riser_mm], // [0.8, 2.6]
    [base_profile_width_mm, base_profile_height_mm]  // [2.95, 4.75]
];

// Sweep the profile around a rounded rectangular path
module sweep_rounded(inner_size) {
    // inner_size = inner rectangle dimensions [x, y]
    // Children must be a 2D shape in Quadrant 1
    // Generates 4 straight segments (linear_extrude) + 4 corners (rotate_extrude)

    for (i = [0:3]) {
        rotate([0, 0, 90*i])
        translate([inner_size.x/2, -inner_size.y/2, 0]) {
            // Straight segment along each edge
            rotate([90, 0, 90])
                linear_extrude(height = inner_size[i%2 == 0 ? 1 : 0])
                    children();
            // Quarter-turn corner
            rotate_extrude(angle = 90)
                children();
        }
    }
}

// Use: sweep_rounded(inner_dims) base_profile_polygon();
```

#### Approach B: Hull over corner cylinders at varying heights (vector76/ostat)

```openscad
// Build profile from hulled cylinder stacks at pad corners
module pad_oversize(num_x, num_y, margins=0) {
    pad_corner = pad_corner_position_mm;  // 17mm from cell center
    radialgap = margins * 0.25;           // 0.25mm clearance for female parts

    // Bottom bevel: hull two cylinder stacks
    hull() {
        cornercopy(pad_corner, num_x, num_y)
            cylinder(d=1.6 + radialgap*2, h=0.1, $fn=32);       // z=0
        cornercopy(pad_corner, num_x, num_y)
            cylinder(d=3.2 + radialgap*2, h=1.9, $fn=32);       // z=0 to 1.9 (bevel1_top=0.8, extends to 0.8+1.1)
    }

    // Upper bevel: hull two cylinder stacks
    hull() {
        cornercopy(pad_corner, num_x, num_y)
            translate([0,0,2.6])
                cylinder(d=3.2 + radialgap*2, h=0.1, $fn=32);   // z=2.6
        cornercopy(pad_corner, num_x, num_y)
            translate([0,0,5.0])
                cylinder(d=8.2 + radialgap*2, h=0.2, $fn=32);   // z=5.0 (with bonus)
    }
}

// Helper: place children at all 4 corners
module cornercopy(r, num_x=1, num_y=1) {
    for (xi=[0:num_x-1], yi=[0:num_y-1])
        for (xx=[-1,1], yy=[-1,1])
            translate([xx*r + xi*grid_pitch_mm, yy*r + yi*grid_pitch_mm, 0])
                children();
}
```

### 5.2 Stacking Lip Construction

```openscad
// Stacking lip profile (2D, for sweeping around bin top edge)
STACKING_LIP_PROFILE = [
    [0, 0],                                                // Inner tip
    [lip_lower_taper_mm, lip_lower_taper_mm],              // [0.7, 0.7]
    [lip_lower_taper_mm, lip_lower_taper_mm + lip_riser_mm], // [0.7, 2.5]
    [lip_depth_mm, lip_nominal_height_mm]                   // [2.6, 4.4]
];

// The lip includes a support section below to connect to the bin wall:
//   support_height = lip_support_height_mm + tan(45) * lip_depth_mm
//                  = 1.2 + 2.6 = 3.8mm

// Lip styles (from ostat):
//   "normal"          - Full lip for standard stacking
//   "reduced"         - Shorter lip for low-profile bins (< 1.8U)
//   "reduced_double"  - Double-reduced for very low bins
//   "minimum"         - Minimal lip
//   "none"            - No lip (auto-selected when < 1.2U)
```

### 5.3 Grid Copy & Iteration Patterns

```openscad
// Place children at each cell in a grid
module gridcopy(num_x, num_y) {
    for (xi = [0:num_x-1], yi = [0:num_y-1])
        translate([xi * grid_pitch_mm, yi * grid_pitch_mm, 0])
            children();
}

// Place children at all 4 corners of a rectangle
module cornercopy(r, num_x=1, num_y=1) {
    for (xx = [-1, 1], yy = [-1, 1])
        translate([xx * r, yy * r, 0])
            children();
}

// Combine grid + corner iteration for magnet/screw holes
module gridcopycorners(num_x, num_y, r, only_outer=false) {
    for (xi = [0:num_x-1], yi = [0:num_y-1])
        for (xx = [-1, 1], yy = [-1, 1])
            if (!only_outer || (xi==0 && xx==-1) || (xi==num_x-1 && xx==1)
                            || (yi==0 && yy==-1) || (yi==num_y-1 && yy==1))
                translate([xi*grid_pitch_mm + xx*r, yi*grid_pitch_mm + yy*r, 0])
                    children();
}
```

### 5.4 Bin Construction Pattern (Additive then Subtractive)

```openscad
// The standard bin construction follows this pattern:
//
// 1. Create solid block with base pads and outer hull
// 2. Subtract stacking socket from top (reuse pad_oversize with margins)
// 3. Subtract magnet/screw holes from base
// 4. Subtract interior cavity (compartments, dividers)
//
// bin = grid_block - stacking_cutout - holes - cavity + dividers

module basic_cup(num_x, num_y, num_z, ...) {
    difference() {
        // Positive: solid block with base pads
        grid_block(num_x, num_y, num_z);

        // Negative: interior cavity with lip
        translate([0, 0, base_total_height_mm])
            partitioned_cavity(num_x, num_y, num_z, divx, divy);
    }
}

module grid_block(num_x, num_y, num_z) {
    total_ht = num_z * grid_zpitch_mm + lip_actual_height_mm;

    union() {
        // Outer shell: rounded rectangle extruded to full height
        linear_extrude(total_ht)
            rounded_square([num_x * grid_pitch_mm - grid_clearance_mm,
                           num_y * grid_pitch_mm - grid_clearance_mm],
                          corner_radius_mm, center=true);

        // Base pads at each grid position
        gridcopy(num_x, num_y)
            pad_oversize(1, 1, margins=0);
    }

    // Subtract stacking socket from top
    difference() {
        children();
        translate([0, 0, total_ht - base_profile_height_mm])
            gridcopy(num_x, num_y)
                pad_oversize(1, 1, margins=1);  // 0.25mm clearance
    }
}
```

### 5.5 Compartment & Divider Pattern

```openscad
// Subdivide bin interior into compartments
module partitioned_cavity(num_x, num_y, num_z, divx, divy) {
    cavity_width  = num_x * grid_pitch_mm - grid_clearance_mm - 2*wall_thin_mm;
    cavity_depth  = num_y * grid_pitch_mm - grid_clearance_mm - 2*wall_thin_mm;
    cavity_height = num_z * grid_zpitch_mm;

    difference() {
        // Full cavity
        translate([wall_thin_mm, wall_thin_mm, 0])
            rounded_cube([cavity_width, cavity_depth, cavity_height],
                        internal_fillet_mm);

        // Divider walls (X direction)
        if (divx > 1)
            for (i = [1:divx-1])
                translate([wall_thin_mm + i * cavity_width/divx - divider_thickness_mm/2,
                          wall_thin_mm, 0])
                    cube([divider_thickness_mm, cavity_depth, cavity_height]);

        // Divider walls (Y direction)
        if (divy > 1)
            for (i = [1:divy-1])
                translate([wall_thin_mm,
                          wall_thin_mm + i * cavity_depth/divy - divider_thickness_mm/2, 0])
                    cube([cavity_width, divider_thickness_mm, cavity_height]);
    }
}
```

### 5.6 Overhang Remedy (Supportless Printing)

For magnet/screw holes, use bridging layers so FDM printers can print without supports:

```openscad
// Sequential bridging: alternating rectangular bridges narrow from outer to inner diameter
module make_hole_printable(inner_r, outer_r, layers=2) {
    for (i = [0:layers-1]) {
        bridge_r = outer_r - i * (outer_r - inner_r) / layers;
        rotate([0, 0, 90 * i])  // Alternate 90° for each layer
            translate([0, 0, i * layer_height_mm])
                cube([bridge_r * 2, outer_r * 2, layer_height_mm], center=true);
    }
}

// Alternative: single thin bridge at magnet-to-screw transition
module overhang_fix(magnet_r, screw_r) {
    translate([0, 0, -0.001])
        render()
        intersection() {
            cube([screw_r*2 + 0.1, magnet_r*2 + 0.1, 0.3], center=true);
            cylinder(r=magnet_r, h=0.3, center=true, $fn=32);
        }
}
```

### 5.7 Efficient Floor (Material-Saving)

```openscad
// Creates a non-flat floor that follows pad contours to save material
// Only usable when no magnets/screws/finger-slides are present
module efficient_floor(num_x, num_y) {
    gridcopy(num_x, num_y) {
        hull() {
            // Bottom: small spheres at pad corners
            translate([0, 0, cup_floor_thickness_mm])
                cornercopy(pad_corner_position_mm)
                    sphere(r=1, $fn=16);
            // Top: full-size corner cylinders
            translate([0, 0, base_profile_height_mm])
                cornercopy(pad_corner_position_mm)
                    cylinder(r=corner_radius_mm, h=4, $fn=32);
        }
    }
}
```

### 5.8 Label Tab Pattern

```openscad
// Label overhang on bin wall for label strips
tab_depth_mm    = 15.85;
tab_angle       = 36;        // degrees
tab_support_mm  = 1.2;
tab_height_mm   = tan(tab_angle) * tab_depth_mm + tab_support_mm;  // ~12.72mm

// Tab polygon (2D cross-section, extruded across bin width)
TAB_POLYGON = [
    [0, 0],
    [0, tab_height_mm],
    [tab_depth_mm, tab_height_mm],
    [tab_depth_mm, tab_height_mm - tab_support_mm]
];
```

### 5.9 Finger Slide / Scoop

```openscad
// Scoop cutout on front wall for easy item retrieval
module finger_slide(width, depth, height) {
    facets = 13;
    pivot = [0, depth, height];
    for (i = [0:facets-1]) {
        angle = 90 * i / (facets - 1);
        rotate([0, 0, 0])
            translate(pivot)
                rotate([angle, 0, 0])
                    translate(-pivot)
                        cube([width, depth, height]);
    }
}
```

### 5.10 OpenSCAD Customizer Integration

Use section comments and inline type hints for the OpenSCAD Customizer GUI:

```openscad
/* [Grid Size] */
bin_width_gu = 2;        // Grid units wide (X)
bin_depth_gu = 3;        // Grid units deep (Y)
bin_height_u = 6;        // Height units (1U = 7mm)

/* [Compartments] */
dividers_x = 1;          // Number of X dividers
dividers_y = 2;          // Number of Y dividers

/* [Features] */
include_magnets = false;  // Add magnet holes
include_screws = false;   // Add screw holes
include_lip = true;       // Add stacking lip
scoop_percent = 0; //[0:5:100]  // Finger scoop depth (%)
label_style = "none"; //[none, full, left, center, right]

/* [Advanced] */
wall_thickness_mm = 0.95; //0.05  // Step size 0.05mm

// Sentinel to hide internal code from Customizer
module end_of_customizer_opts() {}
```

### 5.11 Rounded Shape Helpers

```openscad
// Rounded square (2D) - used extensively for bin cross-sections
module rounded_square(size, radius, center=false) {
    offset(r=radius)
        offset(r=-radius)
            square(size, center=center);
}

// Rounded cube (3D) - for compartment cutouts with filleted corners
module rounded_cube(size, radius) {
    hull() {
        for (x = [radius, size.x - radius])
            for (y = [radius, size.y - radius])
                translate([x, y, 0])
                    cylinder(r=radius, h=size.z, $fn=32);
    }
}

// Shorthand translate
module tz(z) { translate([0, 0, z]) children(); }
```

### 5.12 Wall Patterns (Advanced, from ostat)

Available wall/floor decorative patterns:
- `hexgrid` - Hexagonal grid cutouts
- `grid` - Rectangular grid cutouts
- `voronoi` - Voronoi tessellation
- `brick` / `brickoffset` - Brick-like patterns
- `slats` - Horizontal/vertical slat cutouts

These are created by generating a 2D pattern, then using `linear_extrude` + `intersection` with the wall surface.

### 5.13 File Organization Pattern

Recommended project structure following community conventions:

```
kitchen-grid-finity/
├── CLAUDE.md                          # This file
├── README.md                          # Project overview with images
├── .gitignore
├── build.sh / build.bat               # Build scripts
├── src/
│   ├── gridfinity_constants.scad      # All standard constants (shared)
│   ├── gridfinity_base.scad           # Base profile module (shared)
│   ├── gridfinity_lip.scad            # Stacking lip module (shared)
│   ├── gridfinity_helpers.scad        # Utility modules (shared)
│   └── gridfinity_baseplate.scad      # Baseplate module (shared)
├── cutlery_tray.scad                  # Top-level designs (customizer-ready)
├── cutlery_tray.md
├── utensil_bin.scad
├── utensil_bin.md
├── spice_rack.scad
├── spice_rack.md
└── baseplate.scad
```

Each top-level `.scad` file should `include` the shared modules from `src/`.

---

## 6. Kitchen Drawer Insert Design Guidelines

### Measuring Drawers

Before designing inserts, measure and document:
- **Interior width** (mm) - side to side
- **Interior depth** (mm) - front to back
- **Interior height** (mm) - bottom to top of drawer sides
- **Number of baseplates needed** to cover the drawer floor
- **Drawer slide clearance** - any obstructions from slides or hardware

Calculate grid coverage: `floor(width / 42) x floor(depth / 42)` grid units, then decide on baseplate oversize method (`crop`, `fill`, or `outer`).

### Common Kitchen Insert Types

Design inserts for common kitchen organization needs:

| Insert Type | Typical Grid Size | Height (U) | Notes |
|-------------|-------------------|------------|-------|
| Cutlery tray | 1x4 per slot | 3-4U | Forks, knives, spoons |
| Utensil bin | 2x2 or 2x3 | 8-10U | Spatulas, whisks, tongs |
| Spice rack | 1x1 per jar | 4-6U | Standard spice jars (~42mm dia) |
| Knife block | 2x3 | 6-8U | Slotted for blade storage |
| Wrap/foil holder | 1x2 | 8-10U | Tall narrow for rolls |
| Lid organizer | 3x2 | 6-8U | Vertical slots for pot/pan lids |
| K-cup holder | 1x1 per cup | 3U | Single-serve coffee pods |
| Junk drawer | mixed | 3-4U | Varied compartment sizes |

### Gridfinity Baseplate

Include a baseplate design that can be tiled to cover any drawer floor. The baseplate anchors all bins in place. Options:
- **Plain baseplate**: Just the socket profile, minimal height (5mm)
- **Weighted baseplate**: Heavier base with weight pockets (keeps baseplate from sliding)
- **Screw-down baseplate**: Mounting holes for permanent installation
- **Magnetic baseplate**: Magnet holes in all corners for bin retention

### Kitchen-Specific Design Considerations

- **Food safety**: Use PETG or food-safe PLA; avoid PLA near heat sources
- **Washability**: Rounded internal corners (use `internal_fillet_mm = 2.8`) for easy cleaning
- **Drainage**: Consider adding small drain holes in bins that may get wet
- **Label tabs**: Include label overhangs on bins for easy identification
- **Finger scoops**: Add scoop cutouts on front walls for easy item retrieval
- **No magnets needed**: Kitchen drawers typically don't need magnet retention (friction from baseplate is sufficient)

---

## 7. Workflow Summary

1. **Measure**: Document target drawer dimensions
2. **Plan**: Determine grid layout and which inserts to create
3. **Development**: Create/modify `.scad` files following the modular conventions
4. **Documentation**: Update corresponding `<name>.md` (same directory) with ASCII diagrams and component details
5. **Local Testing**: Run `build.sh` or `build.bat` to generate STL and images locally
6. **README Update**: Ensure `README.md` references the design, documentation link, and release STL download
7. **Release**: GitHub Actions pipeline generates artifacts and attaches to release

---

## 8. Quick Reference: Required Updates Checklist

When creating or modifying a `.scad` file:

- [ ] Coordinate system documented at top
- [ ] Gridfinity standard constants included
- [ ] All dimensions as named variables (no hardcoded numbers)
- [ ] Variable names follow convention: `<module_name>_<description>_mm` or `_gu`
- [ ] Shared dimensions use descriptive prefix (e.g., `wall_thickness_mm`)
- [ ] Grid unit dimensions defined (e.g., `bin_width_gu = 2`)
- [ ] Component index table updated
- [ ] Each module has position/bounding box/alignment comments
- [ ] Connection interfaces defined as functions
- [ ] Debug modules included (axes, bounds, grid overlay, colored, exploded)
- [ ] `<name>.md` created/updated in same directory with ASCII diagrams
- [ ] `README.md` updated with design entry, grid size, and link to docs
