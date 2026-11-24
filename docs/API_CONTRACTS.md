# API_CONTRACTS.md
# عقود واجهات البرمجة — نظام المحاسبة المصغر

| البند | القيمة |
|---|---|
| المشروع | Mini Accounting System |
| الإصدار | 1.0 — مسودة تكامل |
| تاريخ الإعداد | 19 نوفمبر 2026 |
| حالة الوثيقة | تحتاج مراجعة واعتماد |
| مرجع المتطلبات | وثيقة متطلبات النظام v2.0 |
| مرجع قاعدة البيانات | `DATABASE_SCHEMA.md` |
| التقنية المستهدفة | .NET Framework |
| نمط الواجهات | REST API |
| تنسيق البيانات | JSON |
| لغة الواجهة | العربية — RTL |

---

# 1. الغرض من الوثيقة

تحدد هذه الوثيقة عقود API المشتركة بين Frontend وBackend لنظام المحاسبة المصغر، وتوحّد:

- عناوين ومسارات API.
- HTTP methods.
- بنية الطلبات والاستجابات.
- قواعد التحقق من البيانات.
- رموز الأخطاء.
- آلية المصادقة والصلاحيات.
- تدفق اعتماد العمليات المالية والمخزنية.
- العلاقات بين الفواتير والصندوق والديون والمخزون.
- المرشحات والبحث والتقارير.
- متطلبات التكامل والاختبار.

يجب أن يلتزم فريق Frontend بهذه العقود، وألا يعتمد على أسماء حقول أو سلوكيات غير موثقة فيها.

# 2. مبادئ تصميم API

## 2.1 القواعد العامة

1. تستخدم الواجهات نمط REST.
2. تستخدم البيانات بصيغة JSON.
3. تستخدم أسماء الحقول التقنية بصيغة `camelCase`.
4. تستخدم أسماء الموارد في المسارات بصيغة الجمع.
5. تستخدم المسارات باللغة الإنجليزية.
6. تستخدم الواجهة العربية لعرض الرسائل والأسماء.
7. لا ترسل الواجهة إجماليات مالية موثوقة؛ يحسب Backend الإجماليات.
8. لا ترسل الواجهة رصيد المخزون باعتباره حقيقة؛ يستخرجه Backend من الحركات المعتمدة.
9. لا تعتمد الواجهة الفاتورة بمجرد حفظها؛ الاعتماد عملية منفصلة.
10. لا تؤثر المسودات في المخزون أو الصندوق أو أرصدة الأطراف.
11. يجب أن تتحقق الصلاحيات في Backend لكل عملية، وليس في الواجهة فقط.
12. لا يسمح بالحذف النهائي للحركات المالية أو المخزنية المعتمدة.
13. يجب أن تكون العمليات المركبة ذرية قدر الإمكان باستخدام Database Transaction.
14. يجب حفظ المستخدم ووقت التنفيذ للحركات المالية والمخزنية.
15. كل استجابة يجب أن تكون متسقة مع حالة العملية الفعلية في قاعدة البيانات.

## 2.2 عنوان API الأساسي

المسار المقترح:

```text
/api/v1
```

أمثلة:

```text
GET    /api/v1/products
POST   /api/v1/sales-invoices
GET    /api/v1/sales-invoices/1001
POST   /api/v1/sales-invoices/1001/post
```

يجب عدم تغيير معنى المسار بعد اعتماده. أي تغيير غير متوافق يستلزم إصدار API جديداً أو مساراً جديداً.

## 2.3 Content Types

الطلب:

```http
Content-Type: application/json
Accept: application/json
```

الاستجابة:

```http
Content-Type: application/json; charset=utf-8
```

تستخدم UTF-8 لدعم النصوص العربية.

## 2.4 تنسيق التاريخ والوقت

التنسيق المقترح:

```text
2026-11-19T14:30:00
```

ويجب أن يعتمد الفريق المنطقة الزمنية وطريقة تخزين الوقت قبل التنفيذ النهائي. لا يجوز افتراض منطقة زمنية مختلفة بين Frontend وBackend.

## 2.5 تنسيق المبالغ والكميات

ترسل المبالغ والكميات كأرقام JSON، وليس كنصوص منسقة تحتوي فواصل أو رموز عملة.

مثال:

```json
{
  "quantity": 2.5,
  "unitPrice": 150.00
}
```

لا ترسل القيم التالية باعتبارها بيانات حسابية:

```json
{
  "amount": "1,500 YER"
}
```

تتم إضافة رمز العملة وتنسيق العرض في الواجهة بناءً على إعدادات المتجر.

الدقة العشرية وطريقة التقريب لا تزالان قرارين مفتوحين؛ يجب اعتماد القاعدة نفسها في Backend والتقارير والواجهة.

---

# 3. بنية الاستجابة الموحدة

## 3.1 الاستجابة الناجحة

استجابة لطلب إرجاع سجل واحد:

```json
{
  "success": true,
  "data": {
    "id": 1001,
    "name": "منتج تجريبي"
  },
  "message": null
}
```

استجابة لطلب إرجاع قائمة:

```json
{
  "success": true,
  "data": [],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 0,
    "totalPages": 0
  },
  "message": null
}
```

استجابة لعملية إنشاء أو اعتماد:

```json
{
  "success": true,
  "data": {
    "id": 1001,
    "status": "Posted"
  },
  "message": "تم تنفيذ العملية بنجاح"
}
```

## 3.2 الاستجابة عند فشل التحقق

```json
{
  "success": false,
  "data": null,
  "message": "يرجى تصحيح البيانات المدخلة",
  "errors": [
    {
      "field": "quantity",
      "code": "INVALID_QUANTITY",
      "message": "الكمية يجب أن تكون أكبر من صفر"
    }
  ]
}
```

## 3.3 الاستجابة عند عدم العثور على سجل

```json
{
  "success": false,
  "data": null,
  "message": "السجل غير موجود",
  "errors": [
    {
      "code": "RESOURCE_NOT_FOUND",
      "message": "لم يتم العثور على السجل المطلوب"
    }
  ]
}
```

## 3.4 قواعد الاستجابة

- `success`: يحدد نجاح العملية.
- `data`: البيانات الناتجة أو `null`.
- `message`: رسالة عامة قابلة للعرض.
- `errors`: تفاصيل أخطاء التحقق أو التنفيذ.
- `pagination`: تظهر عند إرجاع قائمة مقسمة إلى صفحات.

لا يجب إرسال Stack Trace أو تفاصيل SQL أو معلومات الاتصال بقاعدة البيانات إلى Frontend.

---

# 4. رموز HTTP والأخطاء

| HTTP Status | الاستخدام |
|---|---|
| `200 OK` | قراءة أو تنفيذ ناجح |
| `201 Created` | إنشاء سجل بنجاح |
| `204 No Content` | عملية ناجحة دون بيانات استجابة |
| `400 Bad Request` | بيانات طلب غير صحيحة |
| `401 Unauthorized` | المستخدم غير مسجل الدخول أو الجلسة غير صالحة |
| `403 Forbidden` | المستخدم لا يمتلك الصلاحية |
| `404 Not Found` | السجل غير موجود |
| `409 Conflict` | تعارض في حالة السجل أو تكرار عملية |
| `422 Unprocessable Entity` | فشل قاعدة عمل أو تحقق من البيانات |
| `500 Internal Server Error` | خطأ داخلي غير متوقع |

## 4.1 رموز أخطاء الأعمال المقترحة

