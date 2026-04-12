# Injection Vulnerabilities

## SQL Injection

### Prevention Methods

**1. Parameterized Queries (Prepared Statements)** — PRIMARY DEFENSE
```sql
-- VULNERABLE
query = "SELECT * FROM users WHERE id = " + userId

-- SECURE
query = "SELECT * FROM users WHERE id = ?"
execute(query, [userId])
```

**2. ORM Usage**
- Use ORM methods that automatically parameterize
- Be cautious with raw query methods: Prisma `$queryRawUnsafe`, Sequelize `sequelize.query()`, Django `.raw()`
- Watch for ORM-specific injection points

**3. Input Validation** (defense-in-depth, not primary)
- Validate data types (integer should be integer)
- Whitelist allowed values where applicable

### Injection Points to Watch
- `WHERE` clauses
- `ORDER BY` clauses (cannot use parameters — must whitelist column names)
- `LIMIT`/`OFFSET` values
- Table and column names (cannot parameterize — must whitelist)
- `INSERT` values
- `UPDATE SET` values
- `IN` clauses with dynamic lists
- `LIKE` patterns (escape wildcards: `%`, `_`)

### Additional Defenses
- **Least privilege**: DB user should have minimum required permissions
- **Error handling**: Never expose SQL errors to users
- **WAF**: Consider web application firewall as additional layer

---

## Cross-Site Scripting (XSS)

Every input controllable by the user — directly or indirectly — must be sanitized.

### Input Sources to Protect

**Direct Inputs:**
- Form fields (email, name, bio, comments)
- Search queries
- File names during upload
- Rich text editors / WYSIWYG content

**Indirect Inputs:**
- URL parameters and query strings
- URL fragments (hash values)
- HTTP headers (Referer, User-Agent if displayed)
- Data from third-party APIs displayed to users
- WebSocket messages
- `postMessage` data from iframes
- LocalStorage/SessionStorage values if rendered

**Often Overlooked:**
- Error messages that reflect user input
- PDF/document generators that accept HTML
- Email templates with user data
- Log viewers in admin panels
- JSON responses rendered as HTML
- SVG file uploads (can contain JavaScript)
- Markdown rendering (if allowing HTML)

### Protection Strategies

**1. Output Encoding (Context-Specific)**
- HTML context: HTML entity encode (`<` → `&lt;`)
- JavaScript context: JavaScript escape
- URL context: URL encode
- CSS context: CSS escape
- Use framework's built-in escaping (React JSX, Vue `{{ }}`, Angular templates)

**2. Content Security Policy (CSP)**
```
Content-Security-Policy:
  default-src 'self';
  script-src 'self';
  style-src 'self' 'unsafe-inline';
  img-src 'self' data: https:;
  font-src 'self';
  connect-src 'self' https://api.yourdomain.com;
  frame-ancestors 'none';
  base-uri 'self';
  form-action 'self';
```
- Avoid `'unsafe-inline'` and `'unsafe-eval'` for scripts
- Use nonces or hashes for inline scripts when necessary
- Report violations: `report-uri /csp-report`

**3. Input Sanitization**
- Use established libraries (DOMPurify for HTML)
- Whitelist allowed tags/attributes for rich text
- Strip or encode dangerous patterns

**4. Additional Headers**
- `X-Content-Type-Options: nosniff`
- `X-Frame-Options: DENY` (or use CSP frame-ancestors)

---

## Command Injection

### Vulnerable Patterns
```javascript
// VULNERABLE
exec(`convert ${userFilename} output.png`);
exec(`ping ${userInput}`);

// SECURE — use array-based APIs
execFile('convert', [userFilename, 'output.png']);
// Or avoid shell entirely — use libraries
```

### Prevention
1. **Never pass user input to shell commands**
2. Use language-specific APIs that don't invoke a shell (e.g., `execFile` over `exec` in Node.js)
3. If shell is unavoidable, use strict allowlists for allowed characters
4. Sanitize: reject inputs containing `;`, `|`, `&`, `$`, backticks, `$(`, `\n`

---

## XXE (XML External Entity)

### Vulnerable Scenarios
- SOAP APIs, XML-RPC
- XML file uploads
- Office documents (DOCX, XLSX, PPTX are ZIP with XML)
- SVG files (XML-based)
- SAML assertions
- RSS/Atom feed processing

### Prevention by Language

**Java:**
```java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
dbf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
dbf.setExpandEntityReferences(false);
```

**Python (lxml):**
```python
from lxml import etree
parser = etree.XMLParser(resolve_entities=False, no_network=True)
# Or use defusedxml library
```

**Node.js:**
```javascript
// Use libraries that disable DTD processing by default
// If using libxmljs: { noent: false, dtdload: false }
```

**.NET:**
```csharp
XmlReaderSettings settings = new XmlReaderSettings();
settings.DtdProcessing = DtdProcessing.Prohibit;
settings.XmlResolver = null;
```

### XXE Checklist
- [ ] Disable DTD processing entirely if possible
- [ ] Disable external entity resolution
- [ ] Disable external DTD loading
- [ ] Disable XInclude processing
- [ ] Use latest patched XML parser versions
- [ ] Consider using JSON instead of XML where possible

---

## Prototype Pollution (JavaScript)

### What It Is
Attackers manipulate `__proto__`, `constructor`, or `prototype` properties to inject properties into all JavaScript objects.

### Vulnerable Patterns
```javascript
// VULNERABLE — recursive merge without __proto__ check
function merge(target, source) {
  for (let key in source) {
    if (typeof source[key] === 'object') {
      target[key] = merge(target[key] || {}, source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}
// Attack: merge({}, JSON.parse('{"__proto__":{"isAdmin":true}}'))
// Now ({}).isAdmin === true for ALL objects
```

### Prevention
```javascript
// SECURE — block dangerous keys
function safeMerge(target, source) {
  for (let key in source) {
    if (key === '__proto__' || key === 'constructor' || key === 'prototype') continue;
    if (typeof source[key] === 'object' && source[key] !== null) {
      target[key] = safeMerge(target[key] || {}, source[key]);
    } else {
      target[key] = source[key];
    }
  }
  return target;
}

// Or use Object.create(null) for dictionary objects
const safeObj = Object.create(null);

// Or use Map instead of plain objects for user data
const userData = new Map();
```

### Libraries with Known Issues
- `lodash.merge` (older versions) — update to latest
- `jQuery.extend` (deep mode) — update to latest
- Custom deep merge utilities — audit manually
