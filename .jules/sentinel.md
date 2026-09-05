## 2026-09-01 - Missing rate limiting on Auth & File Upload endpoints
**Vulnerability:** No rate limit on `/api/auth` (RPC for authentication and commands) or `/upload-blob` (file upload).
**Learning:** Adding brute force protections on top of Express with `helmet` and `express-rate-limit` requires adjusting the rate limit max based on RPC design, and ensuring `trust proxy` is enabled since it's deployed behind reverse proxies. CryptPad specifically requires `X-Frame-Options` to be unset (disabled) when using helmet since it utilizes cross-origin iframes.
**Prevention:** Integrate rate limiting globally on auth/file routes by default and use node security standards (e.g. Helmet), considering proxy constraints.
## 2024-04-18 - [DOM-based XSS in Util.stripTags]
**Vulnerability:** Found a Cross-Site Scripting (XSS) vulnerability in `Util.stripTags` within `src/common/common-util.js`. The function used `div.innerHTML = text` to strip tags which triggers event handlers like `<img onerror=...>` before extraction.
**Learning:** Even if a DOM node is not attached to the document body, setting its `innerHTML` with unsanitized user input immediately triggers the browser to load resources and execute `onerror`/`onload` handlers for things like images, leading to XSS.
**Prevention:** Use `DOMParser` (`new DOMParser().parseFromString(text, 'text/html').body.textContent`) when parsing or stripping tags from untrusted HTML on the frontend instead of `innerHTML`.
## 2024-06-25 - Express Trust Proxy Setting
**Vulnerability:** Express application cannot correctly resolve client IP addresses when behind a reverse proxy.
**Learning:** CryptPad is designed to run behind a reverse proxy and needs `app.set('trust proxy', 1)` to correctly resolve client IP addresses for rate limiting and logging.
**Prevention:** Always ensure `trust proxy` is configured correctly for Express applications intended to be deployed behind reverse proxies.

## 2024-09-02 - [Missing Trust Proxy Configuration]
**Vulnerability:** Client IP addresses are not correctly resolved when the application runs behind a reverse proxy, potentially breaking IP-based security measures like rate limiting.
**Learning:** Express applications in this repository must use a configurable setting (e.g., `trustProxy` from configuration) to safely enable IP resolution behind a reverse proxy, avoiding IP spoofing.
**Prevention:** Make `trust proxy` configurable and default to `false` (disabled) unless explicitly enabled in the environment.
## 2024-10-25 - [Mutation XSS in renderMathjax via .innerHTML]
**Vulnerability:** Found a Mutation XSS (mXSS) vulnerability in `renderMathjax` within `www/common/diffMarked.js`. The function modified an SVG's raw string using Regex (`svg.innerHTML.replace(/xlink:href/g, "href")`) and then directly assigned the string to `.innerHTML`.
**Learning:** Directly manipulating raw HTML strings and assigning them back to `.innerHTML` is a vector for Mutation XSS. When parsing SVGs (or any DOM content), the browser's DOM parser can be tricked by maliciously crafted strings that serialize and deserialize unexpectedly. Furthermore, when selecting attributes with colons like `xlink:href` via `querySelectorAll`, they must be escaped as `[xlink\:href]`; using `[*|href]` throws a `SyntaxError` and crashes script execution.
**Prevention:** Avoid assigning raw manipulated strings to `.innerHTML`. Instead, use safer DOM manipulation methods (`getAttribute`, `removeAttribute`, `setAttribute`, and `.appendChild()`) to interact with DOM nodes directly.
## 2024-11-20 - [DOM-based XSS in TOC via innerHTML]
**Vulnerability:** Found a Cross-Site Scripting (XSS) vulnerability in Table of Contents (TOC) rendering (`www/common/diffMarked.js` and `www/pad/inner.js`). The text processed by `Util.stripTags` was being inserted directly into the DOM using `.innerHTML`.
**Learning:** `Util.stripTags` returns a string that may contain unescaped HTML characters (like `<` or `>`) if the DOM parser decides they are just text nodes. When this raw string is passed back to `.innerHTML` in another context, it can be parsed as HTML again, resulting in DOM-based XSS.
**Prevention:** Always use `.textContent` (or equivalent safe manipulation like `document.createTextNode`) instead of `.innerHTML` when inserting text into a DOM node, especially after it has been stripped or processed.
