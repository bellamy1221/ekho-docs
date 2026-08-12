**Status:** LOCKED
**Scope:** consumer authentication and complete account lifecycle for Ekho
**Stack:** Next.js + Supabase Auth + PostgreSQL/RLS
**Research verified:** 12 August 2026
---
# 1. Goal
Authentication in Ekho must be:
* extremely fast;
* safe enough for personal admissions data and private documents;
* compatible with Google, Apple and email/password;
* usable from multiple devices;
* recoverable;
* revocable;
* resistant to account enumeration, brute force and automated abuse;
* independent from application profile data;
* simple enough that authentication never becomes an onboarding questionnaire.
Authentication answers only:
> **Who is this user and is this session valid?**
It must not contain admissions personalization logic.
---
# 2. Fixed architecture
Use:
* **Supabase Auth** as the only authentication provider.
* `auth.users` as the canonical authentication identity.
* UUID `auth.users.id` as the permanent Ekho user identifier.
* `public.profiles.id -> auth.users.id` as the application-level user record.
* PostgreSQL RLS as the final data-authorization boundary.
* `@supabase/ssr` for Next.js server/client authentication.
* **PKCE** for browser OAuth flows.
* Google OAuth.
* Sign in with Apple.
* email + password.
* email ownership verification.
* custom production SMTP.
* refresh-token rotation.
* multiple concurrent sessions.
Do **not** build:
* custom password storage;
* custom JWT signing;
* custom OAuth protocol;
* custom session token format;
* phone authentication in v1;
* username/password authentication;
* security questions.
Supabase supports password auth, social OAuth and PKCE server-side flows; refresh tokens are rotated and access sessions are represented by short-lived JWTs.
---
# 3. Authentication UX
Primary authentication methods:
1. **Continue with Google**
2. **Continue with Apple**
3. divider: `or`
4. email
Do not ask for:
* first name;
* surname;
* nationality;
* school;
* country;
* intended degree;
* graduation year;
* phone;
* username
during authentication.
Those belong to progressive profiling.
After successful authentication:
* return the user to the action/page that caused authentication;
* otherwise go to the normal authenticated Ekho entry point;
* never send the user through a mandatory profile questionnaire.
---
# 4. Public vs authenticated product
University research and other intentionally public Ekho pages should remain usable without authentication.
Require authentication only when persistence or private state is needed, for example:
* save university;
* create an application;
* personalize requirements;
* create tasks;
* upload documents;
* save notes;
* manage account.
This preserves Ekho's immediate-value principle.
---
# 5. Routes
Logical auth routes:
* `/login`
* `/signup`
* `/verify-email`
* `/forgot-password`
* `/reset-password`
* `/auth/callback`
* `/auth/error`
Account routes:
* `/settings/account`
* `/settings/security`
Do not create a complex standalone authentication dashboard.
---
# 6. Safe post-auth redirects
Support:
`?next=/some/internal/path`
After authentication, restore the user's intended action.
Rules:
* only relative internal paths beginning with `/`;
* reject absolute URLs;
* reject protocol-relative URLs such as `//evil.com`;
* never accept an arbitrary external redirect target;
* fallback to the normal Ekho authenticated entry point.
Supabase's current Next.js OAuth callback example explicitly validates that `next` is a relative path before redirecting.
---
# 7. Email/password signup
Flow:
`email + password`
→ validate
→ create Supabase user
→ send verification email
→ show verification state
→ user verifies
→ account becomes usable
→ continue original Ekho flow
Email/password users must verify email ownership before access to private authenticated workspace.
Supabase hosted projects support confirmation-before-sign-in, and OWASP recommends verifying ownership before activating an email-based identity.
---
# 8. Email normalization
Never perform provider-specific tricks such as:
* removing dots from Gmail addresses;
* manually interpreting `+alias`;
* assuming two visually similar addresses belong to the same user.
Rules:
* preserve original email for display;
* treat the domain case-insensitively;
* rely on Supabase identity handling for authentication;
* never use email as a relational primary key;
* all internal ownership relationships use `user_id UUID`.
OWASP specifically warns against inconsistent or provider-specific email canonicalization because it can create identity collisions and account-takeover risks.
---
# 9. Password policy
Ekho password policy:
**Minimum:** 15 characters
**Maximum accepted:** at least 64 characters; Ekho UI should permit up to 128.
Do:
* allow spaces;
* allow Unicode;
* allow paste;
* allow password managers;
* allow browser autofill;
* provide show/hide password;
* optionally show a strength indicator.
Do not require:
* uppercase;
* lowercase;
* digit;
* symbol;
* periodic password changes.
NIST SP 800-63B requires at least 15 characters where a password is the single authentication factor, recommends accepting at least 64 characters, prohibits arbitrary composition rules and periodic mandatory password rotation, and recommends password-manager/paste support.
---
# 10. Leaked passwords
On production Supabase Pro:
**Enable Leaked Password Protection.**
Supabase can reject passwords appearing in the Have I Been Pwned Pwned Passwords dataset. This feature is currently available on Supabase Pro and above.
Do not independently send users' plaintext passwords to another Ekho service.
---
# 11. Google authentication
Use Supabase Google provider.
Request only identity information necessary for authentication:
* `openid`
* email
* basic profile identity
Do not request:
* Google Drive;
* Gmail;
* Calendar;
* Contacts;
* YouTube;
* other unrelated scopes.
Google's authentication implementation is OpenID Connect; Ekho does not need broader Google API authorization merely to identify a user.
---
# 12. Apple authentication
Use Supabase Sign in with Apple OAuth flow for the web.
Required setup includes:
* Apple Developer account;
* primary App ID with Sign in with Apple;
* Services ID;
* verified web configuration;
* signing key;
* Supabase Apple provider configuration.
Apple OAuth secrets used for web authentication require rotation every **6 months**. Missing this rotation can break Apple login, so it must become an operational recurring task.
Do not require a user's real Apple email.
Apple may provide a private relay address.
Do not break account functionality because a user chose **Hide My Email**.
---
# 13. Apple name handling
Do not depend on Apple for the user's name.
In Apple's OAuth web flow, the full name is not reliably available; Apple provides name information only in specific first-authorization/native flows.
Therefore:
* authentication must succeed without a name;
* `display_name` may remain null;
* ask for a preferred name later only if a feature actually needs it.
---
# 14. Identity linking
Supabase automatically links OAuth identities that use the same verified email to one user, reducing duplicate accounts. It deliberately avoids unsafe linking involving unverified identities.
Ekho v1 rule:
* rely on Supabase automatic same-email linking;
* do not build custom account-merging logic;
* do not merge users using email-string comparison in application code.
Supabase manual identity linking is currently documented as **beta**.
Therefore manual merging of accounts with different email addresses is **not part of Ekho v1**.
---
# 15. Email verification
Email verification links:
* expire after **1 hour**;
* are single-use;
* may be resent;
* resend cooldown minimum **60 seconds**;
* expired/used links show a clear recovery action.
Supabase's Email OTP expiration setting also governs confirmation, recovery and email-change links; its default/recommended production value is 3600 seconds or less.
Verification page states:
* `waiting`
* `verified`
* `expired`
* `invalid`
* `rate_limited`
* `delivery_failed`
Never expose the underlying token.
---
# 16. Production email
Do not use Supabase's built-in SMTP service in production.
Supabase currently describes it as best-effort and heavily rate-limited; production applications should configure custom SMTP.
Production auth email sender should use an Ekho-controlled domain.
Auth email templates required:
* confirm signup;
* reset password;
* change email;
* reauthentication.
Enable security notifications for:
* password changed;
* email changed;
* sign-in method linked;
* sign-in method removed;
* MFA factor changes if MFA is added later.
Supabase provides these security notification templates.
---
# 17. Session architecture
Use Supabase access + refresh token sessions.
Fixed Ekho settings:
* access-token JWT expiry: **1 hour**;
* refresh-token rotation: **enabled**;
* default refresh-token reuse window: leave Supabase default;
* multi-device sessions: **allowed**;
* single-session-per-user mode: **disabled**.
Supabase recommends a one-hour JWT expiry for most applications and rotates refresh tokens with reuse detection.
---
# 18. Session lifetime
Production baseline:
* inactivity timeout: **30 days**;
* absolute session timebox: **90 days**;
* after either limit: require authentication again.
These exact durations are **Ekho product/security decisions**, not external standards.
Why:
* admissions is not a banking workflow;
* forcing daily/weekly logins would damage UX;
* leaving sessions indefinitely active on abandoned devices is unnecessary risk.
Supabase provides inactivity and time-box session controls on Pro plans.
---
# 19. Cookie/session implementation
Use:
`@supabase/ssr`
Supabase SSR stores session data in cookies and uses PKCE.
Cookie baseline:
* HTTPS only in production;
* `Secure`;
* `SameSite=Lax` unless a tested auth flow requires otherwise;
* sensible domain/path scoping;
* never expose tokens in URLs, analytics or application logs.
### Important correction to Security & Privacy
Do **not** force `HttpOnly` on Supabase access/refresh-token cookies under Ekho's rich client-side Next.js architecture.
Current Supabase documentation explicitly states that the browser client needs access to refresh tokens to maintain this type of session; HttpOnly token storage is suited to a traditional server-only application.
Therefore update the earlier blanket rule:
> `Auth cookies must always be HttpOnly`
to:
> `Server-only cookies must use HttpOnly where applicable. Supabase SSR auth-token cookies follow the official @supabase/ssr storage model and must not be forced HttpOnly if that prevents client-side refresh.`
---
# 20. CDN/cache security
Never cache a personalized authenticated response containing a refreshed auth cookie as shared/public CDN content.
Protected personalized pages should not use public shared caching.
Responses containing sensitive authenticated information should use appropriate private/no-store semantics.
Supabase warns that caching an SSR response containing `Set-Cookie` can result in another visitor receiving the cached authentication token.
This is a **P0 security requirement**.
---
# 21. Server-side identity validation
For protected Next.js server routes/pages:
use:
`supabase.auth.getClaims()`
for verified JWT identity.
Use:
`supabase.auth.getUser()`
when an up-to-date Auth user record is specifically required.
Do **not** use server-side:
`getSession()`
as proof of identity/authorization.
Supabase explicitly warns that session data read directly from cookies/storage can be spoofed and recommends `getClaims()` for page/data protection.
---
# 22. Authorization is not authentication
Authentication proving:
> user is X
does **not** imply:
> user X may access row Y.
All private user-owned PostgreSQL tables remain protected by RLS.
Every private ownership relation uses:
`user_id = auth.uid()`
according to the separately locked Security & Privacy / Data Architecture policies.
No client-side route guard can replace RLS.
---
# 23. Sign out
Normal `Log out` means:
**current device/session only.**
Call explicitly:
`signOut({ scope: 'local' })`
Do not call `signOut()` without a scope for normal logout because Supabase currently defaults to **global**, which would unexpectedly log the student out from every device.
After local logout:
* clear local user state;
* clear private query caches;
* clear user-specific client stores;
* redirect to public Ekho.
---
# 24. Multi-device behavior
Users may remain authenticated simultaneously on:
* laptop;
* phone;
* tablet;
* additional browser.
Do not enforce one-device-only authentication.
Admissions workflows naturally cross devices and there is no security justification for unnecessary single-session restriction.
---
# 25. Session revocation controls
Settings → Security must support three operations:
### Log out this device
`scope: local`
### Log out other devices
`scope: others`
### Log out everywhere
`scope: global`
Supabase officially supports these three revocation scopes.
---
# 26. Specific device revocation
Do **not** promise:
> "Log out only this particular remote MacBook/iPhone"
in v1.
Supabase's public Auth API exposes `local`, `others` and `global` session revocation scopes, but not a supported end-user API for revoking one arbitrary remote session by session ID.
Therefore v1 supports:
* current;
* all others;
* everywhere.
Do not mutate Supabase-managed `auth.sessions` directly to fake this feature.
---
# 27. JWT revocation limitation
Logging out destroys/revokes affected refresh sessions, but a previously issued access JWT can remain cryptographically valid until its `exp`.
Supabase explicitly documents this behavior.
Therefore:
* access JWT expiry stays at one hour;
* high-risk server actions must additionally verify that the JWT's `session_id` still represents a live session.
High-risk examples:
* delete account;
* change email;
* change password;
* delete private documents;
* security settings;
* future financial/billing-sensitive actions.
Supabase specifically recommends `session_id` validation when immediate post-revocation guarantees are necessary.
---
# 28. Password recovery
Flow:
`Forgot password`
→ enter email
→ always show generic success response
→ send reset link if account exists
→ user opens reset route
→ validate recovery session
→ set new password
→ terminate/refresh affected session state
→ security notification
UI response:
> If an account exists for this email, we've sent password reset instructions.
Never:
> No account exists with this email.
OWASP recommends consistent reset responses to prevent account enumeration and recommends rate limiting reset requests.
---
# 29. Recovery token security
Recovery links must be:
* random/cryptographically generated by auth provider;
* time-limited;
* single-use;
* HTTPS;
* rate-limited;
* restricted to allowlisted Ekho redirect URLs.
Do not log:
* recovery token;
* full reset URL;
* callback query string containing secrets.
OWASP explicitly requires secure, time-limited, single-use reset identifiers and warns against leaking reset tokens via logs/referrers.
---
# 30. Password change
Settings → Security → Change password.
Require:
* active authenticated session;
* current password where the account has a password identity;
* or Supabase secure reauthentication when appropriate.
Enable Supabase **Secure password change**.
Supabase provides both reauthentication nonce support and current-password verification for password changes.
After password change:
* show success;
* send security notification;
* ensure stale sessions behave according to Supabase security-sensitive-session invalidation rules.
---
# 31. Reauthentication / step-up authentication
Require recent reauthentication before:
* changing email;
* changing password;
* deleting account;
* revoking all sessions;
* changing security-critical authentication identities.
OWASP recommends reauthentication after or around high-risk events including email changes, password changes and account recovery.
Do not treat:
* opening the settings page;
* clicking a modal;
* knowing the user ID
as reauthentication.
---
# 32. Email change
Changing email is an identity-security operation.
Flow:
`new email`
→ reauthenticate
→ request change
→ confirmation sent to current email
→ confirmation sent to new email
→ both confirmations complete
→ primary email changes
→ security notification
→ continue session or reauthenticate according to risk state
Keep Supabase **Secure Email Change enabled**.
Supabase sends confirmation to both the old and new address by default when secure email change is enabled. OWASP likewise recommends reauthentication, notification to the existing address and confirmation of the new address.
---
# 33. Never use email as ownership identity
Changing:
`old@example.com`
to:
`new@example.com`
must change **zero** ownership foreign keys.
Everything belongs to:
`user_id UUID`
not email.
This applies to:
* applications;
* tasks;
* requirements;
* saved universities;
* notes;
* uploaded documents;
* personalization;
* subscriptions later.
---
# 34. Account deletion UX
Settings → Account → **Delete account**
It must be easy to find.
Flow:
1. user selects Delete account;
2. explain exactly what is deleted;
3. require reauthentication;
4. final destructive confirmation;
5. immediately remove usable account access;
6. execute deletion pipeline;
7. revoke provider access where required;
8. delete Supabase auth user;
9. clear local session/client state;
10. return to public Ekho.
No hidden support-ticket-only deletion flow.
Apple requires apps that support account creation to permit users to initiate account deletion inside the app; temporary deactivation alone is not sufficient.
GDPR Article 17 also establishes a right to erasure where its conditions apply.
---
# 35. Account deletion order
Deletion must be server-controlled.
Recommended order:
1. revalidate authenticated identity;
2. create internal deletion operation ID;
3. prevent further sensitive mutations;
4. revoke external provider access where required;
5. delete user's private Storage objects;
6. remove/anonymize user application data according to the Privacy/Data Retention specification;
7. delete `auth.users` using server-side admin credentials;
8. invalidate refresh sessions;
9. clear local client data;
10. record non-sensitive completion audit event.
Before deleting a Supabase Auth user, handle the user's private Cloudflare R2 objects and their Ekho metadata according to the document-deletion policy.
---
# 36. Supabase user deletion
Final Auth deletion must happen from trusted server code with:
`auth.admin.deleteUser()`
Never expose the server secret/service credential to the browser.
Do not implement "deleted" only as:
`profiles.deleted = true`
because the existing `auth.users` account could still authenticate.
Supabase explicitly distinguishes actual Auth-user deletion from app-level flags or temporary bans.
---
# 37. Apple deletion requirement
If an account used Sign in with Apple:
Ekho must revoke the associated Apple authorization/token during account deletion.
Apple explicitly requires this for apps supporting Sign in with Apple.
This must be covered before any native iOS/App Store launch.
---
# 38. Database relationship
Application profile:
```text
public.profiles
  id uuid primary key
     references auth.users(id)
     on delete cascade
```
Do not duplicate password hashes, auth tokens or OAuth secrets into `profiles`.
Supabase recommends referencing `auth.users` by its primary key and protecting application-level profile tables with RLS.
---
# 39. Signup profile creation
Create the minimal `profiles` record automatically after successful user creation.
Minimal data:
```text
id
created_at
updated_at
display_name nullable
avatar_url nullable
locale nullable
```
Do not copy unnecessary provider metadata.
Any signup-trigger database function must be extremely small and heavily tested: Supabase warns that a failing user-creation trigger can block signups entirely.
---
# 40. User enumeration protection
Unauthenticated users must not be able to determine whether an email belongs to an Ekho account through differences in:
* signup response;
* login response;
* password-reset response;
* resend-verification response;
* response timing where practical.
Examples:
Bad:
> No user with this email.
Good:
> If an account exists, we'll send instructions.
OWASP explicitly recommends consistent responses and avoiding meaningful timing differences between registered and unregistered emails.
---
# 41. Login errors
Email/password login should return one generic credential error:
> Email or password is incorrect.
Do not separately expose:
* email does not exist;
* password incorrect.
Other errors may remain specific where they do not enumerate accounts:
* network unavailable;
* OAuth cancelled;
* OAuth provider unavailable;
* rate limited;
* email not yet verified;
* session expired.
---
# 42. Abuse protection layers
Auth abuse protection must use defense in depth:
1. Supabase Auth rate limits;
2. account-based throttling;
3. IP-based throttling;
4. progressive delay;
5. Cloudflare Turnstile/risk challenge when suspicious;
6. logging/monitoring.
OWASP notes that IP-only rate limiting can be bypassed by distributed credential-stuffing networks and recommends account-aware throttling.
Supabase supports built-in Auth rate limiting and CAPTCHA integration.
---
# 43. Ekho application-level rate limits
These are **launch defaults**, configurable centrally rather than hard-coded throughout UI code.
### Password login
Per normalized account identifier:
* 5 failures → start progressive delay;
* 10 failures / 10 min → temporary throttle.
Per IP:
* approximately 30 attempts / 10 min before stronger challenge/throttle.
Do not permanently lock accounts.
### Signup
Per email:
* max 5 attempts/hour.
Per IP:
* max 20 account creations/hour before challenge/block.
Shared school/university networks make overly strict IP-only limits unsafe.
### Password reset
Per email:
* max 3 sends/hour.
Per IP:
* max 10/hour.
Always return generic response.
### Verification resend
* minimum 60-second cooldown;
* max 5/hour/user.
### Email change
* max 3/hour/user.
### Account deletion
* no retry loop;
* require valid recent session + reauthentication.
All thresholds must be configurable without a deployment.
---
# 44. Supabase platform rate limits
Do not remove Supabase's own limits.
Current Supabase Auth includes operation-specific limits and returns HTTP `429` when exceeded. Its default resend windows for signup confirmation and password recovery are 60 seconds.
Ekho's limits are an additional application layer, not a replacement.
---
# 45. CAPTCHA / Turnstile
Do not show CAPTCHA to every student by default.
Use Cloudflare Turnstile progressively when:
* repeated failed logins;
* high signup velocity;
* repeated reset requests;
* suspicious automation pattern;
* known abusive IP/risk signal.
This keeps normal signup friction low.
Supabase supports CAPTCHA protection including Cloudflare Turnstile.
---
# 46. Security event logging
Use Supabase Auth Audit Logs for auth events.
Supabase automatically records authentication events in Auth Audit Logs.
Ekho additionally logs application security actions such as:
* `account_delete_requested`
* `account_delete_completed`
* `sessions_revoked_others`
* `sessions_revoked_global`
* `security_reauth_failed`
Never log:
* passwords;
* access tokens;
* refresh tokens;
* OTP values;
* recovery tokens;
* complete verification/reset URLs.
Mask or pseudonymize email addresses in logs where email is necessary.
OWASP recommends monitoring verification/reset activity while avoiding full email addresses and never logging auth/reset tokens.
---
# 47. Analytics privacy
Product analytics may record:
* `signup_started`
* `signup_completed`
* `login_completed`
* `verification_completed`
* `password_reset_completed`
Optional property:
* `provider = google | apple | email`
Do not send to analytics:
* email;
* password;
* Google/Apple identity ID;
* access token;
* refresh token;
* OAuth authorization code.
Authentication metrics should measure funnel performance, not contain authentication credentials.
---
# 48. Sensitive action invariant
Every high-risk server action follows:
```text
request
→ validate JWT identity
→ validate current/live session when required
→ verify RLS/ownership
→ reauthentication requirement if high-risk
→ execute
→ audit
```
Never:
```text
client says userId = X
→ trust X
→ mutate X's data
```
---
# 49. OAuth callback invariant
OAuth callback must:
* accept only expected auth code parameters;
* exchange code through Supabase;
* validate safe `next`;
* never log authorization code;
* never redirect to arbitrary external origins;
* handle cancellation/error safely.
RFC 9700 is the current OAuth 2.0 Security Best Current Practice and requires modern protections such as PKCE/secure refresh-token handling for public clients.
---
# 50. Error states
Every auth flow must explicitly handle:
* offline;
* network timeout;
* provider cancellation;
* provider error;
* invalid callback;
* expired verification;
* expired recovery link;
* already-used token;
* invalid password;
* unverified email;
* duplicate signup;
* rate limit `429`;
* session refresh failure;
* stale session;
* deleted account;
* SMTP delivery failure;
* unexpected Supabase `5xx`.
Never leave an infinite spinner.
---
# 51. Session refresh failure
If refresh fails because the session is invalid/revoked:
* clear authenticated client state;
* redirect to login;
* preserve safe internal `next`;
* show:
> Your session expired. Sign in again.
Do not show internal Supabase error text to the user.
---
# 52. Loading/auth race handling
Application startup must have explicit states:
```text
unknown
authenticated
unauthenticated
```
Never render protected user data while auth state is still `unknown`.
Avoid UI flashes such as:
`logged out → dashboard → logged out`.
---
# 53. Security settings UX
`Settings → Security` contains only useful controls:
* Password
* Connected sign-in methods
* Log out other devices
* Log out everywhere
Do not build a giant security control panel.
`Settings → Account`:
* Email
* Delete account
---
# 54. Connected identities
Display connected methods where available:
* Google
* Apple
* Email/password
Do not expose provider IDs.
Do not let the user remove their final usable authentication method.
Supabase itself requires at least two linked identities before an identity may be unlinked.
---
# 55. MFA
Consumer v1:
**Do not require MFA.**
Reason:
* large additional signup/login friction;
* Ekho is not a financial transaction platform;
* Google/Apple already give many users stronger upstream authentication.
Architecture must remain compatible with optional Supabase TOTP MFA later.
Supabase supports TOTP MFA and AAL1/AAL2 sessions.
Administrative/operations accounts are outside this consumer flow and should use stronger MFA according to the Security specification.
---
# 56. Explicit v1 exclusions
Do not implement unless separately approved:
* phone authentication;
* SMS OTP;
* magic-link-only authentication;
* anonymous Supabase accounts;
* usernames;
* security questions;
* custom password hashing;
* custom auth JWTs;
* custom OAuth implementation;
* account merging by support script;
* manual different-email identity linking;
* per-remote-device individual logout;
* mandatory end-user MFA;
* biometric identity verification;
* KYC.
---
# 57. Required environment separation
Separate OAuth/Auth configuration for:
* local development;
* staging;
* production.
Do not reuse production secrets in preview environments.
Redirect allowlists must contain only intended environment URLs.
Apple/Google production credentials must not be committed to Git.
---
# 58. Service credentials
Supabase server/admin secret may exist only in trusted server runtime.
Never place it in:
* `NEXT_PUBLIC_*`;
* browser bundle;
* HTML;
* client JavaScript;
* analytics;
* logs.
Browser receives only the appropriate Supabase publishable/public credential.
Administrative user deletion and equivalent privileged operations execute server-side.
---
# 59. Required tests — signup
* [ ] Email signup creates one user.
* [ ] Email cannot access protected workspace before verification.
* [ ] Verification succeeds.
* [ ] Expired verification shows resend.
* [ ] Reused verification link fails safely.
* [ ] Duplicate signup does not enumerate account state.
* [ ] Weak/short password rejected.
* [ ] 15-character valid password accepted.
* [ ] Password manager/autofill works.
* [ ] Google signup works.
* [ ] Apple signup works.
* [ ] Apple Hide My Email works.
* [ ] Missing Apple full name does not block auth.
---
# 60. Required tests — login
* [ ] Correct password authenticates.
* [ ] Incorrect email and incorrect password produce equivalent public errors.
* [ ] Google login returns to intended Ekho route.
* [ ] Apple login returns to intended Ekho route.
* [ ] OAuth cancellation recovers cleanly.
* [ ] External malicious `next` redirect rejected.
* [ ] Session persists across browser restart.
* [ ] Revoked session eventually returns to login.
* [ ] Server protected route uses validated identity.
---
# 61. Required tests — identity linking
* [ ] Email account + Google with same verified email does not create duplicate Ekho profile.
* [ ] Google + Apple same verified email follows Supabase linking behavior.
* [ ] Unverified identity cannot hijack existing verified account.
* [ ] Different-email identities are not automatically merged by Ekho code.
* [ ] Removing last authentication method is impossible.
---
# 62. Required tests — recovery
* [ ] Existing and nonexistent reset-email requests have equivalent public response.
* [ ] Reset link works once.
* [ ] Expired reset link fails.
* [ ] Reused reset link fails.
* [ ] Reset endpoint is rate-limited.
* [ ] New password follows password policy.
* [ ] Password-change security notification is sent.
* [ ] Recovery tokens never appear in logs.
---
# 63. Required tests — email change
* [ ] Reauthentication required.
* [ ] New email confirmation required.
* [ ] Existing email notified/confirmed according to Secure Email Change.
* [ ] User UUID remains unchanged.
* [ ] Applications/tasks/documents remain attached to the same user.
* [ ] Security notification sent.
* [ ] Email-change endpoint is rate-limited.
---
# 64. Required tests — sessions
Use at least two browser profiles/devices.
* [ ] Local logout removes only current session.
* [ ] `others` removes other sessions but keeps current.
* [ ] Global logout removes all refresh sessions.
* [ ] JWT expiry remains 1 hour.
* [ ] Refresh-token rotation works.
* [ ] Stolen/reused refresh-token behavior follows Supabase reuse detection.
* [ ] High-risk endpoint rejects revoked `session_id`.
* [ ] Public cached responses never contain another user's auth cookie.
---
# 65. Required tests — deletion
* [ ] User must reauthenticate.
* [ ] Storage files delete successfully.
* [ ] Application-owned records follow deletion policy.
* [ ] `auth.admin.deleteUser()` completes.
* [ ] Refresh sessions disappear.
* [ ] Existing stale JWT cannot perform sensitive mutation.
* [ ] Client state is cleared.
* [ ] Deleted account cannot refresh/sign in as existing account.
* [ ] Apple authorization is revoked where applicable.
* [ ] No password/token appears in deletion logs.
---
# 66. Required tests — RLS
Create User A and User B.
User A must not:
* read User B applications;
* update User B applications;
* read User B tasks;
* update User B tasks;
* access User B documents;
* modify User B personalization.
Test through the same Supabase interfaces available to the application.
Authentication success must never bypass RLS.
---
# 67. Required tests — abuse
* [ ] Login throttle activates.
* [ ] Reset throttle activates.
* [ ] Verification resend cooldown works.
* [ ] Signup velocity protection works.
* [ ] HTTP `429` is handled gracefully.
* [ ] Turnstile challenge appears only after risk threshold.
* [ ] Shared-IP legitimate user can still recover.
* [ ] Rate limits are configurable.
* [ ] Auth abuse produces monitorable events.
---
# 68. P0 failures
Any of these blocks production release:
* cross-user data access;
* service/admin key exposed client-side;
* arbitrary OAuth redirect;
* cached auth token delivered to another user;
* password/token logged;
* account deletion does not revoke account access;
* email can be changed without identity verification;
* password reset reveals whether account exists;
* RLS missing from user-owned table;
* server authorization trusts unvalidated `getSession()` data;
* Apple account deletion without required token revocation where Sign in with Apple is used.
---
# 69. Implementation order for Codex
Implement in this order:
1. Supabase SSR clients.
2. Auth callback / PKCE.
3. protected-route identity verification.
4. email/password signup.
5. verification.
6. login.
7. Google OAuth.
8. Apple OAuth.
9. profile creation.
10. local logout.
11. other/global revocation.
12. password recovery.
13. password change.
14. email change.
15. security notifications.
16. account deletion.
17. rate limiting.
18. Turnstile.
19. audit/security events.
20. complete auth E2E suite.
21. security review against this document.
Do not redesign unrelated Ekho screens while implementing auth.
---
# 70. Definition of Done
Auth & Account Lifecycle is finished only when:
* Google works;
* Apple works;
* verified email/password works;
* OAuth uses PKCE;
* login/signup do not enumerate users;
* passwords follow the locked policy;
* leaked-password protection is enabled in production when Pro is active;
* production custom SMTP works;
* email confirmation works;
* password reset works;
* email change is secure;
* password change requires appropriate reauthentication;
* multi-device works;
* current/others/global logout works;
* protected server routes validate identity correctly;
* RLS remains enforced;
* auth-token responses cannot be shared-cached;
* deletion removes usable account access and data according to policy;
* Apple authorization is revoked during applicable deletion;
* auth abuse is throttled;
* tokens/passwords never reach logs or analytics;
* all P0 tests pass.
---
# 71. Sources / authority hierarchy
Implementation decisions above were checked primarily against:
1. **NIST SP 800-63B — Digital Identity Guidelines** for passwords, authenticators and sessions.
2. **OWASP Authentication Cheat Sheet** for authentication, enumeration, throttling and reauthentication.
3. **OWASP Session Management Cheat Sheet** for session lifecycle and high-risk reauthentication.
4. **OWASP Forgot Password Cheat Sheet** for secure recovery.
5. **OWASP Email Validation and Verification Cheat Sheet** for verification, email change and enumeration.
6. **IETF RFC 9700 — OAuth 2.0 Security Best Current Practice**.
7. **Supabase Auth / SSR / Sessions / User Management current documentation**.
8. **Google OpenID Connect official documentation**.
9. **Apple Sign in with Apple official documentation and account-deletion requirements**.
10. **EU GDPR Article 17** for applicable right-to-erasure requirements.
