## 2026-09-01 - Missing rate limiting on Auth & File Upload endpoints
**Vulnerability:** No rate limit on `/api/auth` (RPC for authentication and commands) or `/upload-blob` (file upload).
**Learning:** Adding brute force protections on top of Express with `helmet` and `express-rate-limit` requires adjusting the rate limit max based on RPC design, and ensuring `trust proxy` is enabled since it's deployed behind reverse proxies. CryptPad specifically requires `X-Frame-Options` to be unset (disabled) when using helmet since it utilizes cross-origin iframes.
**Prevention:** Integrate rate limiting globally on auth/file routes by default and use node security standards (e.g. Helmet), considering proxy constraints.
## 2024-04-18 - [DOM-based XSS in Util.stripTags]
**Vulnerability:** Found a Cross-Site Scripting (XSS) vulnerability in `Util.stripTags` within `src/common/common-util.js`. The function used `div.innerHTML = text` to strip tags which triggers event handlers like `<img onerror=...>` before extraction.
**Learning:** Even if a DOM node is not attached to the document body, setting its `innerHTML` with unsanitized user input immediately triggers the browser to load resources and execute `onerror`/`onload` handlers for things like images, leading to XSS.
**Prevention:** Use `DOMParser` (`new DOMParser().parseFromString(text, 'text/html').body.textContent`) when parsing or stripping tags from untrusted HTML on the frontend instead of `innerHTML`.
