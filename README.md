# HTML File Smuggler

[![Author: tlha](https://img.shields.io/badge/Author-tlha-111111?logo=github&logoColor=white)](https://github.com/c2-tlhah)
[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Vanilla JS](https://img.shields.io/badge/Vanilla-JavaScript-F7DF1E?logo=javascript&logoColor=black)](https://developer.mozilla.org/en-US/docs/Web/JavaScript)
[![HTML5](https://img.shields.io/badge/HTML5-Pure%20Markup-E34F26?logo=html5&logoColor=white)](https://developer.mozilla.org/en-US/docs/Web/HTML)
[![CSS3](https://img.shields.io/badge/CSS3-No%20Frameworks-1572B6?logo=css3&logoColor=white)](https://developer.mozilla.org/en-US/docs/Web/CSS)
[![Security: AES-256-GCM](https://img.shields.io/badge/Security-AES--256--GCM-1abc9c?logo=gnupg&logoColor=white)](https://developer.mozilla.org/en-US/docs/Web/API/Web_Crypto_API)
[![Chromium Compatible](https://img.shields.io/badge/Platform-Chromium%20%7C%20Edge%20%7C%20Chrome-4285F4?logo=googlechrome&logoColor=white)](https://www.google.com/chrome/)
[![File System Access API](https://img.shields.io/badge/API-File%20System%20Access-orange?logo=files&logoColor=white)](https://developer.mozilla.org/en-US/docs/Web/API/File_System_API)
[![Zero Dependencies](https://img.shields.io/badge/Dependencies-0-success?logo=buffer&logoColor=white)](https://github.com/c2-tlhah/html-file-smuggler)
[![Dual-Use Tool](https://img.shields.io/badge/Nature-Dual--Use-red?logo=hackthebox&logoColor=white)](https://attack.mitre.org/techniques/T1027/006/)

> *"The tool itself has no evilness—intent lies solely with the operator."*
>
> The distinctive split red-and-green design reflects the dual-use reality of HTML smuggling: red signifies offensive simulation, red-teaming, and evasion testing; green signifies authorized file recovery, air-gap transportation, and administrative utility.

High-performance, zero-dependency browser utility for client-side HTML file smuggling, AES-256-GCM encryption, chunked streaming, and split-file decoding.

## Overview

HTML File Smuggler is a self-contained, browser-native utility designed to package arbitrary binary files into HTML-wrapped payloads and reconstruct them on destination devices. The tool operates entirely within client-side JavaScript using the Web Crypto API and the Chromium File System Access API.

Because execution is strictly confined to the local browser context, no payload data is transmitted over the network or processed by third-party servers. The system enables reliable file transportation across restricted environments where standard file transfers, executables, or archive attachments are blocked, but HTML, text, or copy-paste operations remain permitted.

## Core Capabilities

- Client-Side Security: 100 percent in-browser execution with zero network telemetry or remote dependencies.
- Strong Cryptography: Optional AES-256-GCM authenticated encryption utilizing PBKDF2 key derivation with 200,000 iterations of SHA-256 and an isolated 16-byte cryptographic salt.
- Memory-Safe Processing: Overcomes browser Out-of-Memory (OOM) limitations on large files through progressive chunk deallocation and direct-to-disk streaming.
- Stream Hardening: Employs 64 KB sliced stream writing to prevent Chromium FileSystemWritableFileStream buffer saturation and premature EOF aborts.
- Modular Operation: Clear separation between the packaging engine (index.html) and the standalone folder extraction tool (decoder.html).

## System Requirements

- Supported Browsers: Google Chrome, Microsoft Edge, Brave, Opera, and other Chromium-based browsers with File System Access API support.
- Operating Systems: Windows, macOS, Linux, and ChromeOS.
- Capacity:
  - Single HTML Mode: Suitable for payloads up to approximately 7 GB on systems with 16 GB of physical RAM.
  - Split Multi-Part Mode: No theoretical file size limit. Tested on multi-gigabyte files with memory consumption remaining under 2 MB during extraction.

## Operational Workflows

The utility provides two distinct packaging models based on payload size and operational constraints.

### Workflow 1: Single HTML File

This workflow wraps the entire payload into a single self-contained HTML file. It is recommended for small-to-medium files where simplicity is preferred.

#### Step 1: Packaging (Source Machine)
1. Open index.html in Chrome or Edge.
2. Select the target file using the File input. The application calculates the source size and estimated encoded size.
3. (Optional) Enter a passphrase in the Password field to enforce AES-256-GCM encryption.
4. (Optional) Enter a message in the Note field to display instructions to the recipient.
5. Click "Build Single HTML".
6. When prompted by the browser, select the destination filename and save location.
7. The encoder processes the source file in 1 MB chunks. If encryption is enabled, each chunk is encrypted using a dedicated 12-byte random initialization vector (IV). Chunks are Base64-encoded and written directly to the output file in 64 KB slices.

#### Step 2: Extraction (Target Machine)
1. Deliver the generated HTML file to the target machine via web download, email attachment, or text transfer.
2. Open the file in Chrome or Edge.
3. If the payload was encrypted, enter the passphrase in the Password field.
4. Click "Retrieve File" and choose a destination path.
5. The embedded extraction logic reads each chunk sequentially, decrypts it, streams it to disk, and immediately releases the memory reference (C[i] = null). Once all chunks are processed, the file is finalized and closed.

### Workflow 2: Split Multi-Part Mode (Large Files)

When files exceed several hundred megabytes, opening a single monolithic HTML file in a web browser will exhaust tab memory and trigger an Out-of-Memory crash. The Split Multi-Part workflow eliminates this limitation by dividing the payload into uniform parts and using a headless text extraction routine.

#### Step 1: Packaging (Source Machine)
1. Open index.html in Chrome or Edge.
2. Select the source file. The interface displays the total part count based on 36 MB raw source slices (approximately 50 MB per encoded part).
3. (Optional) Provide a passphrase and recipient note.
4. Click "Build Split to Folder".
5. When prompted by the browser directory picker, select or create a destination folder.
6. The engine writes sequential part files (such as filename.part001.html, filename.part002.html) into the chosen folder.
7. Upon completion, the engine automatically saves a copy of decoder.html into the same directory.

#### Step 2: Transfer Across Restricted Channels
Copy the generated folder containing all part files and decoder.html to the target system. In environments where file transfer is restricted to copy-paste:
- Open each part file in a text editor on the source machine.
- Copy the text content and paste it into a file with the identical name on the target machine.
- Repeat the process for decoder.html.

#### Step 3: Extraction (Target Machine via decoder.html)
1. Open decoder.html in Chrome or Edge on the target system. Note: Do not attempt to open individual part HTML files.
2. If encryption was applied during packaging, enter the passphrase in the Password field.
3. Click "Open Folder & Decode".
4. When prompted by the browser directory picker, select the directory containing the part files.
5. The decoder executes the following automated pipeline:
   - Scans the directory handle and discovers all files matching the pattern .part*.html.
   - Sorts the discovered parts in correct numerical order.
   - Reads the initial part metadata to verify file integrity and encryption parameters.
   - Derives the cryptographic key using PBKDF2 if a passphrase is required.
   - Opens an output writable stream for the original filename.
   - Iterates through each part file by reading it as a raw text stream via the File API rather than loading it into the DOM.
   - Extracts encoded chunks using delimited string markers, decodes Base64 data, decrypts AES-GCM ciphertexts, and writes to disk in 64 KB segments.
   - Deallocates chunk memory immediately after writing.
6. Closes the writable stream and reports completion. The reconstructed file is restored to its exact original binary format in the same directory.

## Technical Specifications

### Cryptographic Implementation
- Algorithm: AES-256-GCM (Galois/Counter Mode) with 128-bit authentication tags.
- Key Derivation Function: PBKDF2-HMAC-SHA256.
- Iteration Count: 200,000 rounds.
- Salt: 16 cryptographically secure random bytes generated via window.crypto.getRandomValues.
- Initialization Vector: 12 unique random bytes generated per 1 MB payload block.
- Integrity: GCM provides authenticated encryption; any tampering or incorrect password immediately aborts extraction.

### Stream Optimization (64 KB Slicing)
Chromium FileSystemWritableFileStream instances can reject large continuous writes and throw unexpected EOF errors. To resolve this, all write operations pass through a segmented writer:

```javascript
async function w64(wr, data) {
  const bytes = typeof data === 'string' ? new TextEncoder().encode(data) : data;
  for (let i = 0; i < bytes.length; i += 65536) {
    await wr.write(bytes.subarray(i, Math.min(i + 65536, bytes.length)));
  }
}
```

### Memory Management Strategy
1. In Single HTML retrieval, the chunk array is progressively cleared during execution to allow immediate garbage collection:
   ```javascript
   C[i] = null;
   ```
2. In Split Folder decoding, part files are ingested sequentially via File.text() and parsed through string offsets rather than DOM nodes, holding only one slice in memory at any given moment. Peak RAM consumption remains flat at approximately 2 MB.

## Repository Structure

- index.html: File smuggling encoder interface.
- decoder.html: Standalone folder extraction and reconstruction utility.
- LICENSE: MIT License terms.
- README.md: Comprehensive technical documentation and operational guide.
- .gitignore: Git ignore definitions for temporary and system files.

## Author & Maintainer

- Developer: [tlha](https://github.com/c2-tlhah) ([@c2-tlhah](https://github.com/c2-tlhah))
- Bio: Innovating with data, driven by intelligence.
- GitHub: [https://github.com/c2-tlhah](https://github.com/c2-tlhah)

## Acknowledgements & Credits

- Concept and foundational PoC inspiration: [Eddie Chu](https://github.com/eddiechu) ([File-Smuggling](https://github.com/eddiechu/File-Smuggling))
- Threat modeling & taxonomy: MITRE ATT&CK [T1027.006 (HTML Smuggling)](https://attack.mitre.org/techniques/T1027/006/)
- Cryptographic specifications: W3C [Web Cryptography API](https://www.w3.org/TR/WebCryptoAPI/) (AES-256-GCM, PBKDF2)
- Local filesystem streaming: W3C [File System Access API](https://developer.mozilla.org/en-US/docs/Web/API/File_System_API)

## Legal and Security Notice

This utility is published strictly for authorized security assessments, penetration testing, red teaming, digital forensics, and legitimate administrative operations in restricted contexts. Users are solely responsible for ensuring compliance with all applicable legal, regulatory, and corporate policies.

## License

This project is licensed under the MIT License. See the LICENSE file for full terms.
