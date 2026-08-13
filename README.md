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

## Outputs

The notebook produces two local GLB sequences:

- `outputs/rv_mesh_sequence_glb/`: direct per-frame marching-cubes meshes for
  visualization and volume-oriented use.
- `outputs/rv_mesh_sequence_correspondent_glb/`: fixed-topology 30-frame
  baseline for deformation integration. It includes `metadata.json` and
  `rv_correspondence_sequence.npz`.

The correspondence sequence uses a reference-frame projection baseline. It
provides stable vertex IDs for software development but is not a clinically
validated physiological motion field.

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