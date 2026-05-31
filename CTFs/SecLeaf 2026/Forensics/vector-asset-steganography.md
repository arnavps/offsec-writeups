# Vector Asset Steganography Writeup

## 🧠 Challenge Overview
A "corrupted vector asset" was recovered from an internal branding archive. The hint strongly suggested that the file was not actually broken, but that something hidden existed inside the vector structure itself.

Because vector files (specifically SVGs) are text-based XML documents, they are ideal for hiding data in:
* Metadata fields
* Invisible layers (using opacity or display attributes)
* Embedded base64 payloads
* Malformed or unused custom XML attributes

---

## 🔍 Step 1: Identify the File Type
Even though the file was labeled as an image/vector asset, the first step is to confirm its real format programmatically:
```bash
file asset.*
```
This typically reveals it as:
* An SVG image
* An XML document
* Or generic ASCII text

In this challenge, it was confirmed to be a standard SVG/XML-based vector graphic.

---

## 📄 Step 2: Inspect the Raw Structure
Since SVG files are human-readable, we inspect them directly:
```bash
cat asset.svg
```
We immediately notice standard XML elements and tags:
```xml
<svg ...>
  <metadata>...</metadata>
  <g>
    <path .../>
    <text .../>
  </g>
</svg>
```
This confirms it is a vector graphic rather than a binary image.

---

## 🔎 Step 3: Search for Suspicious Data
We search the SVG file for common steganography keywords and patterns:
```bash
grep -iE "base64|flag|hidden|SecLeaf|opacity|display" asset.svg
```

This reveals suspicious elements such as:
1. Hidden text layers (`opacity="0"` or `display="none"`)
2. Unused `<text>` nodes
3. Embedded encoded strings inside attributes

---

## 🧩 Step 4: Extract the Base64 Payload
A key discovery was a massive Base64-encoded string embedded inside the SVG document, typically found in a data URI field:
```text
data:image/png;base64,iVBORw0KGgoAAAANS...
```

We extract this encoded block:
```bash
grep -oP 'base64,.*' asset.svg | cut -d, -f2 > b64.txt
base64 -d b64.txt > hidden.png
```

---

## 🖼️ Step 5: Analyze the Extracted File
After decoding, we inspect the resulting image file:
```bash
file hidden.png
strings hidden.png | grep SecLeaf
```

The decoded PNG image contains the hidden flag.

---

## 🏁 Final Flag
```text
SecLeaf{placeholder_flag}
```

---

## 🧠 Key Takeaway
File extensions and the visual representation of a file do not guarantee its actual underlying structure. Even though it appeared as a corrupted vector graphic, the payload was nested directly inside the SVG XML structure as an encoded asset.

---

## 🧾 Summary of Approach
1. **Verify File Type:** Checked the raw file signature with `file`.
2. **Text-Based Parsing:** Treated the SVG as a structured text file.
3. **Keyword Scanning:** Searched the XML structure for anomalies.
4. **Payload Extraction:** Carved out the Base64 block.
5. **Decoding & Verification:** Decoded the data stream and retrieved the hidden flag.
