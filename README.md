<img width="2816" height="1536" alt="template viewer" src="https://github.com/user-attachments/assets/58299469-7c74-43a6-9f4b-a54798ed81b3" />

# SAM Template Viewer - Technical Documentation

## 1. Working Principle

The **SAM Template Viewer** is a client-side, single-file application designed to visualize AWS Serverless Application Model (SAM) and CloudFormation templates.

### Parsing Logic

Standard YAML parsers fail when encountering AWS-specific tags (Intrinsic Functions) like `!Ref`, `!Sub`, or `!FindInMap` because these are not part of the standard YAML specification.

This viewer operates on the following logic:

- **Input:** Takes raw YAML or JSON text from the editor pane.
- **Schema Extension:** Extends the `js-yaml` default schema and registers AWS-specific intrinsic tags. Instead of executing logic or throwing errors, the parser converts these tags into custom internal object structures (e.g., `{ "aws:Ref": "Value" }`).
- **DOM Generation:** JavaScript traverses the parsed object tree recursively, detecting intrinsic-function objects and rendering them as UI badges or collapsible sections.
- **Interactive Layer:** Event listeners handle expanding/collapsing sections, jumping to code lines, and cloning resources.

---

## 2. Possible Limitations

Although robust for visualization, the viewer has technical limitations:

### Comment Loss in AST
`js-yaml` strips comments during parsing, so the visualization panel cannot display comments.

### Cloning Fragility
To preserve comments and indentation during cloning, the tool performs Regex-based text manipulation on the raw input instead of regenerating YAML. Complex indentation edge cases may cause minor misalignment.

### Performance
Parsing occurs on the browser’s main thread. Very large templates (10,000+ lines) may lead to temporary UI freezing despite debouncing.

### Validation
The viewer validates YAML syntax only. It does **not** validate CloudFormation semantics (e.g., whether `MemorySize` is valid for `AWS::S3::Bucket`).

---

## 3. Technical Specifications

### Architecture

- **Format:** Single-file HTML (`.html`)
- **Dependencies:** No build steps required; runs directly in-browser
- **Languages:** HTML5, CSS3 (CSS Variables), ES6+ JavaScript

### Libraries & CDNs

The tool loads two external libraries via CDN:

- **js-yaml (v4.1.0)**  
  - *Purpose:* YAML → JS object parsing  
  - *CDN:* https://cdnjs.cloudflare.com/ajax/libs/js-yaml/4.1.0/js-yaml.min.js  

- **Font Awesome (v6.4.0)**  
  - *Purpose:* UI icons  
  - *CDN:* https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css  

### Custom YAML Schema

Recognized AWS intrinsic functions:

`Ref, Sub, Join, Select, Split, FindInMap, GetAtt, ImportValue, Base64, Equals, And, Or, Not, If, Condition, GetAZs, Cidr`

### Visualization Features

- **Resource Color Coding:** CSS variables map CloudFormation resource types to border colors.
- **Accordion UI:** Toggle-based expand/collapse (`.expanded` class).
- **Navigation:** Line-height × index approximation syncs editor scroll with visualization.

---

## 4. Dependencies

To run locally, an internet connection is required for the external assets unless downloaded manually.

| Dependency     | Version | Usage              |
|----------------|---------|--------------------|
| js-yaml        | 4.1.0   | Core parsing engine |
| Font Awesome   | 6.4.0   | Iconography        |
| Google Fonts   | -       | “Segoe UI”, “Source Code Pro” |

---

## 5. FAQ

### **Q: Is my SAM template data sent to a server?**  
**A:** No. All processing is fully client-side. No data leaves your machine.

### **Q: Why are resources collapsed by default?**  
**A:** To provide a high-level architectural overview similar to Swagger/OpenAPI editors.

### **Q: Can I use JSON templates instead of YAML?**  
**A:** Yes. YAML is a superset of JSON, and the parser handles JSON templates seamlessly.

### **Q: I cloned a resource, but its indentation is wrong.**  
**A:** This happens when your file mixes tabs/spaces or has inconsistent indentation. Manual adjustment may be needed.

### **Q: Why doesn't the visualizer show syntax errors immediately?**  
**A:** Input parsing is debounced by 500ms to prevent UI flicker and improve usability.

### **Q: How do I add support for a new AWS Intrinsic function?**  
**A:** Add the new tag name to the `awsTags` array in the JavaScript section. The schema auto-registers it on reload.

