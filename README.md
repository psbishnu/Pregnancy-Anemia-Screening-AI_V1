# Pregnancy Anemia Screening AI

**Offline multimodal AI research application for pregnancy anemia screening using seven-view pallor photography and complete blood count (CBC) features.**

Built with **HTML, CSS, JavaScript, ONNX Runtime Web**, and two exported ONNX neural-network checkpoints. Inference runs locally in the browser once the application assets have been installed. This project is a **research prototype, not a clinical diagnostic device**.

## Features

- Seven anatomical photo inputs, shown in this user-interface order: **Left Eye, Left Nail, Left Palm, Right Eye, Right Nail, Right Palm, Tongue**.
- Eight optional CBC laboratory measurement fields plus age; at least **one laboratory CBC measurement** is required for inference.
- Two model routes based on the number of uploaded images:
  - **1–6 photos:** `missing_aware.onnx`
  - **7 photos:** `complete_case.onnx`
- Binary anemia classification and four-class severity output (**Normal, Mild, Moderate, Severe**).
- Image attention weights for the seven views. Attention weights **do not demonstrate causal diagnostic importance**.
- Optional measured hemoglobin (Hb) field for reference only: **Hb is not supplied as an input to the neural network**.
- Local inference; the supplied interface does not intentionally transmit patient images or records to a remote prediction service, and does not save patient records.

## Repository structure

```text
Pregnancy-Anemia-Screening-AI/
├── index.html
├── README.md
├── logos/
│   ├── icmr.png
│   ├── rims.png
│   └── bit.png
├── models/
│   ├── complete_case.onnx
│   ├── complete_case_config.json
│   ├── missing_aware.onnx
│   └── missing_aware_config.json
└── runtime/
    ├── ort.min.js
    ├── ort-wasm-simd-threaded.jsep.mjs
    ├── ort-wasm-simd-threaded.jsep.wasm
    ├── ort-wasm-simd-threaded.mjs
    └── ort-wasm-simd-threaded.wasm
```

The ONNX Runtime Web assets are from **version 1.22.0**. Keep JavaScript and WebAssembly files from the **same version**; the application may need additional runtime assets on some browsers. The logos are optional for inference and must be used with appropriate permission.

## Run locally on Windows

### Requirements

- Python installed and available as `python` in PowerShell.
- A modern browser such as Google Chrome.
- All files in the folder structure above. **No PyTorch installation is needed to run the exported ONNX models in the browser.**

Open PowerShell in the folder containing `index.html`:

```powershell
cd "E:\8. Implementation\3. Anemia Prediction\Python\Anemia_Offline_App"
python -m http.server 8765 --bind 127.0.0.1
```

Manually enter this URL into Chrome:

**http://127.0.0.1:8765/**

Keep the PowerShell terminal open while using the application. Stop the server with **Ctrl+C**.

> **Do not double-click `index.html` to run inference.** Browser `file://` restrictions can prevent the application from loading model and WASM assets. The local Python HTTP server serves files from your own computer; it is not a cloud-inference service.

### First-use runtime installation (only if `runtime/` files are missing)

If your local application still has `setup_runtime.ps1`, run it in PowerShell with internet access to install the ONNX Runtime Web files. Ensure all required `.mjs` and `.wasm` files are copied into `runtime/`. Once the assets are installed locally, the app can be used without internet connectivity.

## How to use

1. Enter a **pseudonymous study ID** and age, if available.
2. Enter available CBC measurements: RBC/TRBC, PCV/HCT, MCV, MCH, MCHC, RDW, reticulocyte count, and reticulocyte percentage. At least one laboratory measurement beyond age is required.
3. Upload 1–7 photos in their **correct anatomical fields**. The form displays left-side images first, then right-side images, then tongue.
4. Click **Run offline AI screening**.
5. Review the binary prediction, model score and threshold, severity scores, and seven-view attention weights.

**Important:** Although the upload boxes are displayed left-first, the models expect inputs internally in this exact order:

```text
Left_Eye, Right_Eye, Left_Nail, Right_Nail, Left_Palm, Right_Palm, Tongue
```

Do not change that internal order or the model configuration's `views` array when modifying the UI.

## Model inputs and preprocessing

- Seven RGB image positions, preprocessed to **224 × 224** pixels, with ImageNet mean and standard-deviation normalization.
- View presence mask for missing photographs.
- Nine tabular inputs: **age plus eight CBC features**.
- Each model has its own JSON configuration containing feature order, median imputation values, normalization means/scales, image preprocessing constants, and the binary decision threshold.
- For uploaded images, the application uses browser Canvas resizing. **The browser preprocessing pipeline has not been confirmed to be pixel-identical to the original Python/PIL pipeline.**

### Model routing and decision thresholds

| Images supplied | Model | Binary score threshold |
|---|---|---:|
| 1–6 | `missing_aware.onnx` | 0.583593 |
| 7 | `complete_case.onnx` | 0.454122 |
| 0 | No inference | — |

Model probabilities are **not established as calibrated clinical probabilities**.

## GitHub Pages deployment

A GitHub repository stores the source and assets; **GitHub Pages deployment is a separate step**.

For a project-site URL such as:

```text
https://YOUR-USERNAME.github.io/Pregnancy-Anemia-Screening-AI/
```

check the `ort.env.wasm.wasmPaths` setting inside `index.html`. The version used locally may contain:

```javascript
ort.env.wasm.wasmPaths = '/runtime/';
```

For a GitHub Pages project site, change it to:

```javascript
ort.env.wasm.wasmPaths = '/Pregnancy-Anemia-Screening-AI/runtime/';
```

The correct location depends on your actual repository name and deployment URL. Verify that the JavaScript, model JSON files, model ONNX files, `.mjs` modules, and `.wasm` binaries all load successfully. A root-relative `/runtime/` path that works on a localhost site generally **will not** work under a GitHub Pages project subpath.

**Privacy and publication warning:** GitHub Pages is publicly accessible; uploaded `.onnx` files and other website assets can be downloaded by visitors. Use a private repository until you have permission to release the trained models and institutional logos. Do not upload identifiable patient photographs, laboratory reports, or other sensitive research data.

## Research validation and limitations

- The two ONNX checkpoints passed **PyTorch–ONNX numerical parity tests on the inputs used during model export**. This does **not** prove full equivalence on real clinical cases.
- Browser inference has been demonstrated locally, but **browser–Python parity with identical real patient photos and CBC values remains to be validated**.
- Performance on independent clinical cohorts, calibration, failure cases, and low-memory Android devices require further investigation.
- The binary and severity prediction heads can disagree.
- Image-attention scores are not evidence of causal feature contribution.
- The optional measured-Hb display is for study reference and is not a substitute for clinical interpretation; anemia cutoffs in pregnancy can depend on gestational trimester and applicable guidance.
- **Do not use this application alone to diagnose, exclude, or treat anemia.** Confirm findings with appropriate laboratory testing and clinical assessment.

## Technologies

HTML5 · CSS3 · JavaScript · ONNX Runtime Web 1.22.0 · WebAssembly · Python `http.server` (local file serving only)

## Acknowledgments and branding

The interface can display logos for **ICMR**, **RIMS Ranchi**, and **BIT Mesra** when image files are present in `logos/`. Their appearance is for project identification only and **does not independently establish institutional endorsement or clinical approval**. Use names and logos in accordance with institutional permissions and project arrangements.

## License

**No open-source license is granted by this README.** Before distributing the code, model weights, third-party runtime assets, or institution-branded materials, confirm the applicable ownership, permissions, and third-party license obligations. ONNX Runtime Web has its own upstream license and notices.
