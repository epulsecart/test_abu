# PROCESS DIAGRAM INPUTS (Fact-Based)

> All bullets cite source file + ReqID or row key. If data is missing it is flagged as OPEN INPUT.

## 1) State Machines to Draw

### OrderItem lifecycle states
- OPEN INPUT: SRS/RTM do not define explicit OrderItem lifecycle states or status enum. (RTM: FR-083, FR-076..FR-081; Data_Dictionary.csv: ORDER_ITEMS, ORDERS STATUS)

### Distribution request/approve/undo states
- Distribution actions include Auto, Manual, and Undo by supervisor. (RTM: FR-013, FR-014, FR-017; SEC-006)
- OPEN INPUT: explicit state names (Requested/Approved/Applied/Undone) are not defined. (RTM: FR-017; Data_Dictionary.csv: DISTRIBUTIONS without status)

### ProductionLog session states
- Start and End timestamps recorded per item/employee/stage. (RTM: FR-025, FR-026; Data_Dictionary.csv: PROD_LOGS.START_AT/END_AT)
- OPEN INPUT: explicit session state enum (IN_PROGRESS/COMPLETED/BLOCKED) is not defined. (RTM: FR-025..FR-027)

### QC states (initial/final/hidden)
- QC types: INITIAL/FINAL/BRANCH; Hidden QC exists as a distinct stage. (Data_Dictionary.csv: QC_CHECKS.QC_TYPE; Stages_List.csv: STG-140_QC_INITIAL, STG-200_QC_FINAL, STG-230_QC_BRANCH, STG-240_QC_HIDDEN)
- QC results: مطابق|إصلاح|مرفوض (Result domain). (Data_Dictionary.csv: QC_CHECKS.RESULT)
- OPEN INPUT: workflow status for QC checks (e.g., Draft/Submitted/Approved) is not defined. (RTM: FR-039..FR-045)

### RepairTicket states
- Repair tickets exist with STATUS and DUE_AT. (Data_Dictionary.csv: REPAIR_TICKETS.STATUS, DUE_AT; RTM: FR-048)
- OPEN INPUT: explicit status values (Open/Assigned/Due/Closed/Re-QC) are not defined. (RTM: FR-048, FR-049)

### PayrollRun states
- Payroll runs exist with STATUS. (Data_Dictionary.csv: PAYROLL_RUNS.STATUS; RTM: FR-064)
- OPEN INPUT: explicit states (Draft/Run/Review/Approved/Exported) are not defined. (RTM: FR-064)

### Notification delivery states
- Delivery states include Sent/Delivered/Read when available. (RTM: FR-054; SRS_TEXT.md section 3.6)
- OPEN INPUT: queued/failed/retry state names are not defined. (RTM: FR-054; Data_Dictionary.csv: NOTIFICATIONS.STATUS)

## 2) Events & Triggers

### Distribution
- Trigger: Auto distribution batch (priority rules). Actor: أمين المستودع. Preconditions: ترتيب حسب DueDate ثم ShipDistance ثم InvoiceNum. Side effects: كتابة DISTRIBUTIONS. (RTM: FR-013; ALG-003)
- Trigger: Manual distribution with reason. Actor: أمين المستودع. Preconditions: سبب مطلوب. Side effects: كتابة DISTRIBUTIONS.REASON + AUDIT_LOG. (RTM: FR-014; SEC-005)
- Trigger: Undo distribution by supervisor. Actor: مدير الإنتاج. Preconditions: صلاحية مشرف + سبب. Side effects: تسجيل Undo في AUDIT_LOG. (RTM: FR-017; SEC-006; SEC-005)
- Failure paths: منع القفز غير المنطقي بين المراحل. (RTM: FR-016)

### Production Logging
- Trigger: دخول عبر Badge أو رقم وظيفي. Actor: عامل تشغيلي. Side effects: تحميل مهارات/مرحلة افتراضية. (RTM: FR-020..FR-022)
- Trigger: Start log. Actor: عامل تشغيلي. Preconditions: المرحلة مخولة + التدريب مكتمل. Side effects: PROD_LOGS.START_AT. (RTM: FR-023; FR-037; FR-025)
- Trigger: End log. Actor: عامل تشغيلي. Preconditions: نهاية بعد بداية. Side effects: PROD_LOGS.END_AT, DURATION_SEC. (RTM: FR-026; FR-027)
- Failure paths: منع تكرار المنتج في نفس المرحلة؛ منع تشغيل نفس الفاتورة في مراحل خياطة مقيدة؛ منع التسجيل لمن لديه مرتجع متأخر. (RTM: FR-029; FR-030; FR-031)

### QC
- Trigger: تسجيل QC (INITIAL/FINAL/BRANCH/HIDDEN). Actor: فاحص جودة أولي/نهائي. Preconditions: نوع QC صحيح. Side effects: QC_CHECKS وQC_DEFECTS وربما REPAIR_TICKETS. (RTM: FR-039..FR-045; Data_Dictionary.csv: QC_CHECKS, QC_DEFECTS)
- Trigger: Hidden QC failure. Actor: فاحص جودة نهائي. Side effects: إنشاء تذكرة إصلاح إلزامية. (RTM: FR-045)
- Failure paths: صور إلزامية عند تسجيل العيب لبعض المعايير. (RTM: FR-047)

