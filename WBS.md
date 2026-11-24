# Work Breakdown Structure (WBS)

## Mini Accounting System --- Release 1

**Document type:** Project scope and deliverable-oriented WBS blueprint\
**Source baseline:** *Mini Accounting System* requirements document,
version 2.0 (4 November 2026), UI/UX team task split, and stakeholder
workflows\
**Product scope:** Arabic, right-to-left (RTL), single-store accounting
and store-management system\
**Status:** Planning baseline for team review; open business and
technical decisions remain subject to owner approval

------------------------------------------------------------------------

## 1. WBS purpose and rules

This WBS decomposes the Release 1 project into manageable deliverables
and work packages. It is intended to support planning, ownership
assignment, estimation, design, implementation, integration, testing,
and acceptance.

-   The WBS is **deliverable-oriented**, not a day-by-day schedule.
-   WBS identifiers are stable references for planning and progress
    tracking.
-   Functional requirement identifiers (`FR-001`--`FR-073`) are retained
    as traceability references.
-   UI/UX work is included because the supplied team task split defines
    a design phase and a Figma merge.
-   Business rules must not be changed or inferred silently. Unresolved
    items are recorded in Section 8.
-   Sales and purchase returns (`FR-055`--`FR-058`) are optional and are
    not part of the committed Release 1 baseline unless approved.
-   This WBS does not prescribe a technology stack, hosting model, or
    local accounting/tax compliance rules.

## 2. Product scope baseline

### 2.1 Included in Release 1

1.  Authentication, user management, and permissions when multiple
    accounts are enabled.
2.  Products, categories, customers, and suppliers.
3.  Sales invoices, including cash, credit, and partial payment.
4.  Purchase invoices, including cash, credit, and partial payment.
5.  Cashbox, receipts, payments, expenses, and other income.
6.  Customer and supplier balances, collections, payments, and
    statements.
7.  Inventory balances, automatic stock movements, stock adjustments,
    and low-stock reporting.
8.  Dashboard and operational/financial reports.
9.  Store settings, invoice numbering, backup creation, and restore.
10. Arabic RTL interface, browser-based printable views, validation, and
    access protection.

### 2.2 Explicitly outside the current scope

-   Multiple stores or branches in one database.
-   Multiple currencies within one transaction; the store uses one
    configured currency.
-   Payroll.
-   Electronic tax integrations.
-   Bank integrations.
-   Specialized point-of-sale integrations.
-   E-commerce.
-   Offline/local versus remote-server operating model until decided.
-   Returns unless the owner approves their inclusion in Release 1.

## 3. WBS hierarchy

### 1.0 Mini Accounting System --- Release 1

#### 1.1 Project management and requirements baseline

-   **1.1.1 Project initiation and governance**
    -   Confirm project objective, stakeholders, decision owner, and
        approval path.
    -   Confirm Release 1 scope and WBS baseline.
    -   Establish team communication, review cadence, and issue/change
        log.
-   **1.1.2 Requirements review and traceability**
    -   Review the requirements document and its functional requirements
        `FR-001`--`FR-073`.
    -   Review non-functional requirements and shared business rules.
    -   Map each requirement to design, implementation, and test work
        packages.
    -   Record assumptions, ambiguities, and dependencies without
        inventing rules.
-   **1.1.3 Open-decision management**
    -   Maintain the decision log in Section 8.
    -   Obtain owner approval for decisions that affect workflows or
        architecture.
    -   Update the baseline and affected work packages after approval.
-   **1.1.4 Delivery planning and reporting**
    -   Prepare estimates, milestones, dependencies, and resource
        assignments.
    -   Track scope, risks, issues, and change requests.
    -   Prepare review and acceptance records.

#### 1.2 UX/UI design and design-system deliverables

-   **1.2.1 Design foundations**
    -   Agree target desktop viewport and responsive expectations.
    -   Define Arabic RTL grid, typography, spacing, color, icons, and
        component conventions.
    -   Define navigation, header, page-title, user-menu, and
        notification patterns.
    -   Create shared buttons, inputs, tables, dialogs, status labels,
        and form patterns.
-   **1.2.2 Foundation and master-data screens --- Teammate 1**
    -   Login and protected application shell.
    -   User list, create/edit user, roles, and permissions.
    -   Product list, search/filter, create/edit, and product details.
    -   Category list and create/edit category.
    -   Customer and supplier lists, forms, profiles, and entry points
        to statements.
    -   Cover `FR-001`--`FR-016`.
