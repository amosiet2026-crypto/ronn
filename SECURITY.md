# Security Implementation

## Overview
This document outlines the security features implemented in the BLACKMARKET website to prevent common web attacks.

## Security Features Implemented

### 1. Content Security Policy (CSP)
- **Location**: HTML `<head>`
- **Benefits**: Prevents inline script execution and controls resource loading
- **Policy Details**:
  - Default source: `'self'`
  - Script sources: `'self'`, `'unsafe-inline'`, `https://fonts.gstatic.com`
  - Style sources: `'self'`, `'unsafe-inline'`, `https://fonts.gstatic.com`
  - Font sources: `https://fonts.gstatic.com`, `https://fonts.googleapis.com`
  - Image sources: `'self'`, `data:`
  - Frame ancestors: `'none'` (prevents clickjacking)
  - Base URI: `'self'`

### 2. Security Headers
- **X-UA-Compatible**: Ensures modern browser rendering
- **X-Content-Type-Options**: `nosniff` - Prevents MIME type sniffing
- **X-Frame-Options**: `DENY` - Prevents clickjacking attacks
- **Referrer-Policy**: `strict-origin-when-cross-origin` - Controls referrer information

### 3. Input Validation & Sanitization

#### HTML Sanitization
```javascript
function sanitizeHTML(str) {
  const div = document.createElement('div');
  div.textContent = str;
  return div.innerHTML;
}
```
- Prevents XSS attacks by converting user input to text nodes before rendering
- Applied to all dynamically generated content

#### Email Validation & Sanitization
```javascript
function isValidEmail(email) {
  const re = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
  return re.test(email);
}

function sanitizeEmail(email) {
  return email.trim().replace(/[\r\n]/g, '').toLowerCase();
}
```
- Validates email format
- Removes line breaks to prevent email header injection
- Applied before sending confirmation emails

#### Wallet Address Validation
```javascript
function isValidWalletAddress(addr, type) {
  // Validates Bitcoin, Ethereum, USDT, and Monero addresses
  // Prevents invalid payment addresses
}
```
- Type-specific validation for different cryptocurrencies
- Prevents typos in payment addresses

### 4. CSRF Protection

#### Token Generation & Validation
```javascript
const CSRFToken = (() => {
  let token = null;
  return {
    generate: () => { /* generates unique token */ },
    get: () => { /* retrieves stored token */ },
    validate: (providedToken) => { /* validates token */ }
  };
})();
```
- Generates unique CSRF tokens on page load
- Stores in `sessionStorage`
- Validates tokens on form submissions
- Prevents cross-site request forgery attacks

### 5. Event Delegation (No Inline Handlers)

#### Before (Vulnerable):
```html
<button onclick="addToCart('p18', this)">Add to Cart</button>
```

#### After (Secure):
```html
<button data-product-id="p18">Add to Cart</button>
```
```javascript
button.addEventListener('click', (e) => {
  const id = e.target.closest('button').dataset.productId;
  addToCart(id, e.target.closest('button'));
});
```

**Benefits**:
- Prevents inline script injection
- Better separation of concerns
- Centralized event handling

### 6. Secure Logging
```javascript
function secureLog(message, level = 'info') {
  const timestamp = new Date().toISOString();
  const logMessage = `[${timestamp}] [${level.toUpperCase()}] ${message}`;
  // Logs to console without exposing sensitive data
}
```

### 7. CSP Violation Monitoring
```javascript
document.addEventListener('securitypolicyviolation', (e) => {
  secureLog(`CSP Violation: ${e.violatedDirective}`, 'warn');
});
```
- Monitors CSP violations
- Helps identify potential attack attempts or misconfigurations

## Common Attack Prevention

### XSS (Cross-Site Scripting)
- ✅ All user input sanitized with `sanitizeHTML()`
- ✅ No inline event handlers (using event delegation)
- ✅ CSP restricts script execution

### CSRF (Cross-Site Request Forgery)
- ✅ CSRF tokens generated and validated
- ✅ Tokens stored in sessionStorage
- ✅ Email-based confirmation prevents automated attacks

### Email Injection
- ✅ Email addresses validated and sanitized
- ✅ Line breaks removed before email generation
- ✅ Safe `mailto:` URL encoding

### Clickjacking
- ✅ `X-Frame-Options: DENY` header
- ✅ CSP `frame-ancestors 'none'`

### MIME Type Sniffing
- ✅ `X-Content-Type-Options: nosniff` header

### Input Injection
- ✅ Form validation before processing
- ✅ Type-specific validation (email, wallet address, etc.)
- ✅ Data attributes used instead of inline handlers

## Security Best Practices Applied

1. **Defense in Depth**: Multiple layers of protection
2. **Input Validation**: All user inputs validated before use
3. **Output Encoding**: Dynamic content properly escaped
4. **Secure Defaults**: CSP defaults to most restrictive
5. **Least Privilege**: Content restrictions minimized to necessary resources
6. **Error Handling**: Secure logging without exposing sensitive data
7. **Separation of Concerns**: HTML structure, CSS styling, JavaScript logic separated

## Recommendations for Production

1. **Backend Integration**:
   - Implement server-side CSRF token validation
   - Validate all inputs on the backend
   - Implement rate limiting on sensitive endpoints

2. **HTTPS**:
   - Always use HTTPS in production
   - Use HSTS headers to force HTTPS

3. **Additional Headers**:
   ```
   Strict-Transport-Security: max-age=31536000; includeSubDomains
   X-XSS-Protection: 1; mode=block
   Expect-CT: max-age=86400, enforce
   Permissions-Policy: geolocation=(), microphone=(), camera=()
   ```

4. **Regular Security Audits**:
   - Test with OWASP ZAP
   - Run npm audit for dependencies
   - Perform penetration testing

5. **Monitoring & Logging**:
   - Log security events server-side
   - Monitor for suspicious patterns
   - Set up alerts for CSP violations

6. **Password Security**:
   - Implement proper password hashing (bcrypt, argon2)
   - Use secure session management
   - Implement account lockout after failed attempts

7. **API Security**:
   - Implement API rate limiting
   - Use API keys / OAuth tokens
   - Validate all API requests server-side

## Testing Security

### Browser Console Tests:
```javascript
// Check CSP headers
fetch('index.html').then(r => console.log(r.headers))

// Test input sanitization
console.log(sanitizeHTML('<script>alert("xss")</script>'))

// Test CSRF token
console.log(CSRFToken.get())

// Test email validation
console.log(isValidEmail('test@example.com'))
console.log(isValidEmail('invalid'))
```

## File Changes Summary

- **index.html**: Added CSP and security headers, implemented secure JavaScript with validation
- **lucid.js**: No changes (external library)
- **style.css**: No changes

## References

- [OWASP Top 10](https://owasp.org/Top10/)
- [Content Security Policy Documentation](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP)
- [Web Security Academy](https://portswigger.net/web-security)
- [NIST Cybersecurity Framework](https://www.nist.gov/cyberframework)

---

**Last Updated**: July 25, 2026
**Security Level**: Enhanced (Development/Demo)

For production deployment, all recommendations must be implemented.
