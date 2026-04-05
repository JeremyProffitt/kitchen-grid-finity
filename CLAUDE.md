# Claude Code Guidelines for Kitchen Gridfinity Project

## Project Overview

This repository contains OpenSCAD designs for Gridfinity-compatible drawer inserts for kitchen organization. The designs follow the Gridfinity standard (42mm grid pitch, stackable bases) and are intended for 3D printing, optimized for the Bambu Labs H2D printer.

### Gridfinity Standard Reference

- **Grid pitch**: 42mm x 42mm per unit
- **Base height**: 5mm (standard baseplate)
- **Stacking lip**: 2.6mm height, designed for secure stacking
- **Base magnet holes**: 6.5mm diameter, 2.4mm deep (optional)
- **Corner radius**: 3.75mm on outer edges
- **Clearance**: 0.5mm between bins and baseplate

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

Every file that uses Gridfinity dimensions must define the standard constants:

```openscad
// ===========================================
// GRIDFINITY STANDARD DIMENSIONS (do not modify)
// ===========================================

grid_pitch_mm         = 42;     // Standard Gridfinity grid unit size
grid_clearance_mm     = 0.5;    // Clearance between bin and baseplate
base_height_mm        = 5;      // Standard baseplate height
stacking_lip_height_mm = 2.6;   // Height of stacking lip
corner_radius_mm      = 3.75;   // Standard corner radius
magnet_hole_diameter_mm = 6.5;  // Magnet hole diameter (optional)
magnet_hole_depth_mm  = 2.4;    // Magnet hole depth (optional)
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

## 5. Kitchen Drawer Insert Design Guidelines

### Measuring Drawers

Before designing inserts, measure and document:
- **Interior width** (mm) - side to side
- **Interior depth** (mm) - front to back
- **Interior height** (mm) - bottom to top of drawer sides
- **Number of baseplates needed** to cover the drawer floor

### Common Kitchen Insert Types

Design inserts for common kitchen organization needs:
- **Cutlery tray**: Divided compartments for forks, knives, spoons, etc.
- **Utensil bin**: Tall bins for spatulas, whisks, tongs
- **Spice rack**: Sized for standard spice jars
- **Knife block**: Slotted insert for knife blade storage
- **Wrap/foil organizer**: Tall narrow bins for rolls
- **Lid organizer**: Vertical slots for pot/pan lids
- **Junk drawer**: Mixed small-item organizer with varied compartment sizes

### Gridfinity Baseplate

Include a baseplate design that can be tiled to cover any drawer floor. The baseplate anchors all bins in place.

---

## 6. Workflow Summary

1. **Measure**: Document target drawer dimensions
2. **Plan**: Determine grid layout and which inserts to create
3. **Development**: Create/modify `.scad` files following the modular conventions
4. **Documentation**: Update corresponding `<name>.md` (same directory) with ASCII diagrams and component details
5. **Local Testing**: Run `build.sh` or `build.bat` to generate STL and images locally
6. **README Update**: Ensure `README.md` references the design, documentation link, and release STL download
7. **Release**: GitHub Actions pipeline generates artifacts and attaches to release

---

## 7. Quick Reference: Required Updates Checklist

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
