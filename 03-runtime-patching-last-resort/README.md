# Pattern 03: Runtime Patching (Last Resort)

## WHAT
Patch rendered UI behavior at runtime (DOM interception, event hooks, client script injection) without a rebuild of target app source.

## WHEN
- Production hotfix is required quickly.
- You cannot run full frontend build/deploy cycle immediately.
- You already have a migration plan to a cleaner pattern.

## WHY
- Fastest emergency lever.
- No source-level coupling initially.

## HOW

### 1) Inject JS via hooks

`hooks.py`

```python
app_include_js = ["/assets/my_app/js/runtime_patch.js"]
```

`public/js/runtime_patch.js`

```js
// Example targets CRM-like Desk DOM from Frappe/CRM v16-era markup.
// Selectors below are implementation details and can break on upstream updates.
// Re-check these selectors whenever you upgrade Frappe/CRM major/minor versions.
function applyPatch() {
  const sidebar = document.querySelector('[data-component="sidebar"], .layout-side-section')
  if (!sidebar || sidebar.querySelector('[data-my-app-link="1"]')) return

  const a = document.createElement('a')
  // Keep href static/internal. If you make this dynamic, validate against an allowlist.
  a.href = '/app/video'
  a.innerText = 'Video'
  a.setAttribute('data-my-app-link', '1')
  a.style.display = 'block'
  a.style.padding = '8px 12px'

  sidebar.appendChild(a)
}

let scheduled = false
const runPatch = () => {
  if (scheduled) return
  scheduled = true
  requestAnimationFrame(() => {
    applyPatch()
    scheduled = false
  })
}

const root = document.querySelector('#body, .desk-container') || document.body
const observer = new MutationObserver(() => runPatch())
observer.observe(root, { childList: true, subtree: true })
applyPatch()
```

## Risks / Limits

- Fragile: selectors break after upstream UI changes.
- Harder testing and debugging.
- Should be temporary technical debt only.
