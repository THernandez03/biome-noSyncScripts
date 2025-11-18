# Biome `noSyncScripts` Rule Issue

## Issue Summary

The Biome linter rule `nursery/noSyncScripts` incorrectly flags `<script type="module">` tags as synchronous scripts, requesting the addition of `defer` or `async` attributes. **This is unnecessary and incorrect** because module scripts are automatically deferred by specification.

## Current Behavior

When running Biome with the `noSyncScripts` rule enabled:

```bash
npm run check
```

**Output:**
```
index.html:10:5 lint/nursery/noSyncScripts

  ℹ Unexpected synchronous script.
  
     8 │   <body>
     9 │     <div id="root"></div>
  > 10 │     <script type="module" src="./src/entrypoint.tsx"></script>
       │     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
    11 │   </body>
    12 │ </html>
  
  ℹ Synchronous scripts can impact your webpage performance. Add the "async" or 
"defer" attribute.
```

## Why This is Incorrect

### Module Scripts Are Already Deferred

According to the **[MDN Web Docs - JavaScript Modules](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules#other_differences_between_modules_and_classic_scripts)**:

> **3. Modules are deferred automatically**
> 
> Modules are automatically executed in defer mode. This means that:
> - Module scripts are downloaded in parallel to parsing the HTML
> - Module scripts are executed after the document has been parsed
> - The relative order of multiple module scripts is preserved

This behavior is defined in the [HTML Standard](https://html.spec.whatwg.org/multipage/scripting.html#attr-script-type) and is consistent across all modern browsers.

### Comparison Table

| Script Type | Blocks HTML Parsing | Execution Timing | Needs `defer`/`async` |
|-------------|---------------------|------------------|----------------------|
| `<script src="...">` | ✅ Yes | Immediately | ✅ **Yes** |
| `<script type="module" src="...">` | ❌ No | After DOM ready (deferred) | ❌ **No** |
| `<script defer src="...">` | ❌ No | After DOM ready | N/A |
| `<script async src="...">` | ❌ No | ASAP (unordered) | N/A |

## Expected Behavior

The `noSyncScripts` rule should **ignore** `<script>` tags that have `type="module"` because:

1. They are inherently non-blocking
2. They are automatically deferred by the browser
3. Adding `defer` or `async` to a module script is redundant and has no effect
4. Module scripts already provide optimal performance characteristics

## Reproduction

### Setup

```bash
npm install
```

### Run Check

```bash
npm run check
```

You'll see the false positive warning on line 10 of `index.html`.

## Configuration

**`biome.jsonc`:**
```jsonc
{
  "$schema": "https://biomejs.dev/schemas/2.3.6/schema.json",
  "files": {
    "includes": ["./index.html"]
  },
  "linter": {
    "enabled": true,
    "rules": {
      "nursery": {
        "noSyncScripts": "on"
      }
    }
  }
}
```

**`index.html`:**
```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>Document</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="./src/entrypoint.tsx"></script>
  </body>
</html>
```

## Proposed Solution

The `noSyncScripts` rule should be updated to:

1. **Skip validation** for `<script>` tags with `type="module"`
2. Only flag classic `<script>` tags without `defer` or `async` attributes
3. Optionally provide an informational message explaining that module scripts are already deferred

## References

- [MDN - JavaScript Modules: Other differences between modules and classic scripts](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules#other_differences_between_modules_and_classic_scripts) (Point #3)
- [HTML Living Standard - Script Types](https://html.spec.whatwg.org/multipage/scripting.html#attr-script-type)
- [MDN - `<script>`: The Script element](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/script#module)

## Environment

- **Biome Version:** 2.3.6
- **Rule:** `nursery/noSyncScripts`
- **Affected Files:** HTML files with `<script type="module">`

---

**Note:** This repository exists solely to demonstrate and document this issue for the Biome team. No workaround or fix is implemented here intentionally, as the rule itself needs to be corrected.
