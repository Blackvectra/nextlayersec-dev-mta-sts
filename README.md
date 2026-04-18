# MTA-STS Policy Host -- nextlayersec.dev

This repository serves the MTA-STS policy for `nextlayersec.dev` via GitHub Pages.
Policy is accessible at: `https://mta-sts.nextlayersec.dev/.well-known/mta-sts.txt`

Part of the NextLayerSec email security framework.
Full documentation: [nextlayersec-email-security](https://github.com/Blackvectra/nextlayersec-email-security)

---

## What is MTA-STS?

MTA-STS (Mail Transfer Agent Strict Transport Security) is an email security standard
defined in [RFC 8461](https://datatracker.ietf.org/doc/html/rfc8461) that allows domain
owners to enforce TLS encryption on inbound SMTP connections.

Unlike SPF, DKIM, and DMARC -- which authenticate the sender -- MTA-STS protects
the transport layer. It prevents SMTP downgrade attacks and DNS-based MX hijacking
by forcing sending MTAs to validate TLS certificates before delivering mail.

---

## Repository Structure

```
.nojekyll                  <- disables Jekyll to allow .well-known/ serving
.well-known/
  mta-sts.txt              <- MTA-STS policy file
CNAME                      <- custom domain: mta-sts.nextlayersec.dev
README.md
```

---

## Policy Configuration

```
version: STSv1
mode: enforce
mx: nextlayersec-dev.l-v1.mx.microsoft
max_age: 86400
```

| Field | Value | Notes |
|---|---|---|
| `version` | STSv1 | Only supported version |
| `mode` | enforce | Sending MTAs must validate TLS or reject delivery |
| `mx` | nextlayersec-dev.l-v1.mx.microsoft | DNSSEC-aware M365 endpoint |
| `max_age` | 86400 | Policy cache TTL -- 24 hours |

---

## DNS Records

| Type | Name | Value | Notes |
|---|---|---|---|
| CNAME | `mta-sts` | `blackvectra.github.io` | DNS-only, no proxy |
| TXT | `_mta-sts` | `v=STSv1; id=20260419` | Bump id on every policy change |
| TXT | `_smtp._tls` | `v=TLSRPTv1; rua=mailto:support@nextlayersec.dev` | TLS failure reporting |

---

## Deployment Notes

**Why `.nojekyll` is required**
GitHub Pages uses Jekyll by default which silently blocks dotfolders including `.well-known/`.
An empty `.nojekyll` file disables Jekyll processing and allows static file serving.

**Why Cloudflare proxy must be disabled**
GitHub Pages handles TLS termination for custom domains. Cloudflare proxying (orange cloud)
intercepts this and breaks certificate provisioning. CNAME must be DNS-only (grey cloud).

**DNSSEC-aware MX endpoint**
`nextlayersec.dev` uses the DNSSEC-aware Microsoft 365 MX endpoint (`l-v1.mx.microsoft`).
Enabled via Exchange Online PowerShell: `Enable-DnssecForVerifiedDomain -DomainName nextlayersec.dev`

**Updating the policy**
Whenever `mta-sts.txt` is modified, update the `id` value in the `_mta-sts` TXT record.
Sending MTAs cache the policy -- a changed `id` signals them to re-fetch.
Use the current date in `YYYYMMDD` format as a versioning convention.

---

## Validation

```bash
# Verify policy file is serving
curl https://mta-sts.nextlayersec.dev/.well-known/mta-sts.txt

# Verify DNS TXT record
nslookup -type=TXT _mta-sts.nextlayersec.dev

# Full MTA-STS validation
# https://mxtoolbox.com/SuperTool.aspx -> MTA-STS Lookup -> nextlayersec.dev
```

```powershell
# Verify DNSSEC status in M365
Get-DnssecStatusForVerifiedDomain -DomainName nextlayersec.dev | Format-List *
```

---

## Email Security Stack -- nextlayersec.dev

| Control | Status |
|---|---|
| SPF | v=spf1 include:spf.protection.outlook.com -all |
| DKIM | Selector1 + Selector2 active |
| DMARC | p=reject |
| MTA-STS | enforce |
| DNSSEC | Enabled -- signedDelegation |
| DNSSEC-aware MX | l-v1.mx.microsoft endpoint |
| TLS-RPT | Configured |

---

## Related Repositories

- [nextlayersec-email-security](https://github.com/Blackvectra/nextlayersec-email-security) -- Full email security framework documentation
- [blackvectra.github.io](https://github.com/Blackvectra/blackvectra.github.io) -- MTA-STS policy host for nrgtechservices.com
- [nextlayersec-mta-sts](https://github.com/Blackvectra/nextlayersec-mta-sts) -- MTA-STS policy host for nextlayersec.io

---

## References

- RFC 8461 -- SMTP MTA Strict Transport Security
- RFC 8460 -- SMTP TLS Reporting

---

*Maintained by [NextLayerSec](https://nextlayersec.io)*
