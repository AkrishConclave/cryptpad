## 2026-09-01 - Missing rate limiting on Auth & File Upload endpoints
**Vulnerability:** No rate limit on `/api/auth` (RPC for authentication and commands) or `/upload-blob` (file upload).
**Learning:** Adding brute force protections on top of Express with `helmet` and `express-rate-limit` requires adjusting the rate limit max based on RPC design, and ensuring `trust proxy` is enabled since it's deployed behind reverse proxies. CryptPad specifically requires `X-Frame-Options` to be unset (disabled) when using helmet since it utilizes cross-origin iframes.
**Prevention:** Integrate rate limiting globally on auth/file routes by default and use node security standards (e.g. Helmet), considering proxy constraints.
