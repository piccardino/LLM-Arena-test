# WollyTCG production architecture

The GitHub Pages build is the public, deployable exhibition prototype. Pages is static hosting and cannot securely host authentication, mutation endpoints, transactional stock, uploads, or payment secrets. Those concerns must not be faked in client code.

## Production boundary

- **Frontend:** current React/Vite experience; SSR migration to Next.js recommended for per-card OpenGraph metadata.
- **API:** TypeScript service with Zod validation, strict CORS, CSRF-protected cookie sessions, request throttling and role checks on every admin mutation.
- **Database:** PostgreSQL. Tables: `cards`, `card_images`, `sets`, `tags`, `card_tags`, `admin_users`, `sessions`, `orders`, `order_items`, `inventory_reservations`, `audit_logs`, `site_settings`.
- **Stock:** checkout transaction locks card rows, validates availability, creates a time-limited reservation, and decrements only after a verified payment webhook. Idempotency keys protect checkout and webhooks.
- **Auth:** no registration route. Seed/invite admins out-of-band; Argon2id hashes, secure HttpOnly SameSite cookies, session rotation, optional TOTP.
- **Storage:** private S3-compatible originals plus public immutable optimized AVIF/WebP derivatives. Server validates MIME via magic bytes, dimensions and size; strips metadata; never executes uploads.
- **Payments:** `PaymentProvider` interface implemented server-side. Stripe PaymentIntents/Checkout may implement it. Webhook signatures are verified; secrets never enter Vite variables.

## Image pipeline

Upload front, back, or detail image using presigned URLs granted after authorization. Record pending image, scan server-side, generate responsive derivatives, then atomically publish. Keep originals private and lossless. Apply CDN cache headers to content-addressed derivatives.

## Static demo limitations

Cart and wishlist intentionally use local storage. Checkout displays a clear demo message. External sample scans are visual fixtures and should be replaced by seller-owned scans before commercial launch. A genuinely protected `/admin` is intentionally not shipped on GitHub Pages because client-only protection is not security.
