# Selenium application assertions

Open when testing the greeting and background color of the example Tauri app.

[Topic context](selenium-tests-driver-lifecycle.md)

Save as `e2e-tests/test/test.js`, beside [driver.js](selenium-driver-lifecycle.md). These Mocha tests import the driver module and its setup/cleanup functions.

```javascript
import { expect } from 'chai';
import { By } from 'selenium-webdriver';
import { driver, startDriver, stopDriver } from './driver.js';

before(async function () {
  this.timeout(120000);
  await startDriver();
});
after(async function () { await stopDriver(); });

describe('Hello Tauri', () => {
  it('should be cordial', async () => {
    const text = await driver.findElement(By.css('body > h1')).getText();
    expect(text).to.match(/^[hH]ello/);
  });

  it('should be excited', async () => {
    const text = await driver.findElement(By.css('body > h1')).getText();
    expect(text).to.match(/!$/);
  });

  it('should be easy on the eyes', async () => {
    // selenium returns color css values as rgb(r, g, b)
    const text = await driver
      .findElement(By.css('body'))
      .getCssValue('background-color');

    const rgb = text.match(/^rgb\((?<r>\d+), (?<g>\d+), (?<b>\d+)\)$/).groups;
    expect(rgb).to.have.all.keys('r', 'g', 'b');

    const luma = 0.2126 * rgb.r + 0.7152 * rgb.g + 0.0722 * rgb.b;
    expect(luma).to.be.lessThan(100);
  });
});
```
