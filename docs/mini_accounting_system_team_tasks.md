# Mini Accounting System — UI/UX Design Task Split (4 Teammates)

## Project goal

Design the **Arabic, single-store Mini Accounting System** described in the requirements document. Each teammate designs a separate group of screens first. After that, we combine the work into one consistent Figma file/design system.

**Important:** This is the design phase only. Do not independently change business rules or add features that are not in the source document.

---

## 1. Team split

### Teammate 1 — Foundation, Login & Master Data

**Own these screens:**
1. Login screen.
2. Main application layout: sidebar/navigation, header, page title, user menu, notifications.
3. User management: users list, add/edit user, role/permissions.
4. Products: products list, search/filter, add/edit product, product details.
5. Categories: list, add/edit category.
6. Customers: list, add/edit customer, customer profile.
7. Suppliers: list, add/edit supplier, supplier profile.

**Requirements covered:** `FR-001–FR-016`

**What to design:**
- Empty, loading, success, validation-error and no-results states where relevant.
- Reusable table, search/filter bar, form fields, status labels, confirmation dialog and page layout.
- Product fields: code, name, category, unit, selling price, reference cost, quantity, reorder level and status.
- Customer/supplier forms with required name and optional contact details/notes.
- Customer/supplier profile should provide access to their account statement and transaction history (the detailed financial behavior is owned by Teammate 3).

**Handoff to the team:**
- Main navigation and shared layout.
- Reusable form and table components.
- Agreed visual styles, spacing, typography, colors and icon style.

---

### Teammate 2 — Sales & Purchases

**Own these screens:**
1. Sales invoices list with search/filter.
2. Create/edit sales invoice.
3. Sales invoice details and printable invoice view.
4. Purchase invoices list with search/filter.
5. Create/edit purchase invoice.
6. Purchase invoice details.
7. Cancel/reverse sale confirmation and result view.

**Requirements covered:** `FR-017–FR-032`

**What to design:**
- Invoice header: invoice number, date, customer/supplier and status.
- Invoice item rows: product, quantity, unit price/cost, line total.
- Discount, invoice total, amount paid and remaining amount.
- Payment states: fully paid, partially paid and unpaid/credit.
- Clear validation states: no valid item, invalid amount, insufficient stock, duplicate invoice number, etc.
- Sales and purchase detail pages must show the recorded items and payment information.
- Printable sales invoice must match the saved invoice information.

**Important business rules from the source:**
- Confirmed sales decrease stock; unconfirmed/draft sales do not.
- Confirmed purchases increase stock; unconfirmed/draft purchases do not.
- Only the amount actually received/paid affects the cashbox.
- The remaining amount is calculated automatically.
- Cancelling a sale must preserve the original record and show its reversal; do not design permanent deletion.

**Handoff to the team:**
- Invoice form and invoice-item components.
- Invoice status and payment status styles.
- Printable invoice layout.
- List of data/interaction requirements needed from products, customers and suppliers.

---

### Teammate 3 — Cashbox, Financial Transactions & Debts

**Own these screens:**
1. Cashbox overview/current balance.
2. Cash receipts and payments list.
3. Add receipt / add payment.
4. Customer collection (receive money from a customer).
5. Supplier payment.
6. Expenses: list, add/edit and details.
7. Other income: list, add/edit and details.
8. Cashbox statement for a selected date range.
9. Customer account statement.
10. Supplier account statement.
11. Debts overview: customer debts and supplier debts.
12. Debt details and partial/full collection or payment.

**Requirements covered:** `FR-033–FR-048`, plus the customer/supplier statement requirements in `FR-015–FR-016`.

**What to design:**
- Cashbox balance, opening balance, receipts, payments and closing balance.
- Transaction form fields: date, amount, description, type, reference and related party where applicable.
- Expense category, description, date, amount and payment method.
- Other income source, date, amount and collection method.
- Debt summary split into amounts owed by customers and amounts owed to suppliers.
- Party statement with date, description, debit/credit amount and running balance.
- Partial-payment flow that shows the remaining debt before confirmation.

**Important business rules from the source:**
- A financial movement must have an amount greater than zero.
- The cashbox changes only by the actual received or paid amount.
- A customer collection reduces that customer's outstanding balance.
- A supplier payment reduces the amount owed to that supplier.
- Overpayment is rejected by default unless an approved credit-balance policy exists.
- Cashbox payments must not exceed the available balance by default.

**Handoff to the team:**
- Shared money/amount input and transaction table components.
- Receipt/payment confirmation and validation states.
- Standard statement layout and balance-summary cards.

---

### Teammate 4 — Inventory, Dashboard, Reports, Settings & Backup

**Own these screens:**
1. Inventory overview / current stock.
2. Product stock movement history.
3. Stock adjustment (increase/decrease with reason).
4. Low-stock / reorder-level report.
5. Dashboard.
6. Reports home and report filters.
7. Sales report.
8. Purchase report.
9. Expenses and other-income report.
10. Customer balances report.
11. Supplier balances report.
12. Cashbox movement report.
13. Inventory report.
14. Store settings: store name, currency and general settings.
15. Invoice numbering and default reorder-level settings.
16. Backup creation, restore and operation result screens.

**Requirements covered:** `FR-049–FR-054`, `FR-059–FR-073`, and `NFR-05` where relevant.

