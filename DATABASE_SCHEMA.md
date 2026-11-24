سأعيد بناء DATABASE_SCHEMA.md من الصفر، اعتماداً على وثيقة متطلبات نظام المحاسبة المصغر وملفات سير العمل ومهام الفريق، وليس على أي تصميم سابق.

سأفصل بين ما نصت عليه المتطلبات والقرارات التي ما زالت تحتاج اعتماد مالك النظام؛ حتى لا تتحول الافتراضات التقنية إلى قواعد عمل معتمدة. وسيغطي التصميم الكيانات والحقول والعلاقات والقيود والفهارس وسياسة الملكية، مع مراعاة أن النظام لمتجر واحد وعملة واحدة، وأن الحركات المالية والمخزنية المعتمدة لا تُحذف نهائياً.


# DATABASE_SCHEMA.md
# مخطط قاعدة بيانات نظام المحاسبة المصغر

| البند | القيمة |
|---|---|
| اسم المشروع | Mini Accounting System |
| الإصدار | 1.0 — مسودة تصميم |
| تاريخ الإعداد | 19 نوفمبر 2026 |
| حالة الوثيقة | مسودة تحتاج مراجعة واعتماد |
| نطاق النظام | متجر واحد |
| لغة النظام | العربية |
| التقنية المستهدفة | متوافقة مبدئياً مع بيئة .NET Framework |
| محرك قاعدة البيانات | لم يُحسم في المتطلبات؛ أنواع البيانات أدناه منطقية وقابلة للمواءمة مع SQL Server |

---

## 1. الغرض من الوثيقة

تحدد هذه الوثيقة التصميم المنطقي لقاعدة بيانات نظام المحاسبة المصغر، بما يشمل:

- الكيانات والجداول الأساسية.
- الحقول وأنواع البيانات المقترحة.
- المفاتيح الأساسية والخارجية.
- العلاقات بين الكيانات.
- القيود وقواعد سلامة البيانات.
- الفهارس المقترحة.
- سياسة ملكية البيانات والوصول إليها.
- طريقة حفظ الحركات المالية والمخزنية وتتبع مصدرها.
- القرارات التي يجب اعتمادها قبل تثبيت التصميم النهائي.

هذه الوثيقة مبنية على متطلبات الإصدار الأول، ولا تعتبر اعتماداً لسياسات محاسبية أو ضريبية محلية لم ترد في وثيقة المتطلبات.

## 2. مرجعية التصميم

يعتمد التصميم على نطاق النظام وقواعد العمل التالية الواردة في وثيقة المتطلبات:

1. النظام يدير متجرًا واحدًا في مكان موحد.
2. يضبط المتجر عملة واحدة، ولا يدعم تعدد العملات في العملية الواحدة.
3. يدير المنتجات والتصنيفات والعملاء والموردين.
4. يسجل فواتير البيع والشراء وبنودها والمدفوعات المرتبطة بها.
5. يسجل المقبوضات والمدفوعات والمصروفات والإيرادات الأخرى.
6. يتتبع أرصدة العملاء والموردين من الحركات التفصيلية.
7. يحدث المخزون بناءً على الحركات المعتمدة.
8. لا تؤثر المسودات في المخزون أو الأرصدة.
9. يمنع المخزون السالب افتراضياً.
10. لا تحذف الحركة المالية أو المخزنية المعتمدة نهائياً؛ بل تسجل عملية إلغاء أو عكس مع السبب والصلاحية.
11. يجب حفظ المستخدم ووقت التنفيذ للحركات المالية والمخزنية.
12. المرتجعات ميزة اختيارية لا تدخل الإصدار الأول إلا بعد اعتمادها.
13. يجب أن تتطابق التقارير مع السجلات التفصيلية الأصلية.

## 3. مبادئ التصميم

### 3.1 فصل البيانات الأساسية عن الحركات

تُقسم الجداول إلى مجموعات:

- بيانات المتجر والإعدادات.
- المستخدمون والأدوار والصلاحيات.
- المنتجات والتصنيفات.
- العملاء والموردون.
- فواتير البيع والشراء.
- المدفوعات والمقبوضات وتخصيصها.
- الصندوق والمصروفات والإيرادات.
- المخزون وحركاته.
- سجل التدقيق.
- إعدادات النظام.

### 3.2 عدم تخزين الأرصدة القابلة للاشتقاق باعتبارها المصدر الوحيد

الأرصدة المالية والكميات الحالية يجب أن تكون قابلة لإعادة الاحتساب من الحركات المعتمدة.

يمكن الاحتفاظ بقيم مخزنة مؤقتاً لتحسين الأداء، لكن يجب ألا تصبح المصدر الوحيد للحقيقة، ويجب توفير وسيلة لمقارنتها بالحركات الأصلية وإعادة بنائها.

### 3.3 اعتماد الحركات

كل فاتورة أو حركة مالية أو مخزنية تمر بحالة واضحة، مثل:

- `Draft`: مسودة.
- `Posted`: معتمدة وذات أثر فعلي.
- `Cancelled`: ملغاة وفق إجراء موثق.
- `Reversed`: تم عكس أثرها بحركة مقابلة.

المسودة لا تؤثر في الرصيد المالي أو المخزون. ولا يسمح بتعديل أثر حركة معتمدة بصمت.

### 3.4 حفظ مصدر الحركة

كل حركة مالية أو مخزنية يجب أن ترتبط بمصدرها، مثل فاتورة بيع أو شراء أو مصروف أو تسوية مخزون أو عملية قبض أو دفع.

### 3.5 عدم تثبيت القرارات غير المعتمدة

توجد حقول مرنة لاستيعاب قرارات مثل الخصومات والدقة العشرية، لكن لا تُعتبر قيمها أو سياساتها نهائية قبل اعتماد مالك النظام.

---

# 4. اصطلاحات وأنواع البيانات

الأسماء التقنية للجداول والحقول باللغة الإنجليزية وبصيغة `snake_case`، بينما تكون أسماء الواجهة باللغة العربية.

| النوع المنطقي | الاستخدام |
|---|---|
| `BIGINT` | معرفات السجلات |
| `INT` | أعداد صغيرة أو ترتيب |
| `NVARCHAR(n)` | النصوص الداعمة للعربية |
| `DECIMAL(p,s)` | المبالغ والكميات والأسعار |
| `BIT` | القيم المنطقية |
| `DATE` | تاريخ دون وقت |
| `DATETIME2` | تاريخ ووقت التنفيذ |
| `UNIQUEIDENTIFIER` | معرفات خارجية أو رموز تتبع عند الحاجة |
| `VARBINARY` | بيانات ثنائية عند وجود حاجة موثقة |

### 4.1 الدقة العشرية

يُقترح مبدئياً استخدام:

- المبالغ: `DECIMAL(19,4)`.
- أسعار الوحدات: `DECIMAL(19,4)`.
- الكميات: `DECIMAL(19,4)`.
- نسب الخصم: `DECIMAL(9,4)`.

هذه قيم تصميمية أولية وليست قراراً محاسبياً معتمداً. يجب اعتماد دقة العملة والكميات وطريقة التقريب قبل تثبيت أنواع الحقول النهائية.

**ممنوع استخدام `FLOAT` أو `REAL` للمبالغ المالية أو الأسعار أو الكميات التي تدخل في الحسابات.**

### 4.2 الحقول المشتركة

تحتوي الجداول التشغيلية المناسبة على الحقول التالية بحسب طبيعة السجل:

| الحقل | النوع | الغرض |
|---|---|---|
| `created_at` | `DATETIME2` | وقت إنشاء السجل |
| `created_by` | `BIGINT NULL` | المستخدم الذي أنشأ السجل |
| `updated_at` | `DATETIME2 NULL` | وقت آخر تعديل |
| `updated_by` | `BIGINT NULL` | المستخدم الذي أجرى آخر تعديل |
| `is_active` | `BIT` | حالة التفعيل للبيانات الأساسية عند الحاجة |

أما الحركات المعتمدة فتحتاج أيضاً إلى:

| الحقل | النوع | الغرض |
|---|---|---|
| `posted_at` | `DATETIME2 NULL` | وقت الاعتماد |
| `posted_by` | `BIGINT NULL` | المستخدم الذي اعتمد الحركة |
| `cancelled_at` | `DATETIME2 NULL` | وقت الإلغاء إن وجد |
| `cancelled_by` | `BIGINT NULL` | المستخدم الذي نفذ الإلغاء |
| `cancellation_reason` | `NVARCHAR(500) NULL` | سبب الإلغاء |

لا يلزم تكرار كل هذه الحقول في كل جدول؛ تطبق وفق طبيعة الكيان.

---

# 5. مخطط الكيانات والعلاقات

## 5.1 المجموعات الرئيسية

