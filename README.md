# agentic-3d-modeling

A Claude Code skill for building **parametric, 3D-printable models that are numerically
verified, not eyeballed** — with build123d / OpenCascade + trimesh. Covers the verify
loop (render → section → MEASURE), the OCC geometry gotchas that bite silently
(winding-sets-extrude-direction, offset-on-scaled-sphere, blanket-mirror regressions),
snap-fit retention math, FDM support-free printability math, and reverse-engineering a
part's geometry from photos via a credit-card scale ruler + known-hole-pattern homography.

Distilled from working builds: `~/cap-model`, `~/fan-holder`, `~/connector-cap`,
`~/robot-power-box`. Showcase repo: `evnchn-agentic/agentic-cad-fan-stand`.

## The one load-bearing idea

A 3D render that "looks solid" proves nothing — a hollow with no occlusion looks
identical. Every constraint (fit, clearance, retention, printability) must become a
**falsifiable number** (trimesh proximity, a true 2D section, a volume-drop assertion),
not a vibe. That's the whole discipline.

## Install (Claude Code)

```bash
git clone https://github.com/evnchn-agentic/agentic-3d-modeling.git \
  ~/.claude/skills/agentic-3d-modeling
```

## Status

**Private until cleared for public release.** Before flipping public: the venv path and
worked-example repos are evnchn-specific; the *techniques* generalize but the paths and
`(memory: ...)` cross-references don't. Inline or genericize them for an outside reader.
