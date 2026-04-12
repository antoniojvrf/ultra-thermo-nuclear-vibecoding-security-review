# File Upload Security

## Validation Requirements

### 1. File Type Validation
- Check file extension against allowlist
- Validate magic bytes/file signature match expected type
- Never rely on just one check — combine extension + magic bytes + MIME type

### 2. Magic Bytes Reference

| Type | Magic Bytes (hex) |
|------|-------------------|
| JPEG | `FF D8 FF` |
| PNG | `89 50 4E 47 0D 0A 1A 0A` |
| GIF | `47 49 46 38` |
| PDF | `25 50 44 46` |
| ZIP | `50 4B 03 04` |
| DOCX/XLSX | `50 4B 03 04` (ZIP-based) |
| WebP | `52 49 46 46 ... 57 45 42 50` |

### 3. File Size Limits
- Set maximum file size server-side
- Configure web server/proxy limits (nginx `client_max_body_size`)
- Consider per-file-type limits (images smaller than videos)

## Common Bypasses and Attacks

| Attack | Description | Prevention |
|--------|-------------|-----------|
| Extension bypass | `shell.php.jpg` | Check FULL extension, use allowlist |
| Null byte | `shell.php%00.jpg` | Sanitize filename, check for null bytes |
| Double extension | `shell.jpg.php` | Only allow single extension |
| MIME type spoofing | Content-Type set to image/jpeg | Validate magic bytes, not MIME |
| Magic byte injection | Valid magic bytes + malicious payload | Check entire file structure |
| Polyglot files | Valid as both JPEG and JavaScript | Parse as expected type, reject if invalid |
| SVG with JavaScript | `<svg onload="alert(1)">` | Sanitize SVG or disallow entirely |
| XXE via file upload | Malicious DOCX/XLSX (XML inside) | Disable external entities in parser |
| ZIP slip | `../../../etc/passwd` in archive | Validate extracted paths |
| ImageMagick exploits | Crafted images | Keep updated, use policy.xml |
| Filename injection | `; rm -rf /` in filename | Sanitize filenames, use random names |

## Secure Upload Handling

```javascript
import { randomUUID } from 'crypto';
import path from 'path';

const ALLOWED_EXTENSIONS = ['.jpg', '.jpeg', '.png', '.gif', '.webp'];
const MAX_SIZE = 5 * 1024 * 1024; // 5MB

async function handleUpload(file) {
  // 1. Check size
  if (file.size > MAX_SIZE) throw new Error('File too large');

  // 2. Check extension
  const ext = path.extname(file.name).toLowerCase();
  if (!ALLOWED_EXTENSIONS.includes(ext)) throw new Error('Invalid type');

  // 3. Check magic bytes
  const buffer = await file.arrayBuffer();
  const bytes = new Uint8Array(buffer.slice(0, 8));
  if (!isValidMagicBytes(bytes, ext)) throw new Error('Invalid file');

  // 4. Rename with UUID (discard original name)
  const safeName = `${randomUUID()}${ext}`;

  // 5. Store outside webroot or on separate domain/CDN
  await storage.upload(safeName, buffer, {
    contentType: file.type,
    contentDisposition: 'attachment', // forces download
  });

  return safeName;
}
```

### Path Traversal in File Handling

```python
import os

def safe_join(base_directory, user_path):
    base = os.path.abspath(os.path.realpath(base_directory))
    target = os.path.abspath(os.path.realpath(os.path.join(base, user_path)))
    if os.path.commonpath([base, target]) != base:
        raise ValueError("Path traversal detected")
    return target
```

### Prevention Checklist
- [ ] File extension checked against allowlist
- [ ] Magic bytes validated (not just MIME type)
- [ ] File size limits enforced server-side
- [ ] Filenames replaced with random UUIDs
- [ ] Files stored outside webroot or on separate domain
- [ ] `Content-Disposition: attachment` set on served files
- [ ] `X-Content-Type-Options: nosniff` header set
- [ ] SVG files sanitized or rejected
- [ ] Archive extraction validates paths (ZIP slip prevention)
- [ ] Uploaded files are not executable
