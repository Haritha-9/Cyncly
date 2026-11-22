# Exact commands to run BOTH case-study projects

> Use Python **3.11+**, terminal in the folder containing this file. The default `marker` detector works **offline** on the provided synthetic example images. It cannot identify objects in ordinary photographs.

## Option A: macOS/Linux — one command

```bash
bash scripts/run_local.sh
```

This creates `.venv`, installs dependencies, regenerates fixtures, runs tests and starts both APIs. In another terminal:

```bash
.venv/bin/python scripts/smoke_test.py
```

## Option B: Windows PowerShell — one command

```powershell
powershell -ExecutionPolicy Bypass -File .\scripts\run_windows.ps1
```

In a second PowerShell terminal:

```powershell
.\.venv\Scripts\python.exe scripts\smoke_test.py
```

## Option C: run step by step (macOS/Linux)

```bash
python3 -m venv .venv
source .venv/bin/activate
python -m pip install -r requirements.txt
python scripts/generate_samples.py
python -m pytest -q
# Terminal 1:
uvicorn case1.main:app --reload --host 127.0.0.1 --port 8001
# Terminal 2 (activate the same environment):
uvicorn case2.main:app --reload --host 127.0.0.1 --port 8002
# Terminal 3 (activate the same environment):
python scripts/smoke_test.py
```

## Option D: run step by step (Windows PowerShell)

```powershell
py -3 -m venv .venv
.\.venv\Scripts\python.exe -m pip install -r requirements.txt
.\.venv\Scripts\python.exe scripts\generate_samples.py
.\.venv\Scripts\python.exe -m pytest -q
# Terminal 1:
.\.venv\Scripts\python.exe -m uvicorn case1.main:app --reload --host 127.0.0.1 --port 8001
# Terminal 2:
.\.venv\Scripts\python.exe -m uvicorn case2.main:app --reload --host 127.0.0.1 --port 8002
# Terminal 3:
.\.venv\Scripts\python.exe scripts\smoke_test.py
```

## Option E: Docker (if Docker Engine/Compose is installed)

```bash
docker compose up --build
# New terminal:
python scripts/smoke_test.py
# Stop:
docker compose down
```

*For `docker compose`, local Python is still required for `scripts/smoke_test.py`, unless you use the browser and Swagger API docs instead.*

## Open in your browser

- **Case 2 interactive chatbot:** http://127.0.0.1:8002/
- **Case 1 API documentation:** http://127.0.0.1:8001/docs
- **Case 2 API documentation:** http://127.0.0.1:8002/docs
- **Case 1 metrics:** http://127.0.0.1:8001/metrics

## Demo instructions

1. Visit the Case 2 chatbot URL; select **CV + chatbot** and click **Start new session**.
2. Click the **Room** sample image and see categories returned from Case 1.
3. Send `why confidence`, `choose 1`, `correct chair` or `human`.
4. Record a real or clearly hypothetical pilot observation in the form. Do not treat sample observations as evidence.
5. Click **Refresh metrics** and **Go / No-Go**.
6. Open the Case 1 Swagger API; call `/v1/analytics/tenant` for both `alpha` and `beta` to show separate monitoring.

## Test Case 1 from a terminal (macOS/Linux; curl)

```bash
curl -X POST 'http://127.0.0.1:8001/v1/predict' \
  -H 'X-Tenant-ID: alpha' -H 'X-API-Key: alpha-demo-key' \
  -F 'file=@data/samples/room.png'
```

## Canary and rollback (developer demo — no model retraining)

```bash
curl -X POST 'http://127.0.0.1:8001/v1/admin/models/rollout' \
  -H 'X-Admin-Key: local-admin-demo-key' -H 'Content-Type: application/json' \
  -d '{"active":"marker-v1","canary":"marker-v2","canary_percent":100}'

# Upload the same image; model_version should now be marker-v2.

curl -X POST 'http://127.0.0.1:8001/v1/admin/models/rollback' \
  -H 'X-Admin-Key: local-admin-demo-key'
```

Both marker versions are **synthetic**, but the rollout and rollback mechanisms are exercised. This is not a production deployment controller.

## Real photo detection with YOLO (OPTIONAL; requires internet at first use)

```bash
python -m pip install -r requirements-ai.txt
curl -X POST 'http://127.0.0.1:8001/v1/admin/models/rollout' \
  -H 'X-Admin-Key: local-admin-demo-key' -H 'Content-Type: application/json' \
  -d '{"active":"yolo11n","canary":null,"canary_percent":0}'
# Upload a real photo through http://127.0.0.1:8001/docs
```

- YOLO weights (`yolo11n.pt`) download on first use unless locally cached; use an appropriate network and license. The COCO-pretrained model recognizes **generic classes**, not Cyncly-specific products or exact SKUs.
- Optional Faster R-CNN: install compatible `torch`/`torchvision` (see `requirements-rcnn.txt`), then set the active model to `fasterrcnn` using the same admin endpoint. The pretrained weights can download on first use.
- On Windows use PowerShell `Invoke-RestMethod`, `curl.exe` or the Swagger `/docs` pages for the admin commands.

## Data / model evaluation commands (local)

```bash
python scripts/split_manifest.py --input data/annotations.csv --output data/splits.csv
python scripts/score_detections.py --truth data/eval_truth.json --pred data/eval_pred.json
python scripts/benchmark_models.py --models marker-v1 --images data/samples
python -m pytest -q
```

Remember: these demo inputs are synthetic; a meaningful experiment needs a real, reviewed, frozen multi-customer evaluation dataset.
