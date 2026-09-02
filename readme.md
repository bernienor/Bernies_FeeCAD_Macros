# FreeCAD macros

These macros are generated via a "discussion" with Claude Desktop. It proved to 
be a much faster way to a good result in CAD compared to me doing the CAD'ing. 
And the result are compleatly reproduceable and deterministic. 

## WingBuilder.FCMacro

Parametric wing build, driven by a station definition file. Reads a set of
airfoil .dat files and places one profile wire per station, positioned and
scaled per a CSV of (dat_file, y_position, chord[, x_offset, z_offset,
twist_deg, bend_deg]) rows, then lofts them into a wing.

Convention used throughout:
    X = chordwise direction (LE at X=0, TE at X=chord, before any x_offset)
    Y = spanwise direction  (0 at root, increasing toward tip, before bend)
    Z = thickness direction (up/down, before bend)

Station file format (CSV, header row required):
    dat_file,y_position,chord
    root.dat,0,180
    tip.dat,600,80

x_offset (sweep), z_offset (dihedral), twist_deg and bend_deg columns are
optional (default 0). twist_deg rotates a section about its own leading
edge before it's positioned - positive values increase local incidence
(nose-up), negative values give washout. bend_deg rotates the whole
cross-section's plane about the chordwise axis - 0 is a normal flat
station (thickness along Z, growing along Y); as it approaches 90, the
section reorients into a winglet (thickness along Y, growing along Z).
bend_deg only reorients the profile - you still place a winglet's stations
along their own curving path via ordinary y_position/z_offset values.

Run this from the FreeCAD Macro menu (or the Macro toolbar) - it needs the
GUI, since it shows a dialog for picking the station file and options.

## WingMouldMacro.FCMacro

Macro to make mould of wings. It is not as generic as one would hope. Works 
best if there is no twist etc. in the wing. It is a good basis for a custom 
mould macro for a project. 

Builds a two-part mould around an existing wing solid, driven by a
settings dialog.

STAGE 1: enclosure block built from the wing's bounding box, expanded by
six independent extension distances (X+/X-/Y+/Y-/Z+/Z-). All zero
reproduces the bounding box exactly. Boolean-cut the wing out of that
block -> mould blank with a wing-shaped cavity.

STAGE 2: GUI dialog (PySide) to set/load/save settings and trigger builds.

STAGE 3 (this version): splits the mould blank into two halves along the
wing's natural parting line, defined for a vertical (Z) pull direction as
follows -- at each span station, the parting points are the two points on
that cross-section's outline that are extremal along the chordwise axis
(furthest forward / furthest aft). Those are exactly the points where a
straight vertical pull can no longer release the surface without locking.
The parting *line* along the span is built from these points at many
stations; the parting *surface* used to actually cut the blank continues
that line's height flat out to the edges of the mould block (fore/aft of
the airfoil, and past the root/tip), so it fully crosses the block.

STAGE 4: added a simple flat-plane parting option as a fallback next to
the contour-following one (kept, but parked for now -- see notes below).
Default is Z = 0.0 exactly, not auto-centered on the wing's bounding box:
for an untwisted wing whose sections were built with LE and TE on the
Z=0 plane (WingBuilder's convention), the exact flat parting height is
Z=0, not the bounding-box midpoint. The midpoint drifts away from the
true LE/TE line by the airfoil's camber, which is usually small but
non-zero -- and near the trailing edge, where the section pinches down
to near-zero thickness, even a small drift is enough for the flat plane
to miss the wing surface entirely on one side, leaving a thin unsplit
sliver of material along the TE in one of the two halves. If a wing ever
isn't built with LE/TE at Z=0 (or has twist), use "Auto-center on wing"
or the manual Z field to match wherever it actually sits, or switch to
the contour-following knife once that's revisited.

Assumptions worth checking against your wing geometry:
  - Pull/vertical axis is fixed at global Z.
  - Flat parting plane defaults to exact Z=0 (see STAGE 4 note above).
  - Span axis defaults to Y (configurable to X in the dialog).
  - The straight-line interpolation between the two parting points at
    each station is assumed to stay inside the (now-empty) wing cavity.
    This holds for ordinary airfoils; a very reflexed/cambered section
    could in principle violate it. Worth a visual check the first time
    you run this on a new wing.


USAGE:
  1. Select the wing solid, run the macro.
  2. "Build Mould Blank" first.
  3. "Split Into Mould Halves" second -- produces MouldTop and
     MouldBottom, plus MouldPartingSurface (the cutting surface itself,
     kept around for visual inspection).