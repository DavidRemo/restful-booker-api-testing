# Restful-booker API Testing

Manual and automated API testing of [restful-booker](https://restful-booker.herokuapp.com), a public practice API for a hotel booking system, done in Postman. I ran the full lifecycle of a booking (authenticate, create, read, full update, partial update, delete), added negative tests along the way, logged every result as expected vs actual, and then automated the lifecycle with Postman scripts and the Collection Runner.

This is part of my move into QA engineering. I'm coming from seven years in healthcare insurance operations, where I spent my days checking that claims, authorizations and contracts were processed correctly, so testing whether a system does what it should is familiar ground.

## What's in this repo

| File | What it is |
|---|---|
| `Restful-booker-practice.postman_collection.json` | The Postman collection, with requests and automated checks |
| `Restful-booker-test-report.pdf` | The full report: test log, defects, environment notes, automation |
| `runner-results.png` | Screenshot of the Collection Runner: 17 of 17 checks passing |
| `README.md` | This file |

## What I tested

| Endpoint | Method | Purpose |
|---|---|---|
| `/ping` | GET | Health check |
| `/booking` | GET | List booking IDs, with optional filters |
| `/booking/{id}` | GET | Get one booking |
| `/auth` | POST | Create an auth token |
| `/booking` | POST | Create a booking |
| `/booking/{id}` | PUT | Replace a booking completely |
| `/booking/{id}` | PATCH | Change part of a booking |
| `/booking/{id}` | DELETE | Remove a booking |

For each one I checked the status code first, then the response body (field names, values, types), then ran negative tests: nonexistent and text IDs, wrong credentials, missing and fake tokens, incomplete data, and deleting the same booking twice.

## Results at a glance

**Manual testing:** 18 tests logged. Core create, read, update and delete flows work and return the data that was sent. Authorization is enforced: stale, missing and fake tokens were all rejected with 403.

**Automated run:** 17 checks across the booking lifecycle, 17 passed, 0 failed (see `runner-results.png`).

## Defects and quirks found

1. **Failed login returns 200 OK.** `POST /auth` with a wrong password returns `200` with `{"reason": "Bad credentials"}` instead of an error status like 401. Clients that rely on the status code would treat a failed login as a success.
2. **PUT or DELETE on a missing booking returns 405.** Expected `404 Not Found`. The method is valid, the booking is missing, so the status is misleading.
3. **DELETE returns 201 Created.** Expected `200` or `204`. A "created" status on a delete is the opposite of what happened.

Each one has steps to reproduce, expected result, actual result and impact in the report.

## Automation

The scripts live in each request's **Scripts** tab (After response). They check status codes, field types, and that the saved data matches what was sent.

The Get token and Create booking scripts save the token and the new booking ID to collection variables, and later requests use `{{token}}` and `{{bookingId}}`. That means the whole chain runs with no copy-pasting, and it recovers from the server's periodic resets because every run creates fresh data.

Run order in the Collection Runner:

1. Get token
2. Create booking
3. Get one booking
4. Update booking (PUT)
5. Update booking (PATCH)
6. Delete booking
7. Get deleted booking

## Things I learned

- **PUT vs PATCH, proven with real data.** The same body, `{"firstname": "Dave"}`, was rejected by PUT with a 400 (it needs every field) and accepted by PATCH, which changed only the first name.
- **Create isn't verified until you read it back.** A response that echoes your data doesn't prove it was saved.
- **Test setup problems look like bugs.** The shared server resets its data and tokens regularly, which caused a 404 on a booking I'd just created and a 403 from a stale token. I re-created the data and repeated the check before calling anything a defect.
- **A test has to be able to fail.** I changed an expected value on purpose and watched it go red before trusting the test.
- **Read the error type first.** A ReferenceError meant a mistake in my script, while an AssertionError means the API returned something different from what I expected.

## How to run it yourself

1. Install [Postman](https://www.postman.com/downloads/) (free).
2. Download the collection file from this repo.
3. In Postman, click **Import** and select the file.
4. Open the collection's **Run collection** option, put the requests in the order above, and click **Run**.

No manual token or ID copying is needed. The practice server can be slow when it has been idle, so if the first request times out, just run it again.

## Next steps

- Add checks to Get all bookings, which has none yet
- Automate the negative tests (nonexistent ID, no token, fake token) inside the collection
- Try the same endpoints with XML bodies

## Contact

David Nyokabi, Nairobi, Kenya
LinkedIn: _add your link here_
