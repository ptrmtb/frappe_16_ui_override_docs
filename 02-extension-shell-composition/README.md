# Pattern 02: Extension Shell / Composition (Preferred)

## WHAT
Add new UI pages/features from your custom app and integrate them into navigation/workspace, without replacing upstream core components.

## WHEN
- You can deliver requirements via additional screens/actions.
- You want lowest upgrade risk on Frappe v16+.
- You want to avoid coupling with internals of CRM/LMS source tree.

## WHY
- Most stable across upstream updates.
- No large frontend source copy step.
- Easier long-term maintenance for teams.

## HOW

### 1) Add a Desk page in your app

`/home/frappe/frappe-bench/apps/my_app/my_app/my_app/page/video/video.json`

```json
{
  "title": "Video",
  "route": "video",
  "type": "page",
  "icon": "video"
}
```

`/home/frappe/frappe-bench/apps/my_app/my_app/my_app/page/video/video.js`

```js
frappe.pages['video'].on_page_load = function (wrapper) {
  const page = frappe.ui.make_app_page({
    parent: wrapper,
    title: 'Video',
    single_column: true,
  })

  const html = `<iframe width="100%" height="520" src="https://www.youtube.com/embed/OKrGJa2NnIs" title="Video" allowfullscreen></iframe>`
  $(page.body).html(html)
}
```

### 2) Link it in Workspace (or role-based shortcut)

Use Workspace customization so users can access your page from Desk/CRM navigation context.

### 3) Optional: add Vue SPA mount for richer UI

`hooks.py`

```python
app_include_js = ["/assets/my_app/js/video_mount.js"]
```

`public/js/video_mount.js`

```js
// Mount your Vue app only on route #video
if (window.location.hash?.includes('/app/video')) {
  // bootstrap your bundled Vue entry here
  console.log('Mount custom Video app')
}
```
