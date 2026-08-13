# VisHeart RV 4D Reconstruction Notebook

An isolated prototype for producing a patient-specific right-ventricle (RV)
cavity mesh sequence from a VisHeart segmentation NIfTI. It does not modify or
import the production VisHeart system at runtime.

## Input

The notebook expects the VisHeart labelled-mask convention:

- `1`: RV cavity / blood pool
- `2`: LV myocardium
- `3`: LV cavity

Set `MASK_PATH` in `rv_4d_reconstruction.ipynb` to a local system-generated
NIfTI. Patient-derived NIfTI files and generated meshes are intentionally
ignored by Git and must not be uploaded without appropriate approval.

## Validation

The committed notebook contains executed validation outputs from the current
de-identified 30-frame test case:

- Direct mesh vs original RV-label volume: mean difference `1.28%`, maximum
  difference `3.67%`.
- RVEDV `172.02 mL`, RVESV `77.52 mL`, stroke volume `94.50 mL`, and RVEF
  `54.94%`. This is within the approximate 45-70% reconstruction sanity-check
  reference range; it is not a clinical diagnosis.
- Direct mesh integrity: `30/30` frames are watertight, winding-consistent,
  and have zero non-manifold edges.
- RVOT check: no frame has more than one component at or above the configured
  `MIN_COMPONENT_VOXELS` threshold.
- Chosen smoothing: `1` iteration. Settings `1`, `3`, and `5` had identical
  measured volume on this test case, so `1` is the minimum effective setting.
- Correspondence warning: the nearest-neighbour fixed-topology sequence is
  **not deformation-ready**. It causes collapsed vertices and degenerate faces
  in non-reference frames. Use direct meshes for visualization only; use a
  registration or learned motion method before deformation analysis.

## Outputs

The notebook produces two local GLB sequences:

- `outputs/rv_mesh_sequence_glb/`: direct per-frame marching-cubes meshes for
  visualization and volume-oriented use.
- `outputs/rv_mesh_sequence_correspondent_glb/`: experimental
  nearest-neighbour correspondence diagnostic. Read its `metadata.json`; it is
  not a valid deformation-motion handoff for the current test case.

## Run

```powershell
cd E:\Jy\visheart-rv-4d-notebook
py -m venv .venv
.venv\Scripts\python -m pip install -r requirements.txt
.venv\Scripts\jupyter lab
```

## Scope

This is a geometric baseline using mask cleanup and marching cubes. It is not
an RV-trained DeepSDF model. The next research phase is to audit data and train
or validate an RV-specific learned shape-and-motion model before any production
integration.
