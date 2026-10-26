# Learning Pratice 04 - black box testing

### Reservation Requirements

- A reservation must be between 1 and 31 nights, inclusive.
- A reservation must include between 1 and 12 guests, inclusive.
- The reservation costs $200 per night.
- Every valid reservation includes a one-time $50 cleaning fee.
- Each guest over five costs an additional $25 per night.
- The check-in date must be before the checkout date.
- The number of nights is calculated as `checkout date - check-in date`.
- The checkout date is not counted as a reserved night.
- Every night in the requested date range must be available.
- Invalid reservation information raises a `ValueError`.
- An unavailable reservation raises an `UnavailableError`.
- A new valid reservation begins in the `Pending` state.
- Successful payment moves it from `Pending` to `Confirmed`.
- Failed payment moves it from `Pending` to `Cancelled`.
- A confirmed reservation can move to `Checked In` on its check-in date.
- A checked-in reservation moves to `Completed` after checkout.
- A pending or confirmed reservation may be cancelled.
- A cancelled or completed reservation cannot change states.
- Invalid state transitions raise an `InvalidStateError`.

## Step 1: Identify Conditions and Actions

Review the reservation requirements and identify:

- Five conditions that affect whether a reservation is accepted.
- Five actions that may occur based on those conditions.

### Condition Examples

- If the reservation is between 1 and 31 nights.
- All the dates selected must be available.
- A reservation must have at least 1 and up to 12 guests.
- Check-out date must be after check-in date.
- Payment is successful.

### Action Examples

- Raise a `ValueError` when the nights, guest count, or dates are invalid.
- Raise an `UnavailableError` when any night in the date range is not available.
- Calculate the total cost: $200 × nights + $50 cleaning fee + $25 × (guests − 5) × nights when there are more than 5 guests.
- Create the reservation in the `Pending` state when all input is valid.
- Move the reservation to `Confirmed` if payment succeeds, or to `Cancelled` if payment fails.

## Step 2: Create a Decision Table

| Conditions and Actions         | Rule 1 | Rule 2 | Rule 3 | Rule 4 | Rule 5 | Rule 6 |
| ------------------------------ | :----: | :----: | :----: | :----: | :----: | :----: |
| Dates are valid                |   T    |   T    |   F    |   T    |   T    |   T    |
| Number of nights is valid      |   T    |   T    |   -    |   F    |   T    |   T    |
| Number of guests is valid      |   T    |   T    |   -    |   -    |   F    |   T    |
| Requested nights are available |   T    |   T    |   -    |   -    |   -    |   F    |
| More than five guests          |   F    |   T    |   -    |   -    |   -    |   -    |
| **Create reservation**         |   X    |   X    |        |        |        |        |
| **Calculate standard price**   |   X    |   X    |        |        |        |        |
| **Add extra guest fee**        |        |   X    |        |        |        |        |
| **Raise `ValueError`**         |        |        |   X    |   X    |   X    |        |
| **Raise `UnavailableError`**   |        |        |        |        |        |   X    |

- **Rule 1:** valid reservation, 5 or fewer guests. Every check passes, so reservation is created, cost is standard fee.

- **Rule 2:** valid reservation, more than 5 guests, addtitional fee per guest will be added at checkout.

- **Rule 3:** invalid dates. If check-in isn’t before checkout, system cannot calculate number of nights, therefor raises a ValueError.

- **Rule 4:** dates are valid, but the nights are out of range (0 or more than 31), it raises a ValueError.

- **Rule 5:** dates and nights are valid, but the guest count is out of range (0 or more than 12), it raises a ValueError.

- **Rule 6:** all the information is valid, but at least one night is booked, it raises an UnavailableError.

## Step 3: Create Decision Table Test Cases

### TC01: Valid Reservation With No Extra Guest Fee (Rule 1)

- **Check-in:** June 10
- **Checkout:** June 13
- **Number of guests:** 3
- **Availability:** Every requested night is available.
- **Expected Result:** The reservation is created.
- **Expected Price:** `$650`
  - Three nights at $200: `$600`
  - One-time cleaning fee: `$50`

### TC02: Valid Reservation With Extra Guest Fee (Rule 2)

- **Check-in:** June 10
- **Checkout:** June 13
- **Number of guests:** 8
- **Availability:** Every requested night is available.
- **Expected Result:** The reservation is created.
- **Expected Price:** `$875`
  - Three nights at $200: `$600`
  - One-time cleaning fee: `$50`
  - Three extra guests × $25 × three nights: `$225`

### TC03: Checkout Before Check-In (Rule 3)

- **Check-in:** June 10
- **Checkout:** June 7
- **Number of guests:** 2
- **Availability:** Not checked.
- **Expected Result:** No reservation is created.
- **Expected Exception:** `ValueError`

