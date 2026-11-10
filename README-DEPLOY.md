# نشر الموقع على Cloudflare (Worker + ملفات ثابتة)

## سبب الفشل

سجل البناء يقول:

```
Executing user deploy command: npx wrangler deploy
? We have identified a `functions` directory ... Is this correct?
🤖 Using fallback value in non-interactive context: no
```

`npx wrangler deploy` هو أمر نشر **Worker**، ومشروعك كان مرتباً كـ **Pages**
(`index.html` + `functions/api/generate.js`). أمر Worker لا يقرأ مجلد `functions/`
أبداً، وبدون ملف إعدادات لا يجد نقطة دخول، فيتوقف البناء عند هذه النقطة.

## الحل: تحويل بسيط إلى Worker + assets

بنية المشروع الجديدة:

```
wrangler.jsonc      إعدادات Worker (اسم المشروع + الملفات الثابتة)
public/index.html   الموقع كما هو، بدون أي تعديل
src/index.js        نفس كود Groq، لكن كـ Worker بدل Pages Function
```

- `public/` تُنشر كملفات ثابتة تلقائياً (الصفحة الرئيسية = `index.html`).
- `src/index.js` يستقبل `POST /api/generate` فقط، وباقي الطلبات تروح للملفات الثابتة.
- مفتاح `GROQ_API_KEY` يبقى مخفي في إعدادات Cloudflare ولا يظهر في المتصفح.

## خطوات النشر

1. في مستودع GitHub، **احذف** `index.html` القديم من الجذر و**احذف** مجلد `functions/`
   (لأن Workers لا يستخدمه، وبقاؤه يربك الأدوات).
2. ارفع الملفات الثلاثة الجديدة بنفس البنية:
   `wrangler.jsonc` و `src/index.js` و `public/index.html`.
3. تأكد أن `name` في `wrangler.jsonc` مطابق تماماً لاسم الـ Worker
   الموجود في لوحة Cloudflare. إن كان الاسم مختلفاً، غيّره في الملف.
4. في لوحة Cloudflare: Worker ← Settings ← Variables and Secrets ← أضف
   `GROQ_API_KEY` كـ **Secret** (وارفعها مشفّرة).
   اختياري: أضف `GROQ_MODEL` لو أردت تغيير الموديل.
5. أعد النشر (Retry deployment). الأمر `npx wrangler deploy` سينجح هذه المرة.

## إن أردت البقاء على Pages

لا حاجة لتغيير الكود، فقط غيّر أمر النشر في اللوحة إلى:

```
npx wrangler pages deploy . --project-name <اسم-مشروع-Pages>
```

مع بقاء `index.html` و `functions/` كما هما في جذر المستودع.
