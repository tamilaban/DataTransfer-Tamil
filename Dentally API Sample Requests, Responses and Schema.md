# Dentally API: Sample Requests, Responses and Schema

Oct 14, 2026 · @Tamil

## Conventions

This document covers all 86 REST endpoints in the [Dentally API](https://developer.dentally.co/) with a sample request and response for each. Single resources come back wrapped in their singular name, lists in the plural name plus a `meta` object, and request bodies must be wrapped in the singular name.

| Environment | Base URL |
| --- | --- |
| Production | `https://api.dentally.co` |
| Sandbox | `https://api.sandbox.dentally.co` |
| APAC practices | `https://api.apac.dentally.com` |
| Canadian practices | `https://api.ca.dentally.com` |

Every request needs these headers. The samples below leave them out to stay short.

```http
Authorization: Bearer ACCESS_TOKEN
User-Agent: Awesome-Dental-App v1
Content-Type: application/json   (writes only)
```

A missing `User-Agent` returns `403 Forbidden`. Paths are shown with the explicit `/v1` prefix; omitting it returns the latest version.

- **Pagination:** `page` and `per_page` (default 25, max 100; availability max 20). Avoid going past page 100.
- **Sorting:** `sort_by=<attribute>&sort_direction=asc|desc` on most lists.
- **Date filters:** use them on large lists such as appointments, and keep `before`/`after` windows under 3 months.
- **Metadata:** patients and appointments accept a `metadata` hash of up to 3 keys (key max 40 chars, value max 500).
- **Deletes** return `204 No Content`. Deleting a patient only sets `active` to false.
- **Timestamps** are ISO 8601 with a timezone offset.

**Rate limits:** 3,600 requests per hour per user, and 200 per hour for appointment availability. Going over returns `403`.

```json
{ "error": { "type": "invalid_access_error", "message": "Rate limit exceeded" } }
```

**Validation error (422):**

```json
{
  "error": {
    "type": "invalid_request_error",
    "message": "The patient could not be saved",
    "params": { "first_name": ["seems to be missing"] }
  }
}
```

**Access error (403):**

```json
{
  "error": {
    "type": "invalid_access_error",
    "message": "API access is not enabled for this user",
    "documentation_url": "https://developer.dentally.co"
  }
}
```

| Status | Meaning |
| --- | --- |
| 200 | OK |
| 204 | No content (deletes) |
| 400 | Bad request, often a missing parameter |
| 401 | Unauthorized, invalid token |
| 403 | Forbidden, no access or rate limited |
| 404 | Not found |
| 410 | Gone |
| 422 | Unprocessable entity, reason in `error.params` |
| 500 / 503 | Server error / maintenance |

**OAuth scopes:** `appointment` (read, create, update, delete), `financials` (read, create, delete), `patient` (read, create, update, delete = archive), `practice` (read, update), `treatments` (treatments, treatment plans and items), `user` (read, update). Request only what you need.

## Patients and related resources

Patients are the hub of the model. Their account, appointments, treatment plans, recalls, referrals, stats and PRSI eligibilities all hang off `patient_id`.

### Patients

#### Create a patient

Required: `title`, `first_name`, `last_name`, `date_of_birth`, `gender`, `ethnicity`, `address_line_1`, `postcode`, `payment_plan_id`. If you send `emergency_contact_name` or `emergency_contact_relationship`, you must also send `emergency_contact_phone`.

```http
POST /v1/patients

{
  "patient": {
    "title": "Mr", "first_name": "John", "last_name": "Smith",
    "date_of_birth": "1980-01-01", "gender": true, "ethnicity": "99",
    "address_line_1": "21 Oak Avenue", "postcode": "W1A 1AA", "payment_plan_id": 1
  }
}
```

```json
{
  "patient": {
    "id": 3, "account_id": 3, "active": true,
    "address_line_1": "21 Oak Avenue", "address_line_2": null, "county": null,
    "created_at": "2014-09-11T14:48:01.563+01:00", "date_of_birth": "1980-01-01",
    "dentist_id": null, "dentist_recall_date": null, "dentist_recall_interval": 0,
    "email_address": null, "ethnicity": "99", "first_name": "John", "gender": true,
    "home_phone": null, "home_phone_country": "GB",
    "hygienist_id": null, "hygienist_recall_date": null, "hygienist_recall_interval": 0,
    "image_url": "https://www.gravatar.com/avatar/fb830e28cd031264e371ab85563b37b0.jpg?&r=pg&d=identicon&s=190",
    "last_name": "Smith", "legacy_id": null, "medical_alert": false, "medical_alert_text": null,
    "metadata": {}, "middle_name": null, "mobile_phone": null, "mobile_phone_country": "GB",
    "nhs_number": null, "ni_number": null, "occupation": null, "payment_plan_id": 1,
    "postcode": "W1A 1AA", "preferred_name": null, "preferred_phone_number": null,
    "recall_method": "Letter", "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75",
    "title": "Mr", "town": null, "updated_at": "2014-09-11T14:48:01.563+01:00",
    "use_email": true, "use_sms": true, "work_phone": null, "work_phone_country": "GB",
    "emergency_contact_name": null, "emergency_contact_relationship": null,
    "emergency_contact_phone": null, "emergency_contact_phone_country": "GB",
    "emergency_contact_phone_normalized": null
  }
}
```

#### Get a patient

```http
GET /v1/patients/1
```

```json
{
  "patient": {
    "id": 1, "account_id": 1, "active": true,
    "acquisition_source_id": "abdcdeb5-ebe4-4629-a375-15e1506d3961",
    "address_line_1": "135 New Crescent", "address_line_2": "Trlr 882", "county": "Breconshire",
    "consent_to_share_accounts": false, "created_at": "2013-11-04T03:07:33.954+00:00",
    "custom_fields": [
      { "value": true, "field_id": "2c19fdf0-987c-4e5a-aa71-06ad8a58ee22", "position": 0 },
      { "value": false, "field_id": "38b96e14-e31d-477b-a1c0-1538ff4cf360", "position": 1 }
    ],
    "custom_field_1": "92837484", "custom_field_2": null,
    "date_of_birth": "1975-10-23", "dentist_id": null, "dentist_recall_date": null,
    "dentist_recall_interval": 12, "email_address": "patient@dentalpractice.com",
    "ethnicity": "07", "family_id": "abdcdeb5-ebe4-4629-a375-15e1506d3961",
    "first_name": "Alesha", "gender": false, "home_phone": "013332511673", "home_phone_country": "GB",
    "hygienist_id": null, "hygienist_recall_date": null, "hygienist_recall_interval": 12,
    "image_url": "https://www.gravatar.com/avatar/7a37035f9ccdd683c05435e07c60ff90.jpg?&r=pg&d=identicon&s=190",
    "last_name": "Chaney", "legacy_id": null, "marketing": true,
    "medical_alert": false, "medical_alert_text": null, "metadata": {}, "middle_name": "Kiara",
    "mobile_phone": "07306743470", "mobile_phone_country": "GB",
    "nhs_number": "7474158055", "ni_number": null, "occupation": "Doctor", "payment_plan_id": 2,
    "postcode": "DG11 6JM", "pps_number": "4567893W", "preferred_name": null,
    "preferred_phone_number": 3, "recall_method": "Letter",
    "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75", "title": "Ms", "town": "Giffnock",
    "updated_at": "2013-11-04T03:07:33.954+00:00", "use_email": false, "use_sms": true,
    "work_phone": "020303132213", "work_phone_country": "GB",
    "emergency_contact_name": "Jim", "emergency_contact_relationship": "Friend",
    "emergency_contact_phone": "07306743470", "emergency_contact_phone_country": "GB",
    "emergency_contact_phone_normalized": "+447306743470"
  }
}
```

#### Edit a patient

Any create field can be sent; none are required. `PUT` and `PATCH` are both accepted.

```http
PATCH /v1/patients/3

{ "patient": { "first_name": "Frank", "occupation": "Doctor" } }
```

```json
{
  "patient": {
    "id": 3, "account_id": 3, "active": true, "first_name": "Frank", "last_name": "Smith",
    "date_of_birth": "1980-01-01", "gender": true, "ethnicity": "99",
    "address_line_1": "21 Oak Avenue", "postcode": "W1A 1AA", "occupation": "Doctor",
    "payment_plan_id": 1, "recall_method": "Letter", "metadata": {},
    "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75", "title": "Mr",
    "created_at": "2014-09-11T14:48:01.563+01:00", "updated_at": "2014-09-12T09:10:11.000+01:00"
  }
}
```

The response is trimmed here. It has the same shape as Get a patient.

#### Delete (archive) a patient

Sets `active` to false. Set it back to true with an edit to undo.

```http
DELETE /v1/patients/1
```

Response: `204 No Content`

#### List patients

Filters: `query` (names, postcode, date of birth, email or phone; include `*` to return inactive patients), `updated_after`, `created_after`, `payment_plan_id`, `site_id`.

```http
GET /v1/patients?query=al&site_id=e1d854af-15db-4482-8e29-39e74a6c7f75&per_page=25
```

```json
{
  "patients": [
    {
      "id": 1, "account_id": 1, "active": true, "first_name": "Alesha", "last_name": "Chaney",
      "title": "Ms", "date_of_birth": "1975-10-23", "email_address": "patient@dentalpractice.com",
      "mobile_phone": "07306743470", "postcode": "DG11 6JM", "payment_plan_id": 2,
      "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75", "metadata": {}
    }
  ],
  "meta": { "total": 3, "page": 1 }
}
```

Items are trimmed here. Each one has the full patient shape.

### Patient referrals

`referrable_type` is one of `Referrals::PrivateIngroupReferral`, `Referrals::PrivateInhouseReferral`, `Referrals::PrivateExternalReferral`, `Referrals::NhsExternalReferral`. `status` is one of `draft`, `submitted`, `accepted`, `booked`, `completed`, `rejected`.

#### Get a patient referral

```http
GET /v1/patient_referrals/122
```

```json
{
  "patient_referral": {
    "id": 122, "created_at": "2026-03-24T09:55:56.473+00:00", "patient_id": 941,
    "patient_referral_reasons": [
      { "referral_reason_id": "6c7db82d-d013-455d-8d8c-d76556c4ada7", "name": "Capacity constraints" },
      { "referral_reason_id": "4857f623-ae5e-48fe-90bb-642cd05713a1", "name": "Funding requirements" }
    ],
    "reference": "REF-000122",
    "referrable_type": "Referrals::PrivateIngroupReferral",
    "referrable_attributes": {
      "additional_information": null, "consented_by_patient": true,
      "referred_practitioner_id": null, "referred_site_id": "0406e162-58c5-4384-bfe0-e842ab4fa4cf"
    },
    "services_appointment_id": "4590d1cb-0e1f-4604-8536-c53fa38037e6",
    "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75", "status": "submitted",
    "updated_at": "2026-03-24T09:56:44.325+00:00", "user_id": 31813
  }
}
```

#### List patient referrals

Filters: `site_id`, `referrable_type`, `created_after`, `created_before`.

```http
GET /v1/patient_referrals?referrable_type=Referrals::PrivateIngroupReferral&created_after=2026-03-01
```

```json
{
  "patient_referrals": [
    {
      "id": 122, "created_at": "2026-03-24T09:55:56.473+00:00", "patient_id": 941,
      "reference": "REF-000122", "referrable_type": "Referrals::PrivateIngroupReferral",
      "referrable_attributes": { "consented_by_patient": true, "referred_practitioner_id": 1, "referred_site_id": "0406e162-58c5-4384-bfe0-e842ab4fa4cf" },
      "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75", "status": "submitted", "user_id": 31813
    }
  ],
  "meta": { "total": 1, "current_page": 1, "total_pages": 1 }
}
```

### Patient stats

#### Get a patient's stats

```http
GET /v1/patients/52/stats
```

```json
{
  "patient_stat": {
    "patient_id": 52,
    "first_appointment_date": "2016-01-11T16:00:00.000+00:00", "first_exam_date": "2016-01-11T16:00:00.000+00:00",
    "last_appointment_date": "2016-06-09T14:00:00.000+01:00", "last_cancelled_appointment_date": null,
    "last_exam_date": "2016-01-11T16:00:00.000+00:00", "last_fta_appointment_date": "2018-09-18T10:00:00.000+01:00",
    "last_scale_and_polish_date": null, "next_appointment_date": null,
    "next_exam_date": null, "next_scale_and_polish_date": null,
    "nhs_exemption_code": null, "total_invoiced": "0.0", "total_paid": "0.0",
    "created_at": "2015-10-20T23:21:46.690+01:00", "updated_at": "2018-09-19T01:00:36.638+01:00"
  }
}
```

#### List patient stats

Filters: `created_at_after`, `created_at_before`, `updated_at_after`, `updated_at_before`.

```http
GET /v1/patient_stats?updated_at_after=2018-09-01
```

```json
{
  "patient_stats": [
    {
      "patient_id": 52, "last_appointment_date": "2016-06-09T14:00:00.000+01:00",
      "last_exam_date": "2016-01-11T16:00:00.000+00:00", "next_appointment_date": null,
      "total_invoiced": "0.0", "total_paid": "0.0", "nhs_exemption_code": null,
      "created_at": "2015-10-20T23:21:46.690+01:00", "updated_at": "2018-09-19T01:00:36.638+01:00"
    }
  ],
  "meta": { "total": 3, "page": 1 }
}
```

### Acquisition sources

#### List acquisition sources

Ordered by name.

```http
GET /v1/acquisition_sources
```

```json
{
  "acquisition_sources": [
    { "id": "abdcdeb5-ebe4-4629-a375-15e1506d3961", "active": true, "name": "Flyer", "notes": "Posted Aug 2023" }
  ],
  "meta": { "total": 3, "page": 1 }
}
```

### Custom fields

A patient's or treatment plan item's `custom_fields[].field_id` points at one of these records.

#### List custom fields

Filters: `active`, `updated_after`, `updated_before`.

```http
GET /v1/custom_fields?active=true&updated_after=2021-08-21&updated_before=2021-08-25
```

```json
{
  "custom_fields": [
    {
      "id": "5427b8d4-eacd-4e6a-8a4a-f2a37979425c", "active": true,
      "created_at": "2020-05-19T08:32:36.008+01:00", "updated_at": "2020-05-19T08:32:36.008+01:00",
      "customizable_id": "a243bf5f-72a8-4e52-9fd7-62b729e8544c", "customizable_type": "Treatment",
      "default_value": null, "description": null, "name": "Registration Status",
      "options": [
        { "name": "Registered at the practice", "position": 0 },
        { "name": "New patient", "position": 1 },
        { "name": "Emergency referral", "position": 2 }
      ],
      "position": 0, "type": "enum", "width": 5
    }
  ],
  "meta": { "total": 1, "current_page": 1, "total_pages": 1 }
}
```

#### Get a custom field

```http
GET /v1/custom_fields/5427b8d4-eacd-4e6a-8a4a-f2a37979425c
```

```json
{
  "custom_field": {
    "id": "5427b8d4-eacd-4e6a-8a4a-f2a37979425c", "active": true,
    "created_at": "2020-05-19T08:32:36.008+01:00", "updated_at": "2020-05-19T08:32:36.008+01:00",
    "customizable_id": "a243bf5f-72a8-4e52-9fd7-62b729e8544c", "customizable_type": "Treatment",
    "default_value": null, "description": null, "name": "Registration Status",
    "options": [ { "name": "Registered at the practice", "position": 0 }, { "name": "New patient", "position": 1 } ],
    "position": 0, "type": "enum"
  }
}
```

### Recalls

#### List recalls

Filters: `created_after`, `created_before`, `updated_after`, `prebooked` (default false).

```http
GET /v1/recalls?created_after=2021-09-01&prebooked=false
```

```json
{
  "recalls": [
    {
      "id": "4cefeeed-0723-4095-8b0d-ff16943d1af8", "patient_id": 2,
      "appointment_id": "de63f346-5426-4321-45t4-7c0aar13e443", "due_date": "2021-09-30",
      "first_reminder_id": "fe53d337-4693-4478-86b9-8b0aaa13e483",
      "first_reminder_sent_at": "2021-09-30T13:24:57.107+01:00", "first_reminder_type": "Email",
      "second_reminder_id": "45b14ca1-4b59-475c-80af-7f711ba1fd8e",
      "second_reminder_sent_at": "2021-09-30T13:24:57.107+01:00", "second_reminder_type": "Letter",
      "last_reminded_at": "2021-09-30T13:24:57.107+01:00", "latest_reminder_type": "Email",
      "prebooked": false,
      "reminders": [
        { "type": "Email", "id": "fe53d337-4693-4478-86b9-8b0aaa13e483", "date": "2021-09-30T13:24:57.107+01:00", "template_name": "Default recall template" },
        { "type": "Letter", "id": "45b14ca1-4b59-475c-80af-7f711ba1fd8e", "date": "2021-09-30T13:24:57.107+01:00", "template_name": "Default recall template" }
      ],
      "recall_method": "SMS", "recall_type": "Dentist", "status": "Completed",
      "times_contacted": 2, "run_date": null, "workflow_stage_id": null, "workflow_status": "unprocessed"
    }
  ],
  "meta": { "total": 1, "current_page": 1, "total_pages": 1 }
}
```

### Waiting lists

#### Get a waiting list

```http
GET /v1/waiting_lists/751c9241-62de-40aa-8f8f-6a0326032e44
```

```json
{
  "waiting_list": {
    "id": "751c9241-62de-40aa-8f8f-6a0326032e44", "active": true, "appointment_reasons": [],
    "created_at": "2019-09-05T16:31:38.910+01:00", "updated_at": "2023-02-21T14:36:37.041+00:00",
    "default_appointment_duration": 5, "default_appointment_reason": "Exam",
    "default_notes": null, "default_status": "waiting",
    "default_treatment_duration": 5, "default_treatment_reason": "Exam",
    "name": "Grahams List", "site_id": "0406e162-58c5-4384-bfe0-e842ab4fa4cf",
    "target_appointment_days": null, "target_treatment_days": null
  }
}
```

The source docs show this response as an array. See the notes at the end.

#### List waiting lists

Filter: `site_id`.

```http
GET /v1/waiting_lists?site_id=0406e162-58c5-4384-bfe0-e842ab4fa4cf
```

```json
{
  "waiting_lists": [
    {
      "id": "ddcac4b7-9fd6-40ec-b969-38795066502b", "active": true,
      "appointment_reasons": ["Exam", "Scale and Polish"],
      "default_appointment_duration": 5, "default_appointment_reason": "Exam",
      "default_notes": null, "default_status": "waiting",
      "default_treatment_duration": 5, "default_treatment_reason": "Exam",
      "name": "Stuff", "target_appointment_days": null, "target_treatment_days": null,
      "created_at": "2019-09-05T16:24:41.382+01:00", "updated_at": "2019-09-05T16:24:41.382+01:00",
      "site_id": "0406e162-58c5-4384-bfe0-e842ab4fa4cf"
    }
  ],
  "meta": { "total": 2, "current_page": 1, "total_pages": 1 }
}
```

### Waiting list entries

#### Get waiting list entries

`waiting_list_id` is required. The docs list the path as `/v1/waiting_list_entries/{WAITING_LIST_ID}` but the example call uses a query parameter, which is what this sample follows.

```http
GET /v1/waiting_list_entries?waiting_list_id=751c9241-62de-40aa-8f8f-6a0326032e44
```

```json
{
  "waiting_list_entries": [
    {
      "id": "b758069a-aec5-45e8-8874-50196e72936d", "waiting_list_id": "751c9241-62de-40aa-8f8f-6a0326032e44",
      "patient_id": "276e47ce-3976-4317-b474-a349a5fe92c8", "patient_sid": 896, "patient_name": "laura adam",
      "practitioner_id": "0c6ce55b-4751-4347-b287-23cc87688143", "practitioner_sid": 4,
      "appointment_id": "3429c60d-9d0b-4f5b-add3-95f9edd3dc80", "appointment_sid": 46750712,
      "appointment_actual_on": null, "appointment_target_on": null, "appointment_treatment_id": null,
      "specialist_id": null, "specialist_sid": null, "referral_on": null, "site_id": null,
      "status": "booked", "priority": 2, "archived": false, "notes": "i am waiting",
      "listed_on": "2023-03-22", "days_waiting_appointment": 8, "days_waiting_treatment": 0,
      "treatment_actual_on": null, "treatment_target_on": null,
      "history": [
        {
          "changes": { "status": ["waiting", "booked"], "appointment_id": [null, "3429c60d-9d0b-4f5b-add3-95f9edd3dc80"] },
          "history_at": "2023-03-22T15:31:53.344+00:00", "history_by": 1, "changed_count": 3
        }
      ],
      "last_edited_by_id": 1, "user_id": 1,
      "created_at": "2023-03-22T15:30:47.571+00:00", "updated_at": "2023-03-22T15:31:53.352+00:00"
    }
  ],
  "meta": { "total": 2, "current_page": 1, "total_pages": 1 }
}
```

### PRSI eligibilities

Irish PRSI dental benefit checks per patient. `exam_eligible_status` and `hygienist_eligible_status` are one of `Eligible`, `Expired`, `Used`, `None`.

#### List PRSI eligibilities

Filters: `eligible_check_date_after`/`_before`, `expiry_date_after`/`_before`, `hygienist_eligible_status`, `exam_eligible_status`, `patient_consented`, `site_id`, `patient_id`.

```http
GET /v1/prsi_eligibilities?eligible_check_date_after=2024-02-29
```

```json
{
  "prsi_eligibilities": [
    {
      "id": "80d0d7be-b2da-4cc8-833a-fefea0d17456",
      "eligible_check_date": "2024-04-19T00:00:00.000+01:00", "expiry_date": "2025-04-19T00:00:00.000+01:00",
      "exam_eligible_status": "Eligible", "hygienist_eligible_status": "Eligible", "patient_consented": true,
      "patient_id": "0f4a6af2-2a52-4be5-a27a-a49f52b7f381", "site_id": "0406e162-58c5-4384-bfe0-e842ab4fa4cf", "user_id": 1,
      "created_at": "2024-05-02T08:53:43.799+01:00", "updated_at": "2024-05-02T08:53:43.799+01:00"
    }
  ],
  "meta": { "total": 2, "current_page": 1, "total_pages": 1 }
}
```

#### Get a PRSI eligibility

```http
GET /v1/prsi_eligibilities/d7c1781a-2d7d-4d50-8de8-a2b5e5306a91
```

```json
{
  "prsi_eligibility": {
    "id": "d7c1781a-2d7d-4d50-8de8-a2b5e5306a91",
    "eligible_check_date": "2024-04-19T00:00:00.000+01:00", "expiry_date": "2025-04-19T00:00:00.000+01:00",
    "exam_eligible_status": "Eligible", "hygienist_eligible_status": "Eligible", "patient_consented": true,
    "patient_id": "0f4a6af2-2a52-4be5-a27a-a49f52b7f381", "site_id": "0406e162-58c5-4384-bfe0-e842ab4fa4cf", "user_id": 1,
    "created_at": "2024-05-02T08:53:55.585+01:00", "updated_at": "2024-05-02T08:53:55.585+01:00"
  }
}
```

#### Create a PRSI eligibility

Required: `patient_id`, `patient_consented`, `eligible_check_date`, `expiry_date`. Statuses default to `None`.

```http
POST /v1/prsi_eligibilities

{
  "prsi_eligibility": {
    "patient_id": "0f4a6af2-2a52-4be5-a27a-a49f52b7f381", "patient_consented": true,
    "eligible_check_date": "19/04/2024", "expiry_date": "19/04/2025"
  }
}
```

```json
{
  "prsi_eligibility": {
    "id": "02d0f5f6-30e2-46aa-a257-036c3e744d9d",
    "eligible_check_date": "2024-04-19T00:00:00.000+01:00", "expiry_date": "2025-04-19T00:00:00.000+01:00",
    "exam_eligible_status": "None", "hygienist_eligible_status": "None", "patient_consented": true,
    "patient_id": "0f4a6af2-2a52-4be5-a27a-a49f52b7f381", "site_id": "0406e162-58c5-4384-bfe0-e842ab4fa4cf", "user_id": 1,
    "created_at": "2024-05-02T10:16:42.382+01:00", "updated_at": "2024-05-02T10:16:42.382+01:00"
  }
}
```

#### Edit a PRSI eligibility

```http
PATCH /v1/prsi_eligibilities/d7c1781a-2d7d-4d50-8de8-a2b5e5306a91

{ "prsi_eligibility": { "exam_eligible_status": "Expired", "hygienist_eligible_status": "Eligible" } }
```

```json
{
  "prsi_eligibility": {
    "id": "d7c1781a-2d7d-4d50-8de8-a2b5e5306a91",
    "eligible_check_date": "2024-04-19T00:00:00.000+01:00", "expiry_date": "2025-04-19T00:00:00.000+01:00",
    "exam_eligible_status": "Expired", "hygienist_eligible_status": "Eligible", "patient_consented": true,
    "patient_id": "0f4a6af2-2a52-4be5-a27a-a49f52b7f381", "site_id": "0406e162-58c5-4384-bfe0-e842ab4fa4cf", "user_id": 1,
    "created_at": "2024-05-02T08:53:55.585+01:00", "updated_at": "2024-05-02T10:58:22.511+01:00"
  }
}
```

#### Delete a PRSI eligibility

```http
DELETE /v1/prsi_eligibilities/90205d1d-9e07-43ee-9a95-245995cb1f42
```

Response: `204 No Content`

## Appointments and scheduling

Appointments link a patient, a practitioner and optionally a room. `state` is one of `Pending`, `Confirmed`, `Arrived`, `In surgery`, `Completed`, `Cancelled`, `Did not attend`, and changing it stamps the matching `*_at` field. `reason` is one of `Exam`, `Scale & Polish`, `Exam + Scale & Polish`, `Continuing Treatment`, `Emergency`, `Review`, `Other`.

### Appointments

#### Create an appointment

Required: `start_time`, `finish_time`, `practitioner_id`, `reason`. Optional: `patient_id`, `state` (default `Pending`), `notes`, `metadata`, `force_changes` (allows double booking).

```http
POST /v1/appointments

{
  "appointment": {
    "start_time": "2026-10-20T13:00:00.000Z", "finish_time": "2026-10-20T13:10:00.000Z",
    "patient_id": 1, "practitioner_id": 1, "reason": "Exam"
  }
}
```

```json
{
  "appointment": {
    "id": 14493, "uuid": "f7b1b1b0-0b1b-4b1b-8b1b-2a2b5b6b1e1c",
    "appointment_cancellation_reason_id": null, "booked_via_api": true,
    "pending_at": "2026-10-14T13:42:14.424+00:00", "confirmed_at": null, "arrived_at": null,
    "in_surgery_at": null, "completed_at": null, "cancelled_at": null, "did_not_attend_at": null,
    "duration": 10, "start_time": "2026-10-20T13:00:00.000+00:00", "finish_time": "2026-10-20T13:10:00.000+00:00",
    "metadata": {}, "notes": null,
    "patient_id": 1, "patient_name": "Liza Stafford",
    "patient_image_url": "https://www.gravatar.com/avatar/d80c726c2a3a8f426acaa06e16b05b09.jpg?&r=pg&d=identicon&s=190",
    "payment_plan_id": 1, "practitioner_id": 1, "reason": "Exam",
    "room_id": null, "state": "Pending", "treatment_description": null, "user_id": 1
  }
}
```

#### Get an appointment

```http
GET /v1/appointments/14493
```

```json
{
  "appointment": {
    "id": 14493, "uuid": "f7b1b1b0-0b1b-4b1b-8b1b-2a2b5b6b1e1c",
    "appointment_cancellation_reason_id": null, "booked_via_api": true,
    "pending_at": "2026-10-14T13:42:14.424+00:00", "confirmed_at": null, "arrived_at": null,
    "in_surgery_at": null, "completed_at": null, "cancelled_at": null, "did_not_attend_at": null,
    "duration": 10, "start_time": "2026-10-20T13:00:00.000+00:00", "finish_time": "2026-10-20T13:10:00.000+00:00",
    "metadata": {}, "notes": null,
    "patient_id": 1, "patient_name": "Liza Stafford",
    "patient_image_url": "https://www.gravatar.com/avatar/d80c726c2a3a8f426acaa06e16b05b09.jpg?&r=pg&d=identicon&s=190",
    "payment_plan_id": 1, "practitioner_id": 1, "reason": "Exam",
    "room_id": "04b2db45-455a-42f7-b9ce-99fa6b0c741b", "state": "Pending",
    "treatment_description": null, "user_id": 1
  }
}
```

#### Edit an appointment

Updating `state` sets the matching timestamp automatically. `PUT` and `PATCH` are both accepted.

```http
PATCH /v1/appointments/14493

{ "appointment": { "finish_time": "2026-10-20T13:15:00.000Z", "state": "Confirmed" } }
```

```json
{
  "appointment": {
    "id": 14493, "uuid": "f7b1b1b0-0b1b-4b1b-8b1b-2a2b5b6b1e1c",
    "appointment_cancellation_reason_id": null, "booked_via_api": true,
    "pending_at": "2026-10-14T13:42:14.424+00:00", "confirmed_at": "2026-10-14T14:05:00.000+00:00",
    "arrived_at": null, "in_surgery_at": null, "completed_at": null, "cancelled_at": null, "did_not_attend_at": null,
    "duration": 15, "start_time": "2026-10-20T13:00:00.000+00:00", "finish_time": "2026-10-20T13:15:00.000+00:00",
    "metadata": {}, "notes": null, "patient_id": 1, "patient_name": "Liza Stafford",
    "payment_plan_id": 1, "practitioner_id": 1, "reason": "Exam",
    "room_id": "04b2db45-455a-42f7-b9ce-99fa6b0c741b", "state": "Confirmed",
    "treatment_description": null, "user_id": 1
  }
}
```

The `patient_image_url` field is left out here and in the list below to save space.

#### Delete an appointment

Only use this to remove an appointment entirely. To cancel one, set `state` to `Cancelled` instead.

```http
DELETE /v1/appointments/14493
```

Response: `204 No Content`

#### List appointments

Filters (combinable): `on`, `before`, `after`, `updated_after`, `practitioner_id`, `patient_id`, `room_id`, `site_id`, `state` (case sensitive), `cancelled` (true includes cancelled and did-not-attend). Ordered by start time.

```http
GET /v1/appointments?on=2026-10-20&practitioner_id=1&state=Pending
```

```json
{
  "appointments": [
    {
      "id": 14493, "appointment_cancellation_reason_id": null,
      "pending_at": "2026-10-14T13:42:14.424+00:00", "confirmed_at": null, "arrived_at": null,
      "in_surgery_at": null, "completed_at": null, "cancelled_at": null, "did_not_attend_at": null,
      "duration": 15, "start_time": "2026-10-20T13:00:00.000+00:00", "finish_time": "2026-10-20T13:15:00.000+00:00",
      "metadata": {}, "notes": null, "patient_id": 1, "patient_name": "Liza Stafford",
      "payment_plan_id": 1, "practitioner_id": 1, "reason": "Exam",
      "room_id": "04b2db45-455a-42f7-b9ce-99fa6b0c741b", "state": "Pending",
      "treatment_description": null, "user_id": 1,
      "created_at": "2026-10-14T13:42:14.424+00:00", "updated_at": "2026-10-14T13:42:14.424+00:00"
    }
  ],
  "meta": { "total": 3, "page": 1 }
}
```

#### Appointment availability

Required: `start_time` and `finish_time` (both in the future), `practitioner_ids[]`. Optional: `duration` in minutes, `page`, `per_page` (max 20). Limited to 200 requests per hour.

```http
GET /v1/appointments/availability?practitioner_ids[]=1&duration=5&start_time=2026-10-20T12:00:11+00:00&finish_time=2026-11-19T12:00:11+00:00
```

```json
{
  "availability": [
    { "start_time": "2026-10-20T12:00:00.000+00:00", "finish_time": "2026-10-20T14:20:00.000+00:00", "available_duration": 140 },
    { "start_time": "2026-10-20T14:40:00.000+00:00", "finish_time": "2026-10-20T17:00:00.000+00:00", "available_duration": 140 }
  ],
  "meta": { "page": 1 }
}
```

#### List appointment reasons

Filter: `deleted`.

```http
GET /v1/appointment_reasons?deleted=false
```

```json
{
  "appointment_reasons": [
    { "id": "7a0a4a71-ecd0-4b0b-81d5-e1739aee88ba", "colour": "#ad00fc", "deleted": false, "exam": false, "hygiene": false, "position": 50, "reason": "NOT WORKING" },
    { "id": "430c6234-4f3d-424d-b1ea-a4d1bc728d36", "colour": "#25B0E6", "deleted": false, "exam": false, "hygiene": false, "position": 49, "reason": "LUNCH" }
  ],
  "meta": { "total": 50, "page": 1 }
}
```

### Appointment cancellation reasons

#### List appointment cancellation reasons

Filters: `created_after`, `created_before`, `updated_after`, `updated_before`. Ordered by `reason`.

```http
GET /v1/appointment_cancellation_reasons?sort_by=created_at&sort_direction=asc
```

```json
{
  "appointment_cancellation_reasons": [
    {
      "id": "fcafb582-0434-4eb8-b990-01f5fa12904d", "archived": false,
      "reason": "Patient - Cancelled", "reason_type": "cancelled",
      "created_at": "2020-03-23T15:58:49.331+00:00", "updated_at": "2020-03-23T15:58:49.331+00:00"
    }
  ],
  "meta": { "total": 14, "current_page": 1, "total_pages": 1 }
}
```

### Rooms

#### Get a room

```http
GET /v1/rooms/04b2db45-455a-42f7-b9ce-99fa6b0c741b
```

```json
{
  "room": {
    "id": "04b2db45-455a-42f7-b9ce-99fa6b0c741b", "calendar_position": null, "colour": "#A3F137",
    "name": "Surgery 3", "import_id": null, "practice_id": 1,
    "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75",
    "created_at": "2018-01-02T15:27:15.461+00:00", "updated_at": "2018-01-02T15:27:15.461+00:00"
  }
}
```

#### List rooms

Filter: `site_id`.

```http
GET /v1/rooms?site_id=e1d854af-15db-4482-8e29-39e74a6c7f75
```

```json
{
  "rooms": [
    {
      "id": "984deaaf-9218-48fe-9034-20d0fe44f946", "calendar_position": null, "colour": "#a660a6",
      "name": "Surgery 1", "import_id": null, "practice_id": 1,
      "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75",
      "created_at": "2017-01-26T14:49:47.426+00:00", "updated_at": "2017-01-26T14:49:47.426+00:00"
    }
  ],
  "meta": { "total": 4, "current_page": 1, "total_pages": 1 }
}
```

### Practitioner diaries (rota)

Requires the Rota feature to be enabled. Diaries are generated up to 4 years ahead.

#### List practitioner diaries

Required: `after`, `before`. Optional: `practitioner_id`, `site_id`, `include_inactive_practitioners` (default false). Ordered by day.

```http
GET /v1/rota_practitioner_diaries?after=2021-08-21&before=2021-08-25&practitioner_id=1
```

```json
{
  "rota_practitioner_diaries": [
    {
      "id": "d6df1c88-b9b2-423d-8de8-62027e0ba0ab", "day": "2021-08-23",
      "start_time": "2021-08-23T10:00:00.000+01:00", "end_time": "2021-08-23T15:00:00.000+01:00",
      "unavailable": false, "practitioner_id": "1",
      "breaks": [
        { "id": "8c9c7d2a-6d1a-4786-98aa-2cbd6f1af3df", "name": "Lunch", "start_time": "2021-08-23T12:00:00.000+01:00", "end_time": "2021-08-23T13:00:00.000+01:00" }
      ]
    }
  ],
  "meta": { "total": 1, "current_page": 1, "total_pages": 1 }
}
```

### Session appointments

Background blocks in the appointment book such as lunch or meetings.

#### List session appointments

Filters: `on`, `before`, `after`, `practitioner_id`, `room_id`. Ordered by start time.

```http
GET /v1/session_appointments?on=2026-10-20&practitioner_id=1
```

```json
{
  "session_appointments": [
    {
      "id": "9cfefa03-8870-4ec1-9dc3-5abec498b785", "colour": "#0C99EF", "description": "Team meeting",
      "display_type": "background", "practitioner_id": 1,
      "room_id": "04b2db45-455a-42f7-b9ce-99fa6b0c741b",
      "start_time": "2026-10-20T09:00:00.000+00:00", "finish_time": "2026-10-20T09:40:00.000+00:00"
    }
  ],
  "meta": { "total": 3, "page": 1 }
}
```

### Treatment appointments

A treatment appointment is a bookable visit inside a treatment plan. It becomes linked to an appointment once booked.

#### Get a treatment appointment

```http
GET /v1/treatment_appointments/98367
```

```json
{
  "treatment_appointment": {
    "id": 98367, "bookable": true, "completed": false, "completed_at": null, "notes": null, "position": 0,
    "appointment_id": null, "patient_id": 19980, "treatment_plan_id": 52308,
    "created_at": "2019-07-16T15:29:53.432+01:00", "updated_at": "2019-07-16T15:29:53.432+01:00"
  }
}
```

#### List treatment appointments

Filters: `patient_id`, `treatment_plan_id`, `updated_after`, `updated_before`, `created_after`, `created_before`. Ordered by `created_at`.

```http
GET /v1/treatment_appointments?patient_id=19980&treatment_plan_id=52308
```

```json
{
  "treatment_appointments": [
    {
      "id": 98367, "bookable": true, "completed": false, "completed_at": null, "notes": null, "position": 0,
      "appointment_id": null, "patient_id": 19980, "treatment_plan_id": 52308,
      "created_at": "2019-07-16T15:29:53.432+01:00", "updated_at": "2019-07-16T15:29:53.432+01:00"
    }
  ],
  "meta": { "total": 3, "page": 1 }
}
```

## Treatments, plans and fees

A treatment plan belongs to a patient and holds treatment plan items. Each item points at a catalogue treatment, and a fee sets that treatment's price per payment plan.

### Treatment categories

#### Get a treatment category

```http
GET /v1/treatment_categories/3
```

```json
{ "treatment_category": { "id": 3, "name": "Primary" } }
```

#### List treatment categories

```http
GET /v1/treatment_categories
```

```json
{
  "treatment_categories": [ { "id": 1, "name": "Primary" }, { "id": 2, "name": "Secondary" } ],
  "meta": { "total": 2, "current_page": 1 }
}
```

### Treatments

#### Get a treatment

```http
GET /v1/treatments/210
```

```json
{
  "treatment": {
    "id": 210, "active": true, "code": "URGXL", "nomenclature": "Extraction (urgent)",
    "description": "", "nhs_treatment_cat": "9307", "notes": "<div>Lignocaine 2% with 1:80,000 Adrenaline</div>",
    "owner": "practice", "patient_description": "", "patient_nomenclature": "Extraction",
    "region": "tooth", "treatment_category_id": 1, "uda_band": 4,
    "created_at": "2015-06-01T15:48:39.412+01:00", "updated_at": "2015-06-01T15:53:37.046+01:00"
  }
}
```

#### List treatments

Filters: `active` (`true`, `false` or `all`), `query` (searches `code`; disables sorting), `treatment_category_id`. Sort by `code`, `created_at` or `updated_at`. Default order is `code`.

```http
GET /v1/treatments?active=true&treatment_category_id=1&sort_by=code&sort_direction=asc
```

```json
{
  "treatments": [
    {
      "id": 210, "active": true, "code": "URGXL", "nomenclature": "Extraction (urgent)",
      "nhs_treatment_cat": "9307", "owner": "practice", "patient_nomenclature": "Extraction",
      "region": "tooth", "treatment_category_id": 1, "uda_band": 4,
      "created_at": "2015-06-01T15:48:39.412+01:00", "updated_at": "2015-06-01T15:53:37.046+01:00"
    }
  ],
  "meta": { "total": 3, "page": 1 }
}
```

### Treatment plans

#### Get a treatment plan

```http
GET /v1/treatment_plans/77
```

```json
{
  "treatment_plan": {
    "id": 77, "completed": true, "completed_at": "2015-02-17T11:28:37.039+00:00",
    "end_date": null, "last_completed_at": null, "nickname": null,
    "nhs_completed_uda_value": "0.0", "nhs_uda_value": "3.0", "private_treatment_value": "0.0",
    "patient_id": 7979, "practitioner_id": 16, "start_date": "2015-01-18",
    "created_at": "2015-01-18T14:08:16.523+00:00", "updated_at": "2017-01-03T20:38:44.896+00:00"
  }
}
```

#### List treatment plans

Filters: `completed`, `created_after`, `created_before`, `ids[]`, `patient_id`, `practitioner_id`, `start_date`, `updated_since`. Sort by `created_at` (default), `completed_at` or `updated_at`.

```http
GET /v1/treatment_plans?ids[]=77&ids[]=78&completed=true
```

```json
{
  "treatment_plans": [
    {
      "id": 77, "completed": true, "completed_at": "2015-02-17T11:28:37.039+00:00",
      "end_date": null, "last_completed_at": null, "nickname": null,
      "nhs_completed_uda_value": "0.0", "nhs_uda_value": "3.0", "private_treatment_value": "0.0",
      "patient_id": 7979, "practitioner_id": 16, "start_date": "2015-01-18",
      "created_at": "2015-01-18T14:08:16.523+00:00", "updated_at": "2017-01-03T20:38:44.896+00:00"
    }
  ],
  "meta": { "total": 3, "page": 1 }
}
```

### Treatment plan items

`surfaces` is an array of numbers (1 to 5 for incisors, canines, premolars and deciduous teeth, 1 to 8 for molars). `teeth` is an array of Palmer notation strings such as `"LR7"`.

#### Get a treatment plan item

```http
GET /v1/treatment_plan_items/1630192
```

```json
{
  "treatment_plan_item": {
    "id": 1630192, "appear_on_invoice": true, "base_chart": false, "charged": false,
    "completed": false, "completed_at": null,
    "custom_fields": [
      { "value": true, "field_id": "2c19fdf0-987c-4e5a-aa71-06ad8a58ee22", "position": 0 },
      { "value": null, "field_id": "38b96e14-e31d-477b-a1c0-1538ff4cf360", "position": 2 }
    ],
    "duration": 15, "nhs_treatment_cat": "9317", "nomenclature": "Examination",
    "notes": "<p><strong>Soft tissues checked</strong></p>", "patient_nomenclature": "Examination",
    "position": null, "price": "0.0", "region": "patient", "surfaces": null, "teeth": null, "uda_band": 1,
    "invoice_id": null, "patient_id": 45, "payment_plan_id": 1, "practitioner_id": 1, "referrer_id": 1,
    "treatment_appointment_id": 98361, "treatment_id": 1, "treatment_plan_id": 52302,
    "created_at": "2019-06-07T09:02:00.968+01:00", "updated_at": "2019-06-07T09:02:00.968+01:00"
  }
}
```

#### List treatment plan items

Filters: `completed_on`, `patient_id`, `practitioner_id`, `treatment_plan_id`, `treatment_appointment_id`, `updated_after`, `updated_before`.

```http
GET /v1/treatment_plan_items?patient_id=45&treatment_plan_id=52302
```

```json
{
  "treatment_plan_items": [
    {
      "id": 1630192, "appear_on_invoice": true, "base_chart": false, "charged": false,
      "completed": false, "completed_at": null, "duration": 15, "nomenclature": "Examination",
      "price": "0.0", "region": "patient", "surfaces": null, "teeth": null, "uda_band": 1,
      "invoice_id": null, "patient_id": 45, "payment_plan_id": 1, "practitioner_id": 1, "referrer_id": 1,
      "treatment_appointment_id": 98361, "treatment_id": 1, "treatment_plan_id": 52302,
      "created_at": "2019-06-07T09:02:00.968+01:00", "updated_at": "2019-06-07T09:02:00.968+01:00"
    }
  ],
  "meta": { "total": 3, "page": 1 }
}
```

### Fees

A fee prices one treatment under one payment plan. When `multiple_pricing` is false only `price_one` and `duration_one` apply, whatever the number of teeth or surfaces.

#### Get a fee

```http
GET /v1/fees/a5ee8c65-84e9-49a0-a7b5-cb136e65007c
```

```json
{
  "fee": {
    "id": "a5ee8c65-84e9-49a0-a7b5-cb136e65007c", "payment_plan_id": 1234, "treatment_id": 5678,
    "multiple_pricing": true,
    "duration_one": 10, "duration_two": 20, "duration_three": 30, "duration_four": 40, "duration_five": 50,
    "price_one": "10.00", "price_two": "20.00", "price_three": "30.00", "price_four": "40.00", "price_five": "50.00"
  }
}
```

#### List fees

You must filter by `payment_plan_id` or `treatment_id`. Ordered by id.

```http
GET /v1/fees?payment_plan_id=1234
```

```json
{
  "fees": [
    {
      "id": "a5ee8c65-84e9-49a0-a7b5-cb136e65007c", "payment_plan_id": 1234, "treatment_id": 5678,
      "multiple_pricing": true,
      "duration_one": 10, "duration_two": 20, "duration_three": 30, "duration_four": 40, "duration_five": 50,
      "price_one": "10.00", "price_two": "20.00", "price_three": "30.00", "price_four": "40.00", "price_five": "50.00"
    }
  ],
  "meta": { "total": 1, "page": 1 }
}
```

#### Edit a fee

Send any of `multiple_pricing` and the `duration_*` and `price_*` fields.

```http
PATCH /v1/fees/a5ee8c65-84e9-49a0-a7b5-cb136e65007c

{ "fee": { "price_one": "20.00" } }
```

```json
{
  "fee": {
    "id": "a5ee8c65-84e9-49a0-a7b5-cb136e65007c", "payment_plan_id": 1234, "treatment_id": 5678,
    "multiple_pricing": true,
    "duration_one": 10, "duration_two": 20, "duration_three": 30, "duration_four": 40, "duration_five": 50,
    "price_one": "20.00", "price_two": "20.00", "price_three": "30.00", "price_four": "40.00", "price_five": "50.00"
  }
}
```

### Sundries

Sundries are extra chargeable items that can appear as invoice items. The source docs show both sundry responses as arrays under a `sundry` key, which is reproduced below.

#### Get a sundry

```http
GET /v1/sundries/1
```

```json
{
  "sundry": [
    { "id": "b2cc52a7-f488-419b-b9ac-f3c2f989c78e", "name": "Dentally Sundry", "nickname": "Dentally Sundry", "price": 0.0, "site_id": 1 }
  ],
  "meta": { "total": 50, "page": 1 }
}
```

#### List sundries

Filter: `sundry_id`.

```http
GET /v1/sundries?sundry_id=10
```

```json
{
  "sundry": [
    { "id": "b2cc52a7-f488-419b-b9ac-f3c2f989c78e", "name": "Dentally Sundry", "nickname": "Dentally Sundry", "price": 0.0, "site_id": 1 }
  ],
  "meta": { "total": 50, "page": 1 }
}
```

### Payment plans

Durations are in minutes (5 to 120). Recall intervals are in months.

#### Get a payment plan

```http
GET /v1/payment_plans/1
```

```json
{
  "payment_plan": {
    "id": 1, "active": true, "created_at": "2013-11-05T11:27:05.686+00:00", "colour": "#25B0E6",
    "dentist_recall_interval": 6, "hygienist_recall_interval": 6,
    "emergency_duration": 15, "exam_duration": 10, "exam_scale_and_polish_duration": 20, "scale_and_polish_duration": 10,
    "name": "NHS", "patient_friendly_name": "NHS Plan", "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75"
  }
}
```

#### Create a payment plan

Required: `name`. Defaults: `active` true, recall intervals 6, `emergency_duration` 10, `exam_duration` 10, `exam_scale_and_polish_duration` 20, `scale_and_polish_duration` 10. `patient_friendly_name` defaults to `name`. A null `site_id` applies the plan to all sites.

```http
POST /v1/payment_plans

{ "payment_plan": { "name": "Custom Plan", "colour": "#FFFFFF" } }
```

```json
{
  "payment_plan": {
    "id": 12, "active": true, "created_at": "2026-10-14T12:34:00.000+00:00", "colour": "#FFFFFF",
    "dentist_recall_interval": 6, "hygienist_recall_interval": 6,
    "emergency_duration": 10, "exam_duration": 10, "exam_scale_and_polish_duration": 20, "scale_and_polish_duration": 10,
    "name": "Custom Plan", "patient_friendly_name": "Custom Plan", "site_id": null
  }
}
```

#### Edit a payment plan

```http
PATCH /v1/payment_plans/1

{ "payment_plan": { "colour": "#FFFFFF" } }
```

```json
{
  "payment_plan": {
    "id": 1, "active": true, "created_at": "2013-11-05T11:27:05.686+00:00", "colour": "#FFFFFF",
    "dentist_recall_interval": 6, "hygienist_recall_interval": 6,
    "emergency_duration": 15, "exam_duration": 10, "exam_scale_and_polish_duration": 20, "scale_and_polish_duration": 10,
    "name": "NHS", "patient_friendly_name": "NHS Plan", "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75"
  }
}
```

#### List payment plans

Filters: `site_id`, `active`. Ordered by creation date.

```http
GET /v1/payment_plans?active=true
```

```json
{
  "payment_plans": [
    {
      "id": 1, "active": true, "created_at": "2013-11-05T11:27:05.686+00:00", "colour": "#25B0E6",
      "dentist_recall_interval": 6, "hygienist_recall_interval": 6,
      "emergency_duration": 15, "exam_duration": 10, "exam_scale_and_polish_duration": 20, "scale_and_polish_duration": 10,
      "name": "NHS", "patient_friendly_name": "NHS Plan", "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75"
    }
  ],
  "meta": { "total": 2, "page": 1 }
}
```

## Billing and NHS

Each patient has one account. Invoices and payments post to the account, invoice items link back to treatment plan items, and payment explanations tie a payment to an invoice. Money amounts are strings.

### Accounts

#### Get an account

```http
GET /v1/accounts/10
```

```json
{
  "account": {
    "id": 10, "current_balance": "200.0", "opening_balance": "10.0",
    "patient_id": 16, "patient_name": "Sergio Aguero",
    "planned_nhs_treatment_value": "50.0", "planned_private_treatment_value": "40.0"
  }
}
```

#### List accounts

Filters: `state` (`credit` or `debit`), `patient_id`.

```http
GET /v1/accounts?state=credit
```

```json
{
  "accounts": [
    {
      "id": 10, "current_balance": "200.0", "opening_balance": "10.0",
      "patient_id": 16, "patient_name": "Sergio Aguero",
      "planned_nhs_treatment_value": "50.0", "planned_private_treatment_value": "40.0"
    }
  ],
  "meta": { "total": 3, "total_nhs": "200.00", "total_private": "400.00", "current_page": 1, "total_balance": "854.04" }
}
```

### Invoices

#### List invoices

Filters: `account_id`, `patient_id`, `site_id`, `dated_on_after`, `dated_on_before`, `updated_after`. Default sort is `dated_on`; `created_at` and `updated_at` are also allowed. The list omits `invoice_items`.

```http
GET /v1/invoices?dated_on_after=2019-05-01&patient_id=637
```

```json
{
  "invoices": [
    {
      "id": 10253, "amount": "11.4", "amount_outstanding": "0.0",
      "dated_on": "2019-05-10", "due_on": "2019-05-17", "footnote": "", "nhs_amount": null,
      "paid": true, "paid_on": "2019-05-10", "payment_terms": "", "reference": "09944",
      "sent_at": null, "status": "new",
      "account_id": 637, "patient_id": 637, "site_id": "0fbff38e-3995-4d32-8db7-3b7aed9b35a3", "user_id": 4983,
      "created_at": "2019-05-10T08:41:27.934+01:00", "updated_at": "2019-05-10T08:48:43.322+01:00"
    }
  ],
  "meta": { "total": 1, "current_page": 1 }
}
```

#### Get an invoice

```http
GET /v1/invoices/2043
```

```json
{
  "invoice": {
    "id": 2043, "amount": "11.12", "amount_outstanding": "0.0",
    "dated_on": "2018-02-05", "due_on": "2018-02-12", "footnote": "", "nhs_amount": "11.12",
    "paid": true, "paid_on": "2018-02-05", "payment_terms": "", "reference": "01946",
    "sent_at": null, "status": "new",
    "account_id": 2986, "patient_id": 2986, "site_id": "0fbff38e-3995-4d32-8db7-3b7aed9b35a3", "user_id": 4985,
    "created_at": "2018-02-05T14:12:00.607+00:00", "updated_at": "2018-09-15T00:33:34.872+01:00",
    "invoice_items": [
      {
        "id": "b2cc52a7-f488-419b-b9ac-f3c2f989c78e", "name": "Routine Examination",
        "item_price": "0.0", "quantity": 1, "total_price": "0.0", "nhs_charge": false,
        "invoice_id": 2043, "practitioner_id": 4, "sundry_id": null,
        "treatment_plan_id": 3946, "treatment_plan_item_id": 55296, "user_id": null,
        "created_at": "2018-09-15T00:33:34.847+01:00", "updated_at": "2018-09-15T00:33:34.847+01:00"
      }
    ]
  }
}
```

### Invoice items

#### List invoice items

Filters: `invoice_id`, `updated_after`.

```http
GET /v1/invoice_items?invoice_id=2043&updated_after=2021-08-21T13:14:15.123Z
```

```json
{
  "invoice_items": [
    {
      "id": "b2cc52a7-f488-419b-b9ac-f3c2f989c78e", "name": "Routine Examination",
      "item_price": "0.0", "quantity": 1, "total_price": "0.0", "nhs_charge": false,
      "invoice_id": 2043, "practitioner_id": 4, "sundry_id": null,
      "treatment_plan_id": 3946, "treatment_plan_item_id": 55296, "user_id": null,
      "created_at": "2018-09-15T00:33:34.847+01:00", "updated_at": "2018-09-15T00:33:34.847+01:00"
    }
  ]
}
```

### Payments

A payment is credited to an account. Its `explanations` array shows how the payment is matched to invoices.

#### Create a payment

Required: `account_id`, `amount`, `dated_on`, `method` (for example `Credit Card`, `Debit Card`, `American Express`, `Cash`), `patient_id`, `payment_plan_id`. Optional: `practitioner_id`, `site_id`, `transaction_number`.

```http
POST /v1/payments

{
  "payment": {
    "account_id": 1, "amount": "100.00", "dated_on": "2026-10-14",
    "method": "Cash", "patient_id": 1, "payment_plan_id": 1
  }
}
```

```json
{
  "payment": {
    "id": 1, "account_id": 1, "amount": "100.0", "amount_unexplained": "100.0",
    "dated_on": "2026-10-14", "deleted": false, "fully_explained": false, "method": "Cash",
    "patient_id": 1, "payment_plan_id": 1, "practitioner_id": null, "reference": "00001",
    "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75", "status": null, "transaction_number": null,
    "user_id": 1, "explanations": []
  }
}
```

#### Get a payment

```http
GET /v1/payments/1
```

```json
{
  "payment": {
    "id": 1, "account_id": 4511, "amount": "18.5", "amount_unexplained": "0.0",
    "dated_on": "2015-01-19", "deleted": false, "fully_explained": true, "method": "Debit Card",
    "patient_id": 5106, "payment_plan_id": 1, "practitioner_id": 1, "reference": "00001",
    "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75", "status": null, "transaction_number": "6932", "user_id": 271,
    "explanations": [
      {
        "id": 57, "amount": "18.5", "comments": null, "invoice_id": 51, "invoice_reference": "00051",
        "payment_id": 1, "payment_reference": "00001", "user_id": 253
      }
    ]
  }
}
```

#### Delete a payment

Removes the payment and its explanations.

```http
DELETE /v1/payments/1
```

Response: `204 No Content`

#### List payments

Filters: `account_id`, `patient_id`, `payment_plan_id`, `practitioner_id`, `site_id`, `dated_after`, `dated_before`. Look payments up by patient rather than account where you can.

```http
GET /v1/payments?dated_after=2015-01-01&patient_id=5106
```

```json
{
  "payments": [
    {
      "id": 1, "account_id": 4511, "amount": "18.5", "amount_unexplained": "0.0",
      "dated_on": "2015-01-19", "deleted": false, "fully_explained": true, "method": "Debit Card",
      "patient_id": 5106, "payment_plan_id": 1, "practitioner_id": 1, "reference": "00001",
      "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75", "status": null, "transaction_number": "6932", "user_id": 271,
      "explanations": [
        { "id": 57, "amount": "18.5", "comments": null, "invoice_id": 51, "invoice_reference": "00051", "payment_id": 1, "payment_reference": "00001", "user_id": 253 }
      ]
    }
  ],
  "meta": { "current_page": 1, "total": 3, "total_amount": "854.04" }
}
```

### Payment allocations

Allocations record funds moving between an account, site or practitioner. `transfer_from_type` and `transfer_to_type` name the entity the id refers to.

#### Get a payment allocation

```http
GET /v1/payment_allocations/1f9db5de-b974-49e7-ac84-faee723ce3b1
```

```json
{
  "payment_allocation": {
    "id": "1f9db5de-b974-49e7-ac84-faee723ce3b1", "amount": "23.0", "description": "Payment #04847",
    "invoice_item_id": null, "patient_id": 1330, "payment_explanation_id": null, "reversal_of_id": null,
    "transfer_from_id": 1331, "transfer_from_type": "Account",
    "transfer_to_id": "1bf77f81-5a46-410d-80e3-a6a257d7f328", "transfer_to_type": "Site",
    "created_at": "2021-03-01T12:30:02.720+00:00", "updated_at": "2021-03-01T12:30:02.720+00:00"
  }
}
```

#### List payment allocations

Filters: `transfer_to_id`, `transfer_to_type`, `transfer_from_id`, `transfer_from_type`, `transfer_id` (either side), `transfer_type`, `created_at_before`, `created_at_after`.

```http
GET /v1/payment_allocations?transfer_to_id=2&transfer_to_type=Practitioner
```

```json
{
  "payment_allocations": [
    {
      "id": "1f9db5de-b974-49e7-ac84-faee723ce3b1", "amount": "23.0", "description": "Payment #04847",
      "invoice_item_id": null, "patient_id": 1330, "payment_explanation_id": null, "reversal_of_id": null,
      "transfer_from_id": 1331, "transfer_from_type": "Account",
      "transfer_to_id": 2, "transfer_to_type": "Practitioner",
      "created_at": "2021-03-01T12:30:02.720+00:00", "updated_at": "2021-03-01T12:30:02.720+00:00"
    }
  ],
  "meta": { "total": 108, "current_page": 1, "total_pages": 5 }
}
```

### Payment explanations

An explanation says what a payment was for, and can optionally point at an invoice.

#### Get a payment explanation

The docs give this path in the singular, `payment_explanation`.

```http
GET /v1/payment_explanation/1
```

```json
{
  "explanation": {
    "id": 1, "amount": "100", "comments": "Deposit for treatment",
    "invoice_id": null, "invoice_reference": null, "payment_id": 1, "payment_reference": "0001", "user_id": 1
  }
}
```

#### Create a payment explanation

Both fields are required. `amount` must not exceed the unexplained amount on the payment.

```http
POST /v1/payments/1/explanations/

{ "explanation": { "amount": "100", "comments": "Deposit for treatment" } }
```

```json
{
  "explanation": {
    "id": 1, "amount": "100", "comments": "Deposit for treatment",
    "invoice_id": null, "invoice_reference": null, "payment_id": 1, "payment_reference": "0001", "user_id": 1
  }
}
```

#### Delete a payment explanation

```http
DELETE /v1/payments/1/explanations/1
```

Response: `204 No Content`

### Contracts (NHS)

#### Get a contract

```http
GET /v1/contracts/652e5e2f-1697-417e-8feb-1ecd832bd42d
```

```json
{
  "contract": {
    "id": "652e5e2f-1697-417e-8feb-1ecd832bd42d", "active": true, "contract_number": "12345678910",
    "name": "Contract 1", "start_date": "2015-01-19", "end_date": "2016-01-19",
    "nhs_location_id": "123456", "nhs_site_id": "654321", "notes": "This is a note about a contract",
    "pds_plus": false, "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75",
    "target": "1000", "uda_value": "3", "uoa_target": "300", "uoa_value": "5",
    "created_at": "2015-01-18T14:08:16.523+00:00", "updated_at": "2017-01-03T20:38:44.896+00:00"
  }
}
```

#### List contracts

Filters: `active`, `site_id`. Sort by `created_at`.

```http
GET /v1/contracts?active=true
```

```json
{
  "contracts": [
    {
      "id": "652e5e2f-1697-417e-8feb-1ecd832bd42d", "active": true, "contract_number": "12345678910",
      "name": "Contract 1", "start_date": "2015-01-19", "end_date": "2016-01-19",
      "nhs_location_id": "123456", "nhs_site_id": "654321", "notes": "This is a note about a contract",
      "pds_plus": false, "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75",
      "target": "1000", "uda_value": "3", "uoa_target": "300", "uoa_value": "5",
      "created_at": "2015-01-18T14:08:16.523+00:00", "updated_at": "2017-01-03T20:38:44.896+00:00"
    }
  ],
  "meta": { "total": 3, "page": 1 }
}
```

### NHS claims

`claim_status` is one of `new`, `queued`, `submitted`, `completed`, `error`, `invalid`, `received`, `queried`, `withdrawn`. `care_package` only appears on Welsh claims that have care package data.

#### Get an NHS claim

```http
GET /v1/nhs_claims/5bd1f93d-bfa3-4402-a2fc-5880c5218815
```

```json
{
  "nhs_claim": {
    "id": "5bd1f93d-bfa3-4402-a2fc-5880c5218815", "claim_status": "completed", "sequence_number": 1420,
    "awarded_uda": "12.0", "expected_uda": "12.0", "uda_band": "3", "ortho": false, "continuation_part_number": 1,
    "approval_date": "2012-07-28", "submitted_date": "2012-07-27", "nhs_updated_at": "2012-07-27T11:24:00.000+00:00",
    "status_comments": "UDA 12.00 positive, Employment Support",
    "contract_id": "652e5e2f-1697-417e-8feb-1ecd832bd42d", "patient_id": 1, "practitioner_id": 1,
    "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75", "treatment_plan_id": 1,
    "dentist_charge": "0.0", "awarded_dentist_charge": "0.0", "patient_charge": "53.9",
    "ni_calculated_dentist_fee": "0.0", "ni_calculated_patient_fee": "0.0",
    "scot_amount_authorised": "0.0", "scot_amount_expected": "0.0",
    "care_package": { "packages": [ { "code": "9212", "name": "Routine Examination" } ] },
    "created_at": "2012-07-25T08:54:05.000+00:00", "updated_at": "2012-07-27T11:24:00.000+00:00"
  }
}
```

#### List NHS claims

Filters: `created_after`, `updated_after`, `site_id` (through the claim's practitioner). Sort by `created_at`.

```http
GET /v1/nhs_claims?site_id=e1d854af-15db-4482-8e29-39e74a6c7f75&created_after=2012-01-01
```

```json
{
  "nhs_claims": [
    {
      "id": "5bd1f93d-bfa3-4402-a2fc-5880c5218815", "claim_status": "completed", "sequence_number": 1420,
      "awarded_uda": "12.0", "expected_uda": "12.0", "uda_band": "3", "ortho": false,
      "contract_id": "652e5e2f-1697-417e-8feb-1ecd832bd42d", "patient_id": 1, "practitioner_id": 1,
      "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75", "treatment_plan_id": 1,
      "patient_charge": "53.9", "dentist_charge": "0.0",
      "created_at": "2012-07-25T08:54:05.000+00:00", "updated_at": "2012-07-27T11:24:00.000+00:00"
    }
  ],
  "meta": { "total": 3, "page": 1 }
}
```

## Practice structure

A practice has many sites and users. A practitioner is a user who sees patients, and is tied to a site. `role` is one of `Administrator`, `Dentist`, `Hygienist`, `Nurse`, `Practice Manager`, and `permission_level` runs from 0 to 4.

### Practice

#### Get the practice

Returns the practice of the authenticated user.

```http
GET /v1/practice
```

```json
{
  "practice": {
    "id": "c687bd0b-b35c-44ac-81d5-aebc3b9310a6", "name": "Dentally Dental", "slug": "dentalpractice",
    "address_line_1": "126 The Crescent", "address_line_2": "", "town": "Pontyberem", "postcode": "W1A 1AA",
    "iso_country_code": "GB", "time_zone": "London", "nhs": true,
    "email_address": "hello@dentalpractice.com", "patient_email_address": "dentalpractice@dentallymail.co.uk",
    "phone_number": "020134848228", "website": "dentalpractice.com",
    "logo_url": "https://dentally.co/api-practice-logo.png",
    "custom_patient_field_label_1": "Insurance Number", "custom_patient_field_label_2": null,
    "medical_history_expiry_days": 365,
    "opening_hours": {
      "Monday": { "open": "09:00", "close": "17:00" }, "Tuesday": { "open": "09:00", "close": "17:00" },
      "Wednesday": { "open": "09:00", "close": "17:00" }, "Thursday": { "open": "09:00", "close": "17:00" },
      "Friday": { "open": "09:00", "close": "17:00" }
    }
  }
}
```

### Sites

#### Get a site

```http
GET /v1/sites/e1d854af-15db-4482-8e29-39e74a6c7f75
```

```json
{
  "site": {
    "id": "e1d854af-15db-4482-8e29-39e74a6c7f75", "active": true, "practice_id": "c687bd0b-b35c-44ac-81d5-aebc3b9310a6",
    "name": "Dentally Dental", "nickname": "Dentaly", "default_payment_plan_id": 426,
    "address_line_1": "126 The Crescent", "address_line_2": "", "town": "Pontyberem", "postcode": "W1A 1AA",
    "email_address": "site@dentalpractice.com", "phone_number": "020134848228", "website": "dentalpractice.com",
    "logo_url": "https://dentally.co/api-practice-logo.png",
    "opening_hours": {
      "Monday": { "open": "09:00", "close": "17:00" }, "Tuesday": { "open": "09:00", "close": "17:00" },
      "Wednesday": { "open": "09:00", "close": "17:00" }, "Thursday": { "open": "09:00", "close": "17:00" },
      "Friday": { "open": "09:00", "close": "17:00" }
    }
  }
}
```

#### Edit a site

Editable: `address_line_1`, `address_line_2`, `default_payment_plan_id`, `email_address`, `name`, `nickname`, `phone_number`, `postcode`, `town`, `website`.

```http
PATCH /v1/sites/e1d854af-15db-4482-8e29-39e74a6c7f75

{ "site": { "nickname": "Site One" } }
```

```json
{
  "site": {
    "id": "e1d854af-15db-4482-8e29-39e74a6c7f75", "active": true, "practice_id": "c687bd0b-b35c-44ac-81d5-aebc3b9310a6",
    "name": "Dentally Dental", "nickname": "Site One", "default_payment_plan_id": 426,
    "address_line_1": "126 The Crescent", "address_line_2": "", "town": "Pontyberem", "postcode": "W1A 1AA",
    "email_address": "site@dentalpractice.com", "phone_number": "020134848228", "website": "dentalpractice.com",
    "logo_url": "https://dentally.co/api-practice-logo.png",
    "opening_hours": { "Monday": { "open": "09:00", "close": "17:00" }, "Friday": { "open": "09:00", "close": "17:00" } }
  }
}
```

#### List sites

No parameters.

```http
GET /v1/sites
```

```json
{
  "sites": [
    {
      "id": "e1d854af-15db-4482-8e29-39e74a6c7f75", "active": true, "practice_id": "c687bd0b-b35c-44ac-81d5-aebc3b9310a6",
      "name": "Dentally Dental", "nickname": "Site One", "default_payment_plan_id": 426,
      "address_line_1": "126 The Crescent", "address_line_2": "", "town": "Pontyberem", "postcode": "W1A 1AA",
      "phone_number": "020134848228", "website": "dentalpractice.com",
      "logo_url": "https://dentally.co/api-practice-logo.png",
      "opening_hours": { "Monday": { "open": "09:00", "close": "17:00" }, "Friday": { "open": "09:00", "close": "17:00" } }
    }
  ],
  "meta": { "total": 2, "page": 1 }
}
```

### Users

#### Get the authenticated user

```http
GET /v1/user
```

```json
{
  "user": {
    "id": 1, "email": "user@dentalpractice.com", "title": "Mx", "first_name": "Gregory", "middle_name": null,
    "last_name": "Hays", "mobile_phone": null, "role": "Dentist", "permission_level": 4,
    "practice_id": "c687bd0b-b35c-44ac-81d5-aebc3b9310a6", "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75",
    "allowed_sites": ["e1d854af-15db-4482-8e29-39e74a6c7f75"],
    "image_url": "https://www.gravatar.com/avatar/0242720e04226d4d190f9ffc15a2644b.jpg?&r=pg&d=identicon&s=100",
    "last_login": "2015-01-02T18:16:50.235+00:00",
    "created_at": "2015-01-01T12:02:52.402+00:00", "updated_at": "2015-01-02T12:02:52.402+00:00"
  }
}
```

#### Get a user

```http
GET /v1/users/1
```

```json
{
  "user": {
    "id": 1, "email": "user@dentalpractice.com", "title": "Mx", "first_name": "Gregory", "middle_name": null,
    "last_name": "Hays", "mobile_phone": null, "role": "Dentist", "permission_level": 4,
    "practice_id": "c687bd0b-b35c-44ac-81d5-aebc3b9310a6", "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75",
    "allowed_sites": ["e1d854af-15db-4482-8e29-39e74a6c7f75"],
    "image_url": "https://www.gravatar.com/avatar/0242720e04226d4d190f9ffc15a2644b.jpg?&r=pg&d=identicon&s=100",
    "last_login": "2015-01-02T18:16:50.235+00:00",
    "created_at": "2015-01-01T12:02:52.402+00:00", "updated_at": "2015-01-02T12:02:52.402+00:00"
  }
}
```

#### Edit a user

Editable: `first_name`, `middle_name`, `last_name`, `mobile_phone`, `role`, `permission_level`.

```http
PATCH /v1/users/1

{ "user": { "first_name": "Greg" } }
```

```json
{
  "user": {
    "id": 1, "email": "user@dentalpractice.com", "title": "Mx", "first_name": "Greg", "middle_name": null,
    "last_name": "Hays", "mobile_phone": null, "role": "Dentist", "permission_level": 4,
    "practice_id": "c687bd0b-b35c-44ac-81d5-aebc3b9310a6", "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75",
    "allowed_sites": ["e1d854af-15db-4482-8e29-39e74a6c7f75"],
    "last_login": "2015-01-02T18:16:50.235+00:00",
    "created_at": "2015-01-01T12:02:52.402+00:00", "updated_at": "2026-10-14T12:34:00.000+00:00"
  }
}
```

#### List users

Filters: `created_after`, `updated_after`. Ordered by id.

```http
GET /v1/users?updated_after=2015-01-01
```

```json
{
  "users": [
    {
      "id": 1, "email": "user@dentalpractice.com", "title": "Mx", "first_name": "Greg", "last_name": "Hays",
      "role": "Dentist", "permission_level": 4,
      "practice_id": "c687bd0b-b35c-44ac-81d5-aebc3b9310a6", "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75",
      "allowed_sites": ["e1d854af-15db-4482-8e29-39e74a6c7f75"],
      "created_at": "2015-01-01T12:02:52.402+00:00", "updated_at": "2015-01-02T12:02:52.402+00:00"
    }
  ],
  "meta": { "total": 3, "page": 1 }
}
```

### Practitioners

#### Get a practitioner

```http
GET /v1/practitioners/1
```

```json
{
  "practitioner": {
    "id": 1, "active": true, "colour": "#9CCB3B", "gdc_number": null, "nhs_number": null,
    "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75",
    "default_contract_id": "8aee9505-a6de-4517-87fb-764790fc9b04",
    "contract_targets": [
      { "id": 1, "contract_id": "8aee9505-a6de-4517-87fb-764790fc9b04", "uda_target": 1, "uoa_target": 1, "default": true }
    ],
    "specialisms": [
      { "id": "304bc656-09e5-4094-82fd-ca6735c627a9", "name": "Oral Medicine", "category": "certified", "created_at": "2026-01-30T14:08:53.775+00:00" }
    ],
    "user": {
      "id": 4, "email": "user@dentalpractice.com", "title": "Mx", "first_name": "Greg", "middle_name": null,
      "last_name": "Hays", "mobile_phone": null, "role": "Dentist", "permission_level": 4,
      "image_url": "https://www.gravatar.com/avatar/0242720e04226d4d190f9ffc15a2644b.jpg?&r=pg&d=identicon&s=100",
      "last_login": "2015-01-05T11:03:01.020+00:00",
      "created_at": "2013-11-05T11:26:52.904+00:00", "updated_at": "2015-01-02T12:02:52.402+00:00"
    }
  }
}
```

#### Edit a practitioner

Editable: `active`, `colour` (hex), `gdc_number`, `nhs_number`.

```http
PATCH /v1/practitioners/1

{ "practitioner": { "gdc_number": "12345" } }
```

```json
{
  "practitioner": {
    "id": 1, "active": true, "colour": "#9CCB3B", "gdc_number": "12345", "nhs_number": null,
    "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75",
    "default_contract_id": "8aee9505-a6de-4517-87fb-764790fc9b04",
    "contract_targets": [
      { "id": 1, "contract_id": "8aee9505-a6de-4517-87fb-764790fc9b04", "uda_target": 1, "uoa_target": 1, "default": true }
    ],
    "specialisms": [],
    "user": { "id": 4, "email": "user@dentalpractice.com", "first_name": "Greg", "last_name": "Hays", "role": "Dentist", "permission_level": 4 }
  }
}
```

#### List practitioners

Filters: `site_id` (a single id, or an array with `site_id[]`), `created_after`, `updated_after`. Ordered by id.

```http
GET /v1/practitioners?site_id[]=8af3d10a-9c1d-415f-941e-2094d45dc08a&site_id[]=e1d854af-15db-4482-8e29-39e74a6c7f75
```

```json
{
  "practitioners": [
    {
      "id": 1, "active": true, "colour": "#9CCB3B", "gdc_number": null, "nhs_number": null,
      "site_id": "e1d854af-15db-4482-8e29-39e74a6c7f75",
      "default_contract_id": "8aee9505-a6de-4517-87fb-764790fc9b04",
      "contract_targets": [
        { "id": 1, "contract_id": "8aee9505-a6de-4517-87fb-764790fc9b04", "uda_target": 1, "uoa_target": 1, "default": true }
      ],
      "user": { "id": 4, "email": "user@dentalpractice.com", "first_name": "Greg", "last_name": "Hays", "role": "Dentist", "permission_level": 4 }
    }
  ],
  "meta": { "total": 2, "page": 1 }
}
```

## Webhooks and rate limit

Webhooks push events to your HTTPS endpoint (plain HTTP is allowed in the sandbox only). Supported events: `appointment.created`, `appointment.updated`, `appointment.deleted`, `patient.created`, `patient.updated`, `patient.deleted`, `payment.created`, `payment.updated`, `payment.deleted`.

### Webhook endpoints

#### Create a webhook

All fields are sent in the `webhook` object. `secret` signs each delivery.

```http
POST /v1/webhooks

{
  "webhook": {
    "active": true, "name": "My example webhook", "url": "https://example.com/webhooks",
    "events": ["patient.created", "patient.updated", "patient.deleted"],
    "secret": "pLjUpNQu8a4YjeCrf2KseA"
  }
}
```

```json
{
  "webhook": {
    "id": 2, "active": true, "name": "My example webhook", "url": "https://example.com/webhooks",
    "events": ["patient.created", "patient.updated", "patient.deleted"],
    "failed_deliveries": 0, "successful_deliveries": 0, "last_delivered_at": null
  }
}
```

#### Get a webhook

```http
GET /v1/webhooks/1
```

```json
{
  "webhook": {
    "id": 1, "active": true, "name": "My example webhook", "url": "https://example.com/webhooks",
    "events": ["appointment.created", "appointment.updated", "appointment.deleted"],
    "failed_deliveries": 0, "successful_deliveries": 343, "last_delivered_at": "2017-12-13T12:34:56.000+00:00"
  }
}
```

#### Delete a webhook

```http
DELETE /v1/webhooks/1
```

Response: `204 No Content`

#### List webhooks

The docs show each list item wrapped in its own `webhook` key, reproduced here.

```http
GET /v1/webhooks
```

```json
{
  "webhooks": [
    {
      "webhook": {
        "id": 1, "active": true, "name": "My example webhook", "url": "https://example.com/webhooks",
        "events": ["appointment.created", "appointment.updated", "appointment.deleted"],
        "failed_deliveries": 0, "successful_deliveries": 343, "last_delivered_at": "2017-12-13T12:34:56.000+00:00"
      }
    }
  ],
  "meta": { "current_page": 1, "total": 3 }
}
```

### What Dentally sends to your endpoint

A `POST` with a JSON body holding the changed object under `data`, the `event` name, the object type and the user who triggered it. Your handler must return a 2xx status. Other codes trigger up to 10 retries at growing intervals, and the webhook is disabled after more than 50 failed deliveries. Deliveries can arrive out of order. Unknown events should be ignored with `204`.

```http
POST https://example.com/webhooks
User-Agent: Dentally-Webhook
Content-Type: application/json
X-Dentally-Signature: 63e0b2c79e8b73993777ce312d6b3ad421742d9a8f2657a3e3c14909c6578ab6

{
  "event": "appointment.created", "object": "appointment",
  "data": {
    "id": 2181, "patient_id": 1, "patient_name": "Carolina James", "practitioner_id": 1,
    "payment_plan_id": 1, "reason": "Exam", "state": "Pending", "duration": 30,
    "start_time": "2015-07-08 09:30:00 +0100", "finish_time": "2015-07-08 10:00:00 +0100",
    "pending_at": "2015-07-01 12:00:00 +0100", "notes": null, "treatment_description": null, "user_id": 1
  },
  "user": {
    "id": 1, "email": "user@dentalpractice.com", "first_name": "Gregory", "last_name": "Hays",
    "role": "Dentist", "permission_level": 4
  }
}
```

The signature is an HMAC SHA256 hex digest of the raw request body, keyed with the webhook's `secret`. Check it before processing. A minimal check in Python:

```python
import hmac, hashlib

def is_valid(raw_body: bytes, secret: str, header: str) -> bool:
    expected = hmac.new(secret.encode(), raw_body, hashlib.sha256).hexdigest()
    return hmac.compare_digest(expected, header)
```

### Rate limit status

#### Check your rate limit

Does not count against your limit. Note the path has no `/v1` prefix in the docs.

```http
GET /rate_limit
```

```json
{
  "resources": {
    "core": { "limit": 3600, "remaining": 3595, "reset": 1494590400 },
    "appointment_availability": { "limit": 200, "remaining": 190, "reset": 1494590400 }
  }
}
```

Every response also carries `X-RateLimit-Limit`, `X-RateLimit-Remaining` and `X-RateLimit-Reset` (UTC epoch seconds) headers.

## Schema diagrams

Five views of the related entities, split by theme so each stays readable. Each arrow runs from the parent (one side) to the child (many side) and is labelled with the foreign key. Colours: blue = patient records, orange = scheduling, green = treatment, purple = billing, grey = practice structure. Relationships are inferred from the `*_id` fields in the API docs, since the docs publish no formal ERD.

### 1. Patients and their records

&#91;image: Patients and their records\]

### 2. Scheduling

&#91;image: Scheduling\]

### 3. Treatment, fees and NHS claims

&#91;image: Treatment, fees and NHS claims\]

### 4. Billing and payments

&#91;image: Billing and payments\]

### 5. Practice structure

&#91;image: Practice structure\]

**Notes on the diagrams**

- Payment allocations use polymorphic `transfer_from` and `transfer_to` (id plus type), so they have no single FK target.
- `custom_fields` are polymorphic and attach to several resources, so they are not drawn.
- Appointments reference cancellation reasons by id, but appointment reasons are matched by name rather than id.
- Webhooks are standalone and have no FK relationships to the other entities.
- Only the key fields of each entity are shown. The endpoint samples above list the full field sets.

## Gaps in the source docs

The samples are built from the published examples and attribute tables, not from live calls. The source has several inconsistencies, so test the shapes below against the sandbox before relying on them.

| Where | What the docs show | What this document does |
| --- | --- | --- |
| Get a contract | Response is wrapped in `nhs_claim` | Wrapped in `contract` |
| Get a waiting list | An array of one under `waiting_list` | A single object |
| Sundries (get and list) | Both return an array under the key `sundry` | Reproduced as documented |
| Waiting list entries | Path shows `/{WAITING_LIST_ID}`, example uses `?waiting_list_id=` | Query parameter |
| Payment explanation (get) | Singular path `/v1/payment_explanation/{id}`, while create and delete use `/v1/payments/{id}/explanations` | Reproduced as documented |
| Payment allocations list | Example URL has a typo, `payment_allocationss` | `payment_allocations` |
| Webhooks list | Each item is wrapped in its own `webhook` key | Reproduced as documented |
| NHS claim | Example `claim_status` is `complete` but the listed values say `completed`. `created_at` is malformed | `completed` and a valid timestamp |
| Create a payment plan | Example response is `NHS`, `active: false`, id 1 for a request that creates "Custom Plan" | Response matches the request and the documented defaults |
| Appointments | `uuid` and `booked_via_api` are in the attribute table but missing from most examples | Included |
| Invoices and items | Tables type amounts as integer and ids as uuid, but examples use strings and numeric ids. `status` appears in examples only | Followed the examples |
| Patients | `marketing` and `medical_alert` are typed integer in the parameter tables but are booleans in responses | Booleans |
| Appointment availability | Example dates are in 2015, but start and finish must be in the future | Dates moved to October 2026 |
| Treatment category | The object example has no wrapper, but get and list do | Wrapped |

Also worth knowing: the `/rate_limit` endpoint has no `/v1` prefix, and the Practitioner list takes either `site_id` or `site_id[]`.