-   **1.2.3 Sales and purchase screens --- Teammate 2**
    -   Sales invoice list, search/filter, create/edit, details, and
        printable view.
    -   Purchase invoice list, search/filter, create/edit, and details.
    -   Invoice line-item, discount, totals, payment, and
        remaining-balance patterns.
    -   Sale cancellation/reversal confirmation and result.
    -   Cover `FR-017`--`FR-032`.
-   **1.2.4 Cashbox and debt screens --- Teammate 3**
    -   Cashbox overview and transaction list.
    -   Receipt/payment forms and customer collection/supplier payment
        flows.
    -   Expense and other-income screens.
    -   Cashbox statement, customer/supplier statements, debt overview,
        and debt details.
    -   Cover `FR-033`--`FR-048`, with statement access also covering
        `FR-015`--`FR-016`.
-   **1.2.5 Inventory, dashboard, reports, settings, and backup screens
    --- Teammate 4**
    -   Inventory overview, product movement history, stock adjustment,
        and low-stock report.
    -   Dashboard and reports home/filter patterns.
    -   Sales, purchase, expense/income, customer-balance,
        supplier-balance, cashbox, and inventory reports.
    -   Store settings, invoice numbering, and backup/restore flows.
    -   Cover `FR-049`--`FR-054` and `FR-059`--`FR-073`; address
        `NFR-05` where relevant.
-   **1.2.6 Design integration and prototype**
    -   Merge teammate screens into one Figma file.
    -   Replace duplicated patterns with shared components.
    -   Connect navigation and cross-module flows.
    -   Review RTL, responsive behavior, terminology, and visual
        consistency.
    -   Prototype the end-to-end scenarios in Section 7.
    -   Record unresolved requirements in a shared Open Decisions page.
-   **1.2.7 Design handoff**
    -   Deliver approved screen designs, component library, interaction
        notes, and states.
    -   Deliver screen-to-requirement traceability.
    -   Obtain design review and stakeholder sign-off.

#### 1.3 Application foundation and shared services

-   **1.3.1 Technical solution and environment**
    -   Confirm the operating model and deployment target after owner
        decision.
    -   Select and document the approved architecture and technology
        stack.
    -   Establish development, test, and production environments as
        applicable.
    -   Configure source control, build process, and environment
        configuration.
-   **1.3.2 Application shell and navigation**
    -   Implement Arabic RTL application shell.
    -   Implement sidebar/navigation, header, page title, and user menu.
    -   Implement shared notifications and page layout.
-   **1.3.3 Authentication and session management**
    -   Implement login and credential validation (`FR-001`).
    -   Protect restricted pages and operations (`FR-002`).
    -   Implement logout and session invalidation (`FR-005`).
-   **1.3.4 User and permission management**
    -   Create, update, and deactivate user accounts (`FR-003`).
    -   Prevent duplicate login identifiers and retain change history.
    -   Implement roles/permissions when multi-user operation is enabled
        (`FR-004`).
    -   Enforce authorization at the operation/service level, not only
        by hiding UI controls.
-   **1.3.5 Shared validation and audit information**
    -   Validate required fields, positive monetary movements, and
        non-negative quantities where applicable.
    -   Record execution time and acting user for financial and
        inventory operations.
    -   Preserve source records and reversal/adjustment history.
    -   Provide consistent success, validation-error, failure, empty,
        and loading states.
-   **1.3.6 Shared presentation and printing**
    -   Implement reusable tables, search/filter controls, forms,
        dialogs, and status indicators.
    -   Implement browser-based printable layouts for invoices and
        reports.
    -   Apply consistent currency and date presentation according to
        approved settings.

#### 1.4 Master data management

-   **1.4.1 Product management**
    -   Create, view, edit, search, and filter products (`FR-006`).
    -   Store product code, name, category, unit, selling price,
        reference cost, quantity, reorder level, and status (`FR-007`).
    -   Enforce unique product code (`FR-008`).
    -   Prevent final deletion of products with linked transactions;
        allow deactivation (`FR-011`).
-   **1.4.2 Category management**
    -   Create, view, and edit categories (`FR-009`).
    -   Prevent deletion of categories in use until products are moved
        or the category is deactivated.
-   **1.4.3 Low-stock threshold behavior**
    -   Compare current quantity with each product's reorder level.
    -   Display low-stock alerts when quantity is at or below the
        threshold (`FR-010`).
-   **1.4.4 Customer management**
    -   Create, view, edit, and search customers (`FR-012`).
    -   Store required name and optional contact details/notes
        (`FR-014`).
