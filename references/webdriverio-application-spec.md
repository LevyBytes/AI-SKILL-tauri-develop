# WebdriverIO application spec

Open when testing the example app greeting and background color with WebdriverIO.

[Topic context](webdriverio-suite-configuration-execution.md)

### Spec

The code that genuinely exercises your application lives inside a spec. These specs get picked up by the test runner, which executes them however it deems appropriate. Let's write our spec inside the folder we pointed to earlier.

`test/specs/example.e2e.js`:

```javascript
// calculates the luma from a hex color `#abcdef`
function luma(hex) {
  if (hex.startsWith('#')) {
    hex = hex.substring(1);
  }

  const rgb = parseInt(hex, 16);
  const r = (rgb >> 16) & 0xff;
  const g = (rgb >> 8) & 0xff;
  const b = (rgb >> 0) & 0xff;
  return 0.2126 * r + 0.7152 * g + 0.0722 * b;
}

describe('Hello Tauri', () => {
  it('should be cordial', async () => {
    const header = await $('body > h1');
    const text = await header.getText();
    expect(text).toMatch(/^[hH]ello/);
  });

  it('should be excited', async () => {
    const header = await $('body > h1');
    const text = await header.getText();
    expect(text).toMatch(/!$/);
  });

  it('should be easy on the eyes', async () => {
    const body = await $('body');
    const backgroundColor = await body.getCSSProperty('background-color');
    expect(luma(backgroundColor.parsed.hex)).toBeLessThan(100);
  });
});
```

That `luma` function up top merely assists one of the tests and has nothing to do with exercising the application itself. Coming from other testing frameworks, you'll likely recognize comparable exposed helpers in use here, namely `describe`, `it`, and `expect`. As for the remaining APIs, things such as `$` and the methods it offers, those are documented in the [WebdriverIO API docs](https://webdriver.io/docs/api).
