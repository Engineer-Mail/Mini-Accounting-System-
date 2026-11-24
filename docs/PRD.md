# Product Requirements Document (PRD)
## Mini Accounting System — Release 1

| Document field | Value |
|---|---|
| Product | Mini Accounting System |
| Release | Release 1 / MVP |
| Document status | Draft for stakeholder approval |
| Source requirements | System Requirements Document v2.0; UI/UX Team Task Split; Stakeholder Workflows |
| Business scope | Single store |
| Interface language | Arabic (RTL) |
| Currency | One store-configured currency (not yet approved) |
| Prepared | 24 November 2026 |

---

## 1. Product Overview

The Mini Accounting System is an Arabic, single-store application for consolidating day-to-day store operations into one system. It manages products and stock, customers and suppliers, sales and purchase invoices, cash receipts and payments, expenses, other income, outstanding debts, reports, store settings, and backup/restore.

The system records a transaction once and, upon confirmation, updates its related stock, cashbox, and customer or supplier balance. It preserves the traceability of confirmed transactions and corrections.

Returns (`FR-055–FR-058`) are **optional and not included in the baseline Release 1 until explicitly approved**.

## 2. Problem Statement

Store operations maintained across separate records require repeated data entry and make it difficult to reconcile stock quantities, cash, outstanding customer balances, supplier balances, and business performance. The system aims to link each invoice to the related stock and financial movements and provide reliable, timely reports.

## 3. Product Vision

Provide a straightforward, secure, Arabic-first accounting and store-management application that enables store personnel to record transactions once, maintain consistent balances and stock, and review operational and financial status without relying on disconnected records.

## 4. Goals and Business Outcomes

| ID | Goal | Success measure / acceptance outcome |
|---|---|---|
| BR-01 | Consolidate store operations | Each confirmed invoice can be traced to its stock, cashbox, and party-balance effects, as applicable. |
| BR-02 | Reduce calculation errors | Totals and outstanding amounts are calculated automatically and agree with recorded transactions. |
| BR-03 | Understand stock and debts | Reports show quantities and balances by relevant period or filters. |
| BR-04 | Protect and recover records | Backup and restore complete successfully, and their results are reported accurately. |

## 5. Scope

### 5.1 Included in Release 1

- Login, session protection, logout, user management and permissions when multiple accounts are enabled.
- Product and category management, including product code, unit, prices, reference cost, stock, reorder level and active status.
- Customer and supplier records and statements.
- Sales invoices: cash, partial and credit; invoice search, detail and print; cancellation/reversal with retained original record.
- Purchase invoices: cash, partial and credit.
- Cashbox balance and movement records, receipts, payments, customer collections, supplier payments, expenses and other income.
- Customer and supplier debts, allocation of receipts/payments and balance details.
- Inventory movements, stock adjustment with reason, stock-level alerts, movement history and inventory reports.
- Dashboard and report suite, filters and browser-printable views.
- Store name, currency, invoice-number start and default reorder-level settings.
- Backup creation and restore with validation, confirmation and truthful result messaging.

### 5.2 Optional / approval-dependent

- Sales returns and purchase returns, including stock disposition and refund/credit effects (`FR-055–FR-058`).

### 5.3 Out of scope unless separately approved

- Multiple stores or branches in one database.
- Multiple currencies in a single transaction.
- Payroll, electronic tax integration, bank integration, specialized POS hardware/workflows or e-commerce.
- An assumed deployment model (offline/local or remote server); this must be decided.
- Any functionality not identified in the source requirements.

## 6. Stakeholders and Personas

| Stakeholder | Primary needs | Core system areas |
|---|---|---|
| Store owner | Understand sales, expenses, debts, cashbox and inventory value/status | Dashboard and reports |
| System administrator | Manage users, settings and backup/restore | User management, settings, backup |
| Accountant | Review financial records, balances and reports (workflow proposed; permissions not yet approved) | Reports, statements, cashbox |
| Cashier | Record collections, payments, expenses and review cashbox | Cashbox, debts, receipts/payments, expenses |
| Sales employee | Create sales invoices, record amounts actually received and print invoices | Sales |
| Purchasing/stock employee | Record purchases, track stock and perform adjustments | Purchases, inventory |
| Development and QA team | Implement and verify requirements and transaction integrity | All modules |

**Role/permission note:** Stakeholder needs do not automatically grant edit, approval, reversal or administrative privileges. Detailed permissions remain an owner decision.

## 7. User Experience and Product Principles

- Arabic interface and right-to-left (RTL) layout.
- Single-store experience and a single store-configured currency.
- Consistent navigation, tables, forms, buttons, dialogs, status labels and printable layouts.
- Confirmed financial and stock records must not be permanently deleted; use reversal/cancellation while retaining original records.
- All money and stock-affecting operations must validate before confirmation and record user and time.
- Forms and reports must have clear feedback, including empty, loading, validation-error, success and failure states where applicable.
- No invented business rules: unresolved policies must be explicitly documented and approved.
- Reports and print views must reflect the same underlying records and selected filters.

## 8. Functional Requirements

The following summarizes the source requirements by ID. The detailed acceptance conditions in the source requirements document remain authoritative.

### 8.1 Authentication and Users — FR-001–FR-005

- **FR-001:** Authenticate users using login credentials; provide a generic failed-login message.
- **FR-002:** Protect pages and actions with a valid authenticated session.
- **FR-003:** Create, edit and deactivate accounts; login names are unique; changes are traceable.
- **FR-004:** Assign roles or permissions when multiple accounts are enabled; enforce them server-side for every action.
- **FR-005:** Log out and invalidate the active session.


### 8.2 Products and Categories — FR-006–FR-011

- **FR-006:** Create, edit, view and search products.
- **FR-007:** Maintain product code, name, category, unit, selling price, reference cost, quantity, reorder level and status; reject negative values where specified.
- **FR-008:** Enforce unique product code.
- **FR-009:** Create, edit and view categories; protect categories referenced by products (move products or deactivate rather than deleting in-use categories).
- **FR-010:** Alert when stock is at or below the product's reorder level.
- **FR-011:** Prevent permanent deletion of products with transaction history; allow deactivation while preserving history.

### 8.3 Customers and Suppliers — FR-012–FR-016

- **FR-012:** Manage and search customers.
- **FR-013:** Manage and search suppliers.
- **FR-014:** Save required party name and optional contact details and notes; validate contact formats when provided.
- **FR-015:** Calculate party balances from confirmed transactions only.
- **FR-016:** Show party statement by selected period with transaction details and running/resulting balance.

### 8.4 Sales — FR-017–FR-025

- **FR-017:** Create an invoice with one or more valid line items, with customer or general sale as applicable.
- **FR-018:** Save invoice number, date, items, quantities, prices, discounts and total; invoice numbers must be unique; discount behavior depends on approved policy.
- **FR-019:** Support fully paid, partially paid and unpaid/credit invoices.
- **FR-020:** Calculate outstanding amount automatically and prevent negative remaining balance under default rules.
- **FR-021:** Deduct stock only when the sale is confirmed; check availability first under the default no-negative-stock policy.
- **FR-022:** Record only the amount actually received as a cashbox receipt.
- **FR-023:** Provide printable invoice view matching the saved invoice.
- **FR-024:** Search invoices by number, date and customer filters as available; view details without changing records.
- **FR-025:** Cancel/reverse a sale with permission and reason; preserve original and reversal; prevent duplicate reversal; reverse relevant stock and financial effects.

### 8.5 Purchases — FR-026–FR-032

- **FR-026:** Record supplier purchase invoice with products and quantities.
- **FR-027:** Save supplier, date, items, quantities, unit cost and total; calculate totals consistently with approved rounding.
- **FR-028:** Support fully paid, partially paid and credit purchases.
- **FR-029:** Increase stock only when purchase is confirmed.
- **FR-030:** Record only actual supplier payment as cashbox outflow.
- **FR-031:** Calculate supplier invoice outstanding amount.
- **FR-032:** Search and view purchase invoices by number, supplier or date.

### 8.6 Cashbox and Financial Transactions — FR-033–FR-041

- **FR-033:** Calculate cashbox balance from opening balance and confirmed movements.
- **FR-034:** Record dated amount, description, type, reference and party/transaction linkage where relevant.
- **FR-035:** Record customer collection, increase cashbox and reduce the allocated customer debt.
- **FR-036:** Record supplier payment, decrease cashbox and reduce allocated supplier debt.
- **FR-037:** Record and categorize expenses with date, amount, description and payment method; ensure the cashbox/report effect is recorded once.
- **FR-038:** Record other income with source, date, amount, description and collection method; cashbox effect occurs when received.
- **FR-039:** Provide cashbox statement for a selected period, with opening balance, receipts, payments and closing balance.
- **FR-040:** Reject zero or negative financial movement amounts.
- **FR-041:** Prevent payments exceeding available cashbox balance by default unless an explicit approved policy changes this.

### 8.7 Debts — FR-042–FR-048

- **FR-042:** Create customer debt for unpaid/partially paid sales.
- **FR-043:** Create supplier debt for unpaid/partially paid purchases.
- **FR-044:** Apply partial or full customer collections to an existing invoice/debt or record unallocated collection where supported.
- **FR-045:** Apply partial or full supplier payments to an existing supplier debt.
- **FR-046:** Report aggregate customer receivables and supplier payables that reconcile to individual balances.
- **FR-047:** Trace balances to originating invoices and receipts/payments.
- **FR-048:** Reject over-allocation/overpayment by default; any credit-balance behavior must be approved, clear and permission-controlled.

### 8.8 Inventory — FR-049–FR-054

- **FR-049:** Display current product quantity derived from confirmed stock movements.
- **FR-050:** Update stock once per confirmed movement and retain source reference; sales decrease stock, purchases increase it.
- **FR-051:** Adjust stock by increase/decrease with reason; record before/after quantity, user and time.
- **FR-052:** Report products at or below reorder level.
- **FR-053:** Prevent negative stock by default; any exception requires approved policy.
- **FR-054:** Show product movement history by period, including incoming, outgoing, adjustments and source.

### 8.9 Returns — FR-055–FR-058 (Approval-dependent)

- **FR-055:** Register sales return linked to original sale and limit return quantity to the sold quantity less prior returns.
- **FR-056:** Restock only items accepted for resale; record why non-resalable items are excluded.
- **FR-057:** Record refund or customer credit effect without exceeding applicable amounts under the approved policy.
- **FR-058:** Register purchase return linked to original purchase and update stock and supplier balance consistently.
- **Release gate:** Do not include these flows in Release 1 until the product owner approves the feature and its rules.

### 8.10 Dashboard and Reports — FR-059–FR-068

- **FR-059:** Dashboard summary of sales, purchases, expenses, cashbox, debts and low stock, reconciling with reports.
- **FR-060:** Sales report by period; exclude drafts and cancelled invoices from net sales.
- **FR-061:** Purchase report by period, with supplier/product filtering as applicable.
- **FR-062:** Expenses and other-income report by period, type, category and source as applicable.
- **FR-063:** Customer balance report.
- **FR-064:** Supplier balance report.
- **FR-065:** Cashbox movements and balance report.
- **FR-066:** Inventory status and movement report.
- **FR-067:** Apply report filters appropriate to the report.
- **FR-068:** Provide browser-printable reports showing selected period, currency and report data.

### 8.11 Settings and Backup/Restore — FR-069–FR-073

- **FR-069:** Create backup at a user-selected local path; verify writable path and report genuine success/failure.
- **FR-070:** Validate backup file before restore; warn that restore replaces current data; require confirmation.
- **FR-071:** Show clear operation result, appropriate failure reason and backup path when relevant; do not expose sensitive technical detail or claim false success.
- **FR-072:** Configure store name, currency and approved general settings; historical records must not be reinterpreted by changing currency without an approved conversion policy.
- **FR-073:** Configure invoice-number start and default reorder level; avoid invoice number duplication.

## 9. Key User Journeys

### UC-01 — Sell a product
1. Select customer or general sale and add product lines.
2. Enter quantities, prices and discount if permitted.
3. Enter actual received amount.
4. Validate invoice, stock, amounts and permissions.
5. Confirm invoice.
6. Record stock deduction, actual receipt and customer outstanding amount as applicable; provide invoice view/print.

**Failure cases:** insufficient stock, invalid amount, duplicate invoice number, invalid item or permission denied. A failed/unconfirmed invoice must not produce confirmed stock/cash/debt effects.

### UC-02 — Purchase products
1. Select supplier and add products, quantities and costs.
2. Enter actual amount paid.
3. Validate and calculate totals and outstanding amount.
4. Confirm purchase.
5. Increase stock, record actual cash payment and supplier debt as applicable.

### UC-03 — Collect customer debt
1. Select customer and debt/invoice.
2. Enter collection amount.
3. Validate amount and allocation against outstanding debt under approved rules.
4. Confirm collection.
5. Increase cashbox and reduce customer balance; show transaction/receipt details.

### UC-04 — Record expense
1. Select expense category.
2. Enter amount, date and description/payment method.
3. Validate amount and cashbox availability under default policy.
4. Save the expense and associated cashbox movement once.
5. Show confirmation and updated balance.

### UC-05 — Adjust stock
1. Select product.
2. Enter increase/decrease and reason.
3. Display before and after quantities.
4. Validate reason and no-negative-stock policy.
5. Confirm and record movement, user and timestamp.

### UC-06 — Backup and restore
1. Select backup creation or restore.
2. Select path or backup file.
3. Validate writable location or backup file.
4. For restore, display replacement warning and request confirmation.
5. Execute and display truthful success/failure result; validate data after restore.

### UC-07 — Returns (conditional)
Only after approval: select original invoice, lines and quantities, reason, stock disposition and financial treatment; validate previous returns and record linked return and its effects.

## 10. Business Rules and Integrity

1. Financial movement amount must be greater than zero.
2. Invoice outstanding = invoice total − allocated confirmed payments.
3. Over-allocation/overpayment is rejected by default unless a policy is explicitly approved.
4. Confirmed sales decrease stock; confirmed purchases increase stock; drafts do not change stock.
5. Negative stock is prohibited by default.
6. Confirmed records are not permanently deleted; correction uses reversal/cancellation retaining original record.
7. Cashbox reflects actual money received or paid, not the full amount of a credit invoice.
8. Party statements, cashbox reports and stock history reconcile with underlying movement records.
9. Returns are excluded from baseline Release 1 until approved.
10. Transaction effects must be linked to their originating record and occur exactly once.

## 11. Data Domains

The implementation is expected to support at least these data domains, based on the source requirements:

- **Users and permissions:** login identifier, protected password, active status, role/permissions when enabled.
- **Products and categories:** unique code, name, category, unit, selling price, reference cost, quantity, reorder level, status.
- **Customers and suppliers:** name, optional contact information and notes.
- **Sales and purchase invoices:** number, date, party, line items, quantities, prices/costs, discount, total, paid, outstanding and status.
- **Cashbox movements:** date, amount, description, type, reference, party and user.
- **Stock movements:** product, incoming/outgoing quantity, reason, date, source and user.
- **Expenses and other income:** date, value, category/source, description and related movement.
- **Audit history:** action/change type, affected record, user, timestamp and cancellation/adjustment reason.

The exact schema, keys, indexes and API contracts are implementation artifacts and must align with the approved database and API design documents.

## 12. Non-functional Requirements (NFR)

These are source-defined targets and constraints; performance and environment assumptions must be confirmed.

| ID | Requirement | Target / constraint |
|---|---|---|
| NFR-01 Performance | Routine page requests | 95% of routine pages within 3 seconds under agreed reference load; standard report within 10 seconds. Reference load must be defined. |
| NFR-02 Security | Authentication and authorization | Passwords are not stored as plain text; idle session timeout configured by administrator; permission checks apply to all actions. |
| NFR-03 Data protection | Store isolation | Data is scoped to one store and accessible only to authorized users. |
| NFR-04 Record integrity | Financial/stock records | Confirmed movements are recorded once with user/time; corrections preserve original record. |
| NFR-05 Backup/recovery | Backup and restore | Procedures documented; actual backup and restore must be tested before acceptance. |
| NFR-06 Usability | Arabic UI and feedback | Clear Arabic interface and actionable validation/error messages. |
| NFR-07 Compatibility | Browser/device support | Must work on modern browsers and devices approved by the owner. |
| NFR-08 Auditability | Change history | Creation, modification, confirmation and cancellation actions record user and timestamp. |
| NFR-09 Privacy | Data minimization | Reports show only data necessary for the authorized role; passwords and verification secrets are not displayed. |
| NFR-10 Calculation accuracy | Currency and rounding | One approved rounding/decimal precision policy must be used consistently across line items, invoices and reports. |

## 13. Acceptance Criteria

Release 1 is acceptable when:

- [ ] Functional requirements `FR-001–FR-073` are implemented and verified, excluding return requirements `FR-055–FR-058` unless approved.
- [ ] Invoice totals, payments and remaining balances reconcile to recorded transactions.
- [ ] Stock quantities reconcile to opening stock and confirmed stock movements.
- [ ] Cashbox balance reconciles to opening balance, receipts, payments, expenses and received other income.
- [ ] Customer and supplier balances reconcile to their detailed transaction histories.
- [ ] Unauthorized users cannot access protected pages or perform restricted actions.
- [ ] Confirmed records can be reversed/cancelled without deleting the original history.
- [ ] Backup creation and restore are successfully tested; restored data is validated.
- [ ] Arabic RTL layout, print views and approved browser/device compatibility are verified.
- [ ] Performance targets are tested against the agreed reference environment and load.
- [ ] The owner has approved all required open decisions.

## 14. Dependencies and Assumptions

- Product owner provides decisions for currency, discount, rounding, invoice numbering, deployment, role permissions and negative-stock policy.
- The target devices, browsers and deployment model are selected before technical design is finalized.
- The store operates as one store and uses one configured currency in the baseline.
- Returns are a gated feature and not assumed to be part of MVP.
- A baseline/reference dataset and performance load must be agreed for testing.
- The UI team shares a consistent design system, RTL conventions, component patterns and page layout before merging module designs.

## 15. Open Decisions / Product-owner Approval

| Decision | Default / source position | Required resolution |
|---|---|---|
| Deployment | Not decided: local/offline or remote server | Select deployment model and connectivity expectations. |
| Target devices and browsers | Not specified | Approve supported browser/device matrix. |
| Currency | One currency per store; actual currency not chosen | Select currency and display format. |
| Discount policy | Not specified | Define discount types, limits and authorization. |
| Rounding / precision | Not specified | Approve precision and rounding method, including line vs invoice rounding. |
| Invoice numbering | Configurable starting number; value not chosen | Approve starting number, format and sequence behavior. |
| Default reorder level | Configurable; value not chosen | Approve default and override behavior. |
| Multiple user accounts | Conditional in scope | Decide whether multiple accounts are enabled. |
| Roles and permissions | Detailed policy not specified | Define roles and permission matrix. |
| Returns | Optional; approval required | Decide whether to include in Release 1 and approve return rules. |
| Overpayment / credit balance | Reject by default | Decide whether customer/supplier credit balance may be allowed. |
| Negative stock | Prevent by default | Decide whether any exception is allowed and who can authorize it. |
| Performance reference | Targets are proposed | Define reference hardware, load and dataset. |
| Opening balances / stock | Initialization approach not detailed | Approve initialization and reconciliation process before go-live. |

## 16. Risks

| Risk | Impact | Mitigation / control |
|---|---|---|
| Unapproved business policies | Rework or inconsistent behavior | Resolve open decisions and record approvals before implementation/testing. |
| Incorrect or duplicate transaction effects | Cash, stock or debt inconsistencies | Atomic transaction handling, source linkage, duplicate prevention and reconciliation tests. |
| Unauthorized actions | Financial or stock manipulation | Enforce permission checks for all endpoints/actions and audit changes. |
| Backup not usable | Data loss | Test restore using real backup files and validate reconciled records. |
| Unclear return handling | Incorrect inventory/refund/debt treatment | Keep returns gated until owner approves policy. |
| Rounding inconsistency | Discrepant totals | Adopt and test a single rounding rule. |
| Unspecified deployment environment | Compatibility or security gaps | Decide deployment model and environment before technical implementation. |

## 17. Release Plan (High-level)

1. **Approval & baseline:** approve PRD, open decisions, roles and deployment assumptions.
2. **UX & design system:** agree Arabic RTL layout, components and screen patterns; design modules according to team split.
3. **Implementation foundation:** authentication, master data, audit model and shared application structure.
4. **Transaction modules:** sales, purchases, cashbox, debts and inventory movements.
5. **Reporting & settings:** dashboard, reports, settings and backup/restore.
6. **Integration & QA:** end-to-end flows, reconciliation, security, performance, backup/recovery and usability tests.
7. **Acceptance & release:** owner acceptance, deployment preparation, training/operational handoff.

## 18. Team Design / Ownership Alignment

- **Teammate 1 — Foundation & Master Data:** login, application layout, users/permissions, products/categories, customers/suppliers (`FR-001–FR-016`).
- **Teammate 2 — Sales & Purchases:** invoice lists, create/edit/details, printing, sale reversal (`FR-017–FR-032`).
- **Teammate 3 — Cashbox, Financial Transactions & Debts:** cashbox, receipts/payments, customer collections, supplier payments, expenses, other income, statements and debts (`FR-033–FR-048`, plus `FR-015–FR-016` statement-related work).
- **Teammate 4 — Inventory, Dashboard, Reports, Settings & Backup:** stock screens and movements (`FR-049–FR-054`), dashboard/reports (`FR-059–FR-068`), settings/backup (`FR-069–FR-073`), relevant `NFR-05`.

Shared design rules: Arabic RTL, single store, one currency, reusable components, consistent number/date/amount formatting, responsive behavior as approved, and no unsupported feature additions.

## 19. End-to-end Integration Scenarios

1. **Sale:** create invoice → confirm → stock decreases → actual receipt enters cashbox → customer debt updates if unpaid/partial → report and statement reflect it.
2. **Purchase:** create purchase invoice → confirm → stock increases → actual payment leaves cashbox → supplier balance updates → reports reflect it.
3. **Customer collection:** allocate collection → cashbox increases → customer debt decreases.
4. **Supplier payment:** allocate payment → cashbox decreases → supplier debt decreases.
5. **Stock adjustment:** record reason and quantity change → stock and movement history update.
6. **Reports:** date/filter selections produce totals and details matching transaction records.
7. **Backup and restore:** create backup → verify result → validate restore file → warn/confirm → restore → verify reconciled data.

---

## Source and traceability

This PRD is based on:
- **System Requirements Document:** Mini Accounting System, version 2.0, Release 1 draft for approval (Arabic).
- **Mini Accounting System — UI/UX Design Task Split (4 Teammates).**
- **Stakeholder Workflows — Mini Accounting System.**

Where the source leaves a business policy undecided, this PRD records it as an open decision rather than assuming a rule.
