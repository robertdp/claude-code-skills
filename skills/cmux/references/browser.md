# Browser Automation

All commands target a browser surface. Specify with `--surface surface:N` or positional `surface:N`.

## Open / Navigate

```sh
cmux browser open https://example.com                      # new browser split
cmux browser open-split https://example.com                # explicit split
cmux browser --surface surface:2 navigate https://google.com
cmux browser --surface surface:2 back
cmux browser --surface surface:2 forward
cmux browser --surface surface:2 reload
cmux browser --surface surface:2 url                       # get current URL
```

## Screenshot

```sh
cmux browser --surface surface:2 screenshot --out /tmp/page.png
```

## DOM Snapshot

```sh
cmux browser --surface surface:2 snapshot                  # full DOM
cmux browser --surface surface:2 snapshot --interactive    # interactive elements only
cmux browser --surface surface:2 snapshot -i --compact     # compact interactive
cmux browser --surface surface:2 snapshot --selector "#main" --max-depth 3
cmux browser --surface surface:2 snapshot --cursor         # include cursor position
```

## Interaction

Most accept `--snapshot-after` to return DOM snapshot after the action.

```sh
cmux browser --surface surface:2 click "#submit"
cmux browser --surface surface:2 dblclick ".item"
cmux browser --surface surface:2 hover ".menu-item"
cmux browser --surface surface:2 focus "#input"
cmux browser --surface surface:2 check "#checkbox"
cmux browser --surface surface:2 uncheck "#checkbox"
cmux browser --surface surface:2 scroll-into-view "#footer"
cmux browser --surface surface:2 type "#search" "query"    # character by character
cmux browser --surface surface:2 fill "#search" "query"    # set value directly
cmux browser --surface surface:2 fill "#search"            # clear input
cmux browser --surface surface:2 press Enter
cmux browser --surface surface:2 keydown Shift
cmux browser --surface surface:2 keyup Shift
cmux browser --surface surface:2 select "#dropdown" "value"
cmux browser --surface surface:2 scroll --dy 500           # scroll down
cmux browser --surface surface:2 scroll --selector "#panel" --dy -200
```

## Wait

```sh
cmux browser --surface surface:2 wait --selector "#loaded"
cmux browser --surface surface:2 wait --text "Success"
cmux browser --surface surface:2 wait --url-contains "/dashboard"
cmux browser --surface surface:2 wait --load-state complete
cmux browser --surface surface:2 wait --function "() => document.ready"
cmux browser --surface surface:2 wait --timeout-ms 5000
```

## Data Extraction

```sh
cmux browser --surface surface:2 get title
cmux browser --surface surface:2 get url
cmux browser --surface surface:2 get text "#main"
cmux browser --surface surface:2 get html "#content"
cmux browser --surface surface:2 get value "#input"
cmux browser --surface surface:2 get attr "#link" href
cmux browser --surface surface:2 get count ".items"
cmux browser --surface surface:2 get box "#element"        # bounding box
cmux browser --surface surface:2 get styles "#element"
```

## Element State

```sh
cmux browser --surface surface:2 is visible "#modal"
cmux browser --surface surface:2 is enabled "#button"
cmux browser --surface surface:2 is checked "#checkbox"
```

## Element Finding

```sh
cmux browser --surface surface:2 find role button
cmux browser --surface surface:2 find text "Sign In"
cmux browser --surface surface:2 find label "Email"
cmux browser --surface surface:2 find placeholder "Search..."
cmux browser --surface surface:2 find testid "submit-btn"
cmux browser --surface surface:2 find first ".item"
cmux browser --surface surface:2 find last ".item"
cmux browser --surface surface:2 find nth ".item" 2
```

## JavaScript

```sh
cmux browser --surface surface:2 eval "document.title"
cmux browser --surface surface:2 eval "window.scrollTo(0, 0)"
cmux browser --surface surface:2 addinitscript "console.log('init')"  # on every nav
cmux browser --surface surface:2 addscript "console.log('once')"      # once now
cmux browser --surface surface:2 addstyle "body { background: red; }"
```

## Frames

```sh
cmux browser --surface surface:2 frame "#iframe-selector"  # enter iframe
cmux browser --surface surface:2 frame main                # back to main
```

## Dialogs

```sh
cmux browser --surface surface:2 dialog accept
cmux browser --surface surface:2 dialog accept "confirmed text"
cmux browser --surface surface:2 dialog dismiss
```

## Downloads

```sh
cmux browser --surface surface:2 download wait --path ./downloads --timeout-ms 10000
```

## Session Data

```sh
# Cookies
cmux browser --surface surface:2 cookies get
cmux browser --surface surface:2 cookies set <cookie-json>
cmux browser --surface surface:2 cookies clear

# Storage
cmux browser --surface surface:2 storage local get "key"
cmux browser --surface surface:2 storage local set "key" "value"
cmux browser --surface surface:2 storage local clear
cmux browser --surface surface:2 storage session get "key"
cmux browser --surface surface:2 storage session set "key" "value"
cmux browser --surface surface:2 storage session clear

# Save/restore full state
cmux browser --surface surface:2 state save /tmp/state.json
cmux browser --surface surface:2 state load /tmp/state.json
```

## Browser Tabs (within surface)

```sh
cmux browser --surface surface:2 tab list
cmux browser --surface surface:2 tab new
cmux browser --surface surface:2 tab switch 1
cmux browser --surface surface:2 tab close
```

## Debugging

```sh
cmux browser --surface surface:2 console list
cmux browser --surface surface:2 console clear
cmux browser --surface surface:2 errors list
cmux browser --surface surface:2 errors clear
cmux browser --surface surface:2 highlight "#element"
cmux browser identify --surface surface:2
```

## Advanced (may return `not_supported` on WKWebView)

```sh
cmux browser --surface surface:2 viewport 1920 1080
cmux browser --surface surface:2 geolocation 35.6762 139.6503
cmux browser --surface surface:2 offline true
cmux browser --surface surface:2 trace start /tmp/trace.zip
cmux browser --surface surface:2 trace stop
cmux browser --surface surface:2 network route "**/*.png" --status 404
cmux browser --surface surface:2 network unroute "**/*.png"
cmux browser --surface surface:2 network requests
cmux browser --surface surface:2 screencast start
cmux browser --surface surface:2 screencast stop
```
