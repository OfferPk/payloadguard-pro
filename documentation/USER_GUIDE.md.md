# PayloadGuard Pro — User & Developer Guide

## 1\. Quickstart: Web Studio

1. Open `source-code/index.html` in any web browser. No server or internet connection required.  
2. Select a preset from the **Preset Schema** dropdown (e.g. *E-Commerce Multi-Item Order* or *User Registration*), or paste your own JSON Schema and JSON payload.  
3. The studio immediately updates:  
   - **Validation Status Banner**: Confirms whether the sample payload strictly conforms to the schema or highlights specific constraint errors with clickable JSON paths.  
   - **Synthesized Code Tab**: Switch between **Zod v3**, **Pydantic v2**, **TypeScript**, and **Golang** with 1-click clipboard copy.  
   - **Fuzz-Test Matrix Tab**: Review synthesized negative, edge-case, and security payloads.  
   - **Automated Unit Tests Tab**: Copy ready-to-run **Jest/Vitest** or **PyTest** test suites.

---

## 2\. Using the Synthesized Schemas in Your Codebase

### TypeScript / Next.js (Zod v3)

import { OrderCheckoutPayloadSchema } from './schemas';

export async function POST(req: Request) {

  const json \= await req.json();

  const parseResult \= OrderCheckoutPayloadSchema.safeParse(json);

  if (\!parseResult.success) {

    return new Response(JSON.stringify(parseResult.error.format()), { status: 422 });

  }

  const order \= parseResult.data; // Fully typed

}

### Python / FastAPI (Pydantic v2)

from fastapi import FastAPI, HTTPException

from models import OrderCheckoutPayload

app \= FastAPI()

@app.post("/checkout")

def create\_order(payload: OrderCheckoutPayload):

    \# Pydantic v2 validates types, constraints, and extra fields automatically

    return {"status": "success", "order\_id": payload.order\_id}

---

## 3\. Standalone Python CLI Tool (`payloadguard-cli.py`)

The CLI utility runs on Python 3.8+ with zero external package dependencies.

### Validate Payloads Offline

\# Validate sample payload against schema

python3 build/payloadguard-cli.py validate schema.json payload.json

\# Machine-readable JSON output

python3 build/payloadguard-cli.py validate schema.json payload.json \--json

### Pre-Commit CI/CD Security Gating

To fail CI/CD build pipelines if mock fixtures or contract tests violate schema constraints:

python3 build/payloadguard-cli.py validate schema.json payload.json \--fail-on-invalid

\# Exits with status code 1 if validation fails

### Synthesize Fuzz-Test Matrices

\# Generate fuzz vectors

python3 build/payloadguard-cli.py fuzz schema.json \--sample payload.json \--json