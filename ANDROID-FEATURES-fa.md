# بروزرسانی کلاینت اندروید — قابلیت‌های جدید

همه‌ی تغییرات فقط در پروژه‌ی اندروید است؛ هیچ تغییری در بک‌اند لازم نیست.

## ۱) قابلیت‌های بک‌اند که حالا در اندروید هست
| قابلیت | روت بک‌اند | کجا در اپ |
|---|---|---|
| لیست دنبال‌کننده‌ها / دنبال‌شونده‌ها | `GET /users/{id}/followers`, `/following` | روی اعداد پروفایل بزن (`PeopleScreen`) |
| مسدودسازی / رفع مسدودی | `POST/DELETE /users/{id}/block` | منوی ⋯ بالای پروفایل دیگران |
| لیست مسدودشده‌ها | `GET /users/me/blocked` | تنظیمات ← حساب‌های مسدودشده |
| گزارش کاربر | `POST /users/{id}/report` | منوی ⋯ پروفایل |
| ویرایش / حذف / گزارش نظر | `PATCH/DELETE/POST /pins/{id}/comments/{cid}` | دکمه‌ی ⋯ کنار هر نظر |
| حذف پین از برد / انتقال به برد دیگر | `DELETE /boards/{id}/pins/{pid}`, `POST .../move` | دکمه‌ی ⋯ روی پین‌ها در برد خودت |
| تنظیمات اعلان پوش سمت سرور | `GET/PUT /notifications/preferences` | تنظیمات ← اعلان‌های پوش |
| ثبت دستگاه برای پوش (FCM) | `POST /devices/` | خودکار بعد از ورود (نیاز به تنظیم، پایین) |

کد این بخش: `data/extras/` (DTO + `ExtrasRepository`)، `ui/ExtrasViewModels.kt` و صفحه‌های `PeopleScreen`، `BlockedUsersScreen`، `PushPrefsScreen`.

### فعال‌کردن پوش (اختیاری)
چهار مقدار `google-services.json` را در `gradle.properties` بگذار:
```
PINBOM_FCM_API_KEY=...
PINBOM_FCM_APP_ID=...
PINBOM_FCM_PROJECT_ID=...
PINBOM_FCM_SENDER_ID=...
```
بدون آن‌ها اپ مثل قبل بیلد می‌شود و فقط ثبت دستگاه انجام نمی‌شود. توجه: نمایش پیام‌های *data-only* در پس‌زمینه به یک `FirebaseMessagingService` نیاز دارد که اضافه نکردم؛ پیام‌های notification-payload را خود سیستم نشان می‌دهد.

## ۲) C++ / XML / Java
- **C++** (`app/src/main/cpp`): موتور فیزیک پارتیکل (`particle_core.h`) + پل JNI + `CMakeLists.txt`. شش افکت: قلب، جرقه، کانفتی، شراره، غبار، حلقه.
- **Java** (`fx/`): `NativeParticles.java` (پل JNI با fallback امن) و `FxHaptics.java` (لرزش‌های هماهنگ با افکت‌ها).
- **XML**: `res/drawable/avd_*.xml` (قلب، بوکمارک، زنگ، تیک)، `res/animator/`، `res/interpolator/`.
- اگر کتابخانه‌ی native لود نشود، نسخه‌ی Kotlin همان فیزیک اجرا می‌شود (کرش نمی‌کند).

## ۳) انیمیشن‌ها
- لایک پین: قلب‌ها + حلقه‌ی موجی + لرزش، ذخیره: جرقه + تیک (`ParticleStage`).
- صفحه‌ی ورود: شراره‌های شناور پس‌زمینه.
- کارت پین‌ها: خم‌شدن ۳بعدی کوچک به سمت انگشت (`pressTilt`).
- زنگ انیمیشنی در تنظیمات پوش (AnimatedVectorDrawable).
- همه‌ی افکت‌ها با «حرکت کم» سیستم و کلید «افکت‌های ویژه» اپ خاموش می‌شوند، و وقتی ذره‌ای زنده نیست هیچ هزینه‌ای ندارند.

## بیلد
`./gradlew assembleRelease` — به NDK و CMake 3.22.1 (از SDK Manager) نیاز دارد.
تست‌ها: `ParticleBackendTest`, `ExtrasModelsTest` و `tools/fx-host-test.cpp` (روی کامپیوتر: دستور بالای فایل).