| المجموعة | الجداول |
|---|---|
| المتجر | `stores`, `store_settings` |
| المستخدمون والصلاحيات | `users`, `roles`, `permissions`, `user_roles`, `role_permissions` |
| البيانات الأساسية | `categories`, `units`, `products`, `parties` |
| المبيعات | `sales_invoices`, `sales_invoice_items` |
| المشتريات | `purchase_invoices`, `purchase_invoice_items` |
| القبض والدفع | `payment_transactions`, `payment_allocations` |
| الصندوق | `cashboxes`, `cashbox_transactions` |
| المصروفات والإيرادات | `expense_categories`, `expenses`, `income_categories`, `other_incomes` |
| المخزون | `inventory_movements`, `stock_adjustments`, `stock_adjustment_items` |
| التدقيق | `audit_logs` |

## 5.2 العلاقات الأساسية

- المتجر الواحد يمتلك مجموعة من المستخدمين والإعدادات والمنتجات والأطراف.
- التصنيف الواحد يرتبط بعدة منتجات.
- المنتج الواحد يمكن أن يظهر في عدة بنود فواتير وحركات مخزون.
- العميل يمكن أن يرتبط بعدة فواتير بيع.
- المورد يمكن أن يرتبط بعدة فواتير شراء.
- فاتورة البيع تتكون من بند واحد أو أكثر.
- فاتورة الشراء تتكون من بند واحد أو أكثر.
- حركة القبض أو الدفع يمكن تخصيصها لفاتورة واحدة أو أكثر.
- كل تخصيص دفع يرتبط بحركة مالية وفاتورة محددة.
- الحركة النقدية ترتبط بالصندوق ومصدرها.
- حركة المخزون ترتبط بالمنتج ومصدر الحركة.
- كل حركة معتمدة تسجل المستخدم الذي أنشأها أو اعتمدها ووقت التنفيذ.

---

# 6. تفاصيل الجداول والكيانات

## 6.1 جدول المتاجر `stores`

يمثل المتجر الذي يستخدم النظام. وجود الجدول لا يعني دعم عدة فروع؛ فالإصدار الأول مخصص لمتجر واحد.

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `store_id` | `BIGINT` | PK | معرف المتجر |
| `store_name` | `NVARCHAR(200)` | NOT NULL | اسم المتجر |
| `currency_code` | `NVARCHAR(10)` | NOT NULL | رمز العملة المعتمدة |
| `currency_name` | `NVARCHAR(50)` | NULL | اسم العملة للعرض |
| `is_active` | `BIT` | NOT NULL | حالة المتجر |
| `created_at` | `DATETIME2` | NOT NULL | تاريخ الإنشاء |
| `updated_at` | `DATETIME2` | NULL | تاريخ التعديل |

**قيود:**
- يجب أن يوجد متجر فعال واحد فقط في نطاق الإصدار الأول.
- لا تسمح العمليات المالية باستخدام عملة مختلفة عن عملة المتجر.
- تغيير العملة لا يعيد تفسير القيم التاريخية تلقائياً؛ سياسة التغيير تحتاج اعتماداً.

## 6.2 جدول إعدادات المتجر `store_settings`

يحفظ إعدادات المتجر التي يمكن تغييرها من شاشة الإعدادات.

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `setting_id` | `BIGINT` | PK | معرف الإعداد |
| `store_id` | `BIGINT` | FK, NOT NULL | المتجر |
| `setting_key` | `NVARCHAR(100)` | NOT NULL | اسم الإعداد |
| `setting_value` | `NVARCHAR(1000)` | NULL | قيمة الإعداد |
| `updated_at` | `DATETIME2` | NOT NULL | تاريخ التعديل |
| `updated_by` | `BIGINT` | FK, NULL | المستخدم المعدل |

**قيود وفهارس:**
- `UNIQUE(store_id, setting_key)`.
- لا تخزن كلمات المرور أو الأسرار الحساسة في هذا الجدول كنص واضح.
- أمثلة الإعدادات: رقم بداية الفاتورة، حد إعادة الطلب الافتراضي، سياسة السماح بالمخزون السالب إذا تم اعتمادها، وإعدادات العرض.

## 6.3 جدول المستخدمين `users`

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `user_id` | `BIGINT` | PK | معرف المستخدم |
| `store_id` | `BIGINT` | FK, NOT NULL | المتجر |
| `username` | `NVARCHAR(100)` | NOT NULL | اسم الدخول |
| `password_hash` | `NVARCHAR(500)` | NOT NULL | تجزئة كلمة المرور |
| `display_name` | `NVARCHAR(200)` | NOT NULL | الاسم الظاهر |
| `is_active` | `BIT` | NOT NULL | حالة الحساب |
| `last_login_at` | `DATETIME2` | NULL | آخر دخول |
| `created_at` | `DATETIME2` | NOT NULL | تاريخ الإنشاء |
| `created_by` | `BIGINT` | FK, NULL | منشئ الحساب |
| `updated_at` | `DATETIME2` | NULL | تاريخ التعديل |
| `updated_by` | `BIGINT` | FK, NULL | آخر معدل |

**قيود:**
- `UNIQUE(store_id, username)`.
- لا تحفظ كلمة المرور كنص واضح.
- الحساب المعطل لا يستطيع تسجيل الدخول.
- تعطيل الحساب لا يحذف السجلات التاريخية المنسوبة إليه.

## 6.4 جدول الأدوار `roles`

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `role_id` | `BIGINT` | PK | معرف الدور |
| `store_id` | `BIGINT` | FK, NOT NULL | المتجر |
| `role_name` | `NVARCHAR(100)` | NOT NULL | اسم الدور |
| `description` | `NVARCHAR(500)` | NULL | وصف الدور |
| `is_system_role` | `BIT` | NOT NULL | هل هو دور أساسي |
| `is_active` | `BIT` | NOT NULL | حالة الدور |

الأدوار المذكورة في المتطلبات تشمل مالك المتجر، مدير النظام، المحاسب، أمين الصندوق، موظف المبيعات، وموظف المشتريات/المخزون. لا تعتبر صلاحيات هذه الأدوار معتمدة حتى اعتماد مصفوفة الصلاحيات.

## 6.5 جدول الصلاحيات `permissions`

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `permission_id` | `BIGINT` | PK | معرف الصلاحية |
| `permission_code` | `NVARCHAR(150)` | UNIQUE, NOT NULL | رمز الصلاحية |
| `permission_name` | `NVARCHAR(200)` | NOT NULL | اسم الصلاحية |
| `module_name` | `NVARCHAR(100)` | NOT NULL | الوحدة |
| `description` | `NVARCHAR(500)` | NULL | وصف الصلاحية |

أمثلة رموز مقترحة: `sales.view`, `sales.create`, `sales.post`, `sales.cancel`, `inventory.adjust`, `users.manage`, `reports.view`.

هذه أسماء تقنية مقترحة وليست مصفوفة صلاحيات معتمدة.

## 6.6 جدول ربط المستخدمين بالأدوار `user_roles`

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `user_id` | `BIGINT` | PK, FK | المستخدم |
| `role_id` | `BIGINT` | PK, FK | الدور |
| `assigned_at` | `DATETIME2` | NOT NULL | وقت الإسناد |
| `assigned_by` | `BIGINT` | FK, NULL | من أسند الدور |

المفتاح الأساسي مركب من `user_id` و`role_id`.

## 6.7 جدول ربط الأدوار بالصلاحيات `role_permissions`

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `role_id` | `BIGINT` | PK, FK | الدور |
| `permission_id` | `BIGINT` | PK, FK | الصلاحية |
| `granted_at` | `DATETIME2` | NOT NULL | وقت منح الصلاحية |
| `granted_by` | `BIGINT` | FK, NULL | مانح الصلاحية |

المفتاح الأساسي مركب من `role_id` و`permission_id`.

---

## 6.8 جدول التصنيفات `categories`

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `category_id` | `BIGINT` | PK | معرف التصنيف |
| `store_id` | `BIGINT` | FK, NOT NULL | المتجر |
| `category_name` | `NVARCHAR(150)` | NOT NULL | اسم التصنيف |
| `description` | `NVARCHAR(500)` | NULL | وصف التصنيف |
| `is_active` | `BIT` | NOT NULL | حالة التصنيف |
| `created_at` | `DATETIME2` | NOT NULL | تاريخ الإنشاء |
| `created_by` | `BIGINT` | FK, NULL | منشئ التصنيف |

**قيد مقترح:** `UNIQUE(store_id, category_name)`.

لا يحذف التصنيف نهائياً إذا كان مرتبطاً بمنتجات أو سجلات سابقة؛ يمكن تعطيله أو نقل المنتجات إلى تصنيف آخر.

## 6.9 جدول وحدات القياس `units`

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `unit_id` | `BIGINT` | PK | معرف الوحدة |
| `store_id` | `BIGINT` | FK, NOT NULL | المتجر |
| `unit_name` | `NVARCHAR(100)` | NOT NULL | اسم الوحدة |
| `unit_symbol` | `NVARCHAR(30)` | NULL | رمز الوحدة |
| `is_active` | `BIT` | NOT NULL | حالة الوحدة |

