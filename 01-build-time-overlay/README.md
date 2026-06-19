# Pattern 01: Build-Time Overlay (Selective Source Override)

## WHAT
Copy upstream app frontend source at build-time, then overwrite only targeted files from your custom app before bundling.

## WHEN
- You must override existing Vue components/pages in CRM/LMS.
- Upstream has no official extension point for that screen.
- Change scope is limited to a small set of files.

## WHY
- Keeps override code in your app.
- Faster than maintaining a full fork.
- Works with current Frappe UI app build pipelines when paths are valid.

## HOW

### 1) Frontend scripts

```json
{
  "devDependencies": {
    "fs-extra": "11.2.0"
  },
  "scripts": {
    "prebuild": "node ./custom-build.js",
    "build": "yarn prebuild && vite build --base=/assets/my_app/frontend/"
  }
}
```

### 2) Overlay script (`frontend/custom-build.js`)

```js
const fs = require('fs-extra')
const path = require('path')

// Example: custom app is at apps/my_app, target app at apps/crm
const targetFrontend = path.resolve(__dirname, '../../crm/frontend/src')
const localSrc = path.resolve(__dirname, './src')
const overrides = path.resolve(__dirname, './src_override')

try {
  if (!fs.existsSync(targetFrontend)) {
    throw new Error(`Target frontend not found: ${targetFrontend}`)
  }

  fs.emptyDirSync(localSrc)
  fs.copySync(targetFrontend, localSrc)
  fs.copySync(overrides, localSrc) // only changed files in src_override

  console.log('Overlay complete')
} catch (error) {
  console.error('Overlay failed:', error.message)
  process.exit(1)
}
```

### 3) Folder layout

```text
frontend/
  src/               # generated during prebuild
  src_override/
    components/Layouts/AppSidebar.vue
    router.js
  custom-build.js
  vite.config.js
```

### 4) Operational guardrails

- Keep `src_override` minimal.
- Pin compatible app versions (your app + target app).
- Re-validate after each upstream update.

```bash
bench build --app my_app
bench --site <site> clear-cache
bench --site <site> clear-website-cache
```
