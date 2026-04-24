# المعمارية (Architecture)

> الهدف: وصف المعمارية المقترحة للنظام بشكل واضح يصلح كمرجع للفريق التقني وللعميل.

## مكونات النظام (High-level)

- **Customer Web/App**: تصفح المنيو، تخصيص الأصناف، الطلب والدفع، متابعة الحالة.
- **Restaurant Web/App**: استقبال الطلبات، التأكيد، الطباعة، إدارة الحالة، إرسال للدليفري.
- **Delivery App (Premium)**: استلام المهام، GPS، تأكيد الالتقاط/التسليم.
- **Backend API**: المصدر الرئيسي للبيانات (Orders, Menu, Offers, Users).
- **Real-time Layer**: بث التحديثات اللحظية.
- **Database**: تخزين البيانات.
- **Integrations**: دفع، خرائط، إشعارات، طابعات.

```mermaid
flowchart TB
  subgraph Clients[Clients]
    CW[Customer Web/App]
    RW[Restaurant Web/App]
    DA[Delivery App (Premium)]
  end

  subgraph Backend[Backend Services]
    API[Backend API]
    RT[Real-time Gateway]
    JOBS[Background Jobs]
  end

  subgraph Data[Data]
    DB[(Database)]
    OBJ[(Object Storage: Images)]
  end

  subgraph Integrations[Integrations]
    PAY[Payment Provider]
    MAPS[Maps/GPS]
    PUSH[Push Notifications]
    PRN[Thermal Printer]
  end

  CW --> API
  RW --> API
  DA --> API

  API <--> DB
  API --> OBJ

  API <--> RT
  RT --> CW
  RT --> RW
  RT --> DA

  JOBS --> API
  API --> PAY
  DA --> MAPS
  API --> PUSH
  RW --> PRN
```

---

## تدفق الطلب (مختصر)

1. العميل ينشئ طلب (كاش/إلكتروني حسب الباقة)
2. المطعم يستقبل الطلب لحظياً ويؤكده
3. حالة الطلب تتحدث للعميل (جاري التحضير)
4. عند الجاهزية: Send to Delivery
5. المندوب يستلم المهمة ويؤكد الالتقاط ثم التسليم

---

## اختيار أسلوب الـ Real-time

خيارات تنفيذ شائعة:

- **WebSockets**: تحكم كامل + مناسب لتحديثات الطلب
- **SSE**: بسيط لتحديثات من السيرفر للعميل
- **Firebase Realtime/Firestore**: أسرع وقت للتسليم مع تكلفة تشغيل مختلفة

المعيار: (وقت التطوير) vs (التحكم) vs (تكلفة التشغيل).

---

## طبقة Jobs (مهام خلفية)

أمثلة مهام:

- إرسال إشعارات (SMS/Push)
- معالجة Webhooks الدفع
- تقارير دورية (Premium)
- تنظيف بيانات GPS القديمة

---

## غير وظيفي (NFR)

- **الاعتمادية**: عدم فقد الطلبات حتى مع انقطاع الإنترنت
- **الأداء**: تحميل منيو سريع + تحديثات لحظية
- **القابلية للتوسع**: دعم عدد كبير من الطلبات والفروع
- **المراقبة**: Logs + Metrics + Alerts

---

## قرارات تصميم مهمة

- الأسعار تُحسب على السيرفر (تجنب التلاعب).
- حفظ Snapshots للأصناف والإضافات داخل الطلب وقت الشراء.
- فصل صلاحيات الموظفين (Premium) عبر RBAC.
