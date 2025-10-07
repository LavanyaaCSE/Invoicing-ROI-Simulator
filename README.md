# Invoicing ROI Simulator — Implementation Plan & Architecture


## 1. Purpose

A focused 2-hour prototype that demonstrates the ROI (savings, payback months, cumulative savings, ROI%) when switching from manual to automated invoicing. The prototype includes a single-page interactive frontend, a lightweight REST API backend, a MongoDB Atlas database for persistence, and a gated report generator (email capture required before download).

## 2. High-level approach

**Time Duration:** 2 hours. Priorities:

1. Implementing the simulation endpoint (`/simulate`) and ensure the calculation uses server-side internal constants and bias so automation is favorable.
2. Build a minimal React single-page UI (Vite) that calls `/simulate` and shows live results.
3. Implement scenario CRUD endpoints + persistence (MongoDB Atlas) and wire simple Save / Load / Delete in the UI.
4. Add an email-gated HTML/PDF report generator endpoint (`/report/generate`).
5. README with run and test instructions.
6. Deploy the Backend in Render and the Frontend in Vercel and Database MongoDB Atlas for the demo.


## 3. Chosen stack & rationale

* **Frontend:** React + Vite
* **Backend:** Flask (Python) + Flask-CORS + Flask-PyMongo
* **Database:** MongoDB Atlas
* **PDF/HTML report:** pdfkit (HTML -> PDF)
* **Email capture:** No real email sending; store the email with scenario/report metadata and return the downloadable file after validation.
* **Dev tools:** venv + npm for frontend and backend.

## 4. Architecture (components)

```
[React Vite] <----> [Flask REST API] <----> [MongoDB Atlas (collections: scenarios, reports)]
                           |
                     [Report Generator (HTML -> PDF)]
```

* Frontend calls `/simulate` (POST) to get computed results.
* Frontend calls `/scenarios` endpoints to save/load/delete scenario objects.
* Report endpoint `/report/generate` requires `{scenario_id, email}`. Server renders an HTML snapshot, stores `{scenario_id, email, timestamp}`, and returns a PDF file or HTML for download.

## 5. API Endpoints

* `POST /simulate`

  * Run simulation and return JSON results

* `POST /scenarios`

  * Save a scenario

* `GET /scenarios`

  * List all scenarios

* `GET /scenarios/:id`

  * Retrieve scenario details

* `DELETE /scenarios/:id`

  * Deletes the saved scenario

* `POST /report/generate`

  * Generate a PDF report (email required)

## 6. Server-side constants 

These must live only on the backend (not exposed in UI):

* `automated_cost_per_invoice = 0.20`
* `error_rate_auto = 0.001`  # 0.1% stored as fraction
* `time_saved_per_invoice_minutes = 8`
* `min_roi_boost_factor = 1.1`

Stored in `.env` or directly in code constants; never exposed to clients.

## 7. Calculation steps (as implemented server-side)

Follow the PRD formulas:

* Convert percentage inputs to fractional form.
* Force results to remain positive with `min_roi_boost_factor`.
* Guarantee ROI > 0 and positive savings for all valid inputs.

---

## 8. MongoDB collections

**Collection: scenarios**

```json
{
  "_id": ObjectId,
  "scenario_name": "Q4_Pilot",
  "inputs": {...},
  "results": {...},
  "created_at": ISODate,
  "updated_at": ISODate
}
```

**Collection: reports**

```json
{
  "_id": ObjectId,
  "scenario_id": ObjectId,
  "email": "user@example.com",
  "filename": "report_Q4_Pilot.pdf",
  "created_at": ISODate
}
```

---

## 9. Frontend UI (single-page)

* Left panel: form inputs for all fields.
* Right panel: result cards showing key metrics.
* Save, Load, and Delete buttons linked to CRUD endpoints.
* Report generation modal for email input.

Validation: client-side for numeric fields; server-side double-check.

---

## 10. Bias & UX considerations

* Backend enforces ROI bias with `min_roi_boost_factor`.
* Frontend labels results as “Estimated” for credibility.

---

## 11. Security & privacy notes

* No authentication.
* Email is for gating only, stored securely in MongoDB.
* `.env` file stores MongoDB URI, DB name, and Flask secret.

---

## 12. README / Run instructions (quick)

### Backend

1. Create `.env` with:

   ```env
   mongodb+srv://lavanyaa7804-db:<db_password>@cluster0.9cfxopc.mongodb.net/?retryWrites=true&w=majority&appName=Cluster0
   DB_NAME=invoicing_roi
   ```
2. Run:

   ```bash
   pip install -r requirements.txt
   flask run
   ```

### Frontend

```bash
cd frontend
npm install
npm run dev
```

### Notes

* API runs at `http://localhost:5000`
* Frontend runs at `http://localhost:5173`

---

## 13. Deliverables

* Flask backend with REST endpoints & MongoDB integration.
* React + Vite frontend for form + results display.
* PDF report generation.
* README with setup guide.

---

## 14. First 15-minute deliverables

1. This documentation file (`INVOICING_ROI_SIMULATOR_PLAN.md`). ✅
2. Flask setup with `/simulate` endpoint and MongoDB connection.
3. Vite + React boilerplate with input form & fetch integration.

