# 🖥 دليل البناء على الحاسوب + رفع Google Play

المشروع الحالي **متوافق 100% مع Android Studio** ويولّد ملف `.aab` مُوقَّع للنشر على Google Play مباشرة.

## 📂 محتويات المشروع بعد فك الضغط

```
CalculatorVault/
├── app/                          ← وحدة التطبيق
│   ├── build.gradle              (يقرأ keystore.properties تلقائياً)
│   ├── proguard-rules.pro
│   └── src/main/
│       ├── AndroidManifest.xml
│       ├── java/com/calcvault/app/    (9 ملفات Kotlin)
│       └── res/                       (layouts, drawables, mipmaps, xml, values)
├── gradlew                       ← سكربت Gradle لـ Linux / macOS
├── gradlew.bat                   ← سكربت Gradle لـ Windows
├── gradle/wrapper/
│   ├── gradle-wrapper.jar        (المطلوب لتشغيل gradlew)
│   └── gradle-wrapper.properties (Gradle 8.2 + Java 17)
├── build.gradle                  (المستوى الأعلى)
├── settings.gradle
├── gradle.properties
├── local.properties.example      ← انسخه وعدّل مسار SDK
├── keystore.properties.example   ← انسخه وأكمل بيانات المفتاح
├── .gitignore                    (يحمي jks/keystore من الرفع بالخطأ)
├── .github/workflows/build.yml   (للـ CI فقط — يمكنك تجاهله)
├── README.md
└── PLAY_STORE_GUIDE.md           (تفاصيل إضافية للنشر)
```

---

## ⚙️ المتطلبات الأساسية (ثبّتها قبل أي شيء)

| الأداة | الإصدار | الرابط |
|---|---|---|
| **JDK** | **17** (مطلوب لـ AGP 8.x) | https://adoptium.net/temurin/releases/?version=17 |
| **Android Studio** | Hedgehog (2023.1.1) أو أحدث — اختياري | https://developer.android.com/studio |
| **Android SDK** | Platform 34 + Build Tools 34.0.0 | يُثبَّت تلقائيًا عبر Android Studio |
| **Gradle** | 8.2 | يأتي مع المشروع (gradlew)، لا يطلب تثبيت منفصل |

