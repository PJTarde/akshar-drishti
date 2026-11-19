# Akshar Drishti (अक्षर दृष्टी): layout-aware OCR for Indian government documents

Akshar Drishti reads scanned or photographed **Marathi, Hindi and English** government documents and turns each page into a verified, structured and privacy-safe record. For every page it:

1. **Reads** the text, layout, reading order and tables (IndicOCR).
2. **Verifies** the officer's **signature** (YOLOv8) and the office **stamp** (colour detector), and gives a verdict: COMPLETE or INCOMPLETE.
3. **Extracts key fields**: Aadhaar, PAN, phone, PIN code, date, amount, survey / gat number and email, including values written in Devanagari digits (०–९).
4. **Masks Aadhaar and PAN** numbers in the text, the fields and the page image.
5. **Stores** everything in a searchable SQLite archive and exports JSON, Markdown, Excel and an HTML report.

A web application shows all of this in the browser: upload a page, see the verdict, annotated page, key fields, stamp and signature crops and downloads, and search the archive.

![Masking before and after](test_pages/sample_aadhaar_pan_certificate_RESULT_FIXED_annotated.png)

*Sample output on a fictional test certificate: text regions (blue), table (green), signature (red), stamp (violet), Aadhaar and PAN numbers blacked out.*

##  How it works

```
page image ─► normalise (RGB, ≤ 2000 px)
           ─► IndicDocLayout: regions + reading order   ┐ IndicOCR
           ─► IndicBlockOCR: read each region            ┘ (pretrained)
           ─► YOLOv8s signatures + HSV colour stamps
           ─► merge (containment rules) + crops
           ─► key fields (regex, Devanagari digits)
           ─► Aadhaar / PAN masking (text + image)
           ─► verdict ─► annotated image ─► JSON / MD / Excel / HTML ─► SQLite archive
```

A detailed, illustrated walk-through of every step and file is in [`docs/how-it-works.html`](docs/how-it-works.html) (download and open it in a browser).

## Results on the 30-page synthetic benchmark

| Measure | Result |
| --- | --- |
| Key-field extraction | 147 / 147 (F1 = 1.00) |
| Aadhaar + PAN masked in text | 33 / 33 |
| Coloured stamps found (recall) | 0.72 |
| Signature F1 at IoU 0.5 / 0.3 | 0.44 / 0.67 |
| OCR word error rate (best of 3 systems) | 0.056 (PaddleOCR 0.089, Tesseract 0.162) |
| Time per page on a T4 GPU (fp16) | 6.7 s |

The benchmark pages are generated with exact ground truth (seed 2026), so no real personal data is used. Known limits: black-ink stamps are not detected by the colour method, and the verdict is of a screening aid that a person confirms.

After the evaluation, the web app (`webapp/akshar_web.py`) added an improved image-masking estimate, month-name dates and landline numbers in the field rules, duplicate removal and a context label for each field. These additions were checked on test pages but have not been re-measured on the benchmark.

## Run it (Google Colab, free T4 GPU)

1. Open `notebooks/indic_ocr_colab.ipynb` in Google Colab.
2. **Runtime → Change runtime type → T4 GPU.** Add a Colab secret `HF_TOKEN` (a Hugging Face token) and accept the IndicOCR model terms on Hugging Face.
3. Run cell **1** (should print *Tesla T4*), then cell **2**, then **Runtime → Restart session**.
4. Run cells **3, 4, 4b, S1, S2, A2, A3, A4**, then **W1**. When W1 asks, upload `webapp/akshar_web.py` and `webapp/akshar_web.html`.
5. Open the link W1 prints. Try the pages in `test_pages/`.

If Colab gives a CPU-only machine (error *"Torch not compiled with CUDA enabled"*), use **Runtime → Disconnect and delete runtime** and start again from cell 1.

## Repository layout

| Path | What it is |
| --- | --- |
| `notebooks/indic_ocr_colab.ipynb` | The full system: OCR, marks, fields, masking, archive, Gradio app (A5) and the website launcher (W1) |
| `notebooks/akshardrishti_synthetic_eval.ipynb` | Builds the 30-page synthetic benchmark and scores every component |
| `notebooks/akshardrishti_speed_benchmark.ipynb` | Speed study on the T4 (bf16 vs fp16, batching) |
| `webapp/akshar_web.py`, `webapp/akshar_web.html` | Standalone website: FastAPI routes on the Gradio share server, plus the page itself |
| `docs/how-it-works.html` | Step-by-step explanation of the whole project |
| `test_pages/` | Fictional test certificates with fake Aadhaar / PAN numbers, and annotated results |

## Credits

- **IndicOCR** (IndicDocLayout + IndicBlockOCR) by Bodhan AI and AI4Bharat, used under the Indic Open Model License. It is downloaded from Hugging Face by the notebook and is not included in this repository.
- **YOLOv8s signature detector** by Tech4Humans (Hugging Face), with Ultralytics.
- OpenCV, pandas, Gradio, FastAPI and SQLite.

## Team

Final-year project, Department of Computer Engineering, K. K. Wagh Institute of Engineering Education & Research, 2026–27. Guide: Prof. K. P. Birla.

| Member | Role |
| --- | --- |
| Prasad Tarde | Team lead, ML and integration |
| Kalpesh Suryawanshi | Detection and vision |
| Ayush Shirsath | Data and evaluation |
| Nikhil Pawar | Application and testing |

All names, numbers and office in `test_pages/` are fictional and made for software testing.