### TC04: Too Many Nights (Rule 4)

- **Check-in:** June 10
- **Checkout:** July 12
- **Number of guests:** 2
- **Availability:** Not checked.
- **Expected Result:** No reservation is created. The stay is 32 nights, which is over the 31-night limit.
- **Expected Exception:** `ValueError`

### TC05: Too Many Guests (Rule 5)

- **Check-in:** June 10
- **Checkout:** June 13
- **Number of guests:** 13
- **Availability:** Not checked.
- **Expected Result:** No reservation is created.
- **Expected Exception:** `ValueError`

### TC06: Requested Night Already Booked (Rule 6)

- **Check-in:** June 10
- **Checkout:** June 13
- **Number of guests:** 4
- **Availability:** June 11 is already booked. June 10 and 12 are available.
- **Expected Result:** No reservation is created.
- **Expected Exception:** `UnavailableError`

## Step 4: Identify Reservation States and Events

- Identify five possible reservation states.
- Identify five events that may cause the reservation to change states.

### Reservation States

- `Pending`: When a new valid reservation is waiting for payment.
- `Confirmed`: When payment went through and the reservation is booked.
- `Checked In`: When the guest has arrived on the check-in date.
- `Completed`: When the guest has checked out. This is a final state.
- `Cancelled`: When the payment failed or the reservation was cancelled. This is a final state.

### Events That Change Reservation States

- Payment succeeds: `Pending` → `Confirmed`
- Payment fails: `Pending` → `Cancelled`
- Guest cancels: `Pending` or `Confirmed` → `Cancelled`
- Guest checks in: `Confirmed` → `Checked In` (only available on the check-in date)
- Guest checks out: `Checked In` → `Completed`

## Step 5: Create a State Transition Table

| Current State | Event                      | Expected State | Expected Result                                          |
| ------------- | -------------------------- | -------------- | -------------------------------------------------------- |
| Pending       | Payment succeeds           | Confirmed      | The reservation is confirmed.                            |
| Pending       | Payment fails              | Cancelled      | The reservation is cancelled and the dates are released. |
| Pending       | Guest cancels              | Cancelled      | The reservation is cancelled and the dates are released. |
| Confirmed     | Guest cancels              | Cancelled      | The reservation is cancelled and the dates are released. |
| Confirmed     | Guest checks in            | Checked In     | The guest is checked in on the check-in date.            |
| Checked In    | Guest checks out           | Completed      | The reservation is completed.                            |
| Pending       | Guest attempts to check in | Pending        | An `InvalidStateError` is raised.                        |
| Checked In    | Guest attempts to cancel   | Checked In     | An `InvalidStateError` is raised.                        |
| Cancelled     | Guest attempts to check in | Cancelled      | An `InvalidStateError` is raised.                        |
| Completed     | Guest attempts to cancel   | Completed      | An `InvalidStateError` is raised.                        |

## Step 6: Create State Transition Test Cases

| Test ID | Starting State | Event                                                 | Expected State | Expected Result                                                           | Transition Type |
| ------- | -------------- | ----------------------------------------------------- | -------------- | ------------------------------------------------------------------------- | --------------- |
| ST01    | Pending        | Payment succeeds                                      | Confirmed      | The reservation is confirmed.                                             | Valid           |
| ST02    | Pending        | Payment fails                                         | Cancelled      | The reservation is cancelled and the dates are released.                  | Valid           |
| ST03    | Pending        | Guest cancels                                         | Cancelled      | The reservation is cancelled and the dates are released.                  | Valid           |
| ST04    | Confirmed      | Guest cancels                                         | Cancelled      | The reservation is cancelled and the dates are released.                  | Valid           |
| ST05    | Confirmed      | Guest checks in                                       | Checked In     | The guest is checked in on the check-in date.                             | Valid           |
| ST06    | Checked In     | Guest checks out                                      | Completed      | The reservation is completed.                                             | Valid           |
| ST07    | Pending        | Guest attempts to check in                            | Pending        | An `InvalidStateError` is raised.                                         | Invalid         |
| ST08    | Checked In     | Guest attempts to cancel                              | Checked In     | An `InvalidStateError` is raised.                                         | Invalid         |
| ST09    | Cancelled      | Guest attempts to check in                            | Cancelled      | An `InvalidStateError` is raised.                                         | Invalid         |
| ST10    | Completed      | Guest attempts to cancel                              | Completed      | An `InvalidStateError` is raised.                                         | Invalid         |
| ST11    | Pending        | Payment succeeds → Guest checks in → Guest checks out | Completed      | The reservation moves through every step and completes.                   | Valid           |
| ST12    | Pending        | Payment succeeds → Guest cancels                      | Cancelled      | The reservation is confirmed, then cancelled, and the dates are released. | Valid           |
