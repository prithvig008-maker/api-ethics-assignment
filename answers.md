# API Ethics Assignment

## Task 1 — Classify and Handle PII Fields

| Field | Classification | Action | Justification |
|---|---|---|---|
| `full_name` | Direct PII | Drop or Pseudonymize | Directly identifies an individual. Replace with a random token/ID if linkage is needed across records. |
| `email` | Direct PII | Drop | Uniquely identifies a person and enables contact. No research value; drop entirely. |
| `date_of_birth` | Direct PII | Mask (generalize) | Exact DOB can identify individuals, especially combined with other fields. Replace with birth year or age range. |
| `zip_code` | Indirect PII | Mask (generalize) | Alone it's weak, but combined with DOB/diagnosis it can re-identify patients. Truncate to first 3 digits or region. |
| `job_title` | Indirect PII | Mask or Drop | Can narrow identity in combination with other fields. Generalize to broad category or drop. |
| `diagnosis_notes` | Indirect PII (Sensitive) | Pseudonymize + Redact | Contains sensitive health data. Strip any names/identifiers mentioned within the text; pseudonymize the record it belongs to. |

**Key principle:** Even indirect PII fields become high-risk when combined — this is the *aggregation problem*. The research partner should receive the minimum fields necessary (data minimization).

---

## Task 2 — Audit the API Script for Ethical Compliance

### Violation 1 — Unlimited Bulk Scraping Without Rate Limiting or Consent Verification

**Problem:** The script loops through 100 pages unconditionally with no delays, no check on whether bulk extraction is permitted, and no verification that the API's Terms of Service allow mass data collection. Most public health APIs explicitly prohibit automated bulk harvesting of patient records, and hammering the endpoint without rate limiting is both an abuse of the service and potentially illegal under laws like HIPAA or GDPR.

**Corrected Code:**
```python
import requests
import time

API_URL = "https://healthstats-api.example.com/records"
API_KEY = "free_tier_key_abc123"
MAX_PAGES = 10          # Respect any page/record limits stated in the TOS
RATE_LIMIT_DELAY = 1.0  # Seconds between requests — check TOS for required rate

records = []
for page in range(1, MAX_PAGES + 1):
    response = requests.get(API_URL, params={"page": page, "key": API_KEY})

    if response.status_code == 429:  # Too Many Requests
        print("Rate limit hit — pausing before retrying.")
        time.sleep(10)
        continue

    response.raise_for_status()
    data = response.json()
    records.extend(data["results"])

    time.sleep(RATE_LIMIT_DELAY)  # Be a polite API consumer
```

### Violation 2 — Permanent Storage of Raw PII Without Anonymization

**Problem:** The line `save_to_database(records)` stores the full raw API response — including direct PII like names, emails, and dates of birth — permanently in the company database. This violates data minimization and storage limitation principles under GDPR and HIPAA. Raw patient PII must be anonymized or pseudonymized before storage.

**Corrected Code:**
```python
import hashlib

def pseudonymize(record):
    cleaned = record.copy()

    # Drop fields with no research value
    cleaned.pop("email", None)

    # Pseudonymize the name with a one-way hash token
    if "full_name" in cleaned:
        token = hashlib.sha256(cleaned["full_name"].encode()).hexdigest()[:12]
        cleaned["patient_token"] = token
        del cleaned["full_name"]

    # Generalize date_of_birth to birth year only
    if "date_of_birth" in cleaned:
        cleaned["birth_year"] = cleaned["date_of_birth"].split("-")[0]
        del cleaned["date_of_birth"]

    # Truncate zip_code to first 3 digits
    if "zip_code" in cleaned:
        cleaned["zip_code"] = cleaned["zip_code"][:3] + "XX"

    return cleaned

# Apply pseudonymization before any storage
cleaned_records = [pseudonymize(r) for r in records]
save_to_database(cleaned_records)
```
