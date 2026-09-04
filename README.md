# Lab 2 Starter: Availability Calculator

A small reservation component. Given a room's bookings and the day's business hours,
`AvailabilityCalculator.freeSlots` computes when the room is free. It is the code you
work in for Lab 2.

It ships with a generated test suite that passes, and a property-based test harness
(jqwik) with one example property. Everything is green. Your job in Lab 2 is to decide
whether green actually means correct.

**Read `ARCHITECTURE.md` before the code.**

## Build and test

```
mvn test
```

`mvn test` runs both files, the ordinary example-based tests (`AvailabilityCalculatorTest`)
and the property-based tests (`AvailabilityProperties`). A code-coverage report is written
to `target/site/jacoco/index.html`.

## Continuous integration

This repository has CI configured in `.github/workflows/ci.yml`. GitHub disables workflows on a
fresh fork, so enable them once on your fork (the handout shows where). After that, every
push runs `mvn test`. You will watch the gate go red when your new property finds the bug, then
green once you fix it.

## Where things are

- Component: `src/main/java/edu/cmu/cs214/availability/`
- Example-based tests: `src/test/java/edu/cmu/cs214/availability/AvailabilityCalculatorTest.java`
- Property-based tests: `src/test/java/edu/cmu/cs214/availability/AvailabilityProperties.java`
- Setup: `SETUP.md`

See the Lab 2 handout on the course page for the three milestones you show a TA.

## Milestone 3: Audit of Generated Suite

### Weaknesses in the generated suite
1. **Controllability Gap**: The suite never tests the calculator with an empty list of bookings (`[]`). The test inputs always provide at least one booking, which means the edge case where the loop is entirely skipped is never verified.
2. **Controllability Gap**: The suite largely ignores scenarios where a significant chunk of free time exists at the very end of the day. In tests like `bookingUntilEndOfDayLeavesTheMorningFree` and `gapsBetweenBookingsAreReturned`, the final booking always perfectly touches `DAY_END`. The test suite didn't adequately force the system to check if it correctly appends remaining time after the last booking.
3. **Observability Gap**: In the `returnedSlotsNeverOverlapABooking` test, the input is `[600, 660)`, which leaves the afternoon free. The bug was actually triggered here, and the calculator omitted the afternoon slot! However, the test's assertions *only* check if the returned slots overlap with the booking (`assertFalse(slot.overlaps(booking))`). It completely failed to check if the returned slots were actually complete, meaning it ran the buggy code but its assertions couldn't see the wrong result.

### Why high coverage did not save it
High code coverage (like JaCoCo reporting 100% line coverage) only proves that the lines of code were *executed* during the test suite. It does not mean that every logical state or edge case (like an empty list) was reached (controllability), nor does it guarantee that the test actually *asserts* the output is correct (observability). Coverage measures execution, not correctness.

### Tools Used
- **Agent/Tool**: Antigravity IDE
- **Model**: Gemini 3.1 Pro (High)