### Repair
- Trigger: إنشاء تذكرة إصلاح بعد QC فاشل. Actor: فاحص جودة نهائي. Side effects: REPAIR_TICKETS + إشعار WhatsApp. (RTM: FR-043; FR-048; FR-052)
- Trigger: تجاوز موعد الإصلاح. Actor: النظام/مدير الإنتاج. Preconditions: بعد DUE_AT. Side effects: منع التسجيل إلا باستثناء مدير الإنتاج. (RTM: FR-049; SEC-007)

### Payroll
- Trigger: تشغيل مسير الرواتب. Actor: محاسب. Preconditions: توفر سجلات الإنتاج/الأسعار/NPT. Side effects: إنشاء PAYROLL_RUNS وPAYROLL_LINES. (RTM: FR-059..FR-064; UC-007)

### Notifications (WhatsApp)
- Trigger: تدريب/مرتجع/موعد إصلاح/تحديث محتوى تدريبي. Actor: النظام. Side effects: NOTIFICATIONS + WhatsApp_Templates. (RTM: FR-050..FR-053; WhatsApp_Templates.csv)
- Delivery tracking: Sent/Delivered/Read إذا توفر من المزود. (RTM: FR-054)

### Integrations
- Trigger: Import Orders (Oracle). Actor: نظام المبيعات (Oracle). Preconditions: APIKey + Idempotency 24h. Side effects: ORDERS/ORDER_ITEMS/PRODUCTS. (RTM: FR-076..FR-081; API_Inventory.csv: API-003; SEC-008..SEC-010)
- Trigger: Status Pull. Actor: نظام التفصيل. Preconditions: APIKey. Side effects: قراءة الحركة/ QC/ ETA. (RTM: UC-009; API-001)
- Trigger: Webhook event. Actor: نظام التفصيل. Preconditions: توقيع HMAC صحيح + Idempotency. Side effects: تحديث الحالة + سجل تكامل. (RTM: UC-010; API-002; SEC-010)
- Failure paths: API errors 401/404/409/422/429 حسب API_Inventory. (API_Inventory.csv: API-001..API-004)

## 3) Guard Rules / Constraints

- صلاحية المرحلة: لا يسمح إلا بالمراحل المخولة للموظف. (RTM: FR-023)
- بوابة التدريب: منع التسجيل قبل إكمال التدريب. (RTM: FR-037)
- منع تكرار المنتج في نفس المرحلة. (RTM: FR-029)
- منع توازي نفس الفاتورة في مراحل خياطة مقيدة. (RTM: FR-030; Stages_List.csv: CanRunParallelForSameInvoice)
- منع التسجيل لمن لديه مرتجع متأخر. (RTM: FR-031)
- منع التسجيل بعد موعد الإصلاح إلا باستثناء مدير الإنتاج. (RTM: FR-049; SEC-007)
- فصل الخياط الخارجي عن مراحل خياطة المصنع. (RTM: FR-035; Stages_List.csv: IsInternal/IsExternal/IsSewing)
- قواعد إدراج مراحل المواصفات: غسيل/تطريز/إكسسوار حسب المواصفة. (Spec_to_Stage_Rules.md)
- Idempotency لمدة 24 ساعة لطلبات التكامل. (RTM: SEC-010; Integration_Priority.csv)
- Rate Limit للتكاملات (قيم في API_Inventory/Integration_Priority). (API_Inventory.csv; Integration_Priority.csv)

## 4) Data Touchpoints

### Distribution
- Entities: DISTRIBUTIONS, ORDERS, AUDIT_LOG. (RTM: FR-013, FR-014, FR-017; Data_Dictionary.csv)
- Key attributes: DUE_DATE, SHIP_DISTANCE, INVOICE_NUM, REASON, USER_ID, ACTION. (RTM: FR-013..FR-017)

### Production Logging
- Entities: PROD_LOGS, EMPLOYEES, STAGES, SKILLS. (RTM: FR-020..FR-027; FR-023)
- Key attributes: EMP_ID, STAGE_ID, ITEM_ID, START_AT, END_AT, DURATION_SEC. (Data_Dictionary.csv)

### QC
- Entities: QC_CHECKS, QC_DEFECTS, DEFECTS, REPAIR_TICKETS. (RTM: FR-039..FR-045)
- Key attributes: QC_TYPE, RESULT, PHOTO_URL, DEFECT_ID, RPR_ID. (Data_Dictionary.csv)

### Repair
- Entities: REPAIR_TICKETS, NOTIFICATIONS. (RTM: FR-048, FR-052)
- Key attributes: DUE_AT, STATUS, MSG_ID. (Data_Dictionary.csv)

### Payroll
- Entities: PAYROLL_RUNS, PAYROLL_LINES, RATES, NPT_LOGS. (RTM: FR-059..FR-064; FR-061)
- Key attributes: RUN_ID, EARN_PROD, EARN_PIECE, EARN_NPT, RATE_TYPE, RATE_VALUE. (Data_Dictionary.csv)