-   **1.4.5 Supplier management**
    -   Create, view, edit, and search suppliers (`FR-013`).
    -   Store required name and optional contact details/notes
        (`FR-014`).
-   **1.4.6 Party balances and statements**
    -   Derive customer and supplier balances from approved linked
        transactions (`FR-015`).
    -   Display party transaction history and statement by selected
        period (`FR-016`).
    -   Ensure statement closing balance agrees with the party balance
        for the same period.

#### 1.5 Sales management

-   **1.5.1 Sales invoice creation**
    -   Select customer or general sale.
    -   Add one or more valid product lines and quantities.
    -   Capture invoice number, date, prices, discounts, and totals
        (`FR-017`--`FR-018`).
    -   Enforce unique invoice number and at least one valid line.
-   **1.5.2 Sales payment handling**
    -   Support fully paid, unpaid/credit, and partially paid invoices
        (`FR-019`).
    -   Calculate remaining amount from approved payments (`FR-020`).
    -   Apply the approved payment method and discount rules.
-   **1.5.3 Sales confirmation and stock effect**
    -   Validate available quantity before confirmation.
    -   Decrease inventory only when the sale is confirmed (`FR-021`).
    -   Prevent negative stock by default unless an explicit policy is
        approved (`FR-053`).
-   **1.5.4 Sales cashbox and customer-balance effect**
    -   Record only the amount actually received in the cashbox
        (`FR-022`).
    -   Create/update customer receivable for the unpaid remainder
        (`FR-042`).
    -   Link receipts, invoice, customer, and cashbox movement.
-   **1.5.5 Sales invoice viewing and printing**
    -   Display saved invoice data and payment status.
    -   Print a browser page matching the saved invoice (`FR-023`).
-   **1.5.6 Sales invoice search and details**
    -   Search by invoice number, date, or customer and open details
        (`FR-024`).
-   **1.5.7 Sales cancellation/reversal**
    -   Require appropriate permission and cancellation reason.
    -   Preserve original invoice and create linked reversal entries.
    -   Reverse stock, financial, and customer-balance effects according
        to amounts actually recorded (`FR-025`).
    -   Prevent duplicate cancellation.

#### 1.6 Purchase management

-   **1.6.1 Purchase invoice creation**
    -   Select supplier and add products, quantities, and unit costs
        (`FR-026`--`FR-027`).
    -   Validate invoice lines and totals using the approved rounding
        policy.
-   **1.6.2 Purchase payment handling**
    -   Support fully paid, unpaid/credit, and partially paid invoices
        (`FR-028`).
    -   Calculate supplier remaining balance (`FR-031`).
-   **1.6.3 Purchase confirmation and stock effect**
    -   Increase inventory only when the purchase is confirmed
        (`FR-029`).
    -   Link stock receipt movements to the source purchase invoice.
-   **1.6.4 Purchase cashbox and supplier-balance effect**
    -   Record only the amount actually paid in the cashbox (`FR-030`).
    -   Create/update supplier payable for the unpaid remainder
        (`FR-043`).
    -   Link payment, invoice, supplier, and cashbox movement.
-   **1.6.5 Purchase invoice search and details**
    -   Search by invoice number, supplier, or date and display details
        (`FR-032`).

#### 1.7 Cashbox and financial transactions

-   **1.7.1 Cashbox balance**
    -   Calculate balance from opening balance and approved movements
        (`FR-033`).
    -   Keep the displayed balance consistent with the transaction
        ledger.
-   **1.7.2 Receipts and payments**
    -   Record date, amount, description, type, and reference
        (`FR-034`).
    -   Validate amount is greater than zero (`FR-040`).
-   **1.7.3 Customer collections**
    -   Record collection from a customer and allocate it to
        invoice/debt where applicable (`FR-035`, `FR-044`).
    -   Increase cashbox by actual receipt and reduce customer
        receivable by allocated amount.
-   **1.7.4 Supplier payments**
    -   Record payment to supplier and allocate it to invoice/debt where
        applicable (`FR-036`, `FR-045`).
    -   Decrease cashbox by actual payment and reduce supplier payable.
-   **1.7.5 Expense management**
    -   Create and classify expenses with date, amount, description,
        category, and payment method (`FR-037`).
    -   Link cash payment to the corresponding cashbox movement.
-   **1.7.6 Other income**
    -   Record income not generated by sales, including source, date,
        amount, description, and collection method (`FR-038`).
    -   Record cashbox receipt when the income is actually received.
-   **1.7.7 Cashbox statement**
    -   Display opening balance, receipts, payments, and closing balance
        for a selected period (`FR-039`).
    -   Apply the approved balance equation and show movement details.
