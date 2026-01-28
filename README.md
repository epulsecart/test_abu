# حزمة تسليم FES v1.0

الهدف: هذه الحزمة هي المصدر الوحيد للحقيقة لتنفيذ نظام تسجيل الإنتاج وفحص الجودة. يجب اشتقاق جميع المتطلبات والتتبّع وواجهات API ونموذج البيانات والواجهات والتجارب والعمليات من SRS والحفاظ على الاتساق هنا.

## خطوات الاستخدام
1) ضع SRS داخل `99_INTAKE` (نص أو PDF) وفق قائمة الاستلام.
2) حلّل SRS إلى متطلبات ذرّية واملأ `02_RTM/RTM.csv`.
3) أصدر معرفات فريدة من `02_RTM/ID_REGISTRY.md`.
4) حدّث المخرجات التابعة:
   - `03_DATA_MODEL/Data_Dictionary.csv`
   - `04_UI_PAGES/Pages_Index.csv`
   - `05_API/API_Inventory.csv` و `05_API/OpenAPI_stub.yaml`
   - `06_SECURITY/RBAC_Matrix.csv`
   - `07_TESTING/UAT_Scenarios.md` و `07_TESTING/Acceptance_Criteria.md`
   - `08_OPS/Deployment_Runbook.md` و `08_OPS/Config_Parameters.csv`
5) تحقق من التتبّع من كل ReqID إلى UI/API/Data/Test/Ops.

## كيف تعمل المعرفات
جميع معرفات المتطلبات فريدة ولا يعاد استخدامها:
- FR-#### = Functional Requirement
- NFR-#### = Non-Functional Requirement
- UC-#### = Use Case
- ALG-#### = Rule/Algorithm
- API-#### = API Requirement
- P-#### = Page/UI Requirement

قواعد التنسيق:
- أربع خانات رقمية مع أصفار بادئة (مثال: FR-0001).
- تصدر المعرفات في `02_RTM/ID_REGISTRY.md` قبل استخدامها.
- تظهر المعرفات في RTM وجميع المخرجات التابعة.

## كيفية تعبئة RTM
1) فكّك SRS إلى متطلبات ذرّية (كل سطر = مطلب قابل للتحقق).
2) خصص ReqID فريد من السجل.
3) عبّئ كل عمود. استخدم Status=BLOCKED عند نقص بيانات التنفيذ.
4) اربط العناصر التابعة:
   - Entities/Attributes لنموذج البيانات
   - APEXPages لواجهات المستخدم
   - APIAffected لنقاط النهاية
   - RulesAlgorithms للمنطق التجاري
5) اربط حالات الاختبار في TestCaseIDs.

## تعريف الإنجاز (Definition of Done)
يُعد المطلب منجزاً عندما:
- Status = VERIFIED في RTM
- توجد حالات اختبار مرتبطة وتم تمريرها
- تتوفر عناصر UI/API/Data المرتبطة بـ ReqID
- لا توجد أسئلة مفتوحة لنفس ReqID

## قواعد التحكم بالتغيير
- أي تغيير يجب أن ينطلق من تحديث SRS أو طلب تغيير معتمد.
- حدّث SRS ثم RTM ثم المخرجات التابعة.
- لا تحذف المعرفات؛ علّم المتطلب كـ BLOCKED مع توضيح السبب.
- كل تغيير يجب أن يتضمن التاريخ والمالك وسبب التغيير في Notes.
