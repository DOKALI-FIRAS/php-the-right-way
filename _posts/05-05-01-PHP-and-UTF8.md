---
title:   العمل بترميز UTF-8
isChild: true
anchor:  php_and_utf8
---

## العمل بترميز UTF-8 {#php_and_utf8_title}

_هذا الفصل تم كتابته من قبل [أليكس كابال](https://alexcabal.com/) في [أفضل ممارسات PHP](https://phpbestpractices.org/#utf-8) حيث تم استخدامها كأساس لنصائحنا للعمل بترميز UTF-8_.

### كن منتبهاً، دقيقاً ومهتماً بالتفاصيل. فليس هناك طريق واحد.

حتى الآن لم تقم PHP بدعم ترميز Unicode في مستوياتها الأساسية. هناك طرق للتَّأكد من أن النصوص قد تمت معالجتها بالترميز UTF-8، ولكنها عملية ليست سهلة وتحتاج الكثير من البحث والتنقيب في كل مستويات تطبيق الويب، ابتداءً من HTML إلى SQL وحتى PHP.
سوف نقوم بإيجاز نبذة عن تطبيقات عملية.

### UTF-8 على مستوى PHP

لا تحتاج عمليات النصوص، مثل دمج النصوص وتعيين قيم نصية إلى متغيرات، إلى أي شيء خاص لكي تتبع ترميز UTF-8.
لكن كثيرا من دوال النصوص مثل `strpos()` و`strlen()` تحتاج عناية خاصة.
هذه الدوال عادة ما تكون  لها نظيراتها باللاَّحِقة `mb_*` مثلاً: `mb_strpos()` و`mb_strlen()`. هذه الدوال عن طريق إضافة [Multibyte String Extension] وهي مُخَصَّصَة للتعامل مع النصوص بترميز Unicode.

يجب عليك أن تستخدم دوال `mb_*` كلما أردت التَّعامل مع نص بترميز Unicode. مثلا إذا قمت باستخدام `substr()` على نص بترميز UTF-8 هناك احتمال أَنَّ النتيجة ستحتوي رموزَا مشوَّهة. في هذه الحالة الطريقة الصحيحة هي استخدام دالة تأخذ النصوص متعددة البايت بالإعتبار (Multibyte) `mb_substr()`.

الصعوبة تَكْمُن في تَذَكُّر استخدام دوال `mb_*` دائماً عوضاً عن الدوال الأصلية. إذا قمت بنسيان ذلك ذات مرة فأي نص بترميز Unicode يكون مهدَّدًا بالتشوه في أي عملية لاحقة قد تطرأ عليه، مما يعني نتائج غير محمودة!

ليس كل دوال النصوص لديها معالجات في دوال `mb_*`. إذَا لم يكن هناك واحدة توفر لك الاستخدام المطلوب فأنت غير محظوظ!

يجب استخدام الدَّالة `mb_internal_encoding()` في بداية كل مصدر PHP تقوم بكتابته (كملف الإستدعاء الرئيس لبقية الملفات مثلاً)،
تليها دالة `mb_http_output()` إذا كنت ستقوم بطباعة نص على المتصفح.
تعيين الترميز بشكل صريح في كل نصوصك يَقِيك الكثير مِمَّا لا تُحمد عقباه خلال عملية التطوير.

إضافةً إلى ذلك هناك العديد من الدوال المدمجة تتيح إدارة النصوص عبر معطيات اختيارية لتحديد ترميز النصوص. عندها يجب عليك تحديد
UTF-8 عندما يكون هذا الخيار متاحاً. مثلا دالة `htmlentities()` لديها معطى اختياري لتحديد الترميز، سيتوجب عليك دائما تحديد
ترميز UTF-8 للتعامل مع النصوص.
ملاحظة: `htmlentities()` و`htmlspecialchars()` تستخدمان الترميز UTF-8 كترميز نصوص افتراضي منذ إصدار PHP 5.4.0.

أخيراً، إذا كنت تقوم ببناء تطبيق تنوي نشره وكنت غير متأكد من أن إضافة `mbstring` ستكون متوفرة أم لا، يجب عليك التفكير في استخدام حزمة Composer تسمى [patchwork/utf8]. ستقوم هذه الحزمة بإتاحة استخدام `mbstring` إذا كانت متوفرة، أو تقوم باستخدام الدوال الإفتراضية إذا لم تكن الأخرى متوفرة.

[Multibyte String Extension]: http://php.net/book.mbstring
[patchwork/utf8]: https://packagist.org/packages/patchwork/utf8

### UTF-8 على مستوى قاعدة البيانات

إذا كان تطبيقك يقوم بالوصول إلى قاعدة البيانات MySQL، عندها هناك احتمال أنَّ النصوص تُحفظ بترميز غير UTF-8 في قاعدة البيانات
حتى ولو اتبعت كل التحذيرات أعلاه!

لكي تكون متأكداً من أن النصوص تُنتقل من PHP إلى MySQL بترميز UTF-8، تَأَكَّد من أن جميع الجداول في قاعدة البيانات قد تم ضبط ترميزها لتسخدم الترميز `utf8mb4` في كل من الترميز والترتيب (Character set and Collation)، وأن تقوم باستخدام `utf8mb4` عند الاتصال باستخدام PDO، قم بالإطلاع على المثال أدناه فهو _هام جدا_.

ملاحظة: يجب استخدام ترميز `utf8mb4` لدعمِ كاملِ لترميز UTF-8 وليس الترميز `utf8`! أكمل القراءة لمعرفة السبب.

### UTF-8 على مستوى المتصفح

استخدم الدَّالة `mb_http_output()` للتَّأكد من أن تطبيقك يطبع نصوصا بترميز UTF-8 على المتصفح.

سيلزم عندها إخبار المتصفح -باستخدام HTTP Response- أن هذه الصفحة يجب أن تعتبر مُرَمَّزة باستخدام UTF-8.
سابقاً كان يتم إدراج هذا الإجراء عن طريق إدراج [charset في وسم `<meta>`](http://htmlpurifier.org/docs/enduser-utf8.html)
بداخل وسم `<head>` في الصفحة. هذا الإجراء سليم تماماً ولكن إضافة `Content-Type` كترويسة (header) هو عملياً [أكثر سرعة](https://developers.google.com/speed/docs/best-practices/rendering#SpecifyCharsetEarly).

{% highlight php %}
<?php
// قم بإخبار PHP أننا نستخدم ترميز UTF-8 حتى النهاية
mb_internal_encoding('UTF-8');
 
// قم بإخبار PHP أننا سنقوم بطبع مخرجات بترميز UTF-8 على المتصفح
mb_http_output('UTF-8');
 
// نص لتجربة عملية الترميز
$string = 'Êl síla erin lû e-govaned vîn.';
 
// قم بتحويل النص إلى شكل آخر باستخدام دالة البايت المتعدد Multibyte
// نلاحظ أنه يمكننا قطع نص من خارج جدول Ascii لأغراض الشرح فقط
$string = mb_substr($string, 0, 15);
 
// نقوم بالاتصال بقاعدة البيانات ثم نقوم بتخزين النص الذي قمنا بتحويله
// قم بالإطلاع على مثال PDP في هذا المستند لمزيد من المعلومات
// لاحظ أننا استخدمنا `charset=utf8mb4` في بيانات اسم مصدر البيانات (Data Source Name) (DSN)
$link = new PDO(
    'mysql:host=your-hostname;dbname=your-db;charset=utf8mb4',
    'your-username',
    'your-password',
    array(
        PDO::ATTR_ERRMODE => PDO::ERRMODE_EXCEPTION,
        PDO::ATTR_PERSISTENT => false
    )
);
 
// قم بتخزين النص المُحَوَّل بترميز UTF-8 في قاعدة البيانات
// هل قاعدة البيانات والجداول تستخدم ترميز وترتيب `utf8mb4` Charset and collation؟
$handle = $link->prepare('insert into ElvishSentences (Id, Body) values (?, ?)');
$handle->bindValue(1, 1, PDO::PARAM_INT);
$handle->bindValue(2, $string);
$handle->execute();
 
// قم باسترجاع النص الذي قمنا بتخزينه حتى يتم إثبات أنه قد تم تخزينه بطريقة صحيحة
$handle = $link->prepare('select * from ElvishSentences where Id = ?');
$handle->bindValue(1, 1, PDO::PARAM_INT);
$handle->execute();
 
// قم بتخزين النتيجة إلى كائن حتى نتمكن من طباعته لاحقاً في HTML
$result = $handle->fetchAll(\PDO::FETCH_OBJ);

header('Content-Type: text/html; charset=UTF-8');
?>
<!doctype html>
<html>
    <head>
        <meta charset="UTF-8">
        <title>UTF-8 صفحة تجريبية</title>
    </head>
    <body>
        <?php
        foreach($result as $row){
            print($row->Body);  // يجب أن يتم طباعة هذا النص المُحَوَّل بترميز UTF-8 بصور صحيحة في المتصفح
        }
        ?>
    </body>
</html>
{% endhighlight %}

### لمزيد من المصادر

* [PHP Manual: String Operations](http://php.net/language.operators.string)
* [PHP Manual: String Functions](http://php.net/ref.strings)
    * [`strpos()`](http://php.net/function.strpos)
    * [`strlen()`](http://php.net/function.strlen)
    * [`substr()`](http://php.net/function.substr)
* [PHP Manual: Multibyte String Functions](http://php.net/ref.mbstring)
    * [`mb_strpos()`](http://php.net/function.mb-strpos)
    * [`mb_strlen()`](http://php.net/function.mb-strlen)
    * [`mb_substr()`](http://php.net/function.mb-substr)
    * [`mb_internal_encoding()`](http://php.net/function.mb-internal-encoding)
    * [`mb_http_output()`](http://php.net/function.mb-http-output)
    * [`htmlentities()`](http://php.net/function.htmlentities)
    * [`htmlspecialchars()`](http://php.net/function.htmlspecialchars)
* [PHP UTF-8 Cheatsheet](http://blog.loftdigital.com/blog/php-utf-8-cheatsheet)
* [Handling UTF-8 with PHP](http://www.phpwact.org/php/i18n/utf-8)
* [Stack Overflow: What factors make PHP Unicode-incompatible?](http://stackoverflow.com/questions/571694/what-factors-make-php-unicode-incompatible)
* [Stack Overflow: Best practices in PHP and MySQL with international strings](http://stackoverflow.com/questions/140728/best-practices-in-php-and-mysql-with-international-strings)
* [How to support full Unicode in MySQL databases](http://mathiasbynens.be/notes/mysql-utf8mb4)
* [Bringing Unicode to PHP with Portable UTF-8](http://www.sitepoint.com/bringing-unicode-to-php-with-portable-utf8/)
* [Stack Overflow: DOMDocument loadHTML does not encode UTF-8 correctly](http://stackoverflow.com/questions/8218230/php-domdocument-loadhtml-not-encoding-utf-8-correctly)
