# Privilege Escalation via a Flawed Login State Machine

- **Vulnerability class:** Business-logic flaw / authentication bypass (CWE-840, CWE-287)
- **Target:** PortSwigger Web Security Academy, *Authentication bypass via flawed state machine* (a deliberately-vulnerable lab)
- **Impact:** An unprivileged user reaches the administrator role and the admin interface.

---

## Summary
The login flow was a two-step state machine: `POST /login` authenticated the user, then
`GET /role-selector` assigned the user's role before the home page loaded. The session's role
**defaulted to `administrator`** and was only lowered once the role-selector step ran. By
dropping `GET /role-selector` so it never reached the server, the session kept its privileged
default and I browsed straight to the application as an administrator.

## Walkthrough
1. **Mapped the state machine.** A normal login issued `POST /login`, followed by
   `GET /role-selector` (role assignment), and then the home page.
2. **Located the target.** Content discovery revealed `/admin`. Browsing to it directly from
   the role-selection page was denied, so the control was enforced at that point in the flow.
3. **Dropped the role-assignment step.** With Burp Proxy intercept on, I **forwarded**
   `POST /login` but **dropped** the subsequent `GET /role-selector`, then requested the home
   page directly.
4. **Inherited the privileged default.** Because the role-assignment step never ran, the
   session retained its default role of `administrator`. The admin interface was accessible,
   and I performed a privileged action to confirm the escalation.

## Impact
Any user who can authenticate at all can reach the administrator role by skipping a single
request, resulting in complete compromise of administrative functionality and data.

## Root cause
The flow assumed the role-assignment step would **always** run before the user reached any
protected page, so the session was initialized with a **privileged default role** that the
role-selector was expected to overwrite. Skipping that step left the session in its elevated
initial state, a configuration the design never anticipated.

## Remediation
- Default sessions to **least privilege**; never initialize a session to a privileged role that
  a later step is expected to downgrade.
- Enforce the authentication state machine **server-side**: verify that each required step has
  been completed before granting access to any protected resource, rather than assuming the
  client follows the intended sequence.

## Methodology takeaway
In any multi-step authentication or onboarding flow, intercept the sequence and try **dropping
or skipping an intermediate step**, then request a protected resource. If the session carries a
**privileged default** that the skipped step was meant to set or lower, skipping it leaves you
elevated. Dropping a request so it never reaches the server is a distinct technique from
replaying or reordering requests, and it is often what drives a state machine into a state the
developer never pictured.
