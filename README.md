# Exercise 8 — Hospital Appointment Booking (Camunda 8)

## Deliverables

This repository contains the complete modeling artifacts requested for Exercise 8:

- Reusable `send-notification` child process
- `slot_decision` DMN with UNIQUE hit policy
- Parent hospital appointment-booking process
- Call Activities with explicit variable mappings
- Event Subprocess for cancellation
- Telegram REST Outbound Connector configuration
- Test cases and expected outcomes
- Operate screenshot checklist

## Project Structure

```text
workflow/
├── parent/
│   └── appointment-booking.bpmn
├── child/
│   └── send-notification.bpmn
└── decisions/
    └── slot_decision.dmn

docs/
└── TEST-CASES.md

screenshots/
└── README.md
```

## Child Process

Process ID: `send-notification`

Flow:

`Start → REST Connector → End`

Inputs:
- `recipientId`
- `messageText`

The Telegram REST request uses:

`{{secrets.TELEGRAM_BOT_TOKEN}}`

The bot token is deliberately not stored in this repository.

## DMN

Decision ID: `slot_decision`

Hit policy: `UNIQUE`

Inputs:
- `department` — String
- `bookedCount` — Number

Outputs:
- `slotStatus`
- `suggestedAction`

The requested Cardiology behavior is:

| Department | bookedCount | slotStatus | suggestedAction |
|---|---:|---|---|
| Cardiology | < 10 | AVAILABLE | BOOK |
| Cardiology | >= 10 | FULL | NOTIFY_FULL |

## Parent Process

The parent process starts with **Booking requested**, evaluates `slot_decision`, and routes through an exclusive gateway.

### AVAILABLE

`Check slot → AVAILABLE → Book slot → Send confirmation notification`

### FULL

`Check slot → FULL → Send slot-full notification`

### Cancellation

A non-interrupting Event Subprocess named **Handle cancellation** listens for the `booking-cancellation` message.

The message is correlated using:

`patientId`

Cancellation flow:

`Cancellation message → Release slot → Send cancellation notification`

The non-interrupting design allows the cancellation path to execute without terminating the main booking scope.

## Variable Mapping

The final model uses explicit mappings on every Call Activity:

```text
recipientId = patientId
messageText = notification-specific FEEL expression
```

Parent-variable propagation to the child is disabled in the final model:

```text
propagateAllParentVariables="false"
```

This demonstrates the controlled approach where the child receives only the variables it needs.

Camunda supports variable mappings on Call Activities and allows parent-variable propagation to be disabled. See the official documentation:
https://docs.camunda.io/docs/components/modeler/bpmn/call-activities/

## Telegram Secret

Create a Camunda secret named:

`TELEGRAM_BOT_TOKEN`

Do not commit the actual bot token to GitHub.

The BPMN references:

`{{secrets.TELEGRAM_BOT_TOKEN}}`

## Test Cases

### 1. Successful booking

```text
department = Cardiology
bookedCount = 3
```

Expected:

`AVAILABLE → Book slot → Confirmation notification`

### 2. Slot full

```text
department = Cardiology
bookedCount = 12
```

Expected:

`FULL → Slot-full notification`

The appointment should not be booked.

### 3. Booking followed by cancellation

First create a successful booking.

Then publish:

```text
messageName = booking-cancellation
correlationKey = patientId
```

Expected:

`Cancellation message → Release slot → Cancellation notification`

## Operate Evidence

Capture screenshots showing:

1. Parent process instance
2. `Check slot` and gateway result
3. Child `send-notification` instance linked from the Call Activity
4. Successful booking path
5. Slot-full path
6. Cancellation Event Subprocess
7. `Release slot`
8. Cancellation notification child instance

## Important

The BPMN/DMN files are the actual submission artifacts. Operate screenshots must be taken from your own Camunda environment after deployment and execution; they should not be fabricated or committed as fake evidence.
