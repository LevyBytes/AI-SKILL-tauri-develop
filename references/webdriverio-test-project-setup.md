# WebdriverIO project setup

[Topic context](webdriver-testing-index.md) / [Previous source section](selenium-tests-driver-lifecycle.md) / [Next source section](webdriverio-suite-configuration-execution.md)

Context: Tauri v2: WebdriverIO project setup. Read linked sections for surrounding setup, version limits, and caveats; continued examples are not standalone programs.

<a id="source-webdriverio"></a> <a id="source-webdriverio-1"></a> <a id="source-create-a-directory-for-the-tests-1"></a> <a id="source-initializing-a-webdriverio-project"></a>

## WebdriverIO

### WebdriverIO

This WebDriver walkthrough relies on [WebdriverIO](https://webdriver.io/) and the suite it brings along. We presume Node.js is already set up, together with `npm` or `yarn`, though the [finished example project](https://github.com/tauri-apps/webdriver-example) leans on `pnpm` instead.

### Create a Directory for the Tests

Let's set aside a place inside the project where these tests can live. A nested folder is used in this sample because more frameworks are covered later, but ordinarily just one suffices. Make the folder via `mkdir e2e-tests`. Everything that follows presumes your working directory is `e2e-tests`.

### Initializing a WebdriverIO Project

A ready-made `package.json` serves to bootstrap this suite, given that the [WebdriverIO](https://webdriver.io/) configuration choices have already been made and we want to demonstrate an uncomplicated, working setup. A folded-up walkthrough on building it from the ground up sits at the foot of this section.

`package.json`:

```json
{
  "name": "webdriverio",
  "version": "1.0.0",
  "private": true,
  "type": "module",
  "scripts": {
    "test": "wdio run wdio.conf.js"
  },
  "dependencies": {
    "@wdio/cli": "^9.19.0"
  },
  "devDependencies": {
    "@wdio/local-runner": "^9.19.0",
    "@wdio/mocha-framework": "^9.19.0",
    "@wdio/spec-reporter": "^9.19.0"
  }
}
```

There's a script that executes a [WebdriverIO](https://webdriver.io/) config as the suite, surfaced behind the `test` command. The assorted dependencies were pulled in by the `@wdio/cli` command back when we initialized things. Put briefly, they cover the most basic arrangement: a local WebDriver runner, [Mocha](https://mochajs.org/) serving as the framework, and a plain Spec Reporter.

Open me to walk through configuring a project from the ground up

Because the CLI runs interactively, the tooling is yours to pick. Bear in mind this will probably take you off the path the guide follows, leaving you to reconcile those discrepancies on your own.

Now let's bring the [WebdriverIO](https://webdriver.io/) CLI into this npm project.

- npm
- yarn

```sh
npm install @wdio/cli
```

```sh
yarn add @wdio/cli
```

Afterward, to launch the interactive config step that scaffolds a [WebdriverIO](https://webdriver.io/) suite, issue:

- npm
- yarn

```sh
npx wdio config
```

```sh
yarn wdio config
```

