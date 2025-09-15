# Doctolib-RPPS-ID-Validation-Bypass
🩺 Doctolib RPPS ID Bypass — The One Where I Trick the Signup Flow (Kind of)

TL;DR — I fed the signup flow bad RPPS, bad email, and bad phone. Then I whispered sweet nothings at the server, changed its reply from “nope” to “sure”, and the app politely let me continue.
No, I didn’t break the whole internet. Yes, it’s a problem. Yes, it was reported responsibly.

🕵️‍♂️ The Plot

Imagine showing up at a hospital with a fake doctor ID, the receptionist shrugs, and says:

“Fine. Fill this form. We’ll trust you… for now.”

That’s basically what happened.

While testing Doctolib’s professional signup (PRO SANTE CONNECT), I noticed the RPPS (doctor ID) check was easy to sidestep by tampering with the server response. Even when I entered obviously wrong RPPS, email, and phone, I could intercept the response, change the server’s “unknown user” into a friendly “200 OK” and continue to the next step — the e-CPS activation request page.

Spoiler: the activation button stayed blocked in my tests, but the fact the client happily progressed is a big integrity problem.

⚙️ What went wrong (in plain English)

The site asks: “Is this RPPS valid?”

The server replies: “Nope — unknown user.” (status 400)

I say: “Hold my Burp.” I change the response to 200 OK.

The client believes the lie and moves on like nothing happened.

In other words: the app trusts the client-side state and accepts forged server responses. That’s like letting someone change the road signs and then driving off a cliff because the sign said “bridge.”

🧪 PoC — the tiny lie that fooled the client

Original error response

HTTP/1.1 400 Bad Request
Content-Type: application/json

{"code":"BUSI-COMMON-1","message":"unknown user"}


Forged “all good” response

HTTP/1.1 200 OK
Content-Type: application/json

{"code":"","message":""}


Change the reply. Forward. The client keeps going. Magic? No — broken validation.

⚠️ Note: PoC uses sanitized data. Do not test on real user accounts or disclose PII.

🎯 Impact (aka why this is not just a prank)

High — Account Takeover & Impersonation: If an attacker knows (or guesses) a target’s email & phone, they can leapfrog RPPS checks and impersonate them in flows intended only for verified practitioners.

High — Security Bypass: Client trusts forged success responses. That’s a huge trust problem.

Medium — Data Manipulation: Reaching later steps could allow tampering with account info or creating fraudulent requests.

Medium — Escalation: Combined with other flaws, this could lead to greater access.

In short: small tampering → big logical confusion.

🛠️ How to actually fix it (not with duct tape)

Never trust the client. Validate RPPS ID, email, and phone server-side before allowing progression.

Use server-side session state / signed tokens to record verified steps. The client should only be able to show the server its token — not invent the token.

Reject forged responses. If the server says “unknown user”, the client must not accept a later-forged 200 OK for that check.

2FA / out-of-band checks for sensitive flows (e.g., e-CPS activation).

Logging & alerting: flag repeated attempts to progress after verification failures.

Rate-limit & captcha where appropriate to slow down automated tampering.

Fix these and the receptionist will stop trusting people named “TotallyRealDoctor”.

✅ Responsible disclosure

Reported to Doctolib (date: insert date here).

I used only test data / my own identifiers for demos. No patient PII was accessed or leaked.

If you’re reading this and work at Doctolib: hi! friendly bug hunter here — reach out if you want sanitized PoC materials.

🧾 Final thoughts (funny but serious)

This wasn’t a Hollywood heist. No dramatic takeover. No fireworks. Just a subtle business-logic failure: the app trusted responses it shouldn’t have trusted. That kind of mistake compounds quickly in critical domains like healthcare.

So yes — I nudged the flow, the app got confused, and the e-CPS button laughed at me by remaining blocked. But the underlying vulnerability was real: don’t let clients tell you your users are who they claim to be.
