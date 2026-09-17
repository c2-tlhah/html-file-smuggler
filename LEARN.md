# HTML Smuggling: Technical Deep-Dive & Educational Guide

A comprehensive guide explaining the mechanics, architecture, browser constraints, and defense strategies behind HTML file smuggling.

---

## 1. What is HTML Smuggling?

**HTML Smuggling** (classified under **MITRE ATT&CK T1027.006**) is a technique where an arbitrary binary or document is encoded and embedded inside an HTML document. Rather than transmitting the binary directly across the network—where Secure Email Gateways (SEGs), next-generation firewalls (NGFW), and web proxies can inspect and block it—the payload is assembled entirely **client-side** by the recipient's web browser using JavaScript.

```
Traditional File Transfer:
[Server / Sender] ──(Binary File via HTTP/SMTP)──> [Proxy / Gateway: BLOCKED] ──X──> [Client]

HTML Smuggling:
[Server / Sender] ──(Innocuous HTML + JS Text)──> [Proxy / Gateway: ALLOWED] ───> [Client Browser]
                                                                                        │
                                                                   (Browser executes JS │
                                                                    and builds binary)  ▼
                                                                                   [Local File]
```

To the network perimeter, the traffic appears to be ordinary HTML markup and JavaScript code.

---

## 2. Core Mechanics

### Phase 1: In-Memory Payload Representation
Web browsers can handle binary data in memory using `Blob` (Binary Large Object) or `ArrayBuffer` instances:
```javascript
const blob = new Blob([binaryPayload], { type: 'application/octet-stream' });
```

### Phase 2: Programmatic File Delivery
To deliver the file to the local disk without user click interaction, JavaScript creates a temporary anchor (`<a>`) element pointing to a memory-backed object URL:
```javascript
const url = URL.createObjectURL(blob);
const a = document.createElement('a');
a.href = url;
a.download = 'target_file.exe';
document.body.appendChild(a);
a.click();
document.body.removeChild(a);
URL.revokeObjectURL(url);
```

---

## 3. The Out-of-Memory (OOM) Barrier & Anti-OOM Architecture

### Why Browsers Crash on Large Payloads
Browsers are built to render web pages, not to serve as multi-gigabyte binary compilers. When an HTML file containing a massive Base64 string is opened:
1. **DOM Overhead**: The browser parser tokenizes the entire string into memory.
2. **Encoding Expansion**: Base64 encoding inflates file size by ~37%.
3. **Array Duplication**: Converting Base64 strings to Uint8Arrays creates duplicate in-memory buffers.

On a machine with 16 GB RAM, opening a single 2 GB HTML file frequently triggers **Error code: Out of Memory** or tab terminations.

### The Solution: Headless Multi-Part Extraction
HTML File Smuggler overcomes memory saturation using a three-tier architecture:

#### 1. Part Segmentation (36 MB / 50 MB Parts)
Large files are split into uniform chunks of 36 MB raw binary, which serialize to ~50 MB Base64-encoded strings. Each part is saved as an independent HTML file (`filename.part001.html`).

#### 2. Headless Stream Processing (`decoder.html`)
The recipient does **not** open individual part files in browser tabs. Instead, `decoder.html` reads each part sequentially from disk using `File.text()`:
```javascript
const text = await fileHandle.getFile().then(f => f.text());
const chunks = extractChunks(text);
```
Because the part files are read as plain text streams and never rendered in the browser DOM, memory consumption remains flat at **~2 MB peak RAM**, regardless of whether the source file is 100 MB or 10 GB.

#### 3. Immediate Memory Dereferencing
As each 1 MB chunk is decrypted and written to disk, its memory reference is explicitly set to `null` to facilitate immediate garbage collection:
```javascript
for (let i = 0; i < chunks.length; i++) {
  let data = decryptChunk(chunks[i]);
  await writeToDisk(data);
  chunks[i] = null; // Explicit GC hint
}
```

---

## 4. Chromium Stream Buffer Hardening (The 64 KB Rule)

### The Unexpected EOF Problem
When using the Chromium File System Access API (`FileSystemWritableFileStream`), calling `writable.write(largeBuffer)` with chunks larger than a few megabytes causes the underlying stream buffer to overflow, terminating the stream with:
```
TypeError: stream reading error: unexpected EOF
```

### The Slicing Pattern
To guarantee resilience across all Chromium builds (Chrome, Edge, Brave, Opera), all writes are segmented into 64 KB slices:
```javascript
async function w64(wr, data) {
  const bytes = typeof data === 'string' ? new TextEncoder().encode(data) : data;
  for (let i = 0; i < bytes.length; i += 65536) {
    await wr.write(bytes.subarray(i, Math.min(i + 65536, bytes.length)));
  }
}
```

---

## 5. Cryptographic Implementation

To ensure payload confidentiality across untrusted transit environments, optional military-grade encryption is integrated:

| Component | Standard / Parameter | Purpose |
| :--- | :--- | :--- |
| **Algorithm** | AES-256-GCM | Authenticated symmetric encryption preventing tampering. |
| **KDF** | PBKDF2-HMAC-SHA256 | Derives a 256-bit key from user passphrase. |
| **Iterations** | 200,000 rounds | Hardens against offline brute-force attacks. |
| **Salt** | 16 random bytes | Prevents precomputed rainbow table lookups. |
| **IV** | 12 random bytes per chunk | Unique nonce ensures identical plaintext blocks produce distinct ciphertexts. |

All cryptographic primitives are executed natively via `window.crypto.subtle` (W3C Web Crypto API).

---

## 6. Detection, Mitigation & Defense (Blue Team Perspective)

Security teams and system administrators can detect and mitigate HTML smuggling through layered defenses:

### 1. Endpoint Detection & Response (EDR)
- Monitor browser processes (`chrome.exe`, `msedge.exe`) initiating unexpected file writes of executable types (`.exe`, `.dll`, `.iso`, `.zip`, `.ps1`).
- Inspect the **Mark of the Web (MotW)** attached to files dropped via browser downloads.

### 2. Secure Email Gateways (SEG) & Proxy Defenses
- Enforce deep inspection on incoming `.html` and `.htm` attachments for embedded JavaScript indicators:
  - Dynamic `Blob` and `URL.createObjectURL` constructions.
  - Large Base64 encoded payload blocks.
  - References to `showSaveFilePicker` or `showDirectoryPicker`.
- Quarantine password-protected or encrypted HTML attachments from untrusted external senders.

### 3. Attack Surface Reduction (ASR)
- Block child process execution from browser binaries.
- Prohibit browser downloads from directly creating `.hta`, `.vbs`, or script-based file types.

---

## 7. Ethical Philosophy & Dual-Use Context

> *"The tool itself has no evilness—intent lies solely with the operator."*

HTML File Smuggler is designed as a **dual-use technology**:
- **Offensive Security / Red Teaming**: Simulates realistic adversary techniques to validate whether enterprise security controls can detect and contain client-side payload assembly.
- **Defensive & Operational Utility**: Enables secure, tamper-evident file transportation across restrictive air-gap jumpboxes or isolated environments where only plain text transfers are permitted.

Users are strictly responsible for adhering to applicable laws, ethical guidelines, and organizational policies.