-   **1.7.8 Cashbox control**
    -   Reject payments exceeding available cashbox balance by default
        (`FR-041`).
    -   Allow exceptions only under an explicitly approved policy and
        permission.

#### 1.8 Debt and account management

-   **1.8.1 Customer receivables**
    -   Create customer debt for credit/partial sales (`FR-042`).
    -   Track amount due by invoice and customer.
-   **1.8.2 Supplier payables**
    -   Create supplier debt for credit/partial purchases (`FR-043`).
    -   Track amount due by invoice and supplier.
-   **1.8.3 Debt collection and settlement**
    -   Support partial and full customer collections (`FR-044`).
    -   Support partial and full supplier payments (`FR-045`).
    -   Reject over-allocation by default unless an approved
        credit-balance policy exists (`FR-048`).
-   **1.8.4 Debt summaries**
    -   Calculate total customer receivables and supplier payables
        (`FR-046`).
    -   Ensure totals equal the sum of individual party balances.
-   **1.8.5 Debt source tracing**
    -   Show the transactions that formed each party balance (`FR-047`).
    -   Link balances to invoices, receipts, payments, and adjustments.

#### 1.9 Inventory management

-   **1.9.1 Current stock**
    -   Display current quantity for each product from approved stock
        movements (`FR-049`).
-   **1.9.2 Automatic stock movement posting**
    -   Post sale issue and purchase receipt movements on confirmation
        (`FR-050`).
    -   Ensure each source operation affects stock exactly once.
    -   Preserve source references and movement history.
-   **1.9.3 Stock adjustment**
    -   Support quantity increase/decrease with date and mandatory
        reason (`FR-051`).
    -   Store before/after quantities, acting user, and timestamp.
    -   Prevent negative stock by default.
-   **1.9.4 Low-stock report**
    -   List products at or below reorder level with current quantity
        and threshold (`FR-052`).
-   **1.9.5 Negative-stock prevention**
    -   Validate available stock before sale or adjustment (`FR-053`).
    -   Permit negative stock only if an explicit manager-approved
        policy is adopted.
-   **1.9.6 Product movement history**
    -   Display incoming, outgoing, and adjustment movements by product
        and period (`FR-054`).
    -   Reconcile closing quantity to the current stock balance.

#### 1.10 Dashboard and reporting

-   **1.10.1 Dashboard**
    -   Summarize sales, purchases, expenses, cashbox, debts, and low
        stock (`FR-059`).
    -   Ensure dashboard values match detailed reports for the same
        period.
-   **1.10.2 Sales report**
    -   Display approved sales by selected period and available filters
        (`FR-060`).
    -   Exclude drafts and cancelled transactions from net sales as
        specified.
-   **1.10.3 Purchase report**
    -   Display approved purchases by period and applicable
        supplier/product filters (`FR-061`).
-   **1.10.4 Expenses and other-income report**
    -   Display movements by type, period, category, and source
        (`FR-062`).
-   **1.10.5 Customer balances report**
    -   Display customer balances for a date or period and reconcile to
        party balances (`FR-063`).
-   **1.10.6 Supplier balances report**
    -   Display supplier balances and reconcile to party balances
        (`FR-064`).
-   **1.10.7 Cashbox movement report**
    -   Display receipts, payments, and cashbox balance for a selected
        period (`FR-065`).
-   **1.10.8 Inventory report**
    -   Display product stock status and movement information
        (`FR-066`).
-   **1.10.9 Report filtering**
    -   Apply supported date, party, product, and category filters
        (`FR-067`).
-   **1.10.10 Report viewing and printing**
    -   Provide browser-based report views and print output (`FR-068`).
    -   Show selected period and configured currency.
-   **1.10.11 Report reconciliation**
    -   Validate report totals against source invoices, cashbox, debt,
        and stock records.
    -   Verify draft/cancelled records are treated according to the
        relevant rules.

#### 1.11 Store settings, backup, and restore

-   **1.11.1 Store settings**
    -   Configure store name and single store currency.
    -   Configure general settings supported by the approved
        requirements (`FR-069`--`FR-073` as applicable).
-   **1.11.2 Invoice numbering**
    -   Configure invoice numbering/start value according to the
        approved policy.
    -   Prevent duplicate invoice numbers across applicable invoice
        types.
-   **1.11.3 Default reorder-level settings**
    -   Configure default reorder level if approved and supported by the
        settings requirements.
-   **1.11.4 Backup creation**
    -   Select destination and create a backup file.
    -   Check destination availability and write access.
    -   Display actual success/failure and file location; never report
        success before completion.
-   **1.11.5 Backup validation and restore**
    -   Select and validate a backup file.
    -   Check file compatibility/integrity before restore.
    -   Warn that current data will be replaced and require explicit
        confirmation.
    -   Execute restore and display actual result.
    -   Verify restored data after completion.
-   **1.11.6 Settings and backup acceptance**
    -   Verify saved settings appear in related invoices and reports.
    -   Verify backup and restore through a controlled test scenario.

#### 1.12 Optional returns package --- requires owner approval

> This package is excluded from the committed Release 1 baseline until
> explicitly approved.

-   **1.12.1 Sales return registration** (`FR-055`)
    -   Link return to original sales invoice and lines.
    -   Validate returned quantity against sold quantity less previous
        returns.
    -   Capture return reason.
-   **1.12.2 Returned stock disposition** (`FR-056`)
    -   Determine whether returned goods are fit for resale.
    -   Return only accepted quantities to sellable stock.
-   **1.12.3 Sales return financial effect** (`FR-057`)
    -   Apply approved refund or customer-credit treatment.
    -   Ensure financial effect matches the return value.
-   **1.12.4 Purchase return registration** (`FR-058`)
    -   Link return to original purchase and validate unreturned
        received quantity.
    -   Update stock and supplier balance according to approved
        movement.
-   **1.12.5 Returns integration and testing**
    -   Test inventory, cashbox, customer/supplier balance, reports, and
        reversal interactions.

#### 1.13 Quality assurance and testing

-   **1.13.1 Test strategy and traceability**
    -   Prepare test plan and requirement-to-test matrix for
        `FR-001`--`FR-073`.
    -   Define test data, roles, environments, and entry/exit criteria.
-   **1.13.2 Functional testing**
    -   Test authentication, users, permissions, and master data.
    -   Test sales and purchase workflows.
    -   Test cashbox, collections, supplier payments, expenses, and
        other income.
    -   Test debt balances and statements.
    -   Test inventory, stock adjustments, and low-stock behavior.
    -   Test dashboard, reports, settings, backup, and restore.
-   **1.13.3 Business-rule and calculation testing**
    -   Verify invoice totals, discounts, paid amounts, and remaining
        balances.
    -   Verify only actual receipts/payments affect cashbox.
    -   Verify approved sales/purchases affect stock exactly once.
    -   Verify customer/supplier balances reconcile to detailed
        movements.
    -   Verify zero/negative financial movements are rejected.
    -   Verify overpayment, cashbox-limit, and negative-stock defaults.
-   **1.13.4 Authorization and data-integrity testing**
    -   Verify protected pages and unauthorized operations are blocked.
    -   Verify confirmed transactions are not permanently deleted.
    -   Verify cancellation/reversal retains original records and
        reasons.
    -   Verify duplicate submissions do not duplicate financial or stock
        effects.
-   **1.13.5 UI and usability testing**
    -   Verify Arabic labels, RTL layout, navigation, forms, tables, and
        print views.
    -   Verify empty, loading, success, validation-error, and no-results
        states where relevant.
    -   Verify agreed responsive behavior and supported
        browsers/devices.
-   **1.13.6 Backup and recovery testing**
    -   Test backup success and failure scenarios.
    -   Test invalid/incompatible backup rejection.
    -   Test restore warning, confirmation, completion, and post-restore
        verification.
-   **1.13.7 Defect management and regression**
    -   Log, prioritize, fix, and retest defects.
    -   Run regression tests on affected modules and integrated
        workflows.
    -   Produce test summary and release recommendation.

#### 1.14 Integration and end-to-end validation

-   **1.14.1 Sell a product**
    -   Select customer/products, create invoice, calculate
        total/paid/remaining, and confirm.
    -   Verify stock decreases, actual receipt enters cashbox, and
        customer debt updates if applicable.
-   **1.14.2 Purchase products**
    -   Select supplier/products, record quantities/costs/payment, and
        confirm.
    -   Verify stock increases, actual payment leaves cashbox, and
        supplier debt updates if applicable.
-   **1.14.3 Collect customer debt**
    -   Select customer/debt, record partial/full collection, and
        confirm.
    -   Verify cashbox increases and customer balance decreases.
-   **1.14.4 Pay supplier debt**
    -   Select supplier/debt, record payment, and confirm.
    -   Verify cashbox decreases and supplier payable decreases.
-   **1.14.5 Adjust stock**
    -   Select product, enter quantity change and reason, and confirm.
    -   Verify quantity and movement history update.
