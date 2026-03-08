<!-- [agent: content] Generated from SPEC-003 -->

## LinkedIn / X Post

🔒 Shipped a security hardening pass on our auth system today.

Ran a full OWASP audit against our FastAPI + Next.js authentication — JWT refresh rotation, bcrypt, httpOnly cookies, rate limiting — and still found 7 exploitable gaps.

The scariest one: password reset tokens never invalidated each other. Request token A, request token B, use B — token A still works.

Other fixes:
• Login timing leak (bcrypt skip on nonexistent users → enumeration)
• CSRF accepted `same-site` instead of `same-origin`
• Rate limiting silently degraded when Redis went down
• Reset tokens exposed in GET query strings

None of these are novel. They're all in the OWASP cheatsheets. The takeaway: even "good enough" auth has gaps when you actually audit it.

#security #owasp #fastapi #nextjs #authentication
