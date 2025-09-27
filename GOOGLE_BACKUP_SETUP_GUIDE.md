# 📊 دليل إعداد النسخ الاحتياطي Google Sheets & Forms
**نظام الحضور الموسيقي - جامعة الزقازيق**

---

## 🎯 **نظرة عامة**

هذا النظام يوفر حلاً احتياطياً تلقائياً لضمان عدم فقدان بيانات الحضور في حالة تعطل قاعدة البيانات الأساسية. يتم حفظ البيانات في Google Sheets و Google Forms كنسخة احتياطية آمنة.

---

## 📋 **الخطوة 1: إعداد Google Cloud Console**

### **1.1 إنشاء مشروع جديد**
1. اذهب إلى [Google Cloud Console](https://console.cloud.google.com)
2. اضغط على "Select a project" → "New Project"
3. اسم المشروع: `attendance-backup-system`
4. اضغط "Create"

### **1.2 تفعيل APIs المطلوبة**
1. في Google Cloud Console، اذهب إلى "APIs & Services" → "Library"
2. ابحث عن وفعّل هذه APIs:
   - **Google Sheets API**
   - **Google Drive API** 
   - **Google Forms API**

### **1.3 إنشاء Service Account**
1. اذهب إلى "IAM & Admin" → "Service Accounts"
2. اضغط "Create Service Account"
3. املأ التفاصيل:
   ```
   Service account name: attendance-backup
   Service account ID: attendance-backup
   Description: Backup service for attendance system
   ```
4. اضغط "Create and Continue"
5. اختر Role: **Editor**
6. اضغط "Continue" → "Done"

### **1.4 إنشاء JSON Key**
1. في قائمة Service Accounts، اضغط على الحساب الذي أنشأته
2. اذهب إلى تبويب "Keys"
3. اضغط "Add Key" → "Create new key"
4. اختر نوع "JSON"
5. اضغط "Create"
6. **احفظ ملف JSON** - ستحتاجه لاحقاً

---

## 📊 **الخطوة 2: إعداد Google Sheets**

### **2.1 إنشاء جدول البيانات**
1. اذهب إلى [Google Sheets](https://sheets.google.com)
2. أنشئ جدول بيانات جديد
3. اسم الجدول: `نظام حضور الطلاب - نسخة احتياطية`

### **2.2 إعداد ورقة "Attendance Records"**
أنشئ ورقة عمل باسم "Attendance Records" مع الأعمدة التالية:

| العمود | العنوان | الوصف |
|--------|---------|-------|
| A | Date | التاريخ |
| B | Time | الوقت |
| C | Student Name | اسم الطالب |
| D | Student Email | بريد الطالب |
| E | Subject | المادة |
| F | Session Type | نوع الجلسة |
| G | Status | الحالة |
| H | QR Code | رمز QR |
| I | Notes | ملاحظات |
| J | Error Type | نوع الخطأ |

### **2.3 إعداد ورقة "Error Log"**
أنشئ ورقة عمل باسم "Error Log" مع الأعمدة التالية:

| العمود | العنوان | الوصف |
|--------|---------|-------|
| A | Timestamp | الوقت |
| B | Error Type | نوع الخطأ |
| C | Error Message | رسالة الخطأ |
| D | Student Data | بيانات الطالب |
| E | Recovery Status | حالة الاستعادة |

### **2.4 مشاركة الجدول مع Service Account**
1. اضغط على "Share" في الجدول
2. أضف email الخاص بـ Service Account (موجود في ملف JSON)
3. أعطه صلاحية "Editor"
4. اضغط "Send"

### **2.5 الحصول على Sheet ID**
من رابط Google Sheet، انسخ الـ ID:
```
https://docs.google.com/spreadsheets/d/[SHEET_ID]/edit
```

---

## 📝 **الخطوة 3: إعداد Google Form**

### **3.1 إنشاء النموذج**
1. اذهب إلى [Google Forms](https://forms.google.com)
2. أنشئ نموذج جديد
3. العنوان: `نموذج حضور الطلاب - النسخة الاحتياطية`

### **3.2 إضافة الحقول المطلوبة**

**حقل 1: اسم الطالب**
- نوع: Short answer
- مطلوب: نعم

**حقل 2: البريد الإلكتروني**
- نوع: Short answer  
- التحقق: Email
- مطلوب: نعم

**حقل 3: المادة**
- نوع: Multiple choice
- الخيارات:
  ```
  Western Rules & Solfege 1
  Western Rules & Solfege 2
  Rhythmic Movement 1
  Western Rules & Solfege 3
  Western Rules & Solfege 4
  Hymn Singing
  Rhythmic Movement 2
  Western Rules & Solfege 5
  Improvisation 1
  Western Rules & Solfege 6
  Improvisation 2
  ```

**حقل 4: نوع الجلسة**
- نوع: Multiple choice
- الخيارات: محاضرة، عملي، امتحان، أخرى

**حقل 5: التاريخ والوقت**
- نوع: Date and time
- مطلوب: نعم

**حقل 6: حالة الحضور**
- نوع: Multiple choice
- الخيارات: حاضر، متأخر، غائب

**حقل 7: ملاحظات**
- نوع: Paragraph

### **3.3 الحصول على Form ID**
من رابط Google Form، انسخ الـ ID:
```
https://docs.google.com/forms/d/[FORM_ID]/edit
```

### **3.4 الحصول على Entry IDs**
1. افتح الـ Form
2. اضغط F12 (Developer Tools)
3. اذهب إلى تبويب "Network"
4. أرسل نموذج تجريبي
5. ابحث عن الطلب المرسل إلى `/formResponse`
6. انسخ الـ `entry.xxxxx` IDs لكل حقل

---

## ⚙️ **الخطوة 4: تكوين النظام**

### **4.1 إضافة متغيرات البيئة**
أضف هذه المتغيرات إلى ملف `.env.local`:

```env
# Google Backup Configuration
ENABLE_GOOGLE_BACKUP=true
GOOGLE_SHEETS_ID=your_sheet_id_here
GOOGLE_FORM_ID=your_form_id_here

# Google Service Account (paste the entire JSON content)
GOOGLE_SERVICE_ACCOUNT_KEY='{"type":"service_account","project_id":"your-project",...}'

# Form Entry IDs (get these from the form inspection)
GOOGLE_FORM_STUDENT_NAME_ENTRY=entry.123456789
GOOGLE_FORM_STUDENT_EMAIL_ENTRY=entry.987654321
GOOGLE_FORM_SUBJECT_ENTRY=entry.456789123
GOOGLE_FORM_SESSION_TYPE_ENTRY=entry.789123456
GOOGLE_FORM_TIMESTAMP_ENTRY=entry.321654987
GOOGLE_FORM_STATUS_ENTRY=entry.654987321
GOOGLE_FORM_NOTES_ENTRY=entry.147258369
```

### **4.2 تثبيت المكتبات المطلوبة**
```bash
npm install googleapis google-auth-library
```

### **4.3 إعادة تشغيل الخادم**
```bash
npm run dev
```

---

## 🧪 **الخطوة 5: اختبار النظام**

### **5.1 اختبار الاتصال**
1. افتح لوحة الإدارة
2. اذهب إلى "إدارة النسخ الاحتياطي"
3. تحقق من حالة الاتصال

### **5.2 اختبار الحفظ الاحتياطي**
1. أوقف اتصال قاعدة البيانات مؤقتاً
2. جرّب تسجيل حضور طالب
3. تحقق من حفظ البيانات في Google Sheets

### **5.3 اختبار المزامنة**
1. أعد تشغيل قاعدة البيانات
2. اضغط "مزامنة النسخ الاحتياطي"
3. تحقق من نقل البيانات إلى قاعدة البيانات الأساسية

---

## 🔧 **استكشاف الأخطاء**

### **خطأ: "Google Backup Service not configured"**
- تحقق من وجود متغيرات البيئة
- تأكد من صحة ملف Service Account JSON
- أعد تشغيل الخادم

### **خطأ: "Permission denied"**
- تأكد من مشاركة Google Sheets مع Service Account
- تحقق من تفعيل APIs المطلوبة في Google Cloud

### **خطأ: "Form submission failed"**
- تحقق من صحة Form ID
- تأكد من صحة Entry IDs
- تحقق من إعدادات النموذج

---

## 📊 **مراقبة النظام**

### **مؤشرات الحالة**
- 🟢 **قاعدة البيانات متصلة + النسخ الاحتياطي جاهز**: النظام يعمل بكامل طاقته
- 🟡 **قاعدة البيانات غير متصلة + النسخ الاحتياطي جاهز**: يتم الحفظ في النسخة الاحتياطية
- 🔴 **كلا النظامين غير متاح**: تحتاج مراجعة فورية

### **التقارير التلقائية**
- سجل يومي بحالة النظام
- تنبيهات عند فشل النسخ الاحتياطي
- تقرير أسبوعي بالإحصائيات

---

## 🚀 **الخطوات التالية**

بعد إتمام الإعداد، أرسل لي:

1. **Google Sheets URL** الخاص بك
2. **Google Form URL** الخاص بك  
3. **محتويات ملف JSON** الخاص بـ Service Account
4. **Entry IDs** من النموذج
5. **قائمة المواد الدراسية** الكاملة

وسأقوم بتكوين النظام ليعمل مع إعداداتك تلقائياً! 🎉