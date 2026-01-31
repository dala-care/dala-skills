---
name: dala-care-api
description: Manage home care operations - schedule visits, coordinate caregivers, track care recipients, and streamline workforce management with Dala Care
metadata:
  author: careverk
  version: "1.0.0"
  skillport:
    category: api
    tags: [healthcare, graphql, api, scheduling, workforce-management]
---

# Dala Care

Dala Care is a healthcare workforce management platform that helps home care agencies coordinate caregivers, manage care recipients, and optimize scheduling. This skill enables you to automate care operations, access real-time data, and integrate Dala Care into your workflows.

## What You Can Do

- **Schedule & Manage Visits**: Create one-time or recurring visits, check caregiver availability, and modify schedules
- **Track Care Recipients**: Access client information, care plans, contacts, and visit history
- **Coordinate Caregivers**: View caregiver profiles, availability, and assignments
- **Monitor Operations**: Get daily schedules, identify open visits, and track visit status
- **Update Care Plans**: Modify care recipient information and care plan details

---

## Managing Your Organization

### View Your Offices

Access all offices in your organization, including their regional structure.

```graphql
query Offices {
  offices {
    id
    name
    type          # OFFICE or CARE_HOME
    region {
      id
      name
      division {
        id
        name
      }
    }
  }
}
```

---

## Working with Care Recipients

### List Care Recipients

View all care recipients across your offices or filter by specific locations.

```graphql
query CareRecipients($officeIds: [String!]!) {
  careRecipients(filter: { officeIds: $officeIds }) {
    id
    firstName
    lastName
    gender          # MALE, FEMALE, OTHER
    email
    phone
    homePhone
    careRecipientRoles {
      deactivatedAt
      office {
        id
        name
      }
    }
  }
}
```

**Variables:**
```json
{
  "officeIds": []
}
```

---

### View Care Recipient Profile

Access complete client information including contacts, address, care plan, and files.

```graphql
query CareRecipientById($id: ID!) {
  careRecipientById(id: $id) {
    id
    firstName
    lastName
    pid
    birthDate
    phone
    homePhone
    email
    gender

    address {
      addressLine1
      addressLine2
      city
      state
      zipCode
      homeInformation
    }

    contacts {
      id
      firstName
      lastName
      relationshipType  # SPOUSE, PARENT, CHILD, SIBLING, etc.
      email
      phone
      isPrimary
      isPayer
      hasPowerOfAttorney
    }

    primaryContact {
      id
      firstName
      lastName
      phone
      email
    }

    careRecipientRoles {
      deactivatedAt
      office {
        id
        name
        type
      }
    }

    carePlan {
      fields {
        ... on RadioField {
          __typename
          id
          name
          description
          availableValues
          value
          required
        }
        ... on CheckboxField {
          __typename
          id
          name
          description
          availableValues
          values
          required
        }
        ... on TextField {
          __typename
          id
          name
          description
          value
          required
        }
        ... on TextArea {
          __typename
          id
          name
          description
          value
          required
        }
        ... on DateField {
          __typename
          id
          name
          description
          value
          required
        }
      }
    }

    files {
      name
      url
    }
  }
}
```

---

### View Visit History

See all visits for a care recipient within a specific time period.

```graphql
query CareRecipientVisits($careRecipientId: ID!, $from: String!, $to: String!) {
  visitsByCareRecipientId(
    careRecipientId: $careRecipientId
    from: $from
    to: $to
  ) {
    id
    visitDefinitionId
    officeId
    subjectId
    subjectType       # CARE_RECIPIENT or CARE_HOME
    start
    durationMinutes
    cancelledAt
    cancelledReason   # CANCELLED_BY_CARE_RECIPIENT, CANCELLED_BY_CARE_PROVIDER, etc.

    visitorsV2 {
      id
      firstName
      lastName
      clockInTime
      clockOutTime
    }

    visitType {
      id
      title
      code
    }

    labels {
      id
      name
    }

    visitNotes {
      id
      note
      createdAt
      sentiment {
        positive
        negative
        neutral
        mixed
      }
      author {
        id
        firstName
        lastName
      }
    }
  }
}
```

**Variables:**
```json
{
  "careRecipientId": "uuid",
  "from": "2025-01-01T00:00:00.000Z",
  "to": "2025-01-31T23:59:59.999Z"
}
```

---

## Managing Caregivers

### View Caregivers

Access caregiver profiles across your offices.

```graphql
query CareGivers($officeIds: [String!]!) {
  careGivers(filter: { officeIds: $officeIds }) {
    id
    firstName
    lastName
    email
    phone
    homePhone
    birthDate
    pid
    lastLogin
    caregiverRoles {
      userId
      hireDate
      deactivatedAt
      office {
        id
        name
      }
    }
  }
}
```

---

## Scheduling & Daily Operations

### View Daily Schedule

See all visits and caregiver absences for your offices on any given day.

```graphql
query EventsByOfficeIds($officeIds: [ID!]!, $from: String!, $to: String!) {
  eventsByOfficeIds(officeIds: $officeIds, from: $from, to: $to) {
    ... on Visit {
      id
      visitDefinitionId
      officeId
      subjectId
      subjectType
      start
      durationMinutes
      isVisitLog
      cancelledAt
      cancelledReason
      requiredCaregiversCount

      visitorsV2 {
        id
        firstName
        lastName
        clockInTime
        clockOutTime
      }

      subject {
        ... on CareRecipient {
          id
          firstName
          lastName
        }
        ... on CareHome {
          id
          name
        }
      }

      visitType {
        id
        title
        code
      }

      labels {
        id
        name
      }
    }
    ... on Absence {
      id
      userId
      start
      end
      allDay
      reason
      description
    }
  }
}
```

**Visit Status Logic:**
- **Open:** `requiredCaregiversCount > visitorsV2.length` and not cancelled
- **Scheduled:** Future visit with sufficient caregivers assigned
- **Late:** `start + durationMinutes < now` and has `clockInTime` but no `clockOutTime`
- **Missed:** Past visit with no `clockInTime` on any visitor
- **Cancelled:** Has `cancelledAt` value
- **Completed:** `isVisitLog = true` or has `clockOutTime` on visitors

---

### Check Caregiver Availability

Find which caregivers are available for a proposed visit time, including recurring schedules.

```graphql
query CaregiverAvailabilities($officeIds: [ID!]!, $input: VisitorAvailabilityInput!) {
  caregiverAvailabilitiesByOfficeIds(officeIds: $officeIds, input: $input) {
    visitorId
    visitor {
      id
      firstName
      lastName
      email
      phone
    }
    firstVisit         # Available for the first occurrence
    allVisits          # Available for all occurrences (if recurring)
    conflictFirstVisit # Has conflict on first occurrence
    conflictInSeries   # Has conflict somewhere in series
  }
}
```

**Variables:**
```json
{
  "officeIds": ["office-uuid"],
  "input": {
    "start": "2025-01-15T09:00:00.000Z",
    "durationMinutes": 60,
    "end": "2025-03-15T09:00:00.000Z",
    "rRule": {
      "frequency": "WEEKLY",
      "weekdays": ["MONDAY", "WEDNESDAY", "FRIDAY"],
      "interval": 1,
      "weekStart": "MONDAY"
    }
  }
}
```

---

## Creating & Updating Visits

### Schedule a Visit

Create one-time or recurring visits for care recipients.

```graphql
mutation ScheduleVisit($input: VisitCreateInputV2!) {
  visitCreateV2(input: $input) {
    id
    subjectId
    subjectType
    recurrences {
      id
      start
      end
      durationMinutes
      visitorIds
      visitors {
        id
        firstName
        lastName
      }
      rRule {
        frequency
        weekdays
        interval
        weekStart
      }
    }
  }
}
```