-   **1.14.6 Review reports**
    -   Select period and filters.
    -   Verify report totals and details agree with invoices, cashbox,
        debt, and inventory records.
-   **1.14.7 Backup and restore**
    -   Create backup, verify result, select backup, review replacement
        warning, confirm restore, and verify data.
-   **1.14.8 Cross-module consistency**
    -   Verify invoice, stock, cashbox, party balance, and report
        effects remain linked and consistent.

#### 1.15 Release, deployment, and handover

-   **1.15.1 Release readiness**
    -   Confirm approved scope, resolved critical decisions, test
        completion, and defect status.
    -   Obtain product-owner acceptance.
-   **1.15.2 Deployment and configuration**
    -   Deploy using the approved operating model.
    -   Apply store settings and initial configuration.
    -   Configure initial user accounts and permissions as approved.
-   **1.15.3 Operational handover**
    -   Deliver user guidance for owner, system administrator,
        accountant, cashier, sales, and inventory/purchasing users.
    -   Deliver administrator guidance for settings, backup, and
        restore.
    -   Deliver technical handover notes and known limitations.
-   **1.15.4 Final acceptance and closure**
    -   Obtain sign-off against the acceptance checklist.
    -   Archive source baseline, designs, test evidence, and release
        notes.
    -   Record deferred scope and post-release issues.

## 4. Work package ownership

  -----------------------------------------------------------------------
  Workstream        Primary owner     Supporting roles  Main WBS
                                                        references
  ----------------- ----------------- ----------------- -----------------
  Project           Project lead /    All teammates     1.1
  coordination and  product owner                       
  requirements                                          

  UX foundations    Teammate 1        Product owner,    1.2.1--1.2.2, 1.4
  and master data                     developers        

  Sales and         Teammate 2        Teammates 1 and   1.2.3, 1.5--1.6
  purchases                           3, developers     

  Cashbox and debts Teammate 3        Teammates 1 and   1.2.4, 1.7--1.8
                                      2, developers     

  Inventory,        Teammate 4        Teammates 1 and   1.2.5, 1.9--1.11
  dashboard,                          3, developers     
  reports,                                              
  settings, backup                                      

  Design            Teammate 1        All four          1.2.6--1.2.7
  integration       coordinates       teammates         

  Application       Development team  Product owner,    1.3--1.11
  implementation                      designers         

  QA and end-to-end QA/test owner     Developers and    1.13--1.14
  validation                          all teammates     

  Release and       Project lead /    Development and   1.15
  handover          product owner     QA                
  -----------------------------------------------------------------------

**Ownership note:** The four-teammate split in the source file is
explicitly a UI/UX design allocation. The implementation and QA
ownership above is a planning allocation and should be confirmed by the
project lead.

## 5. Key dependencies

1.  Operating model, target devices/browsers, and architecture decisions
    must precede final technical design and environment setup.
2.  The shared RTL design system and application shell should be agreed
    before parallel screen design is finalized.
3.  Products, categories, customers, and suppliers are prerequisites for
    invoice workflows.
4.  Authentication and authorization are prerequisites for protected
    modules and permission-sensitive actions.
5.  Sales and purchase confirmation depend on product and inventory
    validation.
6.  Cashbox and debt workflows depend on invoice/payment relationships
    and consistent transaction references.
7.  Reports and dashboard depend on reliable, integrated source records
    from sales, purchases, cashbox, debts, and inventory.
8.  Backup/restore depends on the approved data storage and deployment
    approach.
9.  End-to-end testing depends on the integrated modules and
    representative test data.
10. Release acceptance depends on critical business decisions being
    approved and the acceptance criteria being met.

## 6. Cross-cutting quality requirements

Apply these controls across all relevant work packages:

-   **Language and layout:** Arabic interface and right-to-left
    direction.
-   **Single-store scope:** No multi-branch or multi-currency
    transaction workflows.
-   **Data integrity:** Approved financial and stock movements remain
    traceable; do not permanently delete confirmed movements.
-   **Auditability:** Store user and execution time for financial and
    inventory operations.
-   **Calculation integrity:** Invoice totals, payments, remaining
    balances, cashbox balance, party balances, and stock quantities
    reconcile to source movements.
-   **Validation:** Reject invalid amounts and invalid quantities
    according to the approved rules.
-   **Authorization:** Enforce permissions for protected operations.
-   **Usability:** Provide relevant empty, loading, success,
    validation-error, and no-results states.
-   **Printing:** Printed invoice/report output matches saved/displayed
    data.
