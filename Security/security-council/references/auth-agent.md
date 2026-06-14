# Auth & Session Agent — Reference

## JWT checklist
- [ ] Algorithm explicitly set to `RS256` or `HS256` — never `none`, never left to default
- [ ] Secret/key is strong — min 256 bits for HS256, RSA 2048+ for RS256
- [ ] `exp` claim present and enforced — tokens expire
- [ ] `iss` (issuer) validated on every token check
- [ ] `aud` (audience) validated where applicable
- [ ] Token not stored in `localStorage` (XSS-accessible) — prefer `httpOnly` cookie
- [ ] Refresh token rotation in place — old refresh token invalidated on use
- [ ] Token revocation mechanism exists for logout

## OAuth checklist
- [ ] `state` parameter used and validated — prevents CSRF
- [ ] Authorization Code flow used (not Implicit flow)
- [ ] `redirect_uri` strictly validated — no wildcard matching
- [ ] PKCE used for public clients (SPAs, mobile)
- [ ] Token not passed in URL fragments beyond the auth callback

## Session checklist
- [ ] Session ID regenerated after successful login
- [ ] Session invalidated on logout (server-side)
- [ ] Session cookie: `httpOnly`, `Secure`, `SameSite=Strict` or `Lax`
- [ ] Session timeout enforced (idle + absolute)
- [ ] Concurrent session handling defined

## Password handling checklist
- [ ] Passwords hashed with bcrypt, scrypt, or Argon2 — never MD5, SHA1, SHA256 alone
- [ ] Salt applied (bcrypt/Argon2 do this automatically)
- [ ] Password strength enforced at input
- [ ] Password reset tokens: cryptographically random, single-use, short-lived (<1hr)
- [ ] Account lockout or progressive delay after failed attempts

## Authorization checklist
- [ ] Every protected route has server-side auth check — not just client-side redirect
- [ ] Role/permission checks happen server-side on every request
- [ ] Admin functions require explicit admin role check — not just "is logged in"
- [ ] Ownership checks on all resource access — user can only access their own data