**Variables for one-time visit:**
```json
{
  "input": {
    "subjectId": "care-recipient-uuid",
    "subjectType": "CARE_RECIPIENT",
    "officeId": "office-uuid",
    "start": "2025-01-15T09:00:00.000Z",
    "durationMinutes": 60,
    "visitorIds": ["caregiver-uuid-1", "caregiver-uuid-2"],
    "requiredCaregiversCount": 1
  }
}
```

**Variables for recurring visit:**
```json
{
  "input": {
    "subjectId": "care-recipient-uuid",
    "subjectType": "CARE_RECIPIENT",
    "officeId": "office-uuid",
    "start": "2025-01-15T09:00:00.000Z",
    "end": "2025-06-15T09:00:00.000Z",
    "durationMinutes": 60,
    "visitorIds": ["caregiver-uuid"],
    "requiredCaregiversCount": 1,
    "rRule": {
      "frequency": "WEEKLY",
      "weekdays": ["MONDAY", "WEDNESDAY", "FRIDAY"],
      "interval": 1,
      "weekStart": "MONDAY"
    }
  }
}
```

---

### Modify an Upcoming Visit

Change details for a scheduled visit that hasn't happened yet.

```graphql
mutation UpdateVisit($input: VisitUpdateInput!) {
  visitUpdate(input: $input) {
    id
    visitDefinitionId
    officeId
    subjectId
    start
    durationMinutes
    requiredCaregiversCount
    visitorsV2 {
      id
      firstName
      lastName
    }
  }
}
```

**Variables:**
```json
{
  "input": {
    "visitInstanceId": "visit-uuid",
    "visitorIds": ["caregiver-uuid-1", "caregiver-uuid-2"],
    "durationMinutes": 90,
    "start": "2025-01-15T10:00:00.000Z",
    "requiredCaregiversCount": 2
  }
}
```

---

### Edit a Completed Visit

Update details for visits that have already occurred.

```graphql
mutation UpdateVisitLog($visitInstanceId: ID!, $input: VisitLogUpdateInput!) {
  visitLogUpdate(visitInstanceId: $visitInstanceId, input: $input) {
    id
    visitDefinitionId
    officeId
    subjectId
    start
    durationMinutes
    visitorsV2 {
      id
      firstName
      lastName
    }
  }
}
```

**Variables:**
```json
{
  "visitInstanceId": "visit-uuid",
  "input": {
    "visitors": [
      {
        "id": "caregiver-uuid",
        "clockInTime": "2025-01-15T09:00:00.000Z",
        "clockOutTime": "2025-01-15T10:00:00.000Z"
      }
    ],
    "startDate": "2025-01-15T09:00:00.000Z",
    "durationMinutes": 60
  }
}
```

---

### Update Care Plans

Modify care plan information for your care recipients.

```graphql
mutation CareRecipientCarePlanUpsert($careRecipientId: String!, $input: CarePlanInput!) {
  carePlanForCareRecipientUpsert(careRecipientId: $careRecipientId, input: $input) {
    id
  }
}
```

**Variables:**
```json
{
  "careRecipientId": "care-recipient-uuid",
  "input": {
    "fields": [
      {
        "id": "field-id-1",
        "type": "RADIO_FIELD",
        "value": "Option A"
      },
      {
        "id": "field-id-2",
        "type": "CHECKBOX_FIELD",
        "value": "Option 1,Option 3"
      },
      {
        "id": "field-id-3",
        "type": "TEXT_FIELD",
        "value": "Some text value"
      },
      {
        "id": "field-id-4",
        "type": "DATE_FIELD",
        "value": "2025-01-15"
      }
    ]
  }
}
```

**Field Types:**
- `RADIO_FIELD`: Single value from `availableValues`
- `CHECKBOX_FIELD`: Comma-separated values from `availableValues`
- `TEXT_FIELD`: Free text (short)
- `TEXT_AREA`: Free text (long)
- `DATE_FIELD`: Date string

---

### Cancel a Visit