-   **Recovery:** Backup and restore outcomes must reflect the actual
    operation result.
-   **Localization:** Currency, decimal precision, and date formatting
    follow approved settings; do not assume unspecified formats or
    rounding rules.

## 7. End-to-end acceptance scenarios

  -------------------------------------------------------------------------
  ID                      Scenario                Expected integrated
                                                  result
  ----------------------- ----------------------- -------------------------
  E2E-01                  Sell a product          Confirmed invoice
                                                  decreases stock; only
                                                  amount received increases
                                                  cashbox; unpaid amount
                                                  updates customer debt;
                                                  invoice/report data
                                                  agree.

  E2E-02                  Purchase products       Confirmed invoice
                                                  increases stock; only
                                                  amount paid decreases
                                                  cashbox; unpaid amount
                                                  updates supplier debt.

  E2E-03                  Collect customer debt   Cashbox increases by
                                                  collection; customer
                                                  receivable decreases by
                                                  allocated amount;
                                                  statement reflects the
                                                  movement.

  E2E-04                  Pay supplier debt       Cashbox decreases by
                                                  payment; supplier payable
                                                  decreases; statement
                                                  reflects the movement.

  E2E-05                  Adjust stock            A reason is mandatory;
                                                  before/after quantity,
                                                  user, time, and movement
                                                  are retained; negative
                                                  stock is blocked by
                                                  default.

  E2E-06                  Review reports          Totals and details
                                                  reconcile to invoices,
                                                  cashbox movements, party
                                                  balances, and inventory
                                                  movements for the same
                                                  filters.

  E2E-07                  Backup and restore      Backup result is genuine;
                                                  restore warns about
                                                  replacing current data,
                                                  requires confirmation,
                                                  and is followed by
                                                  verification.

  E2E-08                  Cancel a sale           Original sale remains;
                                                  authorized reversal is
                                                  recorded once with reason
                                                  and corresponding
                                                  stock/financial/balance
                                                  effects.

  E2E-09                  Permission enforcement  A user cannot access
                                                  protected pages or
                                                  perform actions not
                                                  granted to that account.
  -------------------------------------------------------------------------

## 8. Open decisions and approval log

These items are identified by the supplied requirements/team material
and must not be guessed by the project team.

  --------------------------------------------------------------------------------------
  ID                Decision required      Impacted work packages      Current baseline
                                                                       / action
  ----------------- ---------------------- --------------------------- -----------------
  OD-01             Offline/local          1.3, 1.11, 1.15             Unresolved;
                    operation or remote                                decide before
                    server?                                            architecture and
                                                                       deployment
                                                                       design.

  OD-02             Target devices and     1.2, 1.3, 1.13              Unresolved; agree
                    browsers?                                          desktop target
                                                                       and required
                                                                       responsive
                                                                       coverage.

  OD-03             Approved store         1.2, 1.3, 1.7, 1.10, 1.11   One currency per
                    currency?                                          store; exact
                                                                       currency requires
                                                                       approval.

  OD-04             Discount policy?       1.2.3, 1.5, 1.6, 1.13       Unresolved; do
                                                                       not invent
                                                                       discount limits
                                                                       or authorization.

  OD-05             Rounding and decimal   1.5, 1.6, 1.7, 1.10         Unresolved;
                    precision?                                         affects totals
                                                                       and
                                                                       reconciliation.

  OD-06             Starting invoice       1.5, 1.6, 1.11              Unresolved;
                    number and numbering                               define uniqueness
                    policy?                                            and sequence
                                                                       behavior.

  OD-07             More than one user     1.3, all protected modules  Multi-user
                    account? Which                                     capability is
                    roles/permissions?                                 conditional;
                                                                       detailed roles
                                                                       require approval.

  OD-08             Include sales/purchase 1.12 and affected           Optional; exclude
                    returns in Release 1?  stock/cashbox/debt/report   from baseline
                                           work                        until owner
                                                                       approval.

  OD-09             Allow                  1.7, 1.8                    Default is to
                    overpayment/customer                               reject
                    or supplier credit                                 overpayment;
                    balance?                                           exception
                                                                       requires explicit
                                                                       policy.

  OD-10             Allow negative stock?  1.5, 1.9                    Default is to
                                                                       prevent negative
                                                                       stock; exception
                                                                       requires explicit
                                                                       policy.

  OD-11             Default reorder level  1.4, 1.9, 1.11              Confirm
                    and its configuration                              initial/default
                    behavior?                                          value and whether
                                                                       a global default
                                                                       is required.

  OD-12             Detailed permissions   1.3 and all role-sensitive  Stakeholder
                    for owner, accountant, flows                       workflows
                    cashier, sales, and                                identify needs,
                    inventory users?                                   but not all
                                                                       detailed
                                                                       permissions;
                                                                       approve a
                                                                       permission
                                                                       matrix.

  OD-13             Local accounting/tax   Potentially all financial   Requirements
                    rules and compliance   modules                     document does not
                    requirements?                                      constitute
                                                                       approval of local
                                                                       accounting or tax
                                                                       rules; obtain
                                                                       separate guidance
                                                                       if required.
  --------------------------------------------------------------------------------------

