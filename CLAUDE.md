# ארנק התווים (tav-wallet)

Personal PWA that shows the balance, redemption codes and purchase history of Rami Levy "התו המלא" digital vouchers in one place. Static site, no backend: the page calls the Multipass API directly from the browser and keeps all data in `localStorage` on the device.

## Files
- `index.html` – the whole app (Hebrew, RTL; CSS and JS inline). Barcode/QR via JsBarcode and qrcode-generator from cdnjs.
- Settings has CSV export (UTF-8 BOM for Excel): all activity rows as returned by the API, and a per-voucher summary (original, balance, sum of purchases). Uses the share sheet on iPhone, download elsewhere.
- `sw.js` – service worker, caches the app shell and libraries for offline use. Never caches the API. Bump `CACHE` and the "גרסה N" label in the footer of `index.html` together when shipping changes.
- `manifest.webmanifest`, `icon.svg` – install-to-home-screen metadata.

## Multipass API (mapped 2026-10)
Base `https://restservices.mltp.co.il/api/MyVouchers/`, all `POST` JSON, `companyId: 1290`. CORS reflects any origin (including `null`), so it works from any host or `file://`.
- `token` / `link` fields take the **full** link URL (`https://mycards.mltp.co.il/<code>`), not the bare code.
- `GetVouchersData {token, companyId}` → `resultInfo.resultId` 0 = OK; 604 = link session expired (needs OTP); 602 = unregistered (Register flow); 505 = invalid token. Returns `vouchers` (active), `emptyVouchers`, `expiredVouchers`, one list for **all** vouchers tied to the phone. Fields: `voucherId` (the redemption code, e.g. `NNNNNNNNN-NNNN`), `currentBudget`, `originalBudget`, `creationDate`, `expirationDate`.
- `SendOtp {link, phone}` → `{isSuccess, token}`; `resultCode` 3 = already valid, 2 = phone mismatch.
- `ValidateOtpMyCards {OtpCode, Link, phone}` → `resultInfo.resultId` 0 + `myCardsUrl` (fresh link, store it as the new latest link); 610 bad code; 402 too many attempts.
- Register path (602): `Register {link, phone}` then `ValidateOtpAndRegister {link, token, phone, OtpCode}`.
- `GetVoucherActivity {token, cardId: voucherId, companyId}` → `vouchers[0].activity[]`: `amount`, `activityDate`, `branchName`, `activityType` (0 = purchase, 1 = load), `activityCancelationDate` (`0001-…` = not cancelled).
- The official site shows each code as a QR code and a CODE128 barcode of `voucherId`.
- `ramilevydigital.mltp.co.il` checks balance by code alone but sits behind reCAPTCHA, so it is not usable from the app.

## Status
- App built and tested end-to-end against the real account (30 vouchers, 50 purchases loaded, scan view works).
- Local preview: `python -m http.server 8765` in this folder, then http://localhost:8765.
- Deployed (2026-10-11): public repo https://github.com/Ronen979/tav-wallet, GitHub Pages from `main` / root at https://ronen979.github.io/tav-wallet/. Pushing to `main` redeploys.
- Repo-local git identity uses the noreply address `199085156+Ronen979@users.noreply.github.com` so the personal email stays out of the public history. `gh` CLI is installed at `C:\Program Files\GitHub CLI\gh.exe` (not on PATH) and logged in as Ronen979.

## Next step
Open https://ronen979.github.io/tav-wallet/ on the phone and add it to the home screen.

## Rules
- Never commit personal data: phone numbers, voucher codes, SMS links.
- Keep the app static (no server). Data lives only on the device.
