# Form Definition Component — API Contract

Status: Draft — for instructor review

Tools: Python, FastAPI, and pytest

## 1. Purpose and scope

This component stores the labels, examples, and instructions shown on our sample NBI Clearance form. It also explains whose information to enter, especially for parent details. The form gets this guidance through the API, and we can update it in one place.

In Chapter 5, our group chose to improve the shared form definition. This API is our prototype of that idea; we have not confirmed how the real NBI system stores its form guidance. It does not save applicants’ answers, access government records, process applications, or handle payments. It can explain parent fields, but it cannot provide details an applicant does not know.

## 2. Basic rules

- The paths below use the address of our local API server.
- Responses use `Content-Type: application/json`.
- PUT requests must send a JSON object with `Content-Type: application/json`.
- Form and field IDs must follow the format below. The API does not change their capitalization or remove spaces from them.
- Remove spaces at the start and end of guidance text before checking its length and saving it. Keep spaces inside the text.
- Check the request first. If it is valid, look for the form, then the field.
- We will run this prototype locally without login features. The update operation is for maintaining our sample guidance.

## 3. Field definition

| Property | Type | Meaning |
|---|---|---|
| `field_id` | string | Unique ID for a field in a form |
| `label` | string | Readable field name |
| `example` | string | Sample entry |
| `help_text` | string | Instructions explaining what to enter |
| `information_owner` | string | Whose details to enter |

Example:

```json
{
  "field_id": "birthplace",
  "label": "Place of birth",
  "example": "Brooke's Point, Palawan",
  "help_text": "Enter the city or municipality and province where you were born.",
  "information_owner": "Applicant"
}
```

The sample instructions are for our prototype and still need review. They are not official NBI instructions. The validation rules below apply to the guidance we store, not to applicants’ answers.

## 4. Validation rules

| Input | Rule |
|---|---|
| `form_id`, `field_id` | String of 1–64 characters matching `^[a-z0-9_-]{1,64}$` |
| `label` | Required string; 1–100 characters after trimming |
| `example` | Required string; 1–200 characters after trimming |
| `help_text` | Required string; 1–500 characters after trimming |
| `information_owner` | Required string; 1–100 characters after trimming |
| PUT body | JSON object containing exactly the four guidance properties above |

Reject missing, null, blank, or overly long values. Guidance must be text; do not convert numbers or true/false values into text. Reject extra properties, including `field_id` and `form_id` in the PUT body.

## 5. Get all fields

`GET /forms/{form_id}/fields`

This returns all field definitions for a form.

Input: `form_id` in the URL, following the ID rules in Section 4.

Request body: None.

Example request:

```http
GET /forms/nbi-clearance/fields
Accept: application/json
```

Success: `200 OK`

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

Sort the results by `field_id` in ascending order. If the form exists but has no fields, return `200 OK` with an empty list:

```json
{
  "form_id": "empty-demo",
  "fields": []
}
```

Errors: `404 FORM_NOT_FOUND`; `422 VALIDATION_ERROR` for an invalid identifier.

## 6. Get one field

`GET /forms/{form_id}/fields/{field_id}`

This returns the guidance for one field in the selected form.

Inputs: `form_id` and `field_id` in the URL, following the ID rules in Section 4.

Request body: None.

Example request:

```http
GET /forms/nbi-clearance/fields/birthplace
Accept: application/json
```

Success: `200 OK`

```json
{
  "field_id": "birthplace",
  "label": "Place of birth",
  "example": "Brooke's Point, Palawan",
  "help_text": "Enter the city or municipality and province where you were born.",
  "information_owner": "Applicant"
}
```

Errors: `404 FORM_NOT_FOUND`; `404 FIELD_NOT_FOUND`; `422 VALIDATION_ERROR` for invalid identifiers.

## 7. Update field guidance

`PUT /forms/{form_id}/fields/{field_id}/guidance`

This replaces the label, example, help text, and information-owner note for an existing field. It cannot add a new form or field.

Inputs: `form_id` and `field_id` in the URL, following the ID rules in Section 4.

Example request:

```http
PUT /forms/nbi-clearance/fields/birthplace/guidance
Content-Type: application/json
Accept: application/json
```

