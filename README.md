# VisHeart RV 4D Reconstruction Notebook

An isolated notebook prototype that exports patient-specific RV cavity target meshes
from a VisHeart labelled NIfTI. It does not modify or import the production VisHeart
system and it does not use PyTorch, DeepSDF, or a learned model.

## PyTorch3D Target-Mesh Handoff

The teammate's PyTorch3D template-fitting pipeline should use the direct target meshes:

- `outputs/rv_mesh_sequence_glb/`: one GLB per cardiac frame.
- `outputs/rv_mesh_sequence_obj/`: one OBJ per cardiac frame; this is the convenient
  format for PyTorch3D's native OBJ loader.
- `outputs/rv_mesh_sequence_npz/rv_target_meshes.npz`: object arrays of per-frame
  `vertices` and `faces`, plus `frame_indices`, `affine`, and `voxel_sizes_mm`.

Each direct-output directory includes the same `metadata.json`. It records frame volumes,
ED/ES indices, integrity values, the full NIfTI affine, voxel sizes, and known volume bias.
Coordinates are exactly **NIfTI affine world coordinates**. No display or export rotation is
applied.

Vertex and face counts differ per frame by design. This is correct for independently sampled
Chamfer-loss target meshes and does not need to be made uniform: the template-fitting method
creates its own correspondence and can warm-start between frames.

## Executed Validation: patient005

| Check | Result |
| --- | --- |
| Input | `patient005_4d_segmentation`, 30 frames |
| Mean / max mesh-to-voxel volume difference | `1.28%` / `3.67%` |
| Mesh RVEDV / RVESV | `172.02 mL` / `77.52 mL` |
| Mesh RVEF | `54.94%` |
| Voxel RVEF | `56.68%` |
| Mesh minus voxel RVEF | `-1.74` percentage points |
| Signed error vs volume correlation | `r = -0.754` (larger underestimation at larger volumes) |
| Direct mesh integrity | `30/30` watertight, winding-consistent, zero non-manifold edges |
| RVOT component check | one significant RV component in every frame |
| Smoothing | `1` iteration; settings 1, 3, and 5 had identical measured volume |

The RVEF is within the approximate 45-70% reconstruction sanity-check reference range.
It is not a clinical diagnosis. The measured volume bias is systematic: direct mesh volume
underestimates the original label volume more strongly near end-diastole; consumers should use
`metadata.json` when comparing volume-derived results.

## Cross-Patient Validation

Set `MASK_PATHS` in the notebook configuration to add further local patient NIfTIs. The
executed notebook produces a per-patient summary of frame count, volume agreement, RVEDV/RVESV,
RVEF, integrity, and significant components per frame. Only `patient005` is currently available
locally, so degradation on other patients is **unknown**, not assumed absent.

## Experimental Negative Result

`outputs/rv_mesh_sequence_experimental_nearest_neighbor_glb/` preserves an audited failed
nearest-neighbour correspondence experiment. Do **not** use it as a deformation/motion field.
It failed in `29/30` non-reference frames; the worst case had `8,445` degenerate faces and
`4,269` collapsed vertices, and the largest frame-to-frame step was `33.7 mm`.

PyTorch3D template fitting should build correspondence itself from the direct target meshes.

## Input Convention

The notebook expects the VisHeart labelled-mask convention:

- `1`: RV cavity / blood pool
- `2`: LV myocardium
- `3`: LV cavity

Patient-derived NIfTI files and generated meshes are ignored by Git and must not be uploaded
without appropriate approval.

## Run

```powershell
cd E:\Jy\visheart-rv-4d-notebook
py -m venv .venv
.venv\Scripts\python -m pip install -r requirements.txt
.venv\Scripts\jupyter lab
```