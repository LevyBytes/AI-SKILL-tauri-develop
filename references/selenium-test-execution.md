# Run the Selenium test suite

Open when running the Selenium example and interpreting its test report.

[Topic context](selenium-tests-driver-lifecycle.md)

### Running the Test Suite

With the dependencies in place and the test script ready, it's time to fire it off!

- npm
- yarn

```sh
npm test
```

```sh
yarn test
```

What lands on screen should resemble the following:

```text
➜  selenium git:(main) ✗ yarn test
yarn run v1.22.11
$ Mocha


  Hello Tauri
    ✔ should be cordial (120ms)
    ✔ should be excited
    ✔ should be easy on the eyes


  3 passing (588ms)

Done in 0.93s.
```

As shown, the `Hello Tauri` suite we declared via `describe` saw all 3 cases we wrote with `it` come back green!

By pairing [Selenium](https://selenium.dev/) with a bit of test-suite plumbing, we've stood up e2e testing while leaving our Tauri application entirely untouched!

---
