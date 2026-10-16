# Selenium project and driver setup

[Topic context](webdriver-testing-index.md) / [Previous source section](webdriver-github-actions.md) / [Next source section](selenium-tests-driver-lifecycle.md)

Context: Tauri v2: Selenium project and driver setup. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-selenium"></a> <a id="source-selenium-1"></a> <a id="source-create-a-directory-for-the-tests"></a> <a id="source-initializing-a-selenium-project"></a>

## Selenium

### Selenium

For this WebDriver demonstration we lean on [Selenium](https://selenium.dev/) together with a well-known Node.js test harness. The assumption is that Node.js is on your machine already, plus either `npm` or `yarn`, even though `pnpm` is what the [finished example project](https://github.com/tauri-apps/webdriver-example) happens to use.

### Create a Directory for the Tests

Let's carve out somewhere in the project to hold these tests. This sample nests the folder because additional frameworks come up later on, though in practice a single one is usually all you need. Spin up the folder with `mkdir -p e2e-tests`. From here on, the guide takes for granted that your shell sits within the `e2e-tests` directory.

### Initializing a Selenium Project

To get this suite going we start from a ready-made `package.json`, since the dependencies have already been picked for us and the aim is to present a straightforward, functioning setup. Down at the end of this section sits a folded-away walkthrough covering how to assemble it yourself.

`package.json`:

```json
{
  "name": "selenium",
  "version": "1.0.0",
  "private": true,
  "type": "module",
  "scripts": {
    "test": "mocha"
  },
  "dependencies": {
    "chai": "^5.2.1",
    "mocha": "^11.7.1",
    "selenium-webdriver": "^4.34.0"
  }
}
```

There's a script wiring up [Mocha](https://mochajs.org/) as the framework and surfacing it under the `test` command. Alongside it sit the various packages the tests depend on: [Mocha](https://mochajs.org/) for the test framework itself, [Chai](https://www.chaijs.com/) supplying assertions, and [`selenium-webdriver`](https://www.npmx.dev/package/selenium-webdriver), the [Selenium](https://selenium.dev/) binding for Node.js.

Expand this to learn how to stand up the project from scratch

To pull in the dependencies starting from nothing, simply issue the command below.

- npm
- yarn

```sh
npm install mocha chai selenium-webdriver
```

```sh
yarn add mocha chai selenium-webdriver
```

It's also a good idea to drop a `"test": "mocha"` entry into the `"scripts"` key of `package.json`, which lets you kick off Mocha with nothing more than

- npm
- yarn

```sh
npm test
```

```sh
yarn test
```

