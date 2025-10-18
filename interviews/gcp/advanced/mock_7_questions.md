# Google Cloud Platform (GCP) - Advanced Interview Mock #7

> **Difficulty:** Advanced  
> **Duration:** ~60 minutes  
> **Goal:** Evaluate deep proficiency in secure data mobility, cross-project encryption lifecycle, multi-tenant isolation, and production resilience under constrained governance.

---

## 🧠 Section 1: Core Questions

1. You have a Compute Engine VM whose boot + attached persistent disks are encrypted with a Customer-Managed Encryption Key (CMEK) in KMS inside Project A (Org X). You now need to move (not just clone at the app layer) that encrypted data volume to Project B (different org / different billing) and attach it to a new VM there, preserving encryption posture and minimizing exposure. No plaintext export is allowed, and the destination must retain independent key control. Describe an end-to-end approach: snapshot/export mechanics, handling of CMEK vs Customer-Supplied Encryption Key (CSEK) options, cross-project IAM & KMS key policy adjustments, artifact transfer (e.g., via Cloud Storage / signed URLs), re-encryption strategy (if rotating to a new CMEK), integrity verification, risk points (data remanence / key revocation timing), and rollback.  

---

## ⚙️ Section 2: Scenario

(Coming soon)

---

## 🧩 Section 3: Problem-Solving

(Coming soon)