```graphql
mutation CancelVisit(
  $visitInstanceId: ID!
  $cancelledAt: String!
  $cancelledReason: CancelledReason!
  $cancelFollowing: Boolean!
  $cancelledAdditionalInformation: String
) {
  visitCancel(
    visitInstanceId: $visitInstanceId
    cancelledAt: $cancelledAt
    cancelledReason: $cancelledReason
    cancelFollowing: $cancelFollowing
    cancelledAdditionalInformation: $cancelledAdditionalInformation
  ) {
    id
    cancelledAt
    cancelledReason
  }
}
```

**Variables:**
```json
{
  "visitInstanceId": "visit-uuid",
  "cancelledAt": "2025-01-15T08:00:00.000Z",
  "cancelledReason": "CANCELLED_BY_CARE_RECIPIENT",
  "cancelFollowing": false,
  "cancelledAdditionalInformation": "Patient hospitalized"
}
```

**Cancellation Reasons:**
- `CANCELLED_BY_CARE_RECIPIENT`
- `CANCELLED_BY_CARE_PROVIDER`
- `HOSPITAL_CARE_FACILITY`
- `HOLIDAY`
- `RESCHEDULED`
- `OTHER`

---

## Reference Types

### Recurrence (rRule)

```graphql
input RRuleInput {
  frequency: Frequency!    # DAILY, WEEKLY, MONTHLY
  weekdays: [Weekday!]     # Required for WEEKLY
  interval: Int            # Every N periods (default: 1)
  weekStart: Weekday!      # MONDAY, TUESDAY, etc.
}
```

### Weekdays

`MONDAY`, `TUESDAY`, `WEDNESDAY`, `THURSDAY`, `FRIDAY`, `SATURDAY`, `SUNDAY`

### Relationship Types

`SPOUSE`, `PARENT`, `CHILD`, `SIBLING`, `IN_LAW`, `GRANDPARENT`, `GRANDCHILD`, `FAMILY`, `FRIEND`, `NEIGHBOR`, `MEDICAL`, `THERAPY`, `OTHER`

### Absence Reasons (Caregiver)

`PAID_LEAVE`, `UNPAID_LEAVE`, `PAID_SICK_LEAVE`, `UNPAID_SICK_LEAVE`, `SICK_CHILD`, `UNPAID_SICK_CHILD`, `TRAINING`, `OTHER`

### Absence Reasons (Care Recipient)

`CANCELLED_BY_CARE_RECIPIENT`, `CANCELLED_BY_CARE_PROVIDER`, `HOSPITAL_CARE_FACILITY`, `HOLIDAY`, `OTHER`

### Role Types

- `ADMIN`: Full administrative access
- `SCHEDULING_MANAGER`: Manage schedules and visits
- `SUCCESS_MANAGER`: Manage care recipients and caregivers
- `CAREGIVER`: Field worker providing care
- `CARE_RECIPIENT`: Person receiving care
- `CONTACT`: Family member or emergency contact

### Resource Hierarchy

```
Tenant (organization)
  └── Division
        └── Region
              └── Office
                    └── Care Recipients & Caregivers
```

---

## Common Workflows

### Scheduling a New Visit

1. View your offices to identify the location
2. Find the care recipient who needs the visit
3. Check which caregivers are available at the proposed time
4. Create the visit with your selected caregiver(s)

### Finding Open Shifts

1. Pull the daily schedule for your target date range
2. Look for visits where more caregivers are needed than currently assigned
3. These open visits can then be filled by assigning available caregivers

### Reassigning a Visit

1. Check if the visit is upcoming or already completed
2. For upcoming visits: modify the assigned caregivers
3. For completed visits: update the visit log with actual caregiver attendance

### Updating Client Information

1. Access the care recipient's profile to see current care plan fields
2. Update the fields with new information following the specified field types
3. Save changes to keep care plans current

---

## Technical Reference

### Authentication

Authenticate using your API key in the `Authorization` header. Your API key determines which offices and resources you can access based on your role permissions.

### Regional Endpoints

- **EU:** `https://api.eu.dala.care/graphql`
- **US:** `https://api.us.dala.care/graphql`

### Date Format

All dates use ISO 8601 format: `2025-01-15T08:00:00.000Z`
