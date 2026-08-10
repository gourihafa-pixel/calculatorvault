# 📦 نشر التطبيق كملف AAB على Google Play

## لماذا AAB وليس APK؟

| الخاصية | APK | AAB |
|---|---|---|
| مقبول مباشرة في Google Play | ❌ لا (مُمنوع منذ 2021) | ✅ نعم |
| حجم التحميل للمستخدم | كبير | **أصغر 30-50%** (Google تختار موارد الهاتف) |
| يدعم Dynamic Delivery | ❌ | ✅ تلقائيًا |
| التوقيع الرقمي مطلوب؟ | اختياري | **مطلوب** للـ release |

يقبل Google Play Console **فقط** ملفات `.aab` مُوقّعة للـ release.

---

## 🔑 الخطة الكاملة (3 خطوات من الجوال)

### الخطوة 1 — إنشاء Keystore (مفتاح التوقيع)

**من Termux** (أسهل طريقة، Termux متوفر مجانًا على Play):

```bash
pkg update -y
pkg install openjdk-17 -y

cd /sdcard/Download
keytool -genkey -v \
  -keystore calcvault.jks \
  -keyalg RSA \
  -keysize 2048 \
  -validity 10000 \
  -alias calculavault
```

سيُسألك عن:
- **كلمة مرور Keystore** (مثلاً `MyStrongP@ss2026`) — **احفظها في مكان آمن**
- الاسم، المنظمة، المدينة، البلد — كلها اختيارية، اكتب أي شيء
- **كلمة مرور المفتاح** — اضغط Enter لاستخدام نفس كلمة الـ Keystore

عند الانتهاء ستجد ملف **`calcvault.jks`** في مجلد Downloads.

**⚠️ مهم:** هذا الملف هو حياتك للتحديثات المستقبلية — لو ضاع لا تستطيع تحديث التطبيق أبداً على Google Play! احفظه على Google Drive مشفّراً.

### الخطوة 2 — رفع المفتاح إلى GitHub كـ Secret

**2.1** تحويل jks إلى Base64 (نص عادي يمكن حفظه في Secret):
```bash
base64 -w 0 calcvault.jks > calcvault_base64.txt
# افتح calcvault_base64.txt وانسخ محتواه كاملاً (سيكون طويلاً)
```

**2.2** اذهب لصفحتك في GitHub → **Settings** (ليس صفحة المستودع بل حسابك، أو من داخل المستودع اضغط **Settings**)

**2.3** في اليسار اختر **Secrets and variables → Actions**

**2.4** اضغط **New repository secret** ثم أضف **4 أسرار** التالية بالترتيب:

| الاسم (Name) | القيمة (Secret) |
|---|---|
| `KEYSTORE_BASE64` | النص المنسوخ من `calcvault_base64.txt` (سلسلة طويلة) |
| `KEYSTORE_PASSWORD` | كلمة مرور الـ Keystore التي كتبتها |
| `KEY_ALIAS` | `calculavault` (أو أي alias كتبته) |
| `KEY_PASSWORD` | نفس كلمة مرور الـ Keystore (أو المختلفة إن كتبت كلمة أخرى) |

اضغط **Add secret** لكل واحد.

### الخطوة 3 — أعد البناء وحمّل AAB

1. اضغط على ملف **`app/build.gradle`** في GitHub
2. اضغط أيقونة القلم ✏️ → الصق المحتوى الجديد (انظر الأسفل)
3. اضغط **Commit changes**
4. اضغط ملف **`.github/workflows/build.yml`** → قلم ✏️ → الصق المحتوى الجديد
5. اضغط **Commit changes**
6. انتظر — GitHub سيُعيد تشغيل الـ workflow تلاقائيًا
7. ادخل **Actions** → اضغط الـ run الأخضر
8. **Artifacts** بالأسفل → اضغط **`CalculatorVault-signed-xxxxx`** → حمّل ZIP
9. بداخل ZIP ستجد ملف **`app-release.aab`** ← هذا هو ما ترفعه على Google Play

### الخطوة 4 — الرفع إلى Google Play Console

1. ادخل https://play.google.com/console
2. **All apps → Create app**
3. املأ استمارة: اسم التطبيق، اللغة، الفئة، إلخ
4. بعد الإنشاء، في القائمة اليسرى اختر **Release → Production → Create new release**
5. اضغط **Upload** واسحب ملف `app-release.aab`
6. املأ استمارات المحتوى (التقييم العمري، الوصف، لقطات الشاشة)
7. اضغط **Review release → Start rollout to Production**

عادةً Google تراجع طلبك خلال **3-7 أيام عمل** قبل النشر.
