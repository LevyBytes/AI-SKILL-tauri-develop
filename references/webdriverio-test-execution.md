# Run WebdriverIO tests

Open when running the WDIO suite and interpreting its output.

[Topic context](webdriverio-suite-configuration-execution.md)

### Running the Test Suite

Having both the config and a spec squared away, let's go ahead and run it!

- npm
- yarn

```sh
npm test
```

```sh
yarn test
```

The output we get should look something like this:

```text
➜  webdriverio git:(main) ✗ yarn test
yarn run v1.22.11
$ wdio run wdio.conf.js

Execution of 1 workers started at 2021-08-17T08:06:10.279Z

[0-0] RUNNING in undefined - /develop/tests/specs/example.e2e.js
[0-0] PASSED in undefined - /develop/tests/specs/example.e2e.js

 "spec" Reporter:
------------------------------------------------------------------
[wry 0.12.1 linux #0-0] Running: wry (v0.12.1) on linux
[wry 0.12.1 linux #0-0] Session ID: 81e0107b-4d38-4eed-9b10-ee80ca47bb83
[wry 0.12.1 linux #0-0]
[wry 0.12.1 linux #0-0] » /develop/tests/specs/example.e2e.js
[wry 0.12.1 linux #0-0] Hello Tauri
[wry 0.12.1 linux #0-0]    ✓ should be cordial
[wry 0.12.1 linux #0-0]    ✓ should be excited
[wry 0.12.1 linux #0-0]    ✓ should be easy on the eyes
[wry 0.12.1 linux #0-0]
[wry 0.12.1 linux #0-0] 3 passing (244ms)


Spec Files:   1 passed, 1 total (100% completed) in 00:00:01

Done in 1.98s.
```

The Spec Reporter reports back that every one of the 3 tests in the `test/specs/example.e2e.js` file passed, capped off by the summary line `Spec Files: 1 passed, 1 total (100% completed) in 00:00:01`.

Thanks to the [WebdriverIO](https://webdriver.io/) suite, we've effortlessly brought e2e testing to our Tauri application off the back of a handful of config lines plus one command to launch it! Better still, the application itself needed no changes whatsoever.

---