وجود جدول مستقل للوحدات اقتراح تنظيمي لدعم حقل وحدة القياس المطلوب للمنتج. تفاصيل التحويل بين الوحدات غير محددة في المتطلبات، ولذلك لا يفترض هذا التصميم وجود تحويل تلقائي بينها.

## 6.10 جدول المنتجات `products`

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `product_id` | `BIGINT` | PK | معرف المنتج |
| `store_id` | `BIGINT` | FK, NOT NULL | المتجر |
| `product_code` | `NVARCHAR(100)` | NOT NULL | رمز المنتج |
| `product_name` | `NVARCHAR(200)` | NOT NULL | اسم المنتج |
| `category_id` | `BIGINT` | FK, NULL | التصنيف |
| `unit_id` | `BIGINT` | FK, NOT NULL | وحدة القياس |
| `selling_price` | `DECIMAL(19,4)` | NOT NULL | سعر البيع |
| `reference_cost` | `DECIMAL(19,4)` | NOT NULL | التكلفة المرجعية |
| `reorder_level` | `DECIMAL(19,4)` | NOT NULL | حد إعادة الطلب |
| `is_active` | `BIT` | NOT NULL | حالة المنتج |
| `created_at` | `DATETIME2` | NOT NULL | تاريخ الإنشاء |
| `created_by` | `BIGINT` | FK, NULL | منشئ المنتج |
| `updated_at` | `DATETIME2` | NULL | تاريخ التعديل |
| `updated_by` | `BIGINT` | FK, NULL | آخر معدل |

**ملاحظات:**
- الكمية الحالية لا تعتبر حقلاً أصلياً مستقلاً في هذا الجدول؛ مصدرها حركات المخزون المعتمدة.
- يمكن إضافة `current_quantity` كقيمة مخزنة مؤقتاً لتحسين الأداء، بشرط وجود آلية تحقق وإعادة احتساب.
- `reference_cost` هو التكلفة المرجعية المذكورة في المتطلبات، ولا يعني اعتماد طريقة تقييم مخزون محددة.

**القيود:**
- `UNIQUE(store_id, product_code)`.
- `selling_price >= 0`.
- `reference_cost >= 0`.
- `reorder_level >= 0`.
- لا يسمح بالحذف النهائي للمنتج إذا ارتبط بحركة سابقة؛ يسمح بالتعطيل.

---

## 6.11 جدول الأطراف `parties`

يجمع بيانات العملاء والموردين في كيان واحد؛ لأن الطرف قد يكون عميلاً ومورداً في الوقت نفسه. هذا اختيار تنظيمي للتصميم وليس تغييراً في الوظائف المطلوبة.

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `party_id` | `BIGINT` | PK | معرف الطرف |
| `store_id` | `BIGINT` | FK, NOT NULL | المتجر |
| `party_name` | `NVARCHAR(200)` | NOT NULL | اسم الطرف |
| `phone` | `NVARCHAR(50)` | NULL | رقم الهاتف |
| `email` | `NVARCHAR(150)` | NULL | البريد الإلكتروني |
| `address` | `NVARCHAR(300)` | NULL | العنوان |
| `notes` | `NVARCHAR(1000)` | NULL | ملاحظات |
| `is_customer` | `BIT` | NOT NULL | هل هو عميل |
| `is_supplier` | `BIT` | NOT NULL | هل هو مورد |
| `is_active` | `BIT` | NOT NULL | حالة الطرف |
| `created_at` | `DATETIME2` | NOT NULL | تاريخ الإنشاء |
| `created_by` | `BIGINT` | FK, NULL | منشئ السجل |

**القيود:**
- يجب أن يكون `is_customer = 1` أو `is_supplier = 1` على الأقل.
- لا تحذف بيانات الطرف إذا كانت مرتبطة بفواتير أو حركات؛ يمكن تعطيله.
- أرصدة الطرف تحسب من الحركات المالية المعتمدة ولا تعتمد على قيمة يدوية غير موثقة.

---

# 7. فواتير المبيعات

## 7.1 جدول فواتير البيع `sales_invoices`

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `sales_invoice_id` | `BIGINT` | PK | المعرف الداخلي |
| `store_id` | `BIGINT` | FK, NOT NULL | المتجر |
| `invoice_number` | `NVARCHAR(50)` | NOT NULL | رقم الفاتورة |
| `invoice_date` | `DATETIME2` | NOT NULL | تاريخ الفاتورة |
| `customer_id` | `BIGINT` | FK, NULL | العميل؛ السماح بالفراغ يحتاج اعتماداً لسيناريو البيع المباشر |
| `status` | `NVARCHAR(20)` | NOT NULL | حالة الفاتورة |
| `subtotal` | `DECIMAL(19,4)` | NOT NULL | مجموع البنود قبل خصم الفاتورة |
| `discount_type` | `NVARCHAR(20)` | NULL | نوع الخصم |
| `discount_value` | `DECIMAL(19,4)` | NOT NULL | قيمة الخصم |
| `discount_amount` | `DECIMAL(19,4)` | NOT NULL | مبلغ الخصم المحسوب |
| `total_amount` | `DECIMAL(19,4)` | NOT NULL | إجمالي الفاتورة |
| `notes` | `NVARCHAR(1000)` | NULL | ملاحظات |
| `created_at` | `DATETIME2` | NOT NULL | وقت الإنشاء |
| `created_by` | `BIGINT` | FK, NOT NULL | منشئ الفاتورة |
| `posted_at` | `DATETIME2` | NULL | وقت الاعتماد |
| `posted_by` | `BIGINT` | FK, NULL | معتمد الفاتورة |
| `cancelled_at` | `DATETIME2` | NULL | وقت الإلغاء |
| `cancelled_by` | `BIGINT` | FK, NULL | منفذ الإلغاء |
| `cancellation_reason` | `NVARCHAR(500)` | NULL | سبب الإلغاء |

**القيود:**
- `UNIQUE(store_id, invoice_number)`.
- لا يعتمد إجمالي سالب أو صفر.
- لا تقبل الفاتورة دون بند واحد صالح على الأقل.
- لا تؤثر المسودة في المخزون أو رصيد العميل أو الصندوق.
- لا يتم اعتماد الفاتورة إذا كان اعتمادها يؤدي إلى مخزون سالب، وفق السياسة الافتراضية.
- لا تحذف الفاتورة المعتمدة نهائياً.
- يجب أن تكون بيانات الفاتورة المعتمدة ثابتة؛ التصحيح يكون وفق إجراء إلغاء أو عكس موثق.

## 7.2 جدول بنود فواتير البيع `sales_invoice_items`

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `sales_invoice_item_id` | `BIGINT` | PK | معرف البند |
| `sales_invoice_id` | `BIGINT` | FK, NOT NULL | فاتورة البيع |
| `product_id` | `BIGINT` | FK, NOT NULL | المنتج |
| `description` | `NVARCHAR(300)` | NULL | وصف البند وقت البيع |
| `quantity` | `DECIMAL(19,4)` | NOT NULL | الكمية |
| `unit_price` | `DECIMAL(19,4)` | NOT NULL | سعر الوحدة |
| `discount_type` | `NVARCHAR(20)` | NULL | نوع خصم البند |
| `discount_value` | `DECIMAL(19,4)` | NOT NULL | قيمة الخصم |
| `discount_amount` | `DECIMAL(19,4)` | NOT NULL | مبلغ الخصم |
| `line_total` | `DECIMAL(19,4)` | NOT NULL | صافي قيمة البند |

**القيود:**
- `quantity > 0`.
- `unit_price >= 0`.
- `discount_amount >= 0`.
- لا يسمح بخصم يتجاوز قيمة البند.
- تحفظ الأسعار والأوصاف داخل البند كما كانت وقت العملية؛ حتى لا يؤدي تعديل بيانات المنتج لاحقاً إلى تغيير الفاتورة التاريخية.
- طريقة ترتيب الخصم والتقريب تحتاج اعتماداً.

---

# 8. فواتير المشتريات

## 8.1 جدول فواتير الشراء `purchase_invoices`

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `purchase_invoice_id` | `BIGINT` | PK | المعرف الداخلي |
| `store_id` | `BIGINT` | FK, NOT NULL | المتجر |
| `invoice_number` | `NVARCHAR(50)` | NOT NULL | رقم فاتورة النظام |
| `supplier_invoice_number` | `NVARCHAR(100)` | NULL | رقم فاتورة المورد إن وجد |
| `invoice_date` | `DATETIME2` | NOT NULL | تاريخ الفاتورة |
| `supplier_id` | `BIGINT` | FK, NOT NULL | المورد |
| `status` | `NVARCHAR(20)` | NOT NULL | حالة الفاتورة |
| `subtotal` | `DECIMAL(19,4)` | NOT NULL | مجموع البنود |
| `discount_type` | `NVARCHAR(20)` | NULL | نوع الخصم |
| `discount_value` | `DECIMAL(19,4)` | NOT NULL | قيمة الخصم |
| `discount_amount` | `DECIMAL(19,4)` | NOT NULL | مبلغ الخصم |
| `total_amount` | `DECIMAL(19,4)` | NOT NULL | الإجمالي |
| `notes` | `NVARCHAR(1000)` | NULL | ملاحظات |
| `created_at` | `DATETIME2` | NOT NULL | وقت الإنشاء |
| `created_by` | `BIGINT` | FK, NOT NULL | منشئ الفاتورة |
| `posted_at` | `DATETIME2` | NULL | وقت الاعتماد |
| `posted_by` | `BIGINT` | FK, NULL | معتمد الفاتورة |
| `cancelled_at` | `DATETIME2` | NULL | وقت الإلغاء |
| `cancelled_by` | `BIGINT` | FK, NULL | منفذ الإلغاء |
| `cancellation_reason` | `NVARCHAR(500)` | NULL | سبب الإلغاء |

**القيود:**
- `UNIQUE(store_id, invoice_number)`.
- لا تعتمد فاتورة دون بند صالح.
- لا تؤثر المسودة في المخزون أو رصيد المورد أو الصندوق.
- اعتماد الشراء يزيد المخزون.
- لا تحذف الفاتورة المعتمدة نهائياً.

## 8.2 جدول بنود فواتير الشراء `purchase_invoice_items`

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `purchase_invoice_item_id` | `BIGINT` | PK | معرف البند |
| `purchase_invoice_id` | `BIGINT` | FK, NOT NULL | فاتورة الشراء |
| `product_id` | `BIGINT` | FK, NOT NULL | المنتج |
| `description` | `NVARCHAR(300)` | NULL | وصف البند وقت الشراء |
| `quantity` | `DECIMAL(19,4)` | NOT NULL | الكمية |
| `unit_cost` | `DECIMAL(19,4)` | NOT NULL | تكلفة الوحدة |
| `discount_type` | `NVARCHAR(20)` | NULL | نوع الخصم |
| `discount_value` | `DECIMAL(19,4)` | NOT NULL | قيمة الخصم |
| `discount_amount` | `DECIMAL(19,4)` | NOT NULL | مبلغ الخصم |
| `line_total` | `DECIMAL(19,4)` | NOT NULL | صافي قيمة البند |

**القيود:**
- `quantity > 0`.
- `unit_cost >= 0`.
- لا يسمح بخصم يتجاوز قيمة البند.
- تحفظ تكلفة الوحدة التاريخية داخل بند الفاتورة.

---

# 9. المقبوضات والمدفوعات وتخصيصها

## 9.1 جدول الحركات المالية `payment_transactions`

يمثل عملية قبض أو دفع فعلية. يجب أن يسجل الصندوق المبلغ الذي دخل أو خرج فعلياً فقط، وليس إجمالي الفاتورة الآجلة.

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `payment_transaction_id` | `BIGINT` | PK | معرف الحركة |
| `store_id` | `BIGINT` | FK, NOT NULL | المتجر |
| `transaction_number` | `NVARCHAR(50)` | NOT NULL | رقم الحركة |
| `transaction_type` | `NVARCHAR(20)` | NOT NULL | `Receipt` أو `Payment` |
| `party_id` | `BIGINT` | FK, NULL | العميل أو المورد |
| `transaction_date` | `DATETIME2` | NOT NULL | تاريخ الحركة |
| `amount` | `DECIMAL(19,4)` | NOT NULL | المبلغ الفعلي |
| `payment_method` | `NVARCHAR(50)` | NULL | طريقة الدفع |
| `description` | `NVARCHAR(500)` | NULL | الوصف |
| `status` | `NVARCHAR(20)` | NOT NULL | حالة الحركة |
| `created_at` | `DATETIME2` | NOT NULL | وقت الإنشاء |
| `created_by` | `BIGINT` | FK, NOT NULL | منشئ الحركة |
| `posted_at` | `DATETIME2` | NULL | وقت الاعتماد |
| `posted_by` | `BIGINT` | FK, NULL | معتمد الحركة |
| `cashbox_id` | `BIGINT` | FK, NULL | الصندوق المرتبط |
| `reversal_of_id` | `BIGINT` | FK, NULL | الحركة الأصلية التي يتم عكسها |

**القيود:**
- `amount > 0`.
- نوع الحركة يجب أن يكون قيمة معروفة.
- لا يتجاوز مجموع التخصيصات المبلغ الفعلي للحركة.
- لا يتجاوز المقبوض أو المدفوع المستحق إلا عند اعتماد سياسة تسمح بذلك.
- لا تعتمد حركة دفع إذا كان رصيد الصندوق غير كافٍ وفق السياسة الافتراضية.
- لا تسجل الحركة المعتمدة أكثر من مرة.
- لا تحذف الحركة المعتمدة نهائياً.

## 9.2 جدول تخصيص المدفوعات `payment_allocations`

يفصل بين المبلغ المقبوض أو المدفوع وبين الفواتير التي تم تخصيص ذلك المبلغ لها. يدعم هذا التصميم الدفع الجزئي وتسديد أكثر من فاتورة بحركة واحدة.

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `payment_allocation_id` | `BIGINT` | PK | معرف التخصيص |
| `payment_transaction_id` | `BIGINT` | FK, NOT NULL | حركة القبض أو الدفع |
| `invoice_type` | `NVARCHAR(20)` | NOT NULL | `Sales` أو `Purchase` |
| `sales_invoice_id` | `BIGINT` | FK, NULL | فاتورة البيع عند تخصيص قبض |
| `purchase_invoice_id` | `BIGINT` | FK, NULL | فاتورة الشراء عند تخصيص دفع |
| `allocated_amount` | `DECIMAL(19,4)` | NOT NULL | المبلغ المخصص |
| `created_at` | `DATETIME2` | NOT NULL | وقت التخصيص |
| `created_by` | `BIGINT` | FK, NULL | منشئ التخصيص |

**القيود:**
- يجب تحديد فاتورة واحدة فقط في كل سجل تخصيص.
- إذا كان `invoice_type = Sales` يجب أن يكون `sales_invoice_id` غير فارغ و`purchase_invoice_id` فارغاً.
- إذا كان `invoice_type = Purchase` يجب أن يكون `purchase_invoice_id` غير فارغ و`sales_invoice_id` فارغاً.
- `allocated_amount > 0`.
- لا يزيد مجموع التخصيصات على قيمة حركة القبض أو الدفع.
- لا يزيد مجموع ما خصص لفاتورة على المستحق المتبقي لها، إلا وفق سياسة الرصيد الدائن المعتمدة.

**ملاحظة:** يجب أن تتحقق طبقة الخدمة أو إجراء قاعدة البيانات من أن نوع حركة الدفع يتوافق مع نوع الفاتورة والطرف المرتبط بها.

---

# 10. الصندوق

## 10.1 جدول الصناديق `cashboxes`

وجود جدول مستقل يسمح بتمثيل الصندوق الحالي ورصيده الافتتاحي. لا يعني ذلك دعم عدة صناديق في الإصدار الأول ما لم يعتمد ذلك لاحقاً.

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `cashbox_id` | `BIGINT` | PK | معرف الصندوق |
| `store_id` | `BIGINT` | FK, NOT NULL | المتجر |
| `cashbox_name` | `NVARCHAR(150)` | NOT NULL | اسم الصندوق |
| `opening_balance` | `DECIMAL(19,4)` | NOT NULL | الرصيد الافتتاحي |
| `is_active` | `BIT` | NOT NULL | حالة الصندوق |
| `created_at` | `DATETIME2` | NOT NULL | وقت الإنشاء |
| `created_by` | `BIGINT` | FK, NULL | منشئ الصندوق |

## 10.2 جدول حركات الصندوق `cashbox_transactions`

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `cashbox_transaction_id` | `BIGINT` | PK | معرف الحركة |
| `cashbox_id` | `BIGINT` | FK, NOT NULL | الصندوق |
| `transaction_date` | `DATETIME2` | NOT NULL | تاريخ الحركة |
| `direction` | `NVARCHAR(10)` | NOT NULL | `In` أو `Out` |
| `amount` | `DECIMAL(19,4)` | NOT NULL | المبلغ |
| `source_type` | `NVARCHAR(30)` | NOT NULL | نوع المصدر |
| `source_id` | `BIGINT` | NOT NULL | معرف المصدر |
| `description` | `NVARCHAR(500)` | NULL | الوصف |
| `status` | `NVARCHAR(20)` | NOT NULL | حالة الحركة |
| `created_at` | `DATETIME2` | NOT NULL | وقت الإنشاء |
| `created_by` | `BIGINT` | FK, NOT NULL | منشئ الحركة |
| `reversal_of_id` | `BIGINT` | FK, NULL | الحركة الأصلية عند العكس |

**أنواع المصادر المقترحة:**
- `PaymentReceipt`
- `SupplierPayment`
- `Expense`
- `OtherIncome`
- `OpeningBalance`
- `Adjustment`
- `Reversal`

