# XSS Security Lab — Run 4827

A sanitized, defensive learning summary about cross-site scripting (XSS), based on a local, intentionally vulnerable training application. Personal details and operational exploit examples have been removed.

## Scope

The exercise reviewed three XSS categories in an isolated training environment:

| Category | Where untrusted data is interpreted | Defensive focus |
|---|---|---|
| Reflected | In a response to a request | Encode data for its output context |
| Stored | In content saved and later shown to users | Validate input where appropriate and encode on every output |
| DOM-based | In client-side code that processes untrusted data | Prefer safe DOM APIs and avoid unsafe HTML sinks |

The source notes describe behavior at multiple training security settings. They report that low settings allowed script execution and that stricter settings displayed content as text. These are lab observations, not a test of any production service.

## Impact

If untrusted data is interpreted as executable markup or script, an attacker may cause actions in a victim's browser with the victim's application context. Stored XSS can affect multiple users who view the same content. The actual impact depends on the application, browser protections, user privileges, and the data accessible to the script.

This repository does not contain attack strings, session-theft instructions, filter-bypass examples, or steps for testing third-party systems.

## Defensive guidance

- Encode untrusted data when rendering it, using the encoder appropriate to the HTML, attribute, URL, CSS, or JavaScript context.
- For plain text in the browser, use APIs such as `textContent` rather than HTML-parsing APIs.
- If an application must accept rich HTML, sanitize it with a maintained, context-appropriate sanitizer and a restrictive allowlist.
- Validate input to enforce application rules, but do not treat validation or input filtering as a substitute for output encoding.
- Store the original data where practical and encode it at each output location; encoding only when saving can cause display and double-encoding problems.
- Use a restrictive Content Security Policy as defense in depth. It does not replace safe rendering.
- Mark cookies `HttpOnly`, `Secure`, and with an appropriate `SameSite` setting. These attributes reduce some risks but do not fix XSS.
- Test fixes in an authorized, isolated environment and confirm that untrusted content is rendered inert in every relevant output context.

For PHP HTML text or quoted-attribute output, use context-appropriate escaping, for example:

```php
echo htmlspecialchars($value, ENT_QUOTES | ENT_SUBSTITUTE, 'UTF-8');
```

For client-side plain text:

```js
element.textContent = untrustedValue;
```

## Lab record

- **Lab reference:** 4827
- **Environment:** Local, isolated, intentionally vulnerable training application
- **Areas reviewed:** Reflected, stored, and DOM-based XSS
- **Evidence:** The source notes mention screenshots, but no screenshot files were present in the supplied folder.
- **Sanitization:** Names, location, account details, container identifiers, and target-specific references were excluded. Exploit payloads and instructions were excluded.

## References

- [OWASP Cross Site Scripting Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
- [OWASP Cross Site Scripting overview](https://owasp.org/www-community/attacks/xss/)
- [CWE-79: Improper Neutralization of Input During Web Page Generation](https://cwe.mitre.org/data/definitions/79.html)

## Use

For defensive education only. Practice only in systems you own or are explicitly authorized to test.
