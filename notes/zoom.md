---
tags: [ hammerspoon, zoom ]
---
# Using Hammerspoon to intercept Zoom meeting URLs

I hate how clicking on Zoom links leaves behind a dangling tab in my browser,
so this uses [Hammerspoon](https://www.hammerspoon.org/)'s URL handling
capabilities to route Zoom URLs to open in the Zoom app directly:

```lisp
(set hs.urlevent.httpCallback (fn [scheme host params fullURL]
  (if (string.find fullURL "^https?://.*[.]zoom.us/j/%d+")
      (hs.urlevent.openURLWithBundle fullURL "us.zoom.xos")
      (hs.urlevent.openURLWithBundle fullURL "org.mozilla.firefoxdeveloperedition"))))
```