**القيود:**
- `amount > 0`.
- `direction` محصور في `In` أو `Out`.
- لا تسجل حركة صندوق مستقلة إذا كان مصدرها حركة قبض أو دفع مسجلة بالفعل؛ يجب ضمان وجود أثر واحد فقط.
- يجب أن يكون لكل حركة مصدر يمكن تتبعه.
- الرصيد الحالي = الرصيد الافتتاحي + المقبوضات المعتمدة − المدفوعات المعتمدة.

---

# 11. المصروفات والإيرادات الأخرى

## 11.1 جدول تصنيفات المصروفات `expense_categories`

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `expense_category_id` | `BIGINT` | PK | معرف التصنيف |
| `store_id` | `BIGINT` | FK, NOT NULL | المتجر |
| `category_name` | `NVARCHAR(150)` | NOT NULL | اسم التصنيف |
| `description` | `NVARCHAR(500)` | NULL | الوصف |
| `is_active` | `BIT` | NOT NULL | الحالة |

## 11.2 جدول المصروفات `expenses`

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `expense_id` | `BIGINT` | PK | معرف المصروف |
| `store_id` | `BIGINT` | FK, NOT NULL | المتجر |
| `expense_category_id` | `BIGINT` | FK, NOT NULL | تصنيف المصروف |
| `expense_date` | `DATETIME2` | NOT NULL | تاريخ المصروف |
| `amount` | `DECIMAL(19,4)` | NOT NULL | قيمة المصروف |
| `description` | `NVARCHAR(500)` | NULL | وصف المصروف |
| `payment_transaction_id` | `BIGINT` | FK, NULL | حركة الدفع المرتبطة |
| `cashbox_transaction_id` | `BIGINT` | FK, NULL | حركة الصندوق المرتبطة عند اعتماد هذا الربط |
| `status` | `NVARCHAR(20)` | NOT NULL | الحالة |
| `created_at` | `DATETIME2` | NOT NULL | وقت الإنشاء |
| `created_by` | `BIGINT` | FK, NOT NULL | منشئ المصروف |

**القاعدة:** المصروف المدفوع من الصندوق يجب أن يرتبط بحركة دفع واحدة متوافقة، حتى لا يخصم المبلغ من الصندوق مرتين.

## 11.3 جدول تصنيفات الإيرادات `income_categories`

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `income_category_id` | `BIGINT` | PK | معرف التصنيف |
| `store_id` | `BIGINT` | FK, NOT NULL | المتجر |
| `category_name` | `NVARCHAR(150)` | NOT NULL | اسم التصنيف |
| `description` | `NVARCHAR(500)` | NULL | الوصف |
| `is_active` | `BIT` | NOT NULL | الحالة |

## 11.4 جدول الإيرادات الأخرى `other_incomes`

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `other_income_id` | `BIGINT` | PK | معرف الإيراد |
| `store_id` | `BIGINT` | FK, NOT NULL | المتجر |
| `income_category_id` | `BIGINT` | FK, NOT NULL | تصنيف الإيراد |
| `income_date` | `DATETIME2` | NOT NULL | تاريخ الإيراد |
| `amount` | `DECIMAL(19,4)` | NOT NULL | قيمة الإيراد |
| `description` | `NVARCHAR(500)` | NULL | الوصف |
| `payment_transaction_id` | `BIGINT` | FK, NULL | حركة القبض المرتبطة |
| `cashbox_transaction_id` | `BIGINT` | FK, NULL | حركة الصندوق المرتبطة |
| `status` | `NVARCHAR(20)` | NOT NULL | الحالة |
| `created_at` | `DATETIME2` | NOT NULL | وقت الإنشاء |
| `created_by` | `BIGINT` | FK, NOT NULL | منشئ السجل |

**القاعدة:** لا يضاف الإيراد إلى رصيد الصندوق إلا إذا تم قبضه فعلياً. الإيراد غير المقبوض لا يمثل نقداً داخل الصندوق.

---

# 12. المخزون وحركات المنتجات

## 12.1 جدول حركات المخزون `inventory_movements`

هذا الجدول هو المرجع التفصيلي لحساب كمية المنتج. كل حركة معتمدة تزيد أو تنقص الكمية.

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `inventory_movement_id` | `BIGINT` | PK | معرف الحركة |
| `store_id` | `BIGINT` | FK, NOT NULL | المتجر |
| `product_id` | `BIGINT` | FK, NOT NULL | المنتج |
| `movement_date` | `DATETIME2` | NOT NULL | تاريخ الحركة |
| `movement_type` | `NVARCHAR(30)` | NOT NULL | نوع الحركة |
| `quantity_in` | `DECIMAL(19,4)` | NOT NULL | كمية واردة |
| `quantity_out` | `DECIMAL(19,4)` | NOT NULL | كمية صادرة |
| `source_type` | `NVARCHAR(30)` | NOT NULL | نوع مصدر الحركة |
| `source_id` | `BIGINT` | NOT NULL | معرف مصدر الحركة |
| `source_item_id` | `BIGINT` | NULL | معرف بند المصدر |
| `unit_cost` | `DECIMAL(19,4)` | NULL | تكلفة مرجعية للحركة عند الحاجة |
| `description` | `NVARCHAR(500)` | NULL | الوصف |
| `status` | `NVARCHAR(20)` | NOT NULL | الحالة |
| `created_at` | `DATETIME2` | NOT NULL | وقت الإنشاء |
| `created_by` | `BIGINT` | FK, NOT NULL | منشئ الحركة |
| `posted_at` | `DATETIME2` | NULL | وقت الاعتماد |
| `posted_by` | `BIGINT` | FK, NULL | معتمد الحركة |
| `reversal_of_id` | `BIGINT` | FK, NULL | الحركة الأصلية المعكوسة |

**أنواع الحركات المقترحة:**
- `OpeningStock`: رصيد افتتاحي.
- `Purchase`: شراء.
- `Sale`: بيع.
- `AdjustmentIn`: تسوية زيادة.
- `AdjustmentOut`: تسوية نقص.
- `ReturnIn`: مرتجع وارد عند اعتماد المرتجعات.
- `ReturnOut`: مرتجع صادر عند اعتماد المرتجعات.
- `Reversal`: حركة عكس.

**القيود:**
- يجب أن تكون إحدى الكميتين موجبة والأخرى صفراً.
- لا تكون الكميتان موجبتين معاً.
- لا تكون الكميتان صفراً معاً.
- لا تؤثر المسودات في الرصيد.
- يجب منع تكرار الحركة الناتجة عن اعتماد المصدر نفسه.
- يجب حفظ مرجع المصدر لتسهيل تتبع الحركة.
- لا يسمح بأن يصبح الرصيد سالباً افتراضياً.

## 12.2 جدول تسويات المخزون `stock_adjustments`

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `stock_adjustment_id` | `BIGINT` | PK | معرف التسوية |
| `store_id` | `BIGINT` | FK, NOT NULL | المتجر |
| `adjustment_number` | `NVARCHAR(50)` | NOT NULL | رقم التسوية |
| `adjustment_date` | `DATETIME2` | NOT NULL | تاريخ التسوية |
| `reason` | `NVARCHAR(500)` | NOT NULL | سبب التسوية |
| `status` | `NVARCHAR(20)` | NOT NULL | الحالة |
| `created_at` | `DATETIME2` | NOT NULL | وقت الإنشاء |
| `created_by` | `BIGINT` | FK, NOT NULL | منشئ التسوية |
| `posted_at` | `DATETIME2` | NULL | وقت الاعتماد |
| `posted_by` | `BIGINT` | FK, NULL | معتمد التسوية |

## 12.3 جدول بنود تسوية المخزون `stock_adjustment_items`

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `stock_adjustment_item_id` | `BIGINT` | PK | معرف البند |
| `stock_adjustment_id` | `BIGINT` | FK, NOT NULL | التسوية |
| `product_id` | `BIGINT` | FK, NOT NULL | المنتج |
| `adjustment_direction` | `NVARCHAR(10)` | NOT NULL | `Increase` أو `Decrease` |
| `quantity` | `DECIMAL(19,4)` | NOT NULL | مقدار التعديل |
| `quantity_before` | `DECIMAL(19,4)` | NOT NULL | الكمية قبل التسوية |
| `quantity_after` | `DECIMAL(19,4)` | NOT NULL | الكمية بعد التسوية |
| `inventory_movement_id` | `BIGINT` | FK, NULL | حركة المخزون الناتجة |

**القيود:**
- `quantity > 0`.
- سبب التسوية إلزامي على مستوى رأس التسوية.
- يجب أن تتطابق `quantity_after` مع الكمية السابقة بعد تطبيق مقدار التعديل.
- لا يسمح بتسوية تؤدي إلى كمية سالبة.
- يجب إنشاء حركة مخزون معتمدة لكل بند تسوية معتمد.

---

# 13. المرتجعات — خارج النطاق المؤكد للإصدار الأول

وظائف المرتجعات `FR-055–FR-058` اختيارية وتحتاج إلى اعتماد مالك النظام.

