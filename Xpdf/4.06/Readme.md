# 💥 Xpdf 4.06 Stack Exhaustion (DoS) Vulnerabilities

This repository contains the Proof of Concept (PoC) generator and crash details for multiple stack exhaustion vulnerabilities discovered in **Glyph & Cog Xpdf 4.06**.

Due to unbounded recursion in the `Catalog.cc` module, an attacker can craft malicious PDF files with deeply nested tree structures to cause a reliable Denial of Service (DoS) in applications utilizing the Xpdf library (e.g., `pdftotext`).

## 📌 Vulnerability Summary

* **Target Software:** Xpdf 4.06 (Official Windows release `xpdf-tools-win-4.06` & Source build)
* **Vulnerable Binary:** `pdftotext.exe`
* **Vulnerability Type:** Stack Exhaustion / Uncontrolled Recursion (CWE-674)
* **Impact:** Denial of Service (DoS) / Application Crash (Access Violation 0xC0000005)

## 🛠️ Root Cause Analysis

It is an architectural flaw in `Catalog.cc`, where multiple recursive tree-parsing functions rely on a `touchedObjs` array to prevent circular loops, but completely lack a **recursion depth limit**. 

By creating a PDF with extremely deep (e.g., 5,000+ depth), linear (non-circular) tree structures, an attacker can bypass the loop checks. The uncontrolled recursion leads to a stack overflow.

We have identified three distinct attack vectors triggered by different tree structures:

1. **Vector 1:** Page Label Tree -> `Catalog::readPageLabelTree2` (Line 1072)
2. **Vector 2:** Embedded Files Tree -> `Catalog::readEmbeddedFileTree` (Line 850)
3. **Vector 3:** Page Tree -> `Catalog::loadPage2` (Line 746)

## 🚀 Steps to Reproduce

### 1. Trigger the Crash
No external dependencies are required. Just run the Python script:
```bash
pdftotext.exe poc_page_labels.pdf
```

