# Test Cases

## Case 1 — Successful booking

Input:

```json
{
  "patientId": "P001",
  "patientName": "Test Patient",
  "department": "Cardiology",
  "preferredSlot": "10:00 AM",
  "bookedCount": 3
}
```

Expected path:

```text
Booking requested
→ Check slot
→ AVAILABLE
→ Book slot
→ Send confirmation notification
→ Booking flow completed
```

## Case 2 — Slot full

Input:

```json
{
  "patientId": "P002",
  "patientName": "Test Patient",
  "department": "Cardiology",
  "preferredSlot": "10:00 AM",
  "bookedCount": 12
}
```

Expected path:

```text
Booking requested
→ Check slot
→ FULL
→ Send slot-full notification
→ Booking flow completed
```

## Case 3 — Successful booking followed by cancellation

Start with Case 1.

Then publish a message:

```json
{
  "messageName": "booking-cancellation",
  "correlationKey": "P001",
  "variables": {
    "patientId": "P001"
  }
}
```

Expected path:

```text
Handle cancellation
→ Cancellation message
→ Release slot
→ Send cancellation notification
```
