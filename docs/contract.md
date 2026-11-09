# Form Guidance API Contract

**Status:** Draft for instructor review  
**Tools:** Python, FastAPI, and pytest

## 1. Purpose

Our component stores labels, examples, and help text for a sample NBI Clearance form. It explains what to enter and whose details are needed. Connected forms can get the same guidance through the API, and we can update it in one place.

This follows our Chapter 5 recommendation. It is a proposed prototype, not the actual NBI system. It does not store applicants' answers or process clearance applications. The sample wording is for classroom use, not official NBI guidance.

## 2. Data and basic rules

Each field has these details:

| Property | Meaning |
|---|---|
| `field_id` | Field ID, such as `birthplace` |
| `label` | Name shown on the form |
| `example` | Sample entry |
| `help_text` | Instructions for the field |
| `information_owner` | Whose details to enter |

Requests and responses use JSON. GET requests do not need a body. PUT requests use `Content-Type: application/json`.

| Input | Rule |
|---|---|
| `form_id` and `field_id` | 1–64 characters; lowercase letters, numbers, hyphens, or underscores |
| `label` | Required text, 1–100 characters |
| `example` | Required text, 1–200 characters |
| `help_text` | Required text, 1–500 characters |
| `information_owner` | Required text, 1–100 characters |

Remove spaces at the start and end of guidance text before checking its length. Reject missing, blank, null, or non-text values. The update body must contain only the four guidance properties; IDs stay in the URL.

## 3. Get all fields

**Endpoint:** `GET /forms/{form_id}/fields`

Returns all field definitions for the selected form.

**Example request:** `GET /forms/nbi-clearance/fields`

**Response: 200 OK**

```json
{
  "form_id": "nbi-clearance",
  "fields": [
    {
      "field_id": "birthplace",
      "label": "Place of birth",
      "example": "Brooke's Point, Palawan",
      "help_text": "Enter the city or municipality and province where you were born.",
      "information_owner": "Applicant"
    }
  ]
}
```

If the form exists but has no fields, return `200` with an empty `fields` list. Return `404` if the form does not exist, or `422` if the ID is invalid.

## 4. Get one field

**Endpoint:** `GET /forms/{form_id}/fields/{field_id}`

Returns the guidance for one field.

**Example request:** `GET /forms/nbi-clearance/fields/birthplace`

**Response: 200 OK**

```json
{
  "field_id": "birthplace",
  "label": "Place of birth",
  "example": "Brooke's Point, Palawan",
  "help_text": "Enter the city or municipality and province where you were born.",
  "information_owner": "Applicant"
}
```

Return `404` if the form or field does not exist, or `422` if an ID is invalid.

## 5. Update field guidance

**Endpoint:** `PUT /forms/{form_id}/fields/{field_id}/guidance`

Updates the guidance for an existing field. Send all four guidance properties, even if only one changes.

**Example request:** `PUT /forms/nbi-clearance/fields/birthplace/guidance`

**Request body:**

```json
{
  "label": "Place of birth",
  "example": "Brooke's Point, Palawan",
  "help_text": "Enter your birthplace, not your current address.",
  "information_owner": "Applicant"
}
```

**Response: 200 OK**

```json
{
  "field_id": "birthplace",
  "label": "Place of birth",
  "example": "Brooke's Point, Palawan",
  "help_text": "Enter your birthplace, not your current address.",
  "information_owner": "Applicant"
}
```

Keep the field ID and other fields unchanged. Later GET requests return the updated guidance. If the update fails, leave the stored data unchanged.

Return `404` if the form or field does not exist, or `422` if the request breaks the validation rules. This operation does not create new forms or fields.

## 6. Error responses

All operations use the following error formats. Check inputs first, then look for the form, then the field.

**404 — Form not found**

```json
{
  "error": "FORM_NOT_FOUND",
  "message": "The requested form does not exist."
}
```

**404 — Field not found**

```json
{
  "error": "FIELD_NOT_FOUND",
  "message": "The requested field does not exist."
}
```

**422 — Invalid input**

```json
{
  "error": "VALIDATION_ERROR",
  "message": "help_text must not be blank."
}
```

For `422`, the message explains the input problem and may change depending on the error. Use this same format for invalid IDs, missing values, and broken JSON. FastAPI validation errors must also use this format.

## 7. Sample data and approval

We will use mock data stored in memory, including the `nbi-clearance` form and its `birthplace` field. No real applicant data is needed. Changes last while the app is running; restarting it restores the original sample data. The demonstration will run locally in one server process without login features.

Submit this contract for instructor review before implementation. Once approved, commit the approved version and tag it `contract-v1`.

**Approved by:** To be approve  
**Approval date:** To be approve

## Acknowledgment

We used ChatGPT/Codex to help design a contract.