| الرمز | المعنى |
|---|---|
| `VALIDATION_ERROR` | بيانات غير صحيحة |
| `RESOURCE_NOT_FOUND` | السجل غير موجود |
| `DUPLICATE_USERNAME` | اسم المستخدم مستخدم |
| `DUPLICATE_PRODUCT_CODE` | رمز المنتج مكرر |
| `DUPLICATE_INVOICE_NUMBER` | رقم الفاتورة مكرر |
| `INVALID_INVOICE_STATUS` | حالة الفاتورة لا تسمح بالعملية |
| `INVOICE_EMPTY` | الفاتورة لا تحتوي بنوداً |
| `INVALID_AMOUNT` | المبلغ صفر أو سالب |
| `INVALID_QUANTITY` | الكمية غير صالحة |
| `INSUFFICIENT_STOCK` | المخزون غير كافٍ |
| `INSUFFICIENT_CASHBOX_BALANCE` | رصيد الصندوق غير كافٍ |
| `OVERPAYMENT_NOT_ALLOWED` | المبلغ يتجاوز المستحق |
| `ALLOCATION_EXCEEDS_AMOUNT` | التخصيص يتجاوز مبلغ الحركة |
| `PERMISSION_DENIED` | الصلاحية غير متاحة |
| `ALREADY_POSTED` | العملية معتمدة مسبقاً |
| `ALREADY_CANCELLED` | العملية ملغاة مسبقاً |
| `CANCELLATION_REASON_REQUIRED` | سبب الإلغاء مطلوب |
| `INVALID_SOURCE_REFERENCE` | مرجع المصدر غير صحيح |
| `RESTORE_CONFIRMATION_REQUIRED` | تأكيد الاستعادة مطلوب |
| `BACKUP_FILE_INVALID` | ملف النسخة الاحتياطية غير صالح |
| `OPERATION_IN_PROGRESS` | العملية قيد التنفيذ |

يجب توحيد رموز الأخطاء في Backend وعدم إنشاء أسماء مختلفة للخطأ نفسه بين الوحدات.

---

# 5. المصادقة والجلسات والصلاحيات

## 5.1 تسجيل الدخول

**Endpoint**

```http
POST /api/v1/auth/login
```

**Request**

```json
{
  "username": "admin",
  "password": "user-password"
}
```

**Response**

```json
{
  "success": true,
  "data": {
    "user": {
      "id": 1,
      "username": "admin",
      "displayName": "مدير النظام",
      "isActive": true,
      "roles": [
        {
          "id": 1,
          "name": "SystemAdmin"
        }
      ],
      "permissions": [
        "products.view",
        "products.create"
      ]
    },
    "session": {
      "expiresAt": "2026-11-19T18:00:00"
    }
  },
  "message": "تم تسجيل الدخول بنجاح"
}
```

آلية نقل الجلسة (Cookie أو Token) لم تحسم بعد، ويجب اعتمادها قبل التنفيذ. المثال يوضح شكل البيانات المنطقية ولا يفرض آلية مصادقة بعينها.

**قواعد العمل:**
- لا يسمح بالدخول لحساب معطل.
- لا يكشف النظام هل اسم المستخدم أم كلمة المرور هو الخطأ.
- لا تحفظ كلمة المرور كنص واضح.
- تطبق مدة انتهاء الجلسة بعد اعتماد إعدادها.
- يجب حماية جميع المسارات الخاصة بالبيانات.

**الأخطاء:**
- `401 Unauthorized`
- `INVALID_CREDENTIALS`
- `ACCOUNT_DISABLED`

## 5.2 تسجيل الخروج

```http
POST /api/v1/auth/logout
```

يجب إبطال الجلسة الحالية، بحيث لا يمكن استخدامها بعد تسجيل الخروج.

## 5.3 المستخدم الحالي

```http
GET /api/v1/auth/me
```

يعيد بيانات المستخدم الحالي والأدوار والصلاحيات المسموح بها.

## 5.4 إدارة المستخدمين

| Method | Endpoint | الوظيفة |
|---|---|---|
| GET | `/api/v1/users` | عرض المستخدمين |
| GET | `/api/v1/users/{id}` | تفاصيل مستخدم |
| POST | `/api/v1/users` | إنشاء مستخدم |
| PUT | `/api/v1/users/{id}` | تعديل بيانات مستخدم |
| PATCH | `/api/v1/users/{id}/status` | تفعيل أو تعطيل مستخدم |
| PUT | `/api/v1/users/{id}/roles` | إسناد الأدوار أو الصلاحيات وفق التصميم المعتمد |

**إنشاء مستخدم — Request**

```json
{
  "username": "cashier01",
  "password": "initial-password",
  "displayName": "أمين الصندوق",
  "isActive": true,
  "roleIds": [3]
}
```

**قواعد:**
- اسم الدخول فريد.
- كلمة المرور لا تعاد ضمن استجابات القراءة.
- الحساب المعطل لا يسجل الدخول.
- لا يحذف المستخدم إذا كانت له حركات تاريخية؛ يعطل الحساب.
- لا يفترض أن الدور يمنح صلاحيات قبل اعتماد مصفوفة الصلاحيات.

## 5.5 الأدوار والصلاحيات

| Method | Endpoint | الوظيفة |
|---|---|---|
| GET | `/api/v1/roles` | عرض الأدوار |
| GET | `/api/v1/permissions` | عرض الصلاحيات المعرفة |
| GET | `/api/v1/roles/{id}/permissions` | صلاحيات دور |
| PUT | `/api/v1/roles/{id}/permissions` | تحديث صلاحيات دور |

**تحديث الصلاحيات — Request**

```json
{
  "permissionCodes": [
    "sales.view",
    "sales.create"
  ]
}
```

القيم السابقة أمثلة لرموز تقنية مقترحة. التوزيع النهائي على الأدوار يحتاج اعتماد مالك النظام.

---

# 6. المنتجات والتصنيفات

## 6.1 المنتجات

| Method | Endpoint | الوظيفة |
|---|---|---|
| GET | `/api/v1/products` | قائمة المنتجات والبحث |
| GET | `/api/v1/products/{id}` | تفاصيل المنتج |
| POST | `/api/v1/products` | إنشاء منتج |
| PUT | `/api/v1/products/{id}` | تعديل منتج |
| PATCH | `/api/v1/products/{id}/status` | تفعيل أو تعطيل |
| GET | `/api/v1/products/{id}/stock` | الكمية الحالية |
| GET | `/api/v1/products/{id}/movements` | سجل حركة المنتج |

### البحث عن المنتجات

```http
GET /api/v1/products?search=قهوة&categoryId=2&isActive=true&page=1&pageSize=20
```

### إنشاء منتج

```http
POST /api/v1/products
```

**Request**

```json
{
  "productCode": "PRD-001",
  "productName": "قهوة",
  "categoryId": 2,
  "unitId": 1,
  "sellingPrice": 1500,
  "referenceCost": 1100,
  "openingQuantity": 20,
  "reorderLevel": 5,
  "isActive": true
}
```

`openingQuantity` يسمح به فقط إذا تم اعتماد إدخال الرصيد الافتتاحي بهذه الطريقة. ويجب أن ينشئ Backend حركة افتتاحية موثقة، لا أن يغير رصيداً مخزناً دون حركة.

**Response**

```json
{
  "success": true,
  "data": {
    "id": 100,
    "productCode": "PRD-001",
    "productName": "قهوة",
    "sellingPrice": 1500,
    "referenceCost": 1100,
    "currentQuantity": 20,
    "reorderLevel": 5,
    "isActive": true
  },
  "message": "تم إنشاء المنتج"
}
```

**قواعد:**
- رمز المنتج فريد ضمن المتجر.
- السعر والتكلفة والكمية وحد الطلب لا تكون سالبة.
- لا يحذف المنتج المرتبط بحركة؛ يمكن تعطيله.
- الكمية الحالية تستخرج من حركات المخزون المعتمدة.
- يظهر تنبيه إعادة الطلب عندما تكون الكمية مساوية للحد أو أقل منه.

## 6.2 التصنيفات

| Method | Endpoint | الوظيفة |
|---|---|---|
| GET | `/api/v1/categories` | عرض التصنيفات |
| GET | `/api/v1/categories/{id}` | تفاصيل تصنيف |
| POST | `/api/v1/categories` | إنشاء تصنيف |
| PUT | `/api/v1/categories/{id}` | تعديل تصنيف |
| PATCH | `/api/v1/categories/{id}/status` | تعطيل أو تفعيل |

لا يسمح بالحذف النهائي لتصنيف مستخدم إلا بعد نقل المنتجات أو تعطيله.

## 6.3 وحدات القياس

| Method | Endpoint | الوظيفة |
|---|---|---|
| GET | `/api/v1/units` | عرض وحدات القياس |
| POST | `/api/v1/units` | إنشاء وحدة |
| PUT | `/api/v1/units/{id}` | تعديل وحدة |
| PATCH | `/api/v1/units/{id}/status` | حالة الوحدة |

لا يدعم هذا العقد تحويل الكميات بين وحدات مختلفة؛ لم تعتمد هذه الوظيفة في المتطلبات.

---

# 7. العملاء والموردون

## 7.1 العملاء

| Method | Endpoint | الوظيفة |
|---|---|---|
| GET | `/api/v1/customers` | قائمة العملاء |
| GET | `/api/v1/customers/{id}` | تفاصيل العميل |
| POST | `/api/v1/customers` | إنشاء عميل |
| PUT | `/api/v1/customers/{id}` | تعديل عميل |
| GET | `/api/v1/customers/{id}/balance` | رصيد العميل |
| GET | `/api/v1/customers/{id}/statement` | كشف حساب العميل |

**إنشاء عميل — Request**

```json
{
  "name": "عميل تجريبي",
  "phone": "770000000",
  "email": null,
  "address": null,
  "notes": null
}
```

## 7.2 الموردون

| Method | Endpoint | الوظيفة |
|---|---|---|
| GET | `/api/v1/suppliers` | قائمة الموردين |
| GET | `/api/v1/suppliers/{id}` | تفاصيل المورد |
| POST | `/api/v1/suppliers` | إنشاء مورد |
| PUT | `/api/v1/suppliers/{id}` | تعديل مورد |
| GET | `/api/v1/suppliers/{id}/balance` | رصيد المورد |
| GET | `/api/v1/suppliers/{id}/statement` | كشف حساب المورد |

**إنشاء مورد — Request**

```json
{
  "name": "مورد تجريبي",
  "phone": "770000001",
  "email": null,
  "address": null,
  "notes": null
}
```

يمكن تنفيذ العملاء والموردين باستخدام كيان داخلي مشترك `parties`، لكن يجب أن تبقى مسارات API واضحة حسب دور الطرف.

## 7.3 رصيد الطرف

**Response مقترح**

```json
{
  "success": true,
  "data": {
    "partyId": 20,
    "partyName": "عميل تجريبي",
    "balance": 2500,
    "currencyCode": "YER",
    "asOf": "2026-11-19T23:59:59"
  }
}
```

رمز العملة في المثال توضيحي فقط؛ لا يعتمد `YER` قبل قرار مالك النظام.

## 7.4 كشف الحساب

```http
GET /api/v1/customers/20/statement?fromDate=2026-11-01&toDate=2026-11-19&page=1&pageSize=50
```

**Response**

```json
{
  "success": true,
  "data": {
    "partyId": 20,
    "openingBalance": 1000,
    "transactions": [
      {
        "date": "2026-11-10T10:30:00",
        "type": "SalesInvoice",
        "referenceNumber": "S-1001",
        "description": "فاتورة بيع",
        "debit": 2000,
        "credit": 500,
        "balance": 2500
      }
    ],
    "closingBalance": 2500
  }
}
```

يجب اعتماد اتجاه المدين والدائن وطريقة عرض رصيد العميل والمورد قبل تثبيت صيغة الكشف النهائية.

---

# 8. فواتير المبيعات

## 8.1 المسارات

| Method | Endpoint | الوظيفة |
|---|---|---|
| GET | `/api/v1/sales-invoices` | قائمة فواتير البيع |
| GET | `/api/v1/sales-invoices/{id}` | تفاصيل فاتورة |
| POST | `/api/v1/sales-invoices` | إنشاء مسودة |
| PUT | `/api/v1/sales-invoices/{id}` | تعديل مسودة |
| POST | `/api/v1/sales-invoices/{id}/post` | اعتماد فاتورة |
| POST | `/api/v1/sales-invoices/{id}/cancel` | إلغاء/عكس فاتورة |
| GET | `/api/v1/sales-invoices/{id}/print` | بيانات الطباعة |

## 8.2 إنشاء مسودة فاتورة بيع

```http
POST /api/v1/sales-invoices
```

**Request**

```json
{
  "invoiceDate": "2026-11-19T10:30:00",
  "customerId": 20,
  "items": [
    {
      "productId": 100,
      "quantity": 2,
      "unitPrice": 1500,
      "discountType": null,
      "discountValue": 0
    },
    {
      "productId": 101,
      "quantity": 1,
      "unitPrice": 800,
      "discountType": null,
      "discountValue": 0
    }
  ],
  "invoiceDiscount": {
    "type": null,
    "value": 0
  },
  "notes": "ملاحظات"
}
```

`customerId` يمكن أن يكون فارغاً في حالة البيع العام فقط إذا تم اعتماد هذا السيناريو. لا تفترض الواجهة قبول البيع العام قبل حسم القرار.

**Response**

```json
{
  "success": true,
  "data": {
    "id": 1001,
    "invoiceNumber": "S-1001",
    "status": "Draft",
    "subtotal": 3800,
    "discountAmount": 0,
    "totalAmount": 3800,
    "paidAmount": 0,
    "remainingAmount": 3800,
    "items": []
  },
  "message": "تم حفظ مسودة الفاتورة"
}
```

الأرقام أعلاه بيانات توضيحية.

## 8.3 اعتماد فاتورة البيع

```http
POST /api/v1/sales-invoices/{id}/post
```

**Request**

```json
{
  "paidAmount": 1000,
  "paymentMethod": "Cash",
  "cashboxId": 1
}
```

إذا كان المبلغ المدفوع صفراً، لا تنشأ حركة قبض. وإذا كان أكبر من صفر، يسجل Backend المقبوض الفعلي فقط.

**Response**

```json
{
  "success": true,
  "data": {
    "invoiceId": 1001,
    "invoiceNumber": "S-1001",
    "status": "Posted",
    "totalAmount": 3800,
    "paidAmount": 1000,
    "remainingAmount": 2800,
    "stockMovementsCreated": 2,
    "cashboxTransactionCreated": true
  },
  "message": "تم اعتماد فاتورة البيع"
}
```

## 8.4 قواعد اعتماد فاتورة البيع

عند الاعتماد يجب أن ينفذ Backend ضمن معاملة واحدة:

1. التحقق من وجود الفاتورة وحالتها `Draft`.
2. التحقق من وجود بند واحد صحيح على الأقل.
3. التحقق من صحة المنتجات والكميات والأسعار والخصومات.
4. إعادة حساب الإجماليات في Backend.
5. التحقق من توفر المخزون.
6. التحقق من صحة المبلغ المدفوع وعدم تجاوز المستحق.
7. اعتماد الفاتورة.
8. إنشاء حركات صرف مخزون مرتبطة بالفاتورة وبنودها.
9. إنشاء حركة قبض بالمبلغ المستلم فعلياً فقط، إذا كان أكبر من صفر.
10. تخصيص القبض للفاتورة.
11. تحديث الأرصدة الناتجة عن الحركات.
12. تسجيل المستخدم والوقت في سجل التدقيق.
13. إرجاع حالة الفاتورة والأرصدة النهائية.

إذا فشلت خطوة، يجب التراجع عن جميع تغييرات العملية.

## 8.5 إلغاء فاتورة البيع

```http
POST /api/v1/sales-invoices/{id}/cancel
```

**Request**

```json
{
  "reason": "تم إدخال الفاتورة بالخطأ"
}
```

**قواعد:**
- سبب الإلغاء إلزامي.
- يجب أن يمتلك المستخدم صلاحية الإلغاء.
- لا يسمح بإلغاء الفاتورة مرتين.
- يبقى سجل الفاتورة الأصلي.
- تنشأ حركات عكس للمخزون والأثر المالي والرصيد وفق ما تم تحصيله.
- لا يجوز حذف حركات القبض الأصلية؛ تعالج بحركة عكس موثقة.
- يجب تحديد سلوك الإلغاء إذا كانت هناك تخصيصات أو عمليات لاحقة مرتبطة بالفاتورة قبل التنفيذ النهائي.

---

# 9. فواتير المشتريات

## 9.1 المسارات

| Method | Endpoint | الوظيفة |
|---|---|---|
| GET | `/api/v1/purchase-invoices` | قائمة فواتير الشراء |
| GET | `/api/v1/purchase-invoices/{id}` | تفاصيل فاتورة |
| POST | `/api/v1/purchase-invoices` | إنشاء مسودة |
| PUT | `/api/v1/purchase-invoices/{id}` | تعديل مسودة |
| POST | `/api/v1/purchase-invoices/{id}/post` | اعتماد فاتورة |
| POST | `/api/v1/purchase-invoices/{id}/cancel` | إلغاء/عكس فاتورة |

## 9.2 إنشاء مسودة شراء

```http
POST /api/v1/purchase-invoices
```

**Request**

```json
{
  "invoiceDate": "2026-11-19T11:00:00",
  "supplierId": 30,
  "supplierInvoiceNumber": "SUP-2026-15",
  "items": [
    {
      "productId": 100,
      "quantity": 10,
      "unitCost": 1100,
      "discountType": null,
      "discountValue": 0
    }
  ],
  "invoiceDiscount": {
    "type": null,
    "value": 0
  },
  "notes": null
}
```

## 9.3 اعتماد فاتورة الشراء

```http
POST /api/v1/purchase-invoices/{id}/post
```

**Request**

```json
{
  "paidAmount": 5000,
  "paymentMethod": "Cash",
  "cashboxId": 1
}
```

**قواعد الاعتماد:**
- إعادة حساب إجمالي الفاتورة في Backend.
- التحقق من المورد والمنتجات والكميات والتكاليف.
- التحقق من أن المبلغ المدفوع موجب أو صفر.
- منع دفع أكثر من المستحق افتراضياً.
- التحقق من كفاية رصيد الصندوق عند الدفع النقدي.
- زيادة المخزون بحركات مرتبطة ببنود الشراء.
- تسجيل المدفوع الفعلي فقط كحركة خارجة من الصندوق.
- تحديث رصيد المورد.
- تنفيذ العملية داخل Database Transaction.

**Response**

```json
{
  "success": true,
  "data": {
    "invoiceId": 2001,
    "status": "Posted",
    "totalAmount": 11000,
    "paidAmount": 5000,
    "remainingAmount": 6000,
    "stockMovementsCreated": 1,
    "cashboxTransactionCreated": true
  },
  "message": "تم اعتماد فاتورة الشراء"
}
```

---

# 10. المقبوضات والمدفوعات

## 10.1 المسارات

| Method | Endpoint | الوظيفة |
|---|---|---|
| GET | `/api/v1/payment-transactions` | قائمة الحركات المالية |
| GET | `/api/v1/payment-transactions/{id}` | تفاصيل حركة |
| POST | `/api/v1/payment-transactions/receipts` | تسجيل قبض |
| POST | `/api/v1/payment-transactions/payments` | تسجيل دفع |
| GET | `/api/v1/payment-transactions/{id}/allocations` | تخصيصات الحركة |

## 10.2 تحصيل من عميل

```http
POST /api/v1/payment-transactions/receipts
```

**Request**

```json
{
  "partyId": 20,
  "transactionDate": "2026-11-19T12:00:00",
  "amount": 1000,
  "paymentMethod": "Cash",
  "cashboxId": 1,
  "description": "تحصيل من العميل",
  "allocations": [
    {
      "salesInvoiceId": 1001,
      "amount": 1000
    }
  ]
}
```

**Response**

```json
{
  "success": true,
  "data": {
    "id": 5001,
    "transactionType": "Receipt",
    "amount": 1000,
    "allocatedAmount": 1000,
    "unallocatedAmount": 0,
    "cashboxBalance": 15000
  },
  "message": "تم تسجيل القبض"
}
```

## 10.3 دفع لمورد

```http
POST /api/v1/payment-transactions/payments
```

**Request**

```json
{
  "partyId": 30,
  "transactionDate": "2026-11-19T12:30:00",
  "amount": 2000,
  "paymentMethod": "Cash",
  "cashboxId": 1,
  "description": "سداد للمورد",
  "allocations": [
    {
      "purchaseInvoiceId": 2001,
      "amount": 2000
    }
  ]
}
```

## 10.4 قواعد القبض والدفع

- المبلغ أكبر من صفر.
- يجب أن يكون الطرف من النوع الصحيح.
- يجب أن تكون الفاتورة من نوع يتوافق مع الحركة.
- لا يتجاوز مجموع التخصيصات مبلغ الحركة.
- لا يتجاوز التخصيص المستحق المتبقي افتراضياً.
- المبلغ غير المخصص لا يعالج كرصد دائن إلا وفق سياسة معتمدة.
- الدفع النقدي لا يتجاوز رصيد الصندوق.
- يجب أن تتولد حركة صندوق واحدة فقط لكل قبض أو دفع نقدي.
- يجب أن تتولد آثار الرصيد والتخصيص مرة واحدة فقط.
- يجب حفظ المستخدم والتاريخ والوقت.

## 10.5 تخصيص دفعة لاحقاً

إذا كانت هناك حركة قبض أو دفع غير مخصصة أو مخصصة جزئياً:

```http
POST /api/v1/payment-transactions/{id}/allocations
```

**Request**

```json
{
  "allocations": [
    {
      "salesInvoiceId": 1002,
      "amount": 500
    }
  ]
}
```

لا يقبل النظام تخصيصاً يزيد على المبلغ غير المخصص أو يزيد على مستحق الفاتورة.

---

# 11. الصندوق

## 11.1 المسارات

| Method | Endpoint | الوظيفة |
|---|---|---|
| GET | `/api/v1/cashboxes` | عرض الصناديق |
| GET | `/api/v1/cashboxes/{id}` | تفاصيل الصندوق |
| GET | `/api/v1/cashboxes/{id}/balance` | الرصيد الحالي |
| GET | `/api/v1/cashboxes/{id}/transactions` | حركات الصندوق |
| GET | `/api/v1/cashboxes/{id}/statement` | كشف الصندوق |

## 11.2 الرصيد

```http
GET /api/v1/cashboxes/1/balance
```

**Response**

```json
{
  "success": true,
  "data": {
    "cashboxId": 1,
    "openingBalance": 10000,
    "totalIn": 5000,
    "totalOut": 2000,
    "currentBalance": 13000,
    "currencyCode": "YER"
  }
}
```

القيم والعملة في المثال توضيحية.

## 11.3 كشف الصندوق

```http
GET /api/v1/cashboxes/1/statement?fromDate=2026-11-01&toDate=2026-11-19
```

يعيد:
- الرصيد الافتتاحي.
- المقبوضات.
- المدفوعات.
- المصروفات.
- الإيرادات المقبوضة.
- تفاصيل الحركات.
- الرصيد الختامي.

**المعادلة:**

```text
Closing Balance = Opening Balance + Total In - Total Out
```

تدخل الحركات المعتمدة فقط في الحساب.

---

# 12. المصروفات والإيرادات الأخرى

## 12.1 تصنيفات المصروفات

| Method | Endpoint |
|---|---|
| GET | `/api/v1/expense-categories` |
| POST | `/api/v1/expense-categories` |
| PUT | `/api/v1/expense-categories/{id}` |
| PATCH | `/api/v1/expense-categories/{id}/status` |

## 12.2 تسجيل مصروف

```http
POST /api/v1/expenses
```

**Request**

```json
{
  "expenseCategoryId": 1,
  "expenseDate": "2026-11-19T13:00:00",
  "amount": 500,
  "description": "مصروف نقل",
  "paymentMethod": "Cash",
  "cashboxId": 1
}
```

**قواعد:**
- المبلغ أكبر من صفر.
- التصنيف موجود وفعال.
- يتحقق Backend من كفاية الصندوق.
- يسجل المصروف مرة واحدة.
- يسجل أثر الدفع في الصندوق مرة واحدة.
- يظهر المصروف في تقرير المصروفات.
- لا يقبل النظام مصروفاً يؤدي إلى رصيد صندوق سالب.

## 12.3 تصنيفات الإيرادات الأخرى

| Method | Endpoint |
|---|---|
| GET | `/api/v1/income-categories` |
| POST | `/api/v1/income-categories` |
| PUT | `/api/v1/income-categories/{id}` |
| PATCH | `/api/v1/income-categories/{id}/status` |

## 12.4 تسجيل إيراد آخر

```http
POST /api/v1/other-incomes
```

**Request**

```json
{
  "incomeCategoryId": 1,
  "incomeDate": "2026-11-19T13:30:00",
  "amount": 700,
  "source": "إيراد متنوع",
  "description": "إيراد غير ناتج عن البيع",
  "paymentMethod": "Cash",
  "cashboxId": 1
}
```

**قواعد:**
- لا يسجل في الصندوق إلا المبلغ المقبوض فعلياً.
- لا يكرر أثر الإيراد في الصندوق.
- يظهر في تقرير الإيرادات الأخرى.
- المبلغ صفر أو السالب مرفوض.

---

# 13. المخزون والتسويات

## 13.1 المسارات

| Method | Endpoint | الوظيفة |
|---|---|---|
| GET | `/api/v1/inventory` | عرض أرصدة المنتجات |
| GET | `/api/v1/inventory/movements` | قائمة حركات المخزون |
| GET | `/api/v1/inventory/products/{productId}/movements` | حركات منتج |
| GET | `/api/v1/inventory/low-stock` | المنتجات منخفضة المخزون |
| POST | `/api/v1/stock-adjustments` | إنشاء مسودة تسوية |
| GET | `/api/v1/stock-adjustments/{id}` | تفاصيل تسوية |
| PUT | `/api/v1/stock-adjustments/{id}` | تعديل مسودة تسوية |
| POST | `/api/v1/stock-adjustments/{id}/post` | اعتماد التسوية |

## 13.2 عرض المخزون

```http
GET /api/v1/inventory?search=قهوة&categoryId=2&page=1&pageSize=20
```

**Response**

```json
{
  "success": true,
  "data": [
    {
      "productId": 100,
      "productCode": "PRD-001",
      "productName": "قهوة",
      "currentQuantity": 15,
      "reorderLevel": 5,
      "isLowStock": false
    }
  ],
  "pagination": {
    "page": 1,
    "pageSize": 20,
    "totalItems": 1,
    "totalPages": 1
  }
}
```

## 13.3 سجل حركة المنتج

```http
GET /api/v1/inventory/products/100/movements?fromDate=2026-11-01&toDate=2026-11-19
```

يجب أن تتضمن كل حركة:
- التاريخ والوقت.
- نوع الحركة.
- الكمية الواردة أو الصادرة.
- الكمية الناتجة.
- مرجع المصدر.
- المستخدم.
- الحالة.

## 13.4 تسوية المخزون

```http
POST /api/v1/stock-adjustments
```

**Request**

```json
{
  "adjustmentDate": "2026-11-19T14:00:00",
  "reason": "فرق الجرد",
  "items": [
    {
      "productId": 100,
      "direction": "Increase",
      "quantity": 3
    },
    {
      "productId": 101,
      "direction": "Decrease",
      "quantity": 1
    }
  ]
}
```

**قواعد:**
- السبب إلزامي.
- الكمية أكبر من صفر.
- يعرض Backend الكمية قبل وبعد.
- لا يسمح بنقص يؤدي إلى رصيد سالب.
- لا تؤثر المسودة في المخزون.
- عند الاعتماد تنشأ حركة مخزون لكل بند.
- تحفظ هوية المستخدم ووقت الاعتماد.
- لا تحذف التسوية المعتمدة.

## 13.5 المنتجات منخفضة المخزون

```http
GET /api/v1/inventory/low-stock
```

يعيد المنتجات التي:

```text
Current Quantity <= Reorder Level
```

---

# 14. التقارير ولوحة المعلومات

## 14.1 لوحة المعلومات

```http
GET /api/v1/dashboard?fromDate=2026-11-01&toDate=2026-11-19
```

**Response مقترح**

```json
{
  "success": true,
  "data": {
    "salesTotal": 50000,
    "purchasesTotal": 30000,
    "expensesTotal": 5000,
    "cashboxBalance": 15000,
    "customerReceivables": 10000,
    "supplierPayables": 8000,
    "lowStockProductsCount": 4
  }
}
```

يجب أن تتطابق مؤشرات لوحة المعلومات مع التقارير التفصيلية للفترة نفسها، وأن تستبعد المسودات والعمليات غير المعتمدة وفق قواعد التقرير.

## 14.2 المسارات

| التقرير | Endpoint |
|---|---|
| المبيعات | `GET /api/v1/reports/sales` |
| المشتريات | `GET /api/v1/reports/purchases` |
| المصروفات والإيرادات | `GET /api/v1/reports/expenses-incomes` |
| أرصدة العملاء | `GET /api/v1/reports/customer-balances` |
| أرصدة الموردين | `GET /api/v1/reports/supplier-balances` |
| حركة الصندوق | `GET /api/v1/reports/cashbox` |
| المخزون | `GET /api/v1/reports/inventory` |
| المنتجات منخفضة المخزون | `GET /api/v1/reports/low-stock` |

## 14.3 مرشحات التقارير

أمثلة للمرشحات:

```text
fromDate
toDate
customerId
supplierId
productId
categoryId
page
pageSize
```

تطبق المرشحات المناسبة فقط لكل تقرير.

مثال:

```http
GET /api/v1/reports/sales?fromDate=2026-11-01&toDate=2026-11-19&customerId=20&page=1&pageSize=50
```

## 14.4 بيانات الطباعة

يجب أن تعيد واجهات التقارير البيانات اللازمة لعرض التقرير وطباعته كصفحة متصفح، بما يشمل:
- اسم المتجر.
- العملة.
- عنوان التقرير.
- الفترة المحددة.
- المرشحات المستخدمة.
- وقت إصدار التقرير.
- الإجماليات.
- التفاصيل.

لا يشترط إنشاء PDF في الإصدار الأول؛ المتطلب الحالي هو العرض والطباعة عبر صفحات المتصفح.

---

# 15. المرتجعات — اختيارية وتتطلب اعتماداً

لا يتم تفعيل هذه المسارات إلا بعد اعتماد مالك النظام لإدراج المرتجعات في الإصدار الأول وقواعدها.

## 15.1 مرتجعات البيع

| Method | Endpoint |
|---|---|
| GET | `/api/v1/sales-returns` |
| GET | `/api/v1/sales-returns/{id}` |
| POST | `/api/v1/sales-returns` |
| POST | `/api/v1/sales-returns/{id}/post` |

**Request مقترح**

```json
{
  "salesInvoiceId": 1001,
  "returnDate": "2026-11-19T15:00:00",
  "reason": "منتج غير مناسب",
  "items": [
    {
      "salesInvoiceItemId": 1,
      "quantity": 1,
      "isResalable": true
    }
  ],
  "refundMethod": "Cash"
}
```

يجب التحقق من أن الكمية المرتجعة لا تتجاوز الكمية المباعة بعد خصم المرتجعات السابقة.

## 15.2 مرتجعات الشراء

| Method | Endpoint |
|---|---|
| GET | `/api/v1/purchase-returns` |
| GET | `/api/v1/purchase-returns/{id}` |
| POST | `/api/v1/purchase-returns` |
| POST | `/api/v1/purchase-returns/{id}/post` |

يجب التحقق من أن الكمية المرتجعة لا تتجاوز الكمية المستلمة غير المرتجعة.

لا يعتبر هذا القسم عقداً معتمداً للتنفيذ قبل حسم قرار المرتجعات وقواعد الاسترداد والمخزون والرصيد.

---

# 16. إعدادات المتجر وترقيم الفواتير

## 16.1 إعدادات المتجر

| Method | Endpoint |
|---|---|
| GET | `/api/v1/settings/store` |
| PUT | `/api/v1/settings/store` |

**Request**

```json
{
  "storeName": "اسم المتجر",
  "currencyCode": "YER"
}
```

رمز العملة في المثال توضيحي ولا يمثل قراراً معتمداً.

**قواعد:**
- المتجر يستخدم عملة واحدة.
- تغيير العملة لا يعيد تفسير السجلات التاريخية.
- يجب اعتماد سياسة تغيير العملة قبل إتاحة التغيير بعد وجود حركات مالية.

## 16.2 إعدادات ترقيم الفواتير

```http
GET /api/v1/settings/invoice-numbering
PUT /api/v1/settings/invoice-numbering
```

**Request مقترح**

```json
{
  "salesInvoiceStartNumber": 1001,
  "purchaseInvoiceStartNumber": 2001,
  "defaultReorderLevel": 5
}
```

يجب اعتماد شكل الترقيم، وهل التسلسل منفصل بين البيع والشراء، وكيفية التعامل مع الأرقام السابقة قبل تثبيت العقد النهائي.

---

# 17. النسخ الاحتياطي والاستعادة

## 17.1 إنشاء نسخة احتياطية

```http
POST /api/v1/backups
```

**Request**

```json
{
  "destinationPath": "D:\\Backups"
}
```

**Response عند النجاح**

```json
{
  "success": true,
  "data": {
    "backupId": 1,
    "fileName": "MiniAccounting_20261119.bak",
    "filePath": "D:\\Backups\\MiniAccounting_20261119.bak",
    "status": "Completed",
    "completedAt": "2026-11-19T16:00:00"
  },
  "message": "تم إنشاء النسخة الاحتياطية بنجاح"
}
```

لا يعيد Backend نجاحاً إلا بعد اكتمال إنشاء الملف والتحقق من نتيجة العملية.

## 17.2 الاستعادة

```http
POST /api/v1/backups/restore
```

**Request**

```json
{
  "backupFilePath": "D:\\Backups\\MiniAccounting_20261119.bak",
  "confirmation": true
}
```

**قواعد:**
1. التحقق من وجود الملف.
2. التحقق من سلامة الملف وتوافقه.
3. طلب تأكيد صريح من المستخدم.
4. توضيح أن البيانات الحالية ستستبدل.
5. تنفيذ الاستعادة.
6. إرجاع النتيجة الحقيقية للعملية.
7. عدم إظهار نجاح إذا فشلت الاستعادة.

يفضل فصل خطوة التحقق عن خطوة التنفيذ:

```http
POST /api/v1/backups/validate-restore
```

ثم:

```http
POST /api/v1/backups/restore
```

شكل التحقق المقترح:

```json
{
  "backupFilePath": "D:\\Backups\\MiniAccounting_20261119.bak"
}
```

**Response**

```json
{
  "success": true,
  "data": {
    "isValid": true,
    "requiresConfirmation": true,
    "warning": "ستؤدي الاستعادة إلى استبدال البيانات الحالية"
  }
}
```

مسار الملف المحلي وطريقة وصول Backend إليه يتوقفان على قرار طريقة تشغيل النظام، لذلك لا يعتمد هذا العقد قبل حسم التشغيل المحلي أو الخادم.

## 17.3 نتيجة النسخ والاستعادة

```http
GET /api/v1/backups/{id}
```

يعيد:
- نوع العملية.
- الحالة.
- وقت البداية والانتهاء.
- اسم الملف ومساره عند الإنشاء.
- رسالة نتيجة مناسبة.
- سبب الفشل إن وجد دون كشف معلومات تقنية حساسة.

---

# 18. مصفوفة الصلاحيات المقترحة على مستوى API

الأسماء التالية رموز تقنية مقترحة للصلاحيات، وليست اعتماداً لتوزيعها على الأدوار.

| الوحدة | الصلاحيات المقترحة |
|---|---|
| المصادقة | `auth.login`, `auth.logout` |
| المستخدمون | `users.view`, `users.create`, `users.update`, `users.disable` |
| الأدوار | `roles.view`, `roles.manage` |
| المنتجات | `products.view`, `products.create`, `products.update`, `products.disable` |
| التصنيفات | `categories.view`, `categories.manage` |
| العملاء | `customers.view`, `customers.create`, `customers.update` |
| الموردون | `suppliers.view`, `suppliers.create`, `suppliers.update` |
| المبيعات | `sales.view`, `sales.create`, `sales.update`, `sales.post`, `sales.cancel`, `sales.print` |
| المشتريات | `purchases.view`, `purchases.create`, `purchases.update`, `purchases.post`, `purchases.cancel` |
| الصندوق | `cashbox.view`, `cashbox.receipt`, `cashbox.payment`, `cashbox.statement` |
| المصروفات | `expenses.view`, `expenses.create` |
| الإيرادات | `incomes.view`, `incomes.create` |
| الديون | `debts.view`, `debts.collect`, `debts.pay` |
| المخزون | `inventory.view`, `inventory.adjust`, `inventory.movements` |
| التقارير | `reports.view`, `reports.print` |
| الإعدادات | `settings.view`, `settings.manage` |
| النسخ | `backup.create`, `backup.restore` |

**قاعدة إلزامية:** كل Endpoint محمي يجب أن يتحقق من الصلاحية على الخادم. لا تعتبر إخفاء زر في الواجهة حماية كافية.

---

# 19. قواعد التكامل بين الوحدات

هذه أهم نقطة في النظام؛ لأن العملية الواحدة تؤثر في أكثر من وحدة.

## 19.1 تدفق بيع منتج

```text
Frontend
   |
   v
Create Sales Invoice Draft
   |
   v
Validate Invoice
   |
   v
Post Sales Invoice
   |
   +----> Decrease Inventory
   |
   +----> Create Receipt (paid amount only)
   |
   +----> Update Customer Receivable
   |
   +----> Create Audit Log
   |
   v
Return Invoice + Stock + Payment + Balance
```

## 19.2 تدفق شراء منتجات

```text
Frontend
   |
   v
Create Purchase Invoice Draft
   |
   v
Validate Invoice
   |
   v
Post Purchase Invoice
   |
   +----> Increase Inventory
   |
   +----> Create Payment (paid amount only)
   |
   +----> Update Supplier Payable
   |
   +----> Create Audit Log
   |
   v
Return Invoice + Stock + Payment + Balance
```

## 19.3 تحصيل دين عميل

```text
Create Receipt
   |
   +----> Validate Customer
   |
   +----> Validate Outstanding Amount
   |
   +----> Create Cashbox In Movement
   |
   +----> Allocate Receipt to Invoice/Debt
   |
   +----> Update Customer Balance
   |
   +----> Audit Log
```

## 19.4 دفع مستحق مورد

```text
Create Payment
   |
   +----> Validate Supplier
   |
   +----> Validate Outstanding Amount
   |
   +----> Validate Cashbox Balance
   |
   +----> Create Cashbox Out Movement
   |
   +----> Allocate Payment to Purchase Invoice
   |
   +----> Update Supplier Balance
   |
   +----> Audit Log
```

## 19.5 تسوية المخزون

```text
Create Adjustment Draft
   |
   v
Validate Reason and Quantity
   |
   v
Post Adjustment
   |
   +----> Create Inventory Movement
   |
   +----> Save Before/After Quantity
   |
   +----> Save User and Timestamp
   |
   +----> Audit Log
```

## 19.6 قاعدة الاتساق

يجب ألا تنجح عملية اعتماد فاتورة إذا فشل إنشاء أي أثر أساسي تابع لها. يجب استخدام معاملة قاعدة بيانات واحدة للعملية المركبة.

---

# 20. التكرار وإعادة إرسال الطلبات

قد يرسل Frontend طلب اعتماد مرتين بسبب الضغط المتكرر أو انقطاع الاتصال.

يجب أن يمنع Backend تكرار الآثار المالية والمخزنية.

**اقتراح تكاملي:** دعم رأس طلب:

```http
Idempotency-Key: 6d2d0e4a-7e4d-4b6c-9d9f-123456789abc
```

يستخدم مع عمليات مثل:
- اعتماد فاتورة بيع.
- اعتماد فاتورة شراء.
- تسجيل قبض.
- تسجيل دفع.
- اعتماد تسوية مخزون.
- إنشاء نسخة احتياطية.

إذا تكرر الطلب بالمفتاح نفسه والبيانات نفسها، يعيد Backend النتيجة السابقة دون إنشاء أثر جديد.

هذا اقتراح تقني يجب الاتفاق عليه بين Integration Owner وBackend قبل التنفيذ.

---

# 21. التزامن وحماية البيانات

1. يجب منع اعتماد فاتورتين متزامنتين تؤديان إلى بيع كمية أكبر من المتاح.
2. يجب التحقق من رصيد الصندوق داخل المعاملة عند تنفيذ الدفع.
3. يجب منع تعديل المسودة من مستخدمين متزامنين دون اكتشاف التعارض.
4. يوصى باستخدام `rowVersion` أو آلية مناسبة لاكتشاف التعديل المتزامن.
5. يجب ألا يثق Backend بمعرفات السجلات دون التحقق من انتمائها للمتجر.
6. لا يسمح باسترجاع بيانات متجر آخر.
7. يجب تسجيل المستخدم ووقت التنفيذ في العمليات الحساسة.

## 21.1 مثال اكتشاف التعارض

يمكن أن يعاد:

```json
{
  "success": false,
  "data": null,
  "message": "تم تعديل السجل بواسطة مستخدم آخر",
  "errors": [
    {
      "code": "CONCURRENCY_CONFLICT",
      "message": "أعد تحميل البيانات قبل الحفظ"
    }
  ]
}
```

---

# 22. Pagination والبحث والترتيب

## 22.1 المعاملات العامة

```text
page
pageSize
search
sortBy
sortDirection
```

مثال:

```http
GET /api/v1/sales-invoices?page=1&pageSize=20&sortBy=invoiceDate&sortDirection=desc
```

## 22.2 قواعد

- الصفحة الافتراضية: `1`.
- حجم الصفحة الافتراضي: `20`.
- الحد الأقصى المقترح: `100`.
- الترتيب الافتراضي حسب تاريخ الإنشاء أو تاريخ المستند وفق طبيعة المورد.
- يجب التحقق من أن `sortBy` ضمن الحقول المسموح بها.
- يجب عدم تمرير نص ترتيب SQL مباشرة إلى قاعدة البيانات.

هذه قيم تقنية مقترحة ويجب توحيدها في تنفيذ Backend.

---

# 23. خريطة تغطية المتطلبات الوظيفية

| المتطلب | الوظائف | API المقابل |
|---|---|---|
| FR-001–FR-005 | الدخول والجلسات والمستخدمون | `/auth`, `/users`, `/roles` |
| FR-006–FR-011 | المنتجات والتصنيفات | `/products`, `/categories`, `/inventory` |
| FR-012–FR-016 | العملاء والموردون والأرصدة | `/customers`, `/suppliers`, `/statement` |
| FR-017–FR-025 | البيع | `/sales-invoices` |
| FR-026–FR-032 | الشراء | `/purchase-invoices` |
| FR-033–FR-041 | الصندوق والعمليات المالية | `/cashboxes`, `/payment-transactions`, `/expenses`, `/other-incomes` |
| FR-042–FR-048 | الديون والتحصيل والسداد | `/customers`, `/suppliers`, `/payment-transactions` |
| FR-049–FR-054 | المخزون والتسويات | `/inventory`, `/stock-adjustments` |
| FR-055–FR-058 | المرتجعات الاختيارية | `/sales-returns`, `/purchase-returns` |
| FR-059 | لوحة المعلومات | `/dashboard` |
| FR-060–FR-068 | التقارير والطباعة | `/reports` |
| FR-069–FR-071 | النسخ والاستعادة | `/backups` |
| FR-072–FR-073 | الإعدادات والترقيم | `/settings` |

---

# 24. متطلبات غير وظيفية تؤثر في API

| المعرف | المتطلب | أثره على API |
|---|---|---|
| NFR-01 | الأداء | تحسين الاستعلامات والتقارير وتجنب تحميل كل السجلات دفعة واحدة |
| NFR-02 | الأمان | حماية الجلسات وكلمات المرور والتحقق من الصلاحيات |
| NFR-03 | حماية البيانات | حصر البيانات في المتجر والمستخدم المخول |
| NFR-04 | سلامة السجلات | منع تكرار العمليات والاحتفاظ بالحركات الأصلية |
| NFR-05 | النسخ والاستعادة | واجهات واضحة ونتيجة حقيقية للعملية |
| NFR-06 | سهولة الاستخدام | رسائل خطأ عربية واضحة وقابلة للتصحيح |
| NFR-07 | التوافق | توافق JSON والواجهات مع الأجهزة والمتصفحات المعتمدة |
| NFR-08 | التدقيق | تسجيل الإنشاء والتعديل والاعتماد والإلغاء |
| NFR-09 | الخصوصية | عدم إرسال كلمات المرور أو البيانات السرية |
| NFR-10 | دقة الحساب | تطبيق سياسة تقريب موحدة على كل العمليات |