**What to design:**
- Inventory quantities and movement history (incoming, outgoing and adjustments).
- Stock adjustment form with reason, before/after quantity and confirmation.
- Low-stock list showing current quantity and reorder level.
- Dashboard summary for sales, purchases, expenses, cashbox, debts and low stock.
- Report filters such as date range, party, product and category where applicable.
- Reports with totals, details and a print-friendly browser view.
- Settings forms with clear save/cancel states.
- Backup flow: choose location, create backup, show genuine success/failure result.
- Restore flow: choose backup file, validate it, show warning that current data will be replaced, ask for confirmation, then show result.

**Important business rules from the source:**
- Stock is based on confirmed stock movements.
- Stock cannot become negative by default.
- Every stock adjustment needs a reason and must preserve the movement record.
- Report totals must match the underlying detailed records.
- Restore must warn that current data will be replaced; never show success unless the operation succeeds.

**Handoff to the team:**
- Dashboard cards, chart/table styles and report filter components.
- Inventory movement and stock-adjustment components.
- Settings, backup and restore screen designs.

---

## 2. Shared design rules (everyone must follow)

1. **Language and direction:** Arabic interface, right-to-left (RTL).
2. **Product scope:** One store and one currency setting. Do not design multi-branch or multi-currency workflows.
3. **Consistency:** Use the shared navigation, typography, colors, spacing, buttons, inputs, tables, dialogs and status styles agreed by Teammate 1.
4. **Responsive behavior:** Design the agreed target desktop size first; show how tables/forms adapt to the agreed smaller screen if required.
5. **Every screen should include:** page title, clear primary action, useful empty state, validation/error state and success/confirmation state where applicable.
6. **Data integrity:** Do not use permanent deletion for confirmed financial or stock movements. Show cancellation/reversal while preserving the original record.
7. **Numbers:** Use consistent currency display, decimal precision and date formatting. The exact rounding policy is a decision to confirm; do not invent one.
8. **No invented features:** Keep to the requirements. If a needed rule is unclear, add it to the open-decisions list rather than making up behavior.

---

## 3. How we will work and merge

### Phase A — Agree before designing
- All four teammates read the requirements document.
- Teammate 1 proposes the common design system and application navigation.
- Agree on the target screen size, font, colors, spacing and component styles.
- Agree on shared terminology and Arabic labels.

### Phase B — Design separately
- Each teammate designs only their assigned screens.
- Use the same Figma page/component naming convention.
- Use realistic sample data, but keep it clearly fictional.
- Add notes for interactions, validation rules and unclear requirements.

### Phase C — Merge
1. Teammate 1 creates the main Figma file and shared component library.
2. Each teammate moves/copies their screens into their assigned page in that file.
3. Connect navigation and related flows between modules.
4. Replace duplicated components with shared components.
5. Review RTL, spacing, typography, button styles, tables, forms and status colors.
6. Run the end-to-end scenarios below and fix inconsistencies.

### Suggested Figma page names
- `00 — Design System`
- `01 — Foundation & Master Data`
- `02 — Sales & Purchases`
- `03 — Cashbox & Debts`
- `04 — Inventory, Dashboard & Reports`
- `05 — Prototype & User Flows`

---

## 4. Integration flows to check after merging

The design is not complete until these flows make sense across teammates' screens:

1. **Sell a product:** choose customer/products → create invoice → show total/paid/remaining → confirm → stock decreases → actual received amount appears in cashbox → customer debt updates if unpaid/partial.
2. **Purchase products:** choose supplier/products → enter quantities and costs → show total/paid/remaining → confirm → stock increases → actual payment appears in cashbox → supplier debt updates if unpaid/partial.
3. **Collect customer debt:** open customer/debt → enter collection amount → confirm → cashbox increases → customer balance decreases.
4. **Pay supplier debt:** open supplier/debt → enter payment amount → confirm → cashbox decreases → supplier balance decreases.
5. **Adjust stock:** choose product → enter increase/decrease and reason → confirm → quantity and movement history update.
6. **Review reports:** select date range/filter → report totals and details agree with the relevant invoice, cashbox, debt and stock records.
7. **Backup and restore:** create backup → show result → select backup to restore → warn that current data will be replaced → confirm → show result.

---

## 5. Open decisions — do not guess these

The requirements document identifies decisions that must be settled before finalizing the relevant design/implementation:

- How will the system run: offline/local or on a remote server?
- Which devices and browsers are the target?
- What is the approved currency?
- What is the discount policy?
- What rounding/decimal precision rule will be used?
- What is the starting invoice number and default reorder level?
- Will more than one user account be enabled, and what roles/permissions are needed?
- Will returns be included in Release 1? `FR-055–FR-058` are explicitly optional and require approval.
- Is an overpayment/credit-balance policy allowed? Default behavior is to reject overpayment.
- Is negative stock ever allowed? Default behavior is to prevent it.

Keep these in a shared `Open Decisions` section in Figma so the team and project owner can resolve them together.

---

## 6. Definition of done for each teammate

Before handing over your work, make sure:

- [ ] All assigned screens are designed.
- [ ] Screens follow the shared RTL design system.
- [ ] Main, empty, loading, error and success states are covered where relevant.
- [ ] Forms show required fields and validation messages.
- [ ] Tables include useful search/filter and empty states where required.
- [ ] Main actions and navigation are clear.
- [ ] Related screens connect into a sensible user flow.
- [ ] Any unclear requirement is documented instead of guessed.
- [ ] Components are named and reusable.
- [ ] The teammate has added a short handoff note explaining the screens and interactions.

**Final target:** four independently designed modules that merge into one consistent, connected Arabic accounting-system interface.