```json
{
  "label": "Place of birth",
  "example": "Brooke's Point, Palawan",
  "help_text": "Enter your birthplace, not your current address.",
  "information_owner": "Applicant"
}
```

Success: `200 OK`

```json
{
  "field_id": "birthplace",
  "label": "Place of birth",
  "example": "Brooke's Point, Palawan",
  "help_text": "Enter your birthplace, not your current address.",
  "information_owner": "Applicant"
}
```

Update rules:

- Send all four guidance properties, even if only one has changed.
- Return the updated definition after removing spaces at the start and end of each value.
- Keep the same field ID and form.
- Leave all other field definitions unchanged.
- Save only when all checks pass. If a request fails, leave the stored data unchanged.
- Later GET requests must return the updated guidance.
- Sending the same valid update again must leave the same result.

Errors: `404 FORM_NOT_FOUND`; `404 FIELD_NOT_FOUND`; `422 VALIDATION_ERROR`; `415 UNSUPPORTED_MEDIA_TYPE` if Content-Type is missing or is not application/json. An optional charset parameter is permitted.

## 8. Error responses

All three operations use the error formats below.

### Unknown form — 404 Not Found

```json
{
  "error": "FORM_NOT_FOUND",
  "message": "The requested form does not exist."
}
```

### Unknown field in an existing form — 404 Not Found

```json
{
  "error": "FIELD_NOT_FOUND",
  "message": "The requested field does not exist."
}
```

### Invalid input — 422 Unprocessable Entity

```json
{
  "error": "VALIDATION_ERROR",
  "message": "The request contains invalid input.",
  "details": [
    {
      "field": "help_text",
      "message": "Must not be blank."
    }
  ]
}
```

Include at least one item in `details`, with a field name and a message explaining the problem. Use `body` if the JSON is broken, missing, or not an object. The items can appear in any order. Their messages may vary, but keep the main `error` and `message` exactly as shown.

Invalid JSON sent as application/json also returns this 422 format. Use this format for FastAPI validation errors too.

### Unsupported request content type — 415 Unsupported Media Type

```json
{
  "error": "UNSUPPORTED_MEDIA_TYPE",
  "message": "Send the request body as application/json."
}
```

For PUT requests, check Content-Type first, then the input values, then whether the form and field exist. This contract covers the three routes above; it does not define errors for other routes or HTTP methods.

## 9. Sample data

We will use sample field definitions stored in memory, with no real applicant data. Changes stay available while the app is running. Restarting it brings back the original sample data. Run one server process for the demonstration; this version does not save changes permanently or share them between processes.

Include `nbi-clearance` and its `birthplace` field in the sample data. Tests can use a fake repository with other forms and fields. Keep track of forms separately from their fields so an empty form is not mistaken for a missing one.

## 10. Code structure and tests

Controllers receive requests and send responses. The service checks the rules, while the repository reads and updates the data. The service uses a repository interface so we can replace storage with a fake or mock when testing. Unit tests must run without a database.

We plan to test these service behaviors:

1. Retrieve all definitions for an existing form.
2. Return fields in the documented order.
3. Return an empty list for an existing form without fields.
4. Report an unknown form.
5. Retrieve one existing field.
6. Report an unknown field within an existing form.
7. Update all four guidance properties successfully.
8. Trim surrounding whitespace in guidance.
9. Reject blank guidance without changing stored data.
10. Reject guidance exceeding the allowed length.
11. Preserve unrelated fields during an update.
12. Return updated guidance on a later read.

Also check the API responses: status codes, JSON format, and invalid requests. These checks are separate from the required service-layer unit tests.

## 11. Instructor review

- [ ] Component scope and field definitions reviewed.
- [ ] Three operations, examples, errors, and validation rules reviewed.
- [ ] Mock-data limitations accepted.
- [ ] Instructor approval received before implementation.
- [ ] Approved contract committed and tagged `contract-v1`.

Approval date: Pending

Approved by: Pending

After approval, commit the approved file and create/push the tag:

```bash
git tag contract-v1
git push origin contract-v1
```

Implementation starts after the instructor approves the contract.

## 12. Acknowledgment

We used ChatGPT/Codex to help draft this contract from our Chapter 5 recommendation and the Chapter 6 instructions. Our group will review it and make the changes requested by our instructor.