### Notifications
- Entities: NOTIFICATIONS, WHATSAPP_TEMPLATES. (RTM: FR-050..FR-054; WhatsApp_Templates.csv)
- Key attributes: STATUS, SENT_AT, TEMPLATE_ID. (Data_Dictionary.csv)

### Integrations
- Entities: ORDERS, ORDER_ITEMS, PRODUCTS, INTEGRATION_LOGS. (RTM: FR-076..FR-081; FR-090)
- Key attributes: INVOICE_NUM, ITEM_SEQ, THOBE_NUM, DUE_DATE, REQUEST_ID, STATUS. (Data_Dictionary.csv)

### BI/Reports
- Entities: BI_KPI_DAILY, REPORTS_CATALOG. (RTM: FR-094, FR-090; Reports_List.csv)
- Key attributes: KPI_CODE, KPI_VALUE, REPORT_ID. (Data_Dictionary.csv)

### Offline Sync (PWA)
- Entities: MOBILE_DEVICES, OFFLINE_QUEUE. (RTM: FR-098; API-004)
- Key attributes: DEVICE_ID, IDEMPOTENCY_KEY, STATUS, QUEUED_AT, SYNC_AT. (Data_Dictionary.csv)

## 5) Swimlanes (Actors)

Actors from RBAC + RTM (use in Miro swimlanes):
- مسؤول النظام: إدارة الماستر/القوالب/العيوب/الجزاءات/التكامل. (RBAC_Matrix.csv)
- أمين المستودع: التوزيع الآلي/اليدوي ومراجعة WIP. (RTM: FR-013..FR-015; RBAC_Matrix.csv)
- مدير الإنتاج: اعتماد/Undo التوزيع، استثناء الإصلاح، لوحة التخطيط. (RTM: FR-017; FR-049; FR-055; RBAC_Matrix.csv)
- عامل تشغيلي: تسجيل الإنتاج. (RTM: FR-020..FR-027; RBAC_Matrix.csv)
- خياط (خارجي): تسجيل إنتاج محدود للمراحل الخارجية فقط. (RTM: FR-035; RBAC_Matrix.csv)
- فاحص جودة أولي/نهائي: تسجيل QC وإنشاء تذاكر إصلاح. (RTM: FR-039..FR-048; RBAC_Matrix.csv)
- مسؤول موارد بشرية / محقق موارد بشرية: تدريب وتقارير أخطاء. (RTM: FR-038; FR-075; RBAC_Matrix.csv)
- محاسب: الأسعار والرواتب وNPT. (RTM: FR-059..FR-064; FR-061; RBAC_Matrix.csv)
- المدير العام: تقارير/لوحات. (RBAC_Matrix.csv)
- مسؤول التكامل: مراقبة التكامل وإعادة المحاولة. (RBAC_Matrix.csv)
- نظام المبيعات (Oracle): استيراد الطلبات. (RTM: FR-076..FR-081)
- نظام التفصيل: Pull status وWebhook. (RTM: UC-009; UC-010)

## 6) Diagram Checklist

- كل متطلبات سير العمل في RTM ممثلة (Distribution/Logging/QC/Repair/Payroll/Notifications/Integrations). (RTM: FR-013..FR-098; UC-001..UC-010)
- كل مسار استثنائي ممثل (منع تدريب، منع تسجيل بعد إصلاح، منع تكرار، Undo). (RTM: FR-037; FR-049; FR-029; FR-017)
- كل إشعار يمثل Trigger محدد. (RTM: FR-050..FR-054; WhatsApp_Templates.csv)
- حالات QC وأنواعها متطابقة مع قائمة المراحل. (Stages_List.csv; Data_Dictionary.csv: QC_CHECKS.QC_TYPE)
- مسارات المواصفات تولد مراحل وفق Spec_to_Stage_Rules. (Spec_to_Stage_Rules.md)
- تكاملات API تؤثر على تغييرات الحالة وممثلة في المخططات. (API_Inventory.csv: API-001..API-004; Integration_Priority.csv)

## OPEN INPUTS (Missing Facts)

1) OrderItem lifecycle states enum (start/end/terminal). (RTM: FR-083; Data_Dictionary.csv: ORDER_ITEMS)
2) Distribution approval state names (Requested/Approved/Applied/Undone). (RTM: FR-017)
3) ProductionLog state enum beyond START/END (e.g., InProgress/Completed/Blocked). (RTM: FR-025..FR-027)
4) QC workflow status enum (Draft/Submitted/Approved). (RTM: FR-039..FR-045)
5) RepairTicket status values (Open/Assigned/Due/Closed/Re-QC). (RTM: FR-048; Data_Dictionary.csv: REPAIR_TICKETS.STATUS)
6) PayrollRun status values (Draft/Run/Review/Approved/Exported). (RTM: FR-064; Data_Dictionary.csv: PAYROLL_RUNS.STATUS)
7) Notification status values beyond Sent/Delivered/Read (Queued/Failed/Retry). (RTM: FR-054; Data_Dictionary.csv: NOTIFICATIONS.STATUS)
