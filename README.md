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

A standalone, browser-based utility that packages arbitrary files into self-extracting HTML documents with client-side AES-256-GCM encryption, chunked streaming, and split-file decoding. Runs 100% offline with zero dependencies.

---

## Features

- **Zero Server Footprint**: Runs entirely client-side via Web Crypto API. No data leaves your machine.
- **Authenticated Encryption**: AES-256-GCM with PBKDF2 (200,000 rounds SHA-256, 16-byte salt, unique 12-byte IV per chunk).
- **Anti-OOM Architecture**: Streams directly to disk via File System Access API in 64 KB slices, preventing browser tab crashes on multi-GB files.
- **Two Packaging Modes**: Single self-extracting HTML document or split multi-part folder (~50 MB parts) for arbitrary file sizes.
- **Dedicated Standalone Decoder**: Reads parts as raw text without DOM overhead, extracting files at ~2 MB peak RAM.

---

## Quick Start

### 1. Build Payloads (`index.html`)

1. Open `index.html` in Chrome or Edge.
2. Select your file, optional password (enables AES-256-GCM), and optional note.
3. Choose an output mode:
   - **Build Single HTML**: Generates a single self-extracting `.html` file.
   - **Build Split to Folder**: Slices large files into uniform ~50 MB `.part*.html` files and saves `decoder.html` into the chosen folder.

### 2. Retrieve Files

- **Single HTML**: Open the generated `.html` file in Chrome/Edge, enter the password if set, and click **Retrieve File**.
- **Split Parts**: Open `decoder.html` on the target machine, enter the password if set, click **Open Folder & Decode**, and select the folder with the part files. The file is decrypted and reconstructed directly in the same folder.

---

## Under the Hood

- **Key Derivation**: PBKDF2 with SHA-256, 200,000 iterations, 16-byte cryptographically secure salt.
- **Chunk Cipher**: AES-256-GCM authenticated encryption with a unique 12-byte IV per 1 MB block.
- **Stream Slicing**: Internal `w64()` writer pipes data in 64 KB blocks, eliminating Chromium `FileSystemWritableFileStream` buffer overflow and unexpected EOF errors.
- **Memory Recycling**: Progressive array dereferencing (`C[i] = null`) and headless text streaming allow decoding arbitrarily large files within ~2 MB RAM.

---

## Author & Credits

- Author: [tlha](https://github.com/c2-tlhah) ([@c2-tlhah](https://github.com/c2-tlhah))
- Inspiration: [Eddie Chu](https://github.com/eddiechu) ([File-Smuggling](https://github.com/eddiechu/File-Smuggling))
- Threat Taxonomy: MITRE ATT&CK [T1027.006 (HTML Smuggling)](https://attack.mitre.org/techniques/T1027/006/)

---

## License

Distributed under the [MIT License](LICENSE).
