# Lab 4827: Cross-Site Scripting (XSS) in a Training Application

A sanitized write-up of a local, authorized training exercise covering reflected, stored, and DOM-based cross-site scripting (CWE-79). Personal identifiers, host and container identifiers, credentials, real organization names, and executable attack examples have been removed or replaced with generic values.

> **Scope:** The exercise was described as running against an isolated, intentionally vulnerable training application on the tester's own machine. This is a learning record, not an assessment of a public or production service.

## Objective

Review the three common XSS categories, observe how unsafe rendering behaves at different training security settings, and document prevention techniques.

| Category | General flow | Defensive focus |
|---|---|---|
| Reflected | Untrusted request data is included in the immediate response. | Encode data for its output context before rendering. |
| Stored | Untrusted content is saved and rendered to users later. | Encode every output; consider the number of users who may see the content. |
| DOM-based | Client-side code reads untrusted data and passes it to a browser rendering sink. | Use safe DOM APIs and avoid interpreting untrusted data as HTML or script. |

## Lab setup

| Component | Sanitized description |
|---|---|
| Application | Local DVWA training instance |
| Areas | Reflected XSS, stored XSS, and DOM XSS exercises |
| Security settings | Low, medium, high, and impossible |
| Browser and tools | A web browser and the application's built-in exercises |
| Lab reference | Randomized run number `4827` |

The source notes mention a local container and database reset process. Specific host addresses, container names and identifiers, account names, and passwords are intentionally not reproduced.

## Observations

### Reflected XSS

**Location in the lab:** The reflected-input exercise, where a submitted value is returned in the response.

**Recorded observation:** At the low setting, the notes report that script was interpreted by the browser. At higher settings, the notes report that the same kind of content appeared as text rather than executing.

**Security lesson:** Reflected data is not safe merely because it came from a request parameter. Apply context-appropriate output encoding at the point where it is rendered.

### Stored XSS

**Location in the lab:** The guestbook-style exercise, where submitted name and message fields are saved and shown again.

**Recorded observation:** At the low setting, the notes report browser execution after submission and again after refreshing the page. This demonstrates the persistence property: later visitors to the same content may also encounter the unsafe rendering.

The notes describe cookie-related behavior but include no cookie values here. Cookies and other session data must never be copied into public evidence.

**Security lesson:** Stored unsafe content can affect more than the person who submitted it. Encode content whenever it is displayed, including on later page views. Input filtering alone is not a reliable substitute.

### DOM-based XSS

**Location in the lab:** The language-selection exercise, which reads a value from the page URL and displays it using client-side code.

**Recorded observation:** The notes report script execution at the low setting and text display at stricter settings. No specific URL, payload, or execution string is included.

**Security lesson:** Data processed entirely in the browser can still reach unsafe rendering sinks. Review client-side data flows and prefer APIs that treat values as text.

## Consolidated finding

| Category | Reported low-setting behavior | Reported stricter-setting behavior | Main concern |
|---|---|---|---|
| Reflected | Browser interpreted untrusted response content. | Content was reported as text. | A crafted request can cause unsafe content to be rendered to a user. |
| Stored | Content was reported to execute after submission and on later page views. | The notes report filtering or text display, but also raise an unverified concern about incomplete filtering at a middle setting. | Persistent content may be shown to multiple users. |
| DOM-based | Client-side URL-derived content was reported to execute. | Content was reported as text. | Unsafe client-side sinks may interpret untrusted data without server-side rendering. |

The original notes characterize the overall issue as CWE-79 and give indicative severity estimates of 6.1 for reflected/DOM and 7.2 for stored XSS. These scores and the claim that particular settings are fixed are **not independently validated**. The notes contain inconsistent descriptions of the middle security setting, and the referenced screenshots were not present in the supplied folder. Treat the results as preliminary lab observations, not a verified security rating.

## Impact considerations

- Reflected XSS may affect a user who visits a crafted page or request.
- Stored XSS may affect multiple users who later view the affected content.
- DOM-based XSS originates in client-side processing; server-side logs may not capture the full client-side data flow.
- Actual impact depends on application context, user permissions, browser behavior, and available defenses.

These are general risk descriptions, not instructions for targeting real services or users.

## Remediation

### Encode output for its context

For PHP output in an HTML text or quoted-attribute context, use the appropriate encoder:

```php
echo htmlspecialchars($value, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
```

Encoding must match the destination context. HTML escaping does not automatically make a value safe in JavaScript, CSS, or URL contexts.

### Prefer safe client-side rendering

For plain text, set a text node instead of asking the browser to parse a string as HTML:

```js
element.textContent = untrustedValue;
```

Avoid unsafe HTML-parsing sinks for untrusted values. If rich HTML is a product requirement, use a maintained sanitizer with a restrictive, context-appropriate allowlist.

### Add defense in depth

- Validate input to enforce business rules, but do not rely on input filtering as the XSS control.
- Prefer retaining the original value and encoding it at each output location.
- Consider a restrictive Content Security Policy as an additional layer, not a replacement for safe rendering.
- Set session cookies with appropriate `HttpOnly`, `Secure`, and `SameSite` attributes. These attributes reduce some risks but do not prevent XSS.
- Re-test every affected output context in an authorized, isolated environment after making changes.

## Evidence checklist from the source notes

The notes listed these intended screenshots; none were present in the supplied folder, so they are not included in this repository:

1. Reflected behavior at the low setting.
2. A redacted illustration of the cookie-related observation (never publish session values).
3. Reflected content displayed as text at a stricter setting.
4. Stored behavior after submission.
5. Stored behavior after refreshing the page.
6. A middle-setting stored-input check; the source labels this as potentially incomplete and not conclusively verified.
7. DOM-based behavior at the low setting.
8. DOM-based content displayed as text at a stricter setting.

If evidence is added later, redact personal data, session values, host details, and any material that would enable targeting another system.

## Key learnings

| Concept | Takeaway |
|---|---|
| Reflected vs. stored | Reflected content is returned in a request's response; stored content can be rendered again for later visitors. |
| DOM-based XSS | Client-side code can create XSS when it sends untrusted data to an unsafe browser sink. |
| Output encoding | Encoding at the point of output, for the correct context, is a core defense. |
| Security levels | A setting that blocks one example does not prove an entire class of input is safe. Test all relevant contexts and document uncertainty. |
| Evidence handling | Screenshots and logs should be reviewed and redacted before publication. |

## References

- [OWASP Cross Site Scripting Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [OWASP Cross Site Scripting overview](https://owasp.org/www-community/attacks/xss/)
- [CWE-79: Improper Neutralization of Input During Web Page Generation](https://cwe.mitre.org/data/definitions/79.html)

## Use

For defensive education only. Test only systems you own or are explicitly authorized to assess.