---

# 25. القرارات المفتوحة قبل تثبيت العقود

| المعرف | القرار | الأجزاء المتأثرة في API |
|---|---|---|
| DEC-001 | العملة المعتمدة | إعدادات المتجر، الفواتير، الصندوق، التقارير |
| DEC-002 | دقة المبالغ والكميات | جميع حقول المبالغ والأسعار والكميات |
| DEC-003 | طريقة التقريب | حساب البنود والفواتير والتقارير |
| DEC-004 | سياسة الخصومات | طلبات البيع والشراء وحدود الخصم |
| DEC-005 | الرصيد الدائن والدفع الزائد | القبض والدفع والتخصيص |
| DEC-006 | الرصيد الافتتاحي للصندوق | إعداد الصندوق والرصيد |
| DEC-007 | طرق القبض والدفع | `paymentMethod` |
| DEC-008 | ترقيم الفواتير | إنشاء الفواتير والإعدادات |
| DEC-009 | تعديل الفواتير المعتمدة | تعديل الفاتورة والإلغاء والعكس |
| DEC-010 | إلغاء الفواتير | عمليات العكس وآثارها |
| DEC-011 | البيع دون عميل مسجل | `customerId` في فاتورة البيع |
| DEC-012 | المخزون السالب | اعتماد البيع والتسويات |
| DEC-013 | الرصيد الافتتاحي للمخزون | إنشاء المنتجات والرصيد الافتتاحي |
| DEC-014 | طريقة تقييم تكلفة المخزون | تقارير المخزون والتكلفة |
| DEC-015 | حد إعادة الطلب الافتراضي | المنتجات والإعدادات |
| DEC-016 | وحدات القياس والكميات الكسرية | المنتجات وبنود الفواتير |
| DEC-017 | المرتجعات | جميع مسارات المرتجعات |
| DEC-018 | عدد المستخدمين | الأداء والجلسات |
| DEC-019 | مصفوفة الصلاحيات | جميع المسارات المحمية |
| DEC-020 | مدة الجلسة | المصادقة |
| DEC-021 | طريقة تشغيل النظام | المصادقة والنسخ ومسارات الملفات |
| DEC-024 | سياسة النسخ الاحتياطي | النسخ والاستعادة |
| DEC-025 | الاحتفاظ بالسجلات | سجل التدقيق |
| DEC-026 | المنطقة الزمنية | جميع حقول الوقت |
| DEC-030 | المتطلبات التنظيمية المحلية | إعدادات وفواتير وتقارير عند الحاجة |

لا يجوز لفريق Frontend أو Backend اتخاذ قرار منفرد بشأن هذه النقاط ثم اعتباره قاعدة عمل معتمدة.

---

# 26. معايير قبول عقود API

تعتبر العقود جاهزة للتنفيذ بعد التحقق من الآتي:

- [ ] توحيد عنوان API الأساسي والإصدار.
- [ ] اعتماد أسماء المسارات والحقول.
- [ ] اعتماد بنية الاستجابة الناجحة والفاشلة.
- [ ] اعتماد رموز HTTP وأخطاء الأعمال.
- [ ] تحديد آلية المصادقة والجلسات.
- [ ] اعتماد مصفوفة الصلاحيات.
- [ ] تطابق عقود الفواتير مع مخطط قاعدة البيانات.
- [ ] اعتماد تدفق البيع وتأثيره على المخزون والصندوق والعميل.
- [ ] اعتماد تدفق الشراء وتأثيره على المخزون والصندوق والمورد.
- [ ] اعتماد تدفق القبض والدفع والتخصيص.
- [ ] منع تكرار الحركات عند إعادة إرسال الطلبات.
- [ ] اعتماد سياسة الإلغاء والعكس.
- [ ] اعتماد سياسة المبالغ والتقريب والخصومات.
- [ ] تحديد سلوك الرصيد الافتتاحي.
- [ ] اعتماد قرار المرتجعات أو استبعادها من الإصدار الأول.
- [ ] تحديد آلية النسخ والاستعادة حسب بيئة التشغيل.
- [ ] إعداد اختبارات API للحالات الصحيحة والخاطئة والمتزامنة.
- [ ] التأكد من أن التقارير تطابق السجلات التفصيلية.

---

# 27. مسؤوليات الاعتماد والتكامل

| المسؤول | المسؤولية |
|---|---|
| مالك النظام | اعتماد قواعد العمل والقرارات المالية والتشغيلية |
| قائد المشروع / Integration Owner | اعتماد عقود API المشتركة وتوافقها مع قاعدة البيانات وتدفقات العمل |
| Backend Developer | تنفيذ العقود والتحقق من البيانات والصلاحيات والمعاملات |
| Frontend Developer | استهلاك العقود وعدم اختراع سلوك أو حقول غير متفق عليها |
| Database Developer | ضمان تطابق العمليات مع القيود والعلاقات والفهارس |
| QA | اختبار العقود وتدفقات التكامل والحالات السلبية |

## 27.1 قاعدة إدارة التغيير

أي تعديل على عقد API يجب أن يتضمن:

1. اسم المسار المتأثر.
2. الحقول المضافة أو المحذوفة أو المعدلة.
3. سبب التغيير.
4. أثره على قاعدة البيانات.
5. أثره على Frontend.
6. أثره على الاختبارات.
7. موافقة مسؤول التكامل قبل دمج التغيير.

---

# 28. اعتماد الوثيقة

| جهة الاعتماد | الحالة |
|---|---|
| مالك النظام | بانتظار اعتماد قواعد العمل والقرارات المفتوحة |
| قائد المشروع / Integration Owner | بانتظار مراجعة واعتماد عقود التكامل |
| Backend Developer | بانتظار مراجعة قابلية التنفيذ |
| Frontend Developer | بانتظار مراجعة الاستهلاك |
| QA | بانتظار إعداد خطة اختبارات العقود |

**هذه الوثيقة مسودة تكامل وليست تصريحاً بتثبيت القرارات المفتوحة. يبدأ التنفيذ النهائي بعد اعتماد العقود المشتركة والقرارات التي تؤثر فيها.**


سأبني API_CONTRACTS.md بحيث يكون العقد المشترك بين فريق الـ Backend والـ Frontend، ومتوافقاً مع DATABASE_SCHEMA.md ومتطلبات الإصدار الأول FR-001–FR-073.

سأعتمد REST API بصيغة JSON كاقتراح تكاملي مناسب لبيئة .NET Framework، مع توحيد أسماء المسارات والاستجابات والأخطاء، وتحديد صلاحيات العمليات وآثارها المالية والمخزنية. وسأميز بوضوح بين الوظائف المطلوبة والقرارات التي لا تزال تنتظر اعتماد مالك النظام، خصوصاً المصادقة، والخصومات، والتقريب، والمرتجعات.سأبني API_CONTRACTS.md بحيث يكون العقد المشترك بين فريق الـ Backend والـ Frontend، ومتوافقاً مع DATABASE_SCHEMA.md ومتطلبات الإصدار الأول FR-001–FR-073.

سأعتمد REST API بصيغة JSON كاقتراح تكاملي مناسب لبيئة .NET Framework، مع توحيد أسماء المسارات والاستجابات والأخطاء، وتحديد صلاحيات العمليات وآثارها المالية والمخزنية. وسأميز بوضوح بين الوظائف المطلوبة والقرارات التي لا تزال تنتظر اعتماد مالك النظام، خصوصاً المصادقة، والخصومات، والتقريب، والمرتجعات.


---

**نهاية الوثيقة — API_CONTRACTS.md**