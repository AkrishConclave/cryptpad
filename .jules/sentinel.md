## 2024-04-18 - [DOM-based XSS in Util.stripTags]
**Vulnerability:** Found a Cross-Site Scripting (XSS) vulnerability in `Util.stripTags` within `src/common/common-util.js`. The function used `div.innerHTML = text` to strip tags which triggers event handlers like `<img onerror=...>` before extraction.
**Learning:** Even if a DOM node is not attached to the document body, setting its `innerHTML` with unsanitized user input immediately triggers the browser to load resources and execute `onerror`/`onload` handlers for things like images, leading to XSS.
**Prevention:** Use `DOMParser` (`new DOMParser().parseFromString(text, 'text/html').body.textContent`) when parsing or stripping tags from untrusted HTML on the frontend instead of `innerHTML`.

## 2024-09-02 - [Missing Trust Proxy Configuration]
**Vulnerability:** Client IP addresses are not correctly resolved when the application runs behind a reverse proxy, potentially breaking IP-based security measures like rate limiting.
**Learning:** Express applications in this repository must use a configurable setting (e.g., `trustProxy` from configuration) to safely enable IP resolution behind a reverse proxy, avoiding IP spoofing.
**Prevention:** Make `trust proxy` configurable and default to `false` (disabled) unless explicitly enabled in the environment.
