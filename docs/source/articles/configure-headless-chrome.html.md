---
title: Configure Headless Chrome
created_at: 2020/01/13
excerpt: TODO
---

# Configure Headless Chrome

System tests (see below) are driven by a headless instance of Chrome, which, since Workarea 3.5.0, is configurable as follows:

```
# <workarea-core>/lib/workarea/configuration.rb

# Options passed to the Selenium driver's capabilities
config.headless_chrome_options = { w3c: false }

# Arguments passed to headless Chrome for running system tests
config.headless_chrome_args = [
  'headless',
  'disable-gpu',
  'disable-popup-blocking',
  '--enable-features=NetworkService,NetworkServiceInProcess',
  "--window-size=#{config.capybara_browser_width},#{config.capybara_browser_width}"
]
```

Modify these configurations as needed from an initializer in your app or plugin.
