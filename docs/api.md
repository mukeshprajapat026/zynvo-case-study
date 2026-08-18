# Zynvo API Documentation

## Base URL

`https://zynvo-api.syslotech.com`

## API Version

`/v1`

---

## Authentication

All protected user APIs require:

```text
Authorization: Bearer {firebase_id_token}


1. Health
GET /health
Example:
GET https://zynvo-api.syslotech.com/health
Response:
{
    "status": true,
    "message": "Zynvo API is healthy.",
    "data": {
        "service": "Zynvo API",
        "database": "connected"
    }
}