> 💡 **الخيار الأسهل:** ثبّت [Android Studio فقط](https://developer.android.com/studio) ولا تفعل شيئًا آخر — سيثبت JDK 17 وAndroid SDK تلقائيًا ويُولّد `local.properties` وحده.

### التحقق من JDK
```bash
java -version    # يجب أن يعرض 17.x.x
```

### إذا عندك عدة JDKs على Windows
```powershell
setx JAVA_HOME "C:\Program Files\Eclipse Adoptium\jdk-17.0.X.X-hotspot"
setx PATH "%JAVA_HOME%\bin;%PATH%"
```

### إذا عندك عدة JDKs على macOS / Linux
```bash
# macOS (Homebrew)
brew install temurin@17
export JAVA_HOME=$(/usr/libexec/java_home -v 17)

# Linux (Ubuntu / Debian)
sudo apt install -y openjdk-17-jdk
export JAVA_HOME=/usr/lib/jvm/java-17-openjdk-amd64
```

---

## 🚀 الطريقة اﻷسهل: افتح المشروع في Android Studio

1. افتح **Android Studio** → **File → Open** → اختر مجلد **CalculatorVault** المستخرج
2. في أول مرة، يعرض شريط بالأسفل: **"Gradle Sync"** → انتظر حتى ينتهي (5-15 دقيقة، يحمّل Gradle + Android SDK)
3. اذهب إلى القائمة الجانبية اليمنى → **Gradle** → expand `CalculatorVault > app > Tasks > build`
4. اضغط مرتين على **`assembleDebug`** → يبني ملف APK في المجلد جاهز للتثبيت على جوالك

---

## 🏗️ بناء Signed AAB للنشر على Google Play (الطريقة المطلوبة)

### الخطوة 1 — إنشاء مفتاح التوقيع (Keystore)

تطبيق Calculator Vault الموقّع منذ النشر الأول. **حم المفتاح للأبد** لأن Google Play يفرض استخدام **نفس المفتاح** في كل التحديثات.

افتح **Terminal** (macOS/Linux) أو **PowerShell** (Windows) داخل مجلد المشروع ونفّذ:

**macOS / Linux:**
```bash
cd /path/to/CalculatorVault
keytool -genkey -v \
  -keystore calcvault.jks \
  -keyalg RSA -keysize 2048 \
  -validity 10000 \
  -alias calculavault
```

**Windows (PowerShell أو CMD):**
```powershell
cd C:\path\to\CalculatorVault
keytool -genkey -v -keystore calcvault.jks -keyalg RSA -keysize 2048 -validity 10000 -alias calculavault
```

سيطلب منك **كلمة مرور قوية** (مثلاً `MyStr0ng@Vault2026`) — استخدم نفس كلمة المرور للمفتاح لاحقًا أو كلمة مختلفة.
سيطلب بيانات (الاسم، الشركة، المدينة، البلد) — اضغط Enter لتركها افتراضية أو اكتب أي شيء.

عند الانتهاء، ستجد ملف **`calcvault.jks`** في مجلد المشروع.

### الخطوة 2 — إنشاء `keystore.properties`

في مجلد المشروع:
```bash
cp keystore.properties.example keystore.properties
```

افتح **`keystore.properties`** بأي محرر نصوص (Notepad, VS Code...) وعدّل القيم:
```
storeFile=calcvault.jks
storePassword=MyStr0ng@Vault2026       ← كلمة الـ Keystore
keyAlias=calculavault
keyPassword=MyStr0ng@Vault2026         ← كلمة المفتاح (ممكن نفس السابقة)
```

احفظ الملف. هذا الملف يُستخدم محليًا فقط ولا يُرفع إلى Git (محمي في `.gitignore`).

### الخطوة 3 — بناء Signed AAB

**الطريقة A: من Android Studio:**
1. اضغط القائمة **Build → Generate Signed Bundle / APK...**
2. اختر **Android App Bundle** → اضغط **Next**
3. اضغط **Create new...** أو اختر `calcvault.jks` إن كنت أنشأته يدويًا
4. اختر **release** build variant → اضغط **Create**
5. يظهر المسار: `app/release/app-release.aab`

**الطريقة B: من سطر الأوامر (Terminal / PowerShell):**
```bash
./gradlew :app:bundleRelease                  # Linux / macOS
gradlew.bat :app:bundleRelease                # Windows
```

النتيجة النهائية في:
```
app/build/outputs/bundle/release/app-release.aab
```

هذا الملف **هو الذي ترفعه على Google Play**.

### الخطوة 4 — اختبار AAB قبل النشر (اختياري لكن مُستحسن)

```bash
# يثبت bundletool ويُولّد مجموعة ملفات APK محلية (للاختبار على جهازك)
./gradlew :app:bundleRelease && \
  java -jar bundletool.jar build-apks \
  --bundle=app/build/outputs/bundle/release/app-release.aab \
  --output=app.apks \
  --connected-device
```

أو استخدم [`bundletool`](https://developer.android.com/studio/command-line/bundletool) لرؤية محتوى AAB:
```bash
unzip -l app/build/outputs/bundle/release/app-release.aab
```

---

## 📤 رفع AAB على Google Play Console

1. ادخل https://play.google.com/console من Chrome
2. **All apps → Create app** → اكتب اسم التطبيق (**Calculator Vault**) → **Create**
3. القائمة اليسرى:
   - **Release → Production → Create new release**
   - اضغط **Upload** → اختر `app-release.aab`
4. املأ الاستمارات:
   - **App content**: التصنيف العمري
   - **Store listing**: اسم التطبيق، الوصف، أيقونة (512×512)، لقطات شاشة (شاشة جوال على الأقل 2)
   - **Target audience & content**: Libre
5. **Pricing & distribution**: مجاني، اختر الدول
6. **Review release → Start rollout to Production**

⏳ مراجعات Google عادة خلال **3-7 أيام عمل**.

---

## 🔁 تحديث التطبيق لاحقاً (v1.1, v1.2, ...)

في `app/build.gradle` ارفع الرقمين قبل إعادة البناء:
```groovy
versionCode 2    // رقم صحيح تصاعدي فقط — يجب أن يكون أكبر من السابق
versionName "1.1" // سلسلة نصية مرئية للمستخدم
```

ثم:
```bash
./gradlew :app:bundleRelease
```

ارفع `app-release.aab` الجديد في **Play Console → Release → Production → Create new release** — الجسم يحتاج نفس المفتاح `calcvault.jks`.

---

## 🛠 حل المشاكل الشائعة

| المشكلة | السبب | الحل |
|---|---|---|
| `Unsupported class file major version 61` | JDK قديم | ثبّت JDK 17 وثبّت `JAVA_HOME` |
| `SDK location not found` | ملف `local.properties` ناقص | افتح Android Studio مرة واحدة (يولّد الملف) أو انسخ `local.properties.example` |
| `Keystore was tampered with` | كلمة مرور خاطئة | أعد كتابة `storePassword` و `keyPassword` في `keystore.properties` |
| `Execution failed for task ':app:bundleRelease'` | أي خطأ آخر | شغّل `./gradlew :app:bundleRelease --stacktrace` وشارك المخرج |
| `AAPT: error: resource ... not found` | خطأ في موارد | من Android Studio اضغط **Build → Clean Project** ثم **Rebuild** |
| Gradle التحميل بطيء | اتصال محدود | استخدم VPN أو فعّل **Gradle Daemon** `./gradlew --status` |

---

## ❓ أسئلة شائعة

**س: هل يمكنني بناء APK للتثبيت المباشر بدل AAB؟**
نعم: `./gradlew :app:assembleRelease` ينتج ملف في `app/build/outputs/apk/release/`. ملاحظة: Google Play **لا يقبل** الـ APK في production track منذ أغسطس 2021.

**س: نسيت كلمة المرور، ماذا أفعل؟**
لا توجد طريقة لاسترجاعها. يجب إنشاء keystore جديد ونشر التطبيق **كمشروع جديد** على Google Play (ستحتاج ربما تغيير applicationId في `app/build.gradle` كـ `com.calcvault.app2`).

**س: هل يعمل على Windows؟**
نعم. استخدم `gradlew.bat` بدل `./gradlew`. باقي الخطوات متطابقة.

**س: هل أحتاج إلى Android Studio؟**
لا، يمكن البناء بـ Terminal فقط بعد تثبيت JDK 17 و Android SDK وتحديد مساره في `local.properties`. لكن Android Studio يبسّط كل شيء.

**س: كيف أحذف keystore من Git بالخطأ؟**
1. `git filter-branch` أو `git rm --cached calcvault.jks`
2. `git commit --amend`
3. **غيّر كلمة المرور فورًا** (لكن قد لا يكون كافيًا — اعتبر المفتاح مخترق)

---

## ✅ قائمة المراجعة النهائية قبل النشر على Google Play

- [x] كل أكواد `MainActivity.kt` الثلاثة بصيغة Kotlin 1.9
- [x] لا توجد استيراد أي مكتبة مشبوهة
- [x] لا تطلب صلاحيات غير ضرورية
- [x] جميع الـ icons mipmap/Android Adaptive
- [x] ProGuard files افتراضية (لا تعقيد)
- [ ] Keystore محفوظ بنسخ احتياطية في موضعين مختلفين على الأقل
- [ ] Description مختصر بـ 80 حرف ولقطات شاشة عالية الجودة مرفوعة
- [ ] التصنيف العمري مُحدد
- [ ] `versionCode` و `versionName` عن أول نشر (1 / "1.0")
