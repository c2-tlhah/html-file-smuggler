# HTML File Smuggler 📦

> **High-performance, zero-dependency browser suite for client-side HTML file smuggling, AES-256-GCM encryption, chunked streaming, and split-file decoding.**

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Vanilla JS](https://img.shields.io/badge/Vanilla-JavaScript-f7df1e.svg)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![Web Crypto API](https://img.shields.io/badge/Crypto-AES--256--GCM-success.svg)](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API)
[![File System Access API](https://img.shields.io/badge/API-File%20System%20Access-orange.svg)](https://developer.mozilla.org/en-US/docs/Web/API/File_System_API)

---

## 🚀 Overview

**HTML File Smuggler** leverages HTML5, JavaScript Blobs, and the Web Crypto API to convert arbitrary binary files into standalone, self-extracting HTML packages or segmented multi-part bundles.

When opened or processed in a modern browser, the embedded payloads are extracted, decrypted in memory, and reassembled directly on the target file system without requiring external software, runtimes, or network connectivity.

### Why this exists
- **Restricted Environments**: Transfer files across systems where only text, markup, or copy-paste operations are permitted.
- **Zero-Server Security**: 100% client-side execution. Files never touch any remote server or third-party service.
- **Out-of-Memory (OOM) Protection**: Handles large multi-gigabyte files without browser memory exhaustion by combining chunked memory release and disk streaming via the File System Access API.

---

## ✨ Features

- 🔒 **Military-Grade Encryption**: Optional AES-256-GCM encryption with PBKDF2 key derivation (200,000 SHA-256 rounds and a unique 16-byte cryptographic salt).
- ⚡ **Zero Dependencies**: Pure HTML and Vanilla JavaScript. Runs directly in any Chromium-based browser (Chrome, Edge, Brave, Opera).
- 💾 **Two Flexible Build Modes**:
  1. **Single HTML Package**: Encapsulates the entire file into a single, self-extracting `.html` document.
  2. **Split to Folder (Large Files)**: Automatically slices files into uniform ~50 MB parts (`.part001.html`, `.part002.html`, etc.) and pairs them with `decoder.html`.
- 🛠️ **Standalone Decoder (`decoder.html`)**:
  - Automatically indexes and sequences all part files within a selected folder.
  - Reads parts as plain text streams rather than parsing them into the DOM, keeping peak memory usage at **~2 MB regardless of total file size**.
  - Assembles and decrypts payloads on-the-fly, writing directly to disk.
- 🛡️ **Stream Hardening**: Implements chunked 64 KB writes (`w64()`) to prevent premature EOF stream rejections and buffer aborts inside the Chromium File System Access engine.
- 🧹 **Clean & Focused Interface**: Strict separation of concerns — `index.html` builds/encodes payloads, while `decoder.html` retrieves and reassembles them.

---

## 📁 Repository Structure

| File | Description |
| :--- | :--- |
| [`index.html`](index.html) | **File Smuggling Encoder** — Form to configure passwords, custom messages, and generate single or split payloads. |
| [`decoder.html`](decoder.html) | **Standalone Decoder** — Portable utility to select a folder containing split parts and reconstruct the original file. |
| [`LICENSE`](LICENSE) | MIT License. |
| [`README.md`](README.md) | Project documentation and usage guide. |

---

## 📖 Usage Guide

### 1. Encoding Files (`index.html`)

1. Open [`index.html`](index.html) in Google Chrome or Microsoft Edge.
2. Click **Choose File** to select your target file (documents, archives, executables, disk images, etc.).
3. *(Optional)* Provide a **Password** to enforce AES-256-GCM encryption.
4. *(Optional)* Add a **Note** to display instructions or metadata to the recipient.
5. Choose your build mode:
   - **Build Single HTML**: Select a save destination. Generates a standalone `<filename>.html` file.
   - **Build Split to Folder**: Select an output directory. Generates sequential `.part*.html` files along with a copy of `decoder.html`.
   - *(Optional)* Click **Save decoder.html** anytime to obtain an independent copy of the decoder.

---

### 2. Decoding Files

#### A. Single HTML Files
- Simply double-click the generated `.html` file in Chrome or Edge.
- If password-protected, enter the password.
- Click **Retrieve File** to decrypt and save the original file to disk.

#### B. Split Multi-Part Files
1. Transfer the generated folder (or copy-paste the text of each `.part*.html` and `decoder.html`) onto the target system.
2. Open [`decoder.html`](decoder.html) in Chrome or Edge.
3. If password-protected, enter the decryption password.
4. Click **Open Folder & Decode** and select the folder containing your part files.
5. The decoder will automatically sort the parts, decrypt each chunk, and stream the original file directly into the same folder.

---

## ⚙️ Architecture & Memory Safety

Traditional HTML smuggling techniques embed large Base64 blobs directly into strings or DOM elements. When working with files over a few hundred megabytes, this frequently causes tab crashes (**Error code: Out of Memory**).

HTML File Smuggler prevents memory pressure using two primary mechanisms:

### 1. In-Memory Chunk Deallocation
During extraction, chunks are read sequentially and immediately cleared:
```javascript
for (let i = 0; i < n; i++) {
  let d = b64u8(C[i].d);
  // ... decrypt and stream to disk ...
  C[i] = null; // Explicit garbage collection hint
}
```

### 2. Direct-to-Disk Stream Processing
Rather than accumulating Blobs in memory, chunks are passed through the browser's native `FileSystemWritableFileStream` in 64 KB slices:
```javascript
async function w64(wr, data) {
  const b = typeof data === 'string' ? new TextEncoder().encode(data) : data;
  for (let i = 0; i < b.length; i += 65536) {
    await wr.write(b.subarray(i, Math.min(i + 65536, b.length)));
  }
}
```

---

## 🌐 Browser Compatibility

| Browser | Supported | Notes |
| :--- | :---: | :--- |
| **Google Chrome** | ✅ | Full support for File System Access API and Web Crypto |
| **Microsoft Edge** | ✅ | Full support for File System Access API and Web Crypto |
| **Brave** | ✅ | Full support (Chromium core) |
| **Opera** | ✅ | Full support (Chromium core) |
| **Firefox** | ⚠️ Partial | Fallback memory Blob download supported (single file only) |
| **Safari** | ⚠️ Partial | Fallback memory Blob download supported (single file only) |

---

## 🔒 Security & Educational Disclaimer

This project is published for educational purposes, authorized penetration testing, security auditing, and restricted environments file transfer. Users are responsible for complying with all applicable laws and organizational security policies.

---

## 📄 License

Distributed under the [MIT License](LICENSE). See `LICENSE` for more information.
