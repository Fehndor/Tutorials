# 🧾 ORACLE DATABASE LICENSING CHEAT SHEET (2025)

## 1. 📦 Database Editions

| Edition | Key Traits | Licensing Notes |
|---|---|---|
| Enterprise Edition (EE) | Full feature set (Advanced Security, RAC, Partitioning, etc.) | Base license + additional licenses for most optional features |
| Standard Edition 2 (SE2) | For smaller deployments (max 2 sockets) | Includes basic HA, no extra options purchasable |
| Express Edition (XE) | Free limited version | No license required, resource-limited |

## 2. ⚙️ Licensing Metrics

| Metric | Applies To | Description |
|---|---|---|
| Processor License | Server installations | License every physical processor (or core) Oracle uses; core factor applies |
| Named User Plus (NUP) | Per user/device access | Minimum NUPs per processor apply (usually 25 per core for EE) |

> Tip: You can’t mix Processor and NUP licenses in the same environment.

## 3. ☁️ Cloud Licensing

- **Oracle Cloud (OCI):** BYOL or subscription; counts vCPUs directly  
- **AWS / Azure:** Core factor rules apply; 2 vCPUs = 1 Oracle Processor License (for Intel/AMD)  
- **Hybrid deployments:** Each environment licensed independently unless using Oracle Cloud Free Tier

## 4. 🔒 Optional Features (EE Only)

Each of these requires a separate license on top of Enterprise Edition:

- Oracle Partitioning
- Real Application Clusters (RAC)
- Advanced Compression
- Advanced Security
- Data Masking / Subsetting
- Active Data Guard
- Diagnostics & Tuning Packs

> Note: These options are automatically detected by Oracle’s LMS scripts — disable unused options to avoid audit issues.

## 5. 🧠 Common Mistakes to Avoid

- ❌ Assuming test/dev environments don’t need licenses — they usually do (unless covered by a developer license).  
- ❌ Running RAC on SE2 — not allowed.  
- ❌ Enabling features “by accident” (like using partitioning syntax) — it counts as usage.  
- ❌ Not counting all users under NUP — every human or app user counts.  
- ❌ Misunderstanding virtualization — Oracle does not recognize soft partitioning (e.g., VMware) for license reduction.

## 6. 🧩 Audits and Compliance

- Oracle LMS (License Management Services) performs audits; results rely on their collection tools.  
- Keep your own deployment inventory and feature usage reports (DBA_FEATURE_USAGE_STATISTICS).  
- Regularly review license entitlements vs. actual use.

## 7. ✅ Best Practices

- Document your deployment: edition, cores, users, and enabled options.  
- Disable unused features at database level.  
- Track CPU usage and virtualization boundaries carefully.  
- For cloud, clarify “BYOL” vs. “License Included” terms.  
- Use Oracle’s official Licensing Manual for your version (19c / 21c).

## 8. 📚 Official Resources

- Oracle Database Licensing Information User Manual (21c)  
- Oracle License Definitions and Rules  
- Oracle Technology Network License Terms