## 9. WBS dictionary --- work package completion criteria

A work package is complete when its deliverable is produced, reviewed,
traceable to its requirements, and its relevant acceptance checks pass.

  -----------------------------------------------------------------------
  Work package group                  Minimum completion evidence
  ----------------------------------- -----------------------------------
  1.1 Project and requirements        Approved scope baseline,
                                      traceability matrix, decision log,
                                      and delivery plan.

  1.2 UX/UI                           Merged Figma file, shared design
                                      system, screen states, connected
                                      flows, and design sign-off.

  1.3 Application foundation          Working shell,
                                      authentication/session controls,
                                      permission enforcement, shared
                                      validation, and common UI services.

  1.4 Master data                     CRUD/search flows, field
                                      validation, uniqueness/deletion
                                      safeguards, party balances, and
                                      statements.

  1.5 Sales                           Valid sales lifecycle, payment
                                      states, stock/cashbox/debt effects,
                                      printable invoice, search, and
                                      reversal.

  1.6 Purchases                       Valid purchase lifecycle, payment
                                      states,
                                      stock/cashbox/supplier-balance
                                      effects, and search/details.

  1.7 Cashbox                         Accurate opening/current/closing
                                      balances,
                                      receipt/payment/expense/income
                                      movements, and cash controls.

  1.8 Debts                           Accurate receivables/payables,
                                      partial/full settlement, summaries,
                                      and source tracing.

  1.9 Inventory                       Accurate current stock, linked
                                      movements, reasoned adjustments,
                                      low-stock report, and
                                      negative-stock control.

  1.10 Dashboard/reports              Required summaries and reports,
                                      filters, print views, and
                                      reconciliation evidence.

  1.11 Settings/backup                Saved settings, numbering behavior,
                                      valid backup creation, safe
                                      restore, and post-restore
                                      verification.

  1.12 Optional returns               Only if approved: complete return
                                      workflow with validated stock and
                                      financial effects.

  1.13 QA                             Requirement-based test results,
                                      business-rule tests,
                                      security/data-integrity checks,
                                      regression, and defect summary.

  1.14 Integration                    Successful execution of all
                                      applicable E2E scenarios and
                                      cross-module reconciliation.

  1.15 Release/handover               Owner acceptance, deployment
                                      record, user/admin guidance,
                                      technical handover, and release
                                      notes.
  -----------------------------------------------------------------------

## 10. Requirement traceability summary

  -----------------------------------------------------------------------
  Requirement range                   WBS coverage
  ----------------------------------- -----------------------------------
  `FR-001`--`FR-005`                  1.2.2, 1.3.3--1.3.4

  `FR-006`--`FR-011`                  1.2.2, 1.4.1--1.4.3

  `FR-012`--`FR-016`                  1.2.2, 1.4.4--1.4.6

  `FR-017`--`FR-025`                  1.2.3, 1.5

  `FR-026`--`FR-032`                  1.2.3, 1.6

  `FR-033`--`FR-041`                  1.2.4, 1.7

  `FR-042`--`FR-048`                  1.2.4, 1.8

  `FR-049`--`FR-054`                  1.2.5, 1.9

  `FR-055`--`FR-058`                  1.12 --- optional, approval
                                      required

  `FR-059`--`FR-068`                  1.2.5, 1.10

  `FR-069`--`FR-073`                  1.2.5, 1.11; confirm detailed
                                      requirement wording during
                                      implementation planning

  Non-functional requirements         1.3, 1.13, and Section 6; map each
                                      NFR individually in the test
                                      traceability matrix
  -----------------------------------------------------------------------

------------------------------------------------------------------------

## 11. Baseline note

This WBS is a project-planning decomposition based on the supplied
project files. It does not add new product features or settle unresolved
business rules. Before estimating or committing delivery dates, the
project lead should confirm the open decisions, validate the detailed
wording of all non-functional requirements and `FR-069`--`FR-073`,
assign named implementation/QA owners, and approve the WBS baseline.
