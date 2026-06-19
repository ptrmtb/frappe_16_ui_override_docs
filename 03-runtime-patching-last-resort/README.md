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
// Example: add fallback button into CRM sidebar container when present
function applyPatch() {
  const sidebar = document.querySelector('[data-component="sidebar"], .layout-side-section')
  if (!sidebar || sidebar.querySelector('[data-my-app-link="1"]')) return

  const a = document.createElement('a')
  a.href = '/app/video'
  a.innerText = 'Video'
  a.setAttribute('data-my-app-link', '1')
  a.style.display = 'block'
  a.style.padding = '8px 12px'

  sidebar.appendChild(a)
}

const observer = new MutationObserver(() => applyPatch())
observer.observe(document.body, { childList: true, subtree: true })
applyPatch()
```

## Risks / Limits

- Fragile: selectors break after upstream UI changes.
- Harder testing and debugging.
- Should be temporary technical debt only.
