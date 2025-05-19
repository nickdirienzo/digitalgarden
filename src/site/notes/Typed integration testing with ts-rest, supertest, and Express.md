---
{"dg-publish":true,"permalink":"/typed-integration-testing-with-ts-rest-supertest-and-express/"}
---

We've been happily using `ts-rest` at Mirage Security for about a year now. There's more to be written about why we went with `ts-rest` and how we use it to efficiently build features. This is about testing with it because I couldn't find any clear guides on how to test `ts-rest` and there's an unanswered [question in Discord](https://discord.com/channels/1055855205960392724/1055857825831731200/1259172105677836298) so here's how we approached it.

When building APIs, I've found integration tests to be the more valuable kinds of tests to write than unit tests. You can codify expected end-user API behavior in tests, can validate database state, and generally have more confidence it'll work in production. With agentic coding tools like VSCode Copilot or Windsurf Cascade, the cost of writing these kinds of tests become 0 so we have way more test coverage than I would have expected at this stage while still shipping incredibly quickly.

There were two problems to solve for us to get integration tests working:
1. How do we share database access between our test harness and the API implementations so that we can create data within the test block and have the API handler be able to use them?
2. How do we use a ts-rest client in our test suite so we can have strong typing and generally reduce what we have to test (because our contracts will validate request input)?

I wasn't sure how to solve these in this TypeScript / Node ecosystem. The codebase has been using a singleton database client which you'll see in a number of Drizzle examples (this [one](https://github.com/drizzlenext/drizzle-next) or the [official guide](https://orm.drizzle.team/docs/get-started-postgresql)). That works for until it doesn't. We now needed to share the database connection from outside of the application runtime so we could use it with `supertest`, which it seems that everyone was using for testing (at least per Discord).

To stay in line with [12 Factor](https://12factor.net) principles (which is still extremely good guidance), I wanted the app to not know whether it was using a real database or a test database. To make that happen we used dependency injection. The plan was to have the request context maintain the database client handle, use an app factory (taking inspiration from [Flask](https://flask.palletsprojects.com/en/stable/patterns/appfactories/)), and pass in a database client to either a managed database or a test database. 

There's two approaches you can take when testing with `ts-rest`. You can hit it with `supertest` directly, but then you lose all the client helpers and type safety. Or you can provide `initClient` an `ApiFetcher` with leverages `supertest` under the hood to have the test harness get types. The former is how your non-TS users will use the API, but the latter lets us test really fast using ergonomics we're already using in the frontend. 

I'm simplifying the code as I copy paste so I wouldn't expect it to work with a direct copy-paste to your codebase, but I hope it explains what's supposed to be happening. The full gist can be found here: https://gist.github.com/nickdirienzo/73d6cfb6caf29c56219340a7546ebd9c

The part that brings `supertest` and `ts-rest` together is specifically this bit: <script src="https://gist.github.com/nickdirienzo/73d6cfb6caf29c56219340a7546ebd9c.js?file=tsRest.ts"></script>

