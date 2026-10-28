# Restful-booker API Testing

Manual API testing of [restful-booker](https://restful-booker.herokuapp.com), a public practice API for a hotel booking system, done in Postman. I ran the full lifecycle of a booking (authenticate, create, read, full update, partial update, delete), added negative tests along the way, and logged every result as expected vs actual.

This is part of my move into QA engineering. I'm coming from seven years in healthcare insurance operations, where I spent my days checking that claims, authorizations and contracts were processed correctly, so testing whether a system does what it should is familiar ground.

## What's in this repo

| File | What it is |
|---|---|
| `Restful-booker-practice.postman_collection.json` | The Postman collection with every request I ran |
| `Restful-booker-test-report.pdf` | The full report: test log, defects, environment notes |
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

18 tests logged. Core create, read, update and delete flows work and return the data that was sent. Authorization is enforced: stale, missing and fake tokens were all rejected with 403.

## Defects and quirks found

1. **Failed login returns 200 OK.** `POST /auth` with a wrong password returns `200` with `{"reason": "Bad credentials"}` instead of an error status like 401. Clients that rely on the status code would treat a failed login as a success.
2. **PUT or DELETE on a missing booking returns 405.** Expected `404 Not Found`. The method is valid, the booking is missing, so the status is misleading.
3. **DELETE returns 201 Created.** Expected `200` or `204`. A "created" status on a delete is the opposite of what happened.

Each one has steps to reproduce, expected result, actual result and impact in the report.

## Things I learned

- **PUT vs PATCH, proven with real data.** The same body, `{"firstname": "Dave"}`, was rejected by PUT with a 400 (it needs every field) and accepted by PATCH, which changed only the first name.
- **Create isn't verified until you read it back.** A response that echoes your data doesn't prove it was saved.
- **Test setup problems look like bugs.** The shared server resets its data and tokens regularly, which caused a 404 on a booking I'd just created and a 403 from a token that had gone stale. I re-created the data and repeated the check before calling anything a defect.

## How to run it yourself

1. Install [Postman](https://www.postman.com/downloads/) (free).
2. Download the collection file from this repo.
3. In Postman, click **Import** and select the file.
4. Run `Get token` first, copy the token, and paste it into the `Cookie` header (`token=...`) of the update, patch and delete requests.
5. Run `Create booking`, then use the returned `bookingid` in the later requests.

Do it all in one go. The practice server resets periodically, so bookings and tokens don't last long.

## Next steps

- Add automated assertions in Postman's Scripts tab (status code checks, field type checks)
- Run the whole collection end to end with the Collection Runner
- Try the same endpoints with XML bodies

## Contact

David Nyokabi, Nairobi, Kenya
LinkedIn: _add your link here_
