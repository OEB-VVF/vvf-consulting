Multi-Ledger Accounting for Odoo 19

Blueprint v1.0 — Adjustment-Based Alternative Ledgers

⸻

1. Objective

Enable multi-ledger accounting in Odoo 19 (e.g. Managerial, IFRS) without duplicating operational postings, by introducing:
	•	Alternative ledgers
	•	A ledger-specific Chart of Accounts
	•	A strict one-to-one account mapping
	•	Adjustment-only journal entries
	•	Ledger-specific reporting built on top of statutory accounting

This design preserves Odoo’s core accounting integrity while unlocking professional-grade reporting.

⸻

2. Core Accounting Philosophy

2.1 Single Source of Truth
	•	All operational transactions (sales, purchases, stock, payroll, etc.) are posted once into the statutory ledger.
	•	The statutory ledger remains the legal and operational reference.

2.2 Adjustment-Only Ledgers
	•	Alternative ledgers do not receive operational postings.
	•	Differences between accounting standards are expressed only through adjustment journal entries.

2.3 Ledger Reporting Formula

Ledger View = Statutory Ledger + Ledger-Specific Adjustments

This avoids:
	•	Parallel posting
	•	Data duplication
	•	Reconciliation nightmares

⸻

3. Functional Scope (Phase 1 – MVP)

Phase 1 focuses exclusively on alternative Chart of Accounts and adjustments.

Included:
	•	Ledger definition
	•	Ledger Chart of Accounts
	•	Mandatory account mapping
	•	Ledger-scoped adjustment journal entries

Excluded (later phases):
	•	Automated IFRS recognition
	•	Valuation engines
	•	Scheduling / amortization logic

⸻

4. Functional Architecture Overview
Operational Transactions
(Sales / Purchases / Stock)
        ↓
Statutory Ledger (Odoo Native)
        ↓
Adjustment Journal Entries
(per Alternative Ledger)
        ↓
Ledger-Specific Reporting

## **5. Core Objects & Data Model**

  

### **5.1 Ledger**

  

**Purpose**

Represents an alternative accounting universe (IFRS, Managerial, etc.).

  

**Key Attributes**

- Name
    
- Code
    
- Company
    
- Type (Managerial / IFRS / Other)
    
- Active flag
    

  

> The statutory ledger remains implicit and unchanged.

---

### **5.2 Ledger Chart of Accounts**

  

**Purpose**

Defines the account structure used **only for reporting** within a ledger.

  

**Key Rules**

- Independent from Odoo’s statutory CoA
    
- No direct posting allowed
    
- One ledger = one ledger-specific CoA
    

---

### **5.3 Account Mapping (Critical Component)**

  

**Purpose**

Connects statutory accounts to ledger accounts.

  

**Hard Rule**

  

> For a given company and ledger:

> **each statutory account must map to exactly one ledger account**

  

**Why**

- Guarantees deterministic reporting
    
- Prevents ambiguity
    
- Enforces accounting discipline
    

  

This mapping table is the **keystone** of the architecture.

---

## **6. Journal Entries & Posting Logic**

  

### **6.1 Journal Entry Classification**

  

Journal entries are split into two categories:

1. **Statutory Entries**
    
    - Normal Odoo behavior
        
    - No ledger assigned
        
    
2. **Ledger Adjustment Entries**
    
    - Explicitly linked to one alternative ledger
        
    - Created only via dedicated adjustment journals
        
    

---

### **6.2 Adjustment Entry Rules**

- Must balance
    
- Must reference a ledger
    
- Must never be generated automatically by operational flows
    
- Are fully auditable and reversible
    

---

## **7. Reporting Logic (Conceptual)**

  

When producing reports for a given ledger:

1. Take **all posted statutory entries**
    
2. Add **all adjustment entries for the selected ledger**
    
3. Translate statutory accounts → ledger accounts via the mapping table
    
4. Aggregate and present results using the ledger CoA
    

  

This logic applies identically to:

- Trial Balance
    
- P&L
    
- Balance Sheet
    

---

## **8. User Interface Scope**

  

### **8.1 Ledger Configuration**

- Create / manage alternative ledgers
    
- Enable / disable per company
    

  

### **8.2 Ledger Chart of Accounts**

- Maintain ledger-specific account structures
    
- Structural only (no postings)
    

  

### **8.3 Account Mapping Screen**

  

Mission-critical UX:

- One line per statutory account
    
- Mandatory completeness checks
    
- Clear visibility on missing mappings
    
- Bulk import/export capability
    

  

### **8.4 Adjustment Journal Entries**

- Dedicated journal type
    
- Ledger selection mandatory
    
- Clear visual distinction from statutory entries
    

---

## **9. Governance & Audit Principles**

- Clear separation between statutory and alternative accounting
    
- Full traceability of all adjustments
    
- Easy reconciliation between ledgers
    
- Auditor-friendly by design
    

---

## **10. Forward Compatibility (Not Implemented in v1)**

  

The architecture explicitly allows future extensions such as:

- IFRS 16 lease recognition wizards
    
- IFRS 15 revenue allocation
    
- Impairment and revaluation engines
    
- Automated schedules and reversals
    

  

These will **generate adjustment entries only**, never operational postings.

---

## **11. Summary**

  

This blueprint delivers:

- A clean, Odoo-native multi-ledger framework
    
- Zero duplication of postings
    
- Strong accounting governance
    
- A scalable foundation for IFRS and managerial reporting
    

  

**Phase 1 focuses on structure and discipline.**

**Complexity comes later — on purpose.**