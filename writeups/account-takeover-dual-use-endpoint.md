# Account Takeover via Weak Isolation on a Dual-Use Endpoint

- **Vulnerability class:** Broken access control / business-logic flaw (CWE-639, CWE-840)
- **Target:** PortSwigger Web Security Academy, *Weak isolation on a dual-use endpoint* (a deliberately-vulnerable lab)
- **Impact:** Full account takeover of any user, including the administrator.

---

## Summary
A single `POST /my-account/change-password` endpoint served two use-cases: a user changing
their own password, and (by design) an administrator changing another user's. It decided
*whose* password to change from a client-supplied `username` field. Combined with a second flaw,
where the current-password check was skipped whenever the parameter was absent, this allowed an
unprivileged user to set the password of any account, including `administrator`, and take it
over.

## Walkthrough
1. Authenticated as a low-privilege user and performed a normal password change, capturing the
   request in Burp:

   ```http
   POST /my-account/change-password
   username=wiener&current-password=peter&new-password-1=x&new-password-2=x
   ```
2. **Flaw 1, missing-parameter handling.** Removing the `current-password` parameter entirely
   (not just blanking it) caused the server to **skip the current-password check**. The password
   changed successfully with no credential verification, because the validation branch only ran
   when the parameter was present.
3. **Flaw 2, weak isolation / trusted client identity.** The account modified was determined
   solely by the `username` field in the body, with no authorization check that the requester
   actually *was* that user.
4. Combining both, I replayed the request in Repeater as the low-privilege user:

   ```http
   POST /my-account/change-password
   username=administrator&new-password-1=hijacked&new-password-2=hijacked
   ```
   The administrator's password was changed to a value I chose. I logged in as
   `administrator`, reached the admin panel, and performed a privileged action, confirming
   full takeover.

## Impact
An unauthenticated-to-low-privilege attacker can take over **any account on the application**,
including administrators, leading to complete compromise of application data and functionality.
High severity.

## Root cause
Two independent weaknesses that are dangerous together:
- The current-password check was **conditional on the parameter's presence**, so omitting it
  bypassed verification.
- One endpoint served two trust contexts, and it **trusted a client-controlled identity field**
  to choose the target account instead of deriving it from the authenticated session.

## Remediation
- **Derive the target account from the authenticated session**, never from a client-supplied
  `username`/`id`. If an admin-on-behalf-of flow is required, isolate it behind a distinct,
  authorization-checked endpoint.
- **Require and validate the current password on every self-service change**, and treat a
  missing parameter as a failed check (deny by default) rather than a skipped one.

## Methodology takeaway
The reusable rule I apply to **any** `change-password`, `change-email`, or profile-update
request:
1. **Try removing** the current-password / confirmation parameter (name *and* value). Many
   apps only enforce the check when the field is present.
2. **Try swapping** the `username`/`id` to another user. Dual-use endpoints frequently trust a
   client-supplied identity with no real authorization.

Either flaw alone is a weakness; stacked, they are account takeover. This is a recurring
anti-pattern wherever one endpoint serves both "self" and "admin" use-cases.