إذا اعتمدت، يوصى بإضافة الجداول التالية:

## 13.1 جدول مرتجعات البيع `sales_returns`

- `sales_return_id` — PK.
- `store_id` — FK.
- `return_number`.
- `sales_invoice_id` — FK إلى فاتورة البيع الأصلية.
- `return_date`.
- `reason`.
- `refund_method`.
- `refund_amount`.
- `status`.
- `created_at`, `created_by`.
- `posted_at`, `posted_by`.

## 13.2 جدول بنود مرتجعات البيع `sales_return_items`

- `sales_return_item_id` — PK.
- `sales_return_id` — FK.
- `sales_invoice_item_id` — FK إلى بند البيع الأصلي.
- `product_id` — FK.
- `quantity`.
- `unit_price`.
- `line_total`.
- `is_resalable` — هل البضاعة صالحة لإعادة البيع.

## 13.3 جدول مرتجعات الشراء `purchase_returns`

- `purchase_return_id` — PK.
- `store_id` — FK.
- `return_number`.
- `purchase_invoice_id` — FK إلى فاتورة الشراء الأصلية.
- `return_date`.
- `reason`.
- `status`.
- `created_at`, `created_by`.
- `posted_at`, `posted_by`.

## 13.4 جدول بنود مرتجعات الشراء `purchase_return_items`

- `purchase_return_item_id` — PK.
- `purchase_return_id` — FK.
- `purchase_invoice_item_id` — FK إلى بند الشراء الأصلي.
- `product_id` — FK.
- `quantity`.
- `unit_cost`.
- `line_total`.

**شروط تفعيل المرتجعات:**
- تحديد الكمية المتبقية القابلة للإرجاع بعد خصم المرتجعات السابقة.
- تحديد حالة البضاعة المرتجعة.
- تحديد طريقة استرداد المبلغ.
- تحديد أثر المرتجع على رصيد العميل أو المورد.
- ربط كل أثر مالي أو مخزني بعملية المرتجع الأصلية.
- عدم تفعيل هذه الجداول في التنفيذ قبل اعتماد الميزة وقواعدها.

---

# 14. سجل التدقيق `audit_logs`

يسجل العمليات المهمة التي تحدث على بيانات النظام، بما يتوافق مع متطلبات التدقيق وسلامة السجلات.

| الحقل | النوع | القيود | الوصف |
|---|---|---|---|
| `audit_log_id` | `BIGINT` | PK | معرف السجل |
| `store_id` | `BIGINT` | FK, NOT NULL | المتجر |
| `user_id` | `BIGINT` | FK, NULL | المستخدم |
| `action_type` | `NVARCHAR(50)` | NOT NULL | نوع الإجراء |
| `entity_name` | `NVARCHAR(100)` | NOT NULL | اسم الجدول أو الكيان |
| `entity_id` | `BIGINT` | NOT NULL | معرف السجل المتأثر |
| `old_values` | `NVARCHAR(MAX)` | NULL | القيم السابقة عند الحاجة |
| `new_values` | `NVARCHAR(MAX)` | NULL | القيم الجديدة عند الحاجة |
| `reason` | `NVARCHAR(500)` | NULL | سبب الإجراء |
| `created_at` | `DATETIME2` | NOT NULL | وقت الإجراء |
| `ip_address` | `NVARCHAR(50)` | NULL | عنوان الاتصال عند الحاجة |

**الأحداث التي يجب تسجيلها:**
- إنشاء سجل.
- تعديل بيانات أساسية.
- اعتماد فاتورة.
- إلغاء أو عكس حركة.
- قبض أو دفع.
- تسوية مخزون.
- تغيير إعدادات المتجر.
- إنشاء أو تعطيل مستخدم.
- تغيير الأدوار والصلاحيات.
- عمليات النسخ والاستعادة إذا تم حفظ سجلها في قاعدة البيانات.

لا ينبغي أن يحتوي سجل التدقيق على كلمات مرور أو بيانات تحقق سرية.

---

# 15. العلاقات والمفاتيح الخارجية

| الجدول المصدر | الحقل | الجدول الهدف | نوع العلاقة |
|---|---|---|---|
| `users` | `store_id` | `stores` | عدة مستخدمين لمتجر |
| `store_settings` | `store_id` | `stores` | عدة إعدادات لمتجر |
| `roles` | `store_id` | `stores` | عدة أدوار لمتجر |
| `products` | `category_id` | `categories` | عدة منتجات لتصنيف |
| `products` | `unit_id` | `units` | عدة منتجات لوحدة |
| `parties` | `store_id` | `stores` | عدة أطراف لمتجر |
| `sales_invoices` | `customer_id` | `parties` | عدة فواتير بيع لطرف |
| `sales_invoice_items` | `sales_invoice_id` | `sales_invoices` | عدة بنود لفاتورة |
| `sales_invoice_items` | `product_id` | `products` | عدة بنود لمنتج |
| `purchase_invoices` | `supplier_id` | `parties` | عدة فواتير شراء لطرف |
| `purchase_invoice_items` | `purchase_invoice_id` | `purchase_invoices` | عدة بنود لفاتورة |
| `purchase_invoice_items` | `product_id` | `products` | عدة بنود لمنتج |
| `payment_transactions` | `party_id` | `parties` | عدة حركات لطرف |
| `payment_allocations` | `payment_transaction_id` | `payment_transactions` | عدة تخصيصات لحركة |
| `payment_allocations` | `sales_invoice_id` | `sales_invoices` | تخصيص لفاتورة بيع |
| `payment_allocations` | `purchase_invoice_id` | `purchase_invoices` | تخصيص لفاتورة شراء |
| `cashbox_transactions` | `cashbox_id` | `cashboxes` | عدة حركات لصندوق |
| `inventory_movements` | `product_id` | `products` | عدة حركات لمنتج |
| `stock_adjustment_items` | `stock_adjustment_id` | `stock_adjustments` | عدة بنود لتسوية |
| `stock_adjustment_items` | `product_id` | `products` | عدة تسويات لمنتج |
| `audit_logs` | `user_id` | `users` | عدة سجلات تدقيق لمستخدم |

تضاف مفاتيح خارجية أخرى للحقول التي تحمل معرفات المستخدمين مثل `created_by` و`posted_by` و`updated_by`.

---

# 16. القيود العامة لسلامة البيانات

## 16.1 قيود مالية

1. لا يقبل النظام مبلغاً صفرياً أو سالباً في حركة مالية.
2. قيمة المتبقي = إجمالي الفاتورة − مجموع المدفوعات المخصصة لها.
3. لا يتجاوز مجموع المدفوعات المخصصة المستحق افتراضياً.
4. لا يسمح بتسجيل قبض أو دفع أكبر من المستحق إلا بعد اعتماد سياسة الرصيد الدائن.
5. لا يسجل إجمالي فاتورة آجلة على أنه مبلغ داخل الصندوق.
6. يجب أن يرتبط كل قبض أو دفع معتمد بأثر صندوق واحد فقط إذا كان نقدياً.
7. يجب أن تتطابق حركات الصندوق مع مصادرها المالية.
8. لا يعتمد دفع من الصندوق إذا كان الرصيد غير كافٍ وفق السياسة الافتراضية.

## 16.2 قيود المخزون

1. البيع المعتمد ينشئ حركة صادرة.
2. الشراء المعتمد ينشئ حركة واردة.
3. المسودة لا تنشئ حركة مخزون مؤثرة.
4. لا يسمح بالمخزون السالب افتراضياً.
5. كل تسوية تحتاج سبباً.
6. لا يجوز تكرار حركة المخزون لنفس المصدر والبند.
7. لا تحذف حركة مخزون معتمدة؛ يعالج التصحيح بحركة عكس.

## 16.3 قيود الفواتير

1. كل فاتورة معتمدة تحتوي بنداً صالحاً واحداً على الأقل.
2. رقم الفاتورة فريد ضمن المتجر ونوع الفاتورة.
3. إجمالي الفاتورة يجب أن يساوي مجموع البنود بعد تطبيق الخصومات المعتمدة.
4. لا تعدل الفاتورة المعتمدة بطريقة تمحو القيم السابقة.
5. كل اعتماد يجب أن يتم مرة واحدة فقط.
6. يجب حفظ تاريخ ووقت الاعتماد والمستخدم المسؤول.

## 16.4 قيود الحذف

- يمنع الحذف النهائي للفواتير والحركات المالية والمخزنية المعتمدة.
- يمنع حذف منتج أو طرف مرتبط بحركة تاريخية.
- يسمح بتعطيل البيانات الأساسية بدلاً من حذفها.
- لا يستخدم `ON DELETE CASCADE` على العلاقات التي تمس سجلات مالية أو مخزنية أو سجلات التدقيق.

## 16.5 سلامة المعاملات

يجب أن تتم العمليات المركبة داخل معاملة قاعدة بيانات واحدة `Database Transaction`.

مثال اعتماد فاتورة بيع:

1. التحقق من حالة الفاتورة.
2. التحقق من البنود والكميات والأسعار.
3. التحقق من توفر المخزون.
4. اعتماد الفاتورة.
5. إنشاء حركات المخزون الصادرة.
6. تسجيل القبض الفعلي إن وجد.
7. إنشاء حركة الصندوق المقابلة عند القبض النقدي.
8. تسجيل تخصيص مبلغ القبض للفاتورة.
9. تسجيل حدث التدقيق.
10. تنفيذ `COMMIT` عند نجاح جميع الخطوات، أو `ROLLBACK` عند فشل أي خطوة.

يجب أن يمنع التصميم اعتماد الفاتورة مرتين بسبب إعادة إرسال الطلب أو تكرار الضغط على زر الاعتماد.

---

# 17. الفهارس المقترحة

الفهارس التالية نقطة بداية، ويجب مراجعتها بعد معرفة حجم البيانات وأنماط الاستعلام الفعلية.

| الجدول | الفهرس | النوع | الغرض |
|---|---|---|---|
| `users` | `(store_id, username)` | UNIQUE | منع تكرار اسم الدخول |
| `products` | `(store_id, product_code)` | UNIQUE | منع تكرار رمز المنتج |
| `products` | `(store_id, product_name)` | NON-UNIQUE | البحث بالاسم |
| `products` | `(store_id, category_id, is_active)` | NON-UNIQUE | تصفية المنتجات حسب التصنيف |
| `parties` | `(store_id, party_name)` | NON-UNIQUE | البحث عن العملاء والموردين |
| `sales_invoices` | `(store_id, invoice_number)` | UNIQUE | تفرد رقم فاتورة البيع |
| `sales_invoices` | `(store_id, invoice_date)` | NON-UNIQUE | تقارير المبيعات حسب التاريخ |
| `sales_invoices` | `(customer_id, invoice_date)` | NON-UNIQUE | كشف حساب العميل |
| `purchase_invoices` | `(store_id, invoice_number)` | UNIQUE | تفرد رقم فاتورة الشراء |
| `purchase_invoices` | `(store_id, invoice_date)` | NON-UNIQUE | تقارير المشتريات |
| `purchase_invoices` | `(supplier_id, invoice_date)` | NON-UNIQUE | كشف حساب المورد |
| `sales_invoice_items` | `(sales_invoice_id)` | NON-UNIQUE | جلب بنود الفاتورة |
| `purchase_invoice_items` | `(purchase_invoice_id)` | NON-UNIQUE | جلب بنود الفاتورة |
| `payment_transactions` | `(store_id, transaction_date)` | NON-UNIQUE | تقارير القبض والدفع |
| `payment_transactions` | `(party_id, transaction_date)` | NON-UNIQUE | كشف الطرف |
| `payment_allocations` | `(sales_invoice_id)` | NON-UNIQUE | حساب المدفوع لفاتورة البيع |
| `payment_allocations` | `(purchase_invoice_id)` | NON-UNIQUE | حساب المدفوع لفاتورة الشراء |
| `cashbox_transactions` | `(cashbox_id, transaction_date)` | NON-UNIQUE | كشف الصندوق |
| `inventory_movements` | `(product_id, movement_date)` | NON-UNIQUE | سجل حركة المنتج |
| `inventory_movements` | `(source_type, source_id, source_item_id)` | NON-UNIQUE | تتبع مصدر الحركة |
| `audit_logs` | `(entity_name, entity_id, created_at)` | NON-UNIQUE | سجل تدقيق السجل |
| `audit_logs` | `(user_id, created_at)` | NON-UNIQUE | مراجعة عمليات المستخدم |

يجب تقييم الفهارس الإضافية باستخدام خطط تنفيذ الاستعلامات، وعدم إضافة فهارس كثيرة تؤثر سلباً على عمليات الإدخال والتحديث.

---

# 18. سياسة ملكية البيانات والوصول إليها

## 18.1 المالك الأساسي للبيانات

المتجر هو نطاق ملكية البيانات في الإصدار الأول. كل سجل تشغيلي يتبع المتجر من خلال `store_id` مباشرة أو من خلال علاقة موثوقة بجدول تابع له.

لا توجد في النطاق الحالي مشاركة بيانات بين متاجر أو فروع.

## 18.2 ملكية السجلات

| نوع البيانات | المالك المنطقي | من ينشئها عادةً | ضوابط التعديل |
|---|---|---|---|
| إعدادات المتجر | المتجر | مدير النظام | وفق صلاحية معتمدة |
| المستخدمون والأدوار | المتجر | مدير النظام | وفق صلاحية إدارة المستخدمين |
| المنتجات والتصنيفات | المتجر | المستخدم المخول | لا يحذف المنتج المرتبط بحركة |
| العملاء والموردون | المتجر | المستخدم المخول | يحافظ على السجل التاريخي |
| فواتير البيع | المتجر | موظف المبيعات أو المخول | المسودة قابلة للتعديل؛ المعتمدة لا تعدل بصمت |
| فواتير الشراء | المتجر | موظف المشتريات أو المخول | المسودة قابلة للتعديل؛ المعتمدة لا تعدل بصمت |
| القبض والدفع | المتجر | أمين الصندوق أو المخول | لا تحذف الحركة المعتمدة |
| حركات الصندوق | المتجر | النظام نتيجة حركة مالية أو تسوية معتمدة | لا تعدل أو تحذف بصمت |
| حركات المخزون | المتجر | النظام نتيجة البيع أو الشراء أو التسوية | لا تحذف الحركة المعتمدة |
| سجل التدقيق | المتجر | النظام | سجل تاريخي لا يعدل من واجهة المستخدم |

## 18.3 سياسة المستخدم

- كل مستخدم يعمل ضمن متجر واحد.
- لا يستطيع المستخدم الوصول إلى بيانات متجر آخر.
- لا تكفي إخفاء الأزرار في الواجهة لتطبيق الصلاحيات؛ يجب التحقق منها في الخادم عند كل عملية.
- يجب التحقق من ملكية السجل للمتجر قبل عرضه أو تعديله أو اعتماده.
- لا تمنح صلاحية اعتماد الفواتير أو إلغائها تلقائياً لمجرد امتلاك صلاحية عرضها.
- الصلاحيات التفصيلية النهائية تنتظر اعتماد مصفوفة الأدوار.

## 18.4 سياسة السجل التاريخي

- يحتفظ النظام بهوية منشئ السجل ومعتمده.
- يحتفظ النظام بالتاريخ والوقت.
- يحتفظ النظام بسبب الإلغاء أو التسوية.
- لا يؤدي تعطيل مستخدم إلى حذف سجلاته.
- لا يؤدي تعطيل منتج أو طرف إلى حذف الحركات السابقة المرتبطة به.

---

# 19. طريقة احتساب الأرصدة

## 19.1 رصيد فاتورة البيع

`remaining_amount = total_amount - allocated_receipts`

حيث:

- `total_amount`: إجمالي الفاتورة المعتمدة.
- `allocated_receipts`: مجموع المقبوضات المعتمدة والمخصصة للفاتورة.

## 19.2 رصيد فاتورة الشراء

`remaining_amount = total_amount - allocated_payments`

حيث:

- `total_amount`: إجمالي فاتورة الشراء المعتمدة.
- `allocated_payments`: مجموع المدفوعات المعتمدة والمخصصة للفاتورة.

## 19.3 رصيد الصندوق

`cashbox_balance = opening_balance + total_in - total_out`

تدخل في الحساب الحركات المعتمدة فقط، مع ضمان عدم تكرار مصدر الحركة.

## 19.4 كمية المنتج

`current_quantity = opening_stock + total_in - total_out`

تدخل في الحساب حركات المخزون المعتمدة فقط.

## 19.5 رصيد العميل

يحسب من فواتير البيع المعتمدة والمبالغ المقبوضة المخصصة لها وأي تسويات أو مرتجعات معتمدة بعد اعتمادها.

## 19.6 رصيد المورد

يحسب من فواتير الشراء المعتمدة والمدفوعات المخصصة لها وأي تسويات أو مرتجعات معتمدة بعد اعتمادها.

**ملاحظة مهمة:** اتجاه عرض الرصيد في كشف العميل أو المورد، وطريقة تمثيل الرصيد الدائن، يحتاجان تعريفاً واضحاً في عقد API والتقارير قبل التنفيذ.

---

# 20. النسخ الاحتياطي والاستعادة

النسخ الاحتياطي والاستعادة من متطلبات النظام، لكن تفاصيل تنفيذها تعتمد على محرك قاعدة البيانات وطريقة التشغيل التي لم تحسمها وثيقة المتطلبات.

يمكن إضافة جدول اختياري `backup_logs` لتسجيل العمليات:

| الحقل | النوع | الوصف |
|---|---|---|
| `backup_log_id` | `BIGINT` PK | معرف العملية |
| `store_id` | `BIGINT` FK | المتجر |
| `operation_type` | `NVARCHAR(20)` | Backup أو Restore |
| `file_name` | `NVARCHAR(300)` | اسم الملف |
| `file_path` | `NVARCHAR(1000)` | المسار |
| `status` | `NVARCHAR(20)` | النتيجة |
| `started_at` | `DATETIME2` | وقت البداية |
| `completed_at` | `DATETIME2 NULL` | وقت الانتهاء |
| `user_id` | `BIGINT FK` | المستخدم |
| `message` | `NVARCHAR(1000)` | رسالة النتيجة |

لا يحل هذا الجدول محل النسخة الاحتياطية نفسها؛ فهو يسجل معلومات العملية فقط.

---

# 21. قواعد تنفيذية يجب أن تلتزم بها طبقة Backend

1. لا تنفذ عمليات الاعتماد من خلال تحديث الحالة فقط؛ يجب إنشاء جميع آثارها المرتبطة ضمن معاملة واحدة.
2. لا تثق بالمجاميع المرسلة من الواجهة؛ يعيد الخادم احتساب الإجماليات من البنود.
3. لا تثق برصيد المنتج المرسل من الواجهة؛ يتحقق الخادم من الحركات المعتمدة.
4. لا تثق بالمبلغ المتبقي المرسل من الواجهة؛ يحسب الخادم المدفوع والمتبقي.
5. تتحقق الخدمة من أن الطرف والمنتج والفاتورة والصندوق تنتمي إلى المتجر نفسه.
6. تتحقق الخدمة من صلاحية المستخدم لكل عملية.
7. تمنع الخدمة تكرار اعتماد العملية.
8. تسجل الخدمة جميع الحركات المالية والمخزنية الناتجة عن المصدر نفسه.
9. تستخدم المعاملات لضمان عدم بقاء فاتورة معتمدة دون آثارها المالية والمخزنية.
10. تستخدم آلية مناسبة لمنع تعارض عمليتي بيع متزامنتين على الكمية نفسها.
11. لا تسمح بتعديل الفاتورة المعتمدة أو حركتها التاريخية دون إجراء عكس موثق.
12. لا تسمح بحذف السجلات التي يجب الاحتفاظ بها تاريخياً.

---

# 22. قرارات مفتوحة قبل اعتماد المخطط النهائي

الحقول المقترحة لا تعني أن السياسات المرتبطة بها قد اعتُمدت. القرارات التالية يجب أن يحسمها مالك النظام:

| المعرف | القرار | أثره على قاعدة البيانات | الحالة |
|---|---|---|---|
| DEC-001 | العملة المعتمدة | `stores.currency_code` ودقة المبالغ والعرض | مفتوح |
| DEC-002 | دقة المبالغ والكميات | دقة حقول `DECIMAL` | مفتوح |
| DEC-003 | طريقة التقريب وتوقيته | حساب البنود والإجماليات والتقارير | مفتوح |
| DEC-004 | سياسة الخصومات | خصم البند وخصم الفاتورة والحدود والصلاحيات | مفتوح |
| DEC-005 | الرصيد الدائن والدفع الزائد | تخصيص المدفوعات وأرصدة الأطراف | مفتوح |
| DEC-006 | الرصيد الافتتاحي للصندوق | حركة افتتاحية وتاريخها ومسؤول اعتمادها | مفتوح |
| DEC-007 | طرق القبض والدفع | قيم `payment_method` | مفتوح |
| DEC-008 | ترقيم الفواتير | قواعد الرقم والتسلسل وإعادة البداية | مفتوح |
| DEC-009 | تعديل الفاتورة بعد اعتمادها | سياسة التعديل والعكس | مفتوح |
| DEC-010 | إلغاء الفواتير | شكل العكس وآثاره المالية والمخزنية | مفتوح |
| DEC-011 | البيع دون عميل مسجل | السماح بـ `customer_id = NULL` أو إنشاء عميل افتراضي | مفتوح |
| DEC-012 | المخزون السالب | إعداد المنع والتحقق وقت الاعتماد | مفتوح؛ الافتراضي المنع |
| DEC-013 | الرصيد الافتتاحي للمخزون | طريقة إنشاء حركات `OpeningStock` | مفتوح |
| DEC-014 | طريقة تقييم تكلفة المخزون | الحاجة إلى سجل تكلفة أو متوسط تكلفة أو غيره | مفتوح |
| DEC-015 | حد إعادة الطلب الافتراضي | إعداد المتجر وقيمة المنتج | مفتوح |
| DEC-016 | وحدات القياس والكسور | السماح بالكميات الكسرية والتحويل بين الوحدات | مفتوح |
| DEC-017 | اعتماد المرتجعات | تفعيل جداول المرتجعات وآثارها | مفتوح |
| DEC-018 | عدد المستخدمين | تقدير الأداء والجلسات | مفتوح |
| DEC-019 | مصفوفة الأدوار والصلاحيات | بيانات الأدوار وربطها بالصلاحيات | مفتوح |
| DEC-020 | مدة انتهاء الجلسة | إعداد أمان المستخدمين | مفتوح |
| DEC-021 | طريقة تشغيل النظام | متطلبات الاتصال والنسخ والاستعادة | مفتوح |
| DEC-024 | سياسة النسخ الاحتياطي | سجل النسخ ومكان الحفظ والاحتفاظ | مفتوح |
| DEC-025 | مدة الاحتفاظ بالسجلات | سجل التدقيق والأرشفة | مفتوح |
| DEC-026 | المنطقة الزمنية والتاريخ | طريقة حفظ وعرض الوقت | مفتوح |
| DEC-030 | المتطلبات التنظيمية المحلية | أي حقول أو قيود ضريبية أو محاسبية إضافية | مفتوح |

---

# 23. ما هو خارج نطاق هذا المخطط حالياً؟

لا يضيف هذا التصميم تلقائياً ما يلي:

- عدة متاجر أو فروع ضمن قاعدة واحدة.
- عمليات متعددة العملات.
- الرواتب.
- الضرائب الإلكترونية أو التكامل الضريبي.
- الربط البنكي.
- التجارة الإلكترونية.
- نقاط البيع المتخصصة.
- نظام محاسبة مزدوج القيد ودفتر أستاذ كامل.
- تحويل وحدات القياس.
- مرتجعات مفعلة قبل اعتمادها.
- طريقة تقييم مخزون نهائية قبل اعتماد القرار الخاص بها.

إذا اعتمدت أي من هذه الوظائف لاحقاً، يجب تعديل المتطلبات والمخطط وعقود API والاختبارات معاً.

---

# 24. معايير قبول قاعدة البيانات

يعتبر التصميم جاهزاً للاعتماد بعد التحقق من الآتي:

1. وجود جميع الكيانات المطلوبة في نطاق الإصدار الأول.
2. وجود المفاتيح الأساسية والخارجية والعلاقات الصحيحة.
3. منع تكرار رموز المنتجات وأرقام الفواتير وأسماء الدخول ضمن نطاق المتجر.
4. عدم قبول مبالغ أو كميات غير صالحة.
5. عدم تأثير المسودات في المخزون أو الأرصدة.
6. إنشاء حركة مخزون واحدة فقط عند اعتماد فاتورة البيع أو الشراء.
7. تسجيل المقبوض أو المدفوع الفعلي فقط في الصندوق.
8. دعم الدفع الجزئي وتخصيص الدفعات للفواتير.
9. إمكانية استخراج رصيد العميل والمورد من الحركات التفصيلية.
10. إمكانية استخراج رصيد الصندوق من الرصيد الافتتاحي والحركات.
11. إمكانية استخراج كمية المنتج من حركات المخزون.
12. منع الحذف النهائي للحركات المعتمدة.
13. تسجيل المستخدم ووقت التنفيذ والسبب في العمليات الحساسة.
14. تطبيق صلاحيات المستخدم في الخادم.
15. دعم النسخ الاحتياطي والاستعادة وفق محرك قاعدة البيانات المعتمد.
16. اجتياز اختبارات تطابق التقارير مع السجلات التفصيلية.

---

# 25. اعتماد الوثيقة

| الدور | المسؤولية | الحالة |
|---|---|---|
| مالك النظام | اعتماد قواعد العمل والقرارات المالية والتشغيلية | بانتظار الاعتماد |
| قائد المشروع / مسؤول التكامل | مراجعة العلاقات والكيانات واعتماد التكامل مع API | بانتظار المراجعة |
| مسؤول قاعدة البيانات | مراجعة أنواع البيانات والقيود والفهارس والتنفيذ | بانتظار المراجعة |
| Backend Developer | مراجعة قابلية تنفيذ المعاملات وقواعد العمل | بانتظار المراجعة |
| QA | مراجعة قابلية اختبار القيود والتطابق | بانتظار المراجعة |

**لا تعتبر هذه الوثيقة معتمدة للتنفيذ النهائي قبل مراجعة القرارات المفتوحة وتوقيع الاعتماد.**

---

**نهاية الوثيقة — DATABASE_SCHEMA.md**