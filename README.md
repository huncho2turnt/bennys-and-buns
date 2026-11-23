# Benny's & Buns — Netlify POS + Discord receipts

This Netlify edition uses **Netlify Functions + Netlify Blobs** for a small, staff-operated POS website. No FiveM and no VPS required. It is not a card processor or complete accounting system.

## Deploy (GitHub recommended)
1. Create a **private GitHub repository** and upload the contents of this folder (not the ZIP as a single file). Keep `.env` and secrets out of the repository.
2. In Netlify, choose **Add new project > Import an existing project > GitHub** and select the repository.
3. Netlify reads `netlify.toml`; confirm build command `npm run build`, publish directory `public`, functions directory `netlify/functions`.
4. In Netlify **Project configuration > Environment variables**, add these **server-side** values (set scope to Functions or All):
   - `POS_ACCESS_PIN`: your private staff PIN, **8+ unpredictable characters**
   - `COOKIE_SECRET`: **32+ random characters** (generate with `node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"`)
   - `DISCORD_WEBHOOK_URL`: Discord channel webhook URL, **never put this in public HTML or GitHub**
   - `TAX_RATE`: use `0` for no tax or e.g. `0.0825` for 8.25%, only after verifying applicable tax requirements
5. Redeploy from Netlify after setting environment variables so the Functions use them.
6. Open the Netlify URL, sign in with staff name + PIN, place a **test** order, and verify that a receipt appears in your Discord channel and Sales Reports.
7. For easy team access, share your Netlify website URL in a private staff Discord channel. **Do not share the staff PIN publicly.**

## Menu
Edit `menu.json`. Prices are expressed in **integer cents**, e.g. 1099 is $10.99. Commit/push to GitHub to redeploy.

## Known limitations (important)
- **Payments are recorded, not processed/verified**. Cash, external card terminal, and other methods require manual staff confirmation.
- Shared PIN login is **starter-only authentication**, not individual staff authorization. The in-memory rate limit is per serverless instance and not effective as a production security boundary.
- Netlify Blobs is durable key-value storage and is suitable for a **small pilot**, but reporting scans all receipts and is capped at **3,000 receipts**. For serious usage, migrate records to a transactional database (e.g. Netlify DB/Postgres), add individual accounts, durable rate limiting, backups, returns/refunds, and audit logs.
- Reports use **UTC dates**, not a local restaurant timezone.
- A network retry can create a second order; staff should check Sales Reports before resubmitting a timed-out checkout.
- Discord webhook delivery can fail even if an order was saved; see the receipt's status and use **Resend**. The system does not guarantee exactly-once webhook delivery.
- Netlify Blobs site-wide stores may be shared across deploy contexts; do not perform test sales on preview deployments that point at production data.
- Netlify hosting, functions, and storage may have usage limits/charges. Check your plan before launch.

## Files
- `public/`: original Benny's & Buns browser POS
- `netlify/functions/api.mjs`: authentication, checkout, reports, Discord receipts
- `menu.json`: editable menu
- `netlify.toml`: Netlify build and URL routing
- `.env.example`: required configuration names (never commit real values)
