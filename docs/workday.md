# Workday Integration

The [Workday REST API](https://community.workday.com/sites/default/files/file-hosting/restapi/) seems relatively straightforward, but the general flow we'll adopt is...

1. Start with workerId
2. GET worker core record
3. Extract references:
   - locationId
   - managerId
   - supervisoryOrgId
   - costCenterId
   - positionId
4. Follow references for location/org/manager detail
5. Use RaaS or SOAP for non-standard data:
   - certificates
   - NHS / CDA metrics
   - mentoring relationships
   - messaging
   - tenant-specific finance fields
6. Return one normalised mobile-app payload

## Endpoint Overview

The table below shows some common aggregates for data items and respective endpoints. This isn't an exchaustive - or even tested - list, but it is another assumption that _could_ be right!

| Use Case                   | Recommended Endpoint(s)                                                        |
| -------------------------- | ------------------------------------------------------------------------------ |
| My Profile                 | `GET /workers/{workerId}`                                                      |
| My Practice Location       | `GET /jobs/{jobId}/workspace`, `GET /locations/{locationId}`                   |
| My Team                    | `GET /supervisoryOrganizations/{orgId}/members`                                |
| My Manager                 | `GET /workers/{workerId}` or `GET /supervisoryOrganizations/{orgId}`           |
| Org Chart                  | `GET /supervisoryOrganizations/{orgId}/orgChart`                               |
| Skills & Certifications    | `GET /workers/{workerId}/explicitSkills`, `GET /workers/{workerId}/skillItems` |
| Service History            | `GET /workers/{workerId}/serviceDates`                                         |
| Career Information         | `GET /jobProfiles`, `GET /jobFamilies`                                         |
| Office Finder              | `GET /locations`, `GET /locations/{locationId}`                                |
| Business Unit / Department | `GET /organizations`, `GET /supervisoryOrganizations/{orgId}`                  |

## Endpoint Assumptions

### Getting a Worker (practitioner)

```http
GET /workday/api/staffing/v5/{tenant}/workers/{workerId}
Authorization: Bearer {access_token}
Accept: application/json

{
  "id": "abc123workerid",
  "descriptor": "Jane Smith",
  "workerType": {
    "id": "employee",
    "descriptor": "Employee"
  },
  "primaryJob": {
    "jobTitle": "Consultant",
    "businessTitle": "Clinical Consultant",
    "location": {
      "id": "loc_001",
      "descriptor": "London Practice"
    },
    "supervisoryOrganization": {
      "id": "org_123",
      "descriptor": "Digital Health"
    }
  },
  "manager": {
    "id": "worker_mgr_001",
    "descriptor": "Alex Brown"
  },
  "workEmail": "jane.smith@mydentist.co.uk"
}
```

### Getting a Location (practice)

```http
GET /workday/api/staffing/v5/{tenant}/locations/{locationId}
Authorization: Bearer {access_token}
Accept: application/json

{
  "id": "london_practice_001",
  "name": "London Practice",
  "type": "Practice Location",
  "address": {
    "line1": "1 Example Street",
    "city": "London",
    "postalCode": "SW1A 1AA",
    "country": "GB"
  }
}
```

### Getting Jobs

```http
GET /workday/api/staffing/v5/{tenant}/jobs/{jobId}
Authorization: Bearer {access_token}
Accept: application/json

{
  "id": "JOB-12345",
  "descriptor": "Senior Consultant",
  "jobProfile": {
    "id": "JP-001",
    "descriptor": "Senior Consultant"
  },
  "businessTitle": "Principal Consultant",
  "worker": {
    "id": "WK-123456",
    "descriptor": "Jane Smith"
  },
  "workerType": {
    "id": "Employee",
    "descriptor": "Employee"
  },
  "position": {
    "id": "POS-98765",
    "descriptor": "Senior Consultant Position"
  },
  "supervisoryOrganization": {
    "id": "SUP-1001",
    "descriptor": "Consulting Practice"
  },
  "location": {
    "id": "LOC-001",
    "descriptor": "London Practice"
  },
  "timeType": {
    "id": "FullTime",
    "descriptor": "Full Time"
  }
}
```

### Getting Community (Org. Chart)

```http
GET /workday/api/staffing/v5/{tenant}/supervisoryOrganizations/{supervisoryOrgId}
Authorization: Bearer {access_token}
Accept: application/json

{
  "id": "SUP-1001",
  "descriptor": "Consulting Practice",
  "manager": {
    "id": "WK-999999",
    "descriptor": "Alex Brown"
  },
  "organizationType": {
    "id": "SUP",
    "descriptor": "Supervisory Organization"
  },
  "parentOrganization": {
    "id": "SUP-1000",
    "descriptor": "Professional Services"
  }
}
```

---

### Getting Certificates & Learning

TBC...

### Getting Remission/Finance

TBC...
