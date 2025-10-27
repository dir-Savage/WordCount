## كود ال word count شرح


```java
import java.io.IOException;
```

* ده استيراد لفئة الاستثناء `IOException`. بنقوله علشان الدوال اللي بتعمل I/O (قراءة/كتابة ملفات أو إدخال/إخراج) ممكن ترمي الاستثناء ده. أي ميثود في الكود بتتعامل مع I/O لازم تعلن إنها `throws IOException` أو تتعامل معاه.

```java
import java.util.StringTokenizer;
```

* بيستورد `StringTokenizer` اللي هو أداة بسيطة بتقسم `String` لكلمات (tokens) بناءً على فواصل افتراضية (زي المسافات). هنا هنستخدمه في الـ Mapper علشان نقسم السطر لكلمات.

```java
import org.apache.hadoop.conf.Configuration;
```

* `Configuration` ده كائن إعدادات Hadoop. بيحمل إعدادات افتراضية وملفات config (زي `core-site.xml`, `hdfs-site.xml` لو موجودة). بنستخدمه علشان نخلق `Job` مربوط بـ Hadoop environment.

```java
import org.apache.hadoop.fs.Path;
```

* `Path` بيمثل مسار ملف أو مجلد سواء على HDFS أو على النظام المحلي. بنستخدمه لتحديد مسارات الـ input و output للـ job.

```java
import org.apache.hadoop.io.IntWritable;
```

* `IntWritable` هو نوع بيانات Hadoop قابل للسيريالايز (Writable) بلف على قيمة `int`. Hadoop بيتعامل مع `Writable` مش مع الـ primitive types مباشرة. هنا القيمة اللي بنرسلها للـ reducer هي `1`، بنستخدم `IntWritable(1)`.

```java
import org.apache.hadoop.io.Text;
```

* `Text` هو مكافئ Hadoop للسلاسل (String) لكن قابل للسيريالايز والكفاءة المطلوبة في Hadoop. هنستخدمه كمفتاح (key) لأن المفتاح هو الكلمة نفسها.

```java
import org.apache.hadoop.mapreduce.Job;
```

* `Job` بيمثل مهمة MapReduce كاملة — فيه إعدادات، اسم، المابّر، الريديوسّر، إلخ. بننشئ `Job` عشان نشغل المهمة على الـ cluster أو وضع محلي.

```java
import org.apache.hadoop.mapreduce.Mapper;
```

* استيراد الكلاس الأساسي `Mapper` اللي هنورث منه عشان نعرف منطق الـ map.

```java
import org.apache.hadoop.mapreduce.Reducer;
```

* استيراد الكلاس الأساسي `Reducer` اللي هنورث منه عشان نعرف منطق الـ reduce.

```java
import org.apache.hadoop.mapreduce.lib.input.FileInputFormat;
```

* `FileInputFormat` مسؤول عن قراءة الملفات وتجزئتها إلى `InputSplits`، وبيتعامل مع القراية الفعلية من الملفات. بنستخدمه علشان نضيف مسار الإدخال للـ Job.

```java
import org.apache.hadoop.mapreduce.lib.output.FileOutputFormat;
```

* `FileOutputFormat` مسؤول عن كتابة مخرجات الـ Reducer إلى ملفات (زي `part-r-00000`) في المسار اللي نحدده.

```java
public class FunctionsS {
```

* تعريف الكلاس العام. اسم الكلاس لازم يطابق اسم الملف `FunctionsS.java` لأن الكلاس `public`. الكلاس ده يحتوي على تعريف الـ Mapper، Reducer، و `main()`.

---

**الـ Mapper:**

```java
  public static class TokenizerMapper
       extends Mapper<Object, Text, Text, IntWritable>{
```

* هنا بنعرف كلاس داخلي `TokenizerMapper` غير متصل (static) وبيورث من `Mapper`.
* التوقيع `Mapper<Object, Text, Text, IntWritable>` معناه:

  * `KEYIN` = `Object` (نوع المفتاح الداخل؛ عادة `LongWritable` لو إحنا بنستخدم `TextInputFormat`، لكن هنا بنخلي `Object` لأننا مش مهتمين بالمفتاح).
  * `VALUEIN` = `Text` (كل سطر من الملف كقيمة).
  * `KEYOUT` = `Text` (هنطلع الكلمات كمفتاح).
  * `VALUEOUT` = `IntWritable` (هنطلع قيمة عددية، وهنا دايمًا 1 لكل كلمة).
* `public static` علشان Hadoop يقدر ينشئ instance من الكلاس بسهولة.

```java
    private final static IntWritable one = new IntWritable(1);
```

* تعريف ثابت لكائن `IntWritable` بقيمة 1. بنعيد استخدامه لكل الكلمات بدل ما ننشئ كائن جديد لكل كلمة — ده تحسين للأداء لتقليل الـ object creation.

```java
    private Text word = new Text();
```

* كائن `Text` يتم إعادة استخدامه لحفظ كل كلمة قبل ما نكتبها في الـ context. برضه ده أداء أفضل بدل إنشاء `new Text()` داخل الحلقة كل مرة.

```java
    public void map(Object key, Text value, Context context
                    ) throws IOException, InterruptedException {
```

* تعريف ميثود الـ `map` اللي Hadoop هيستدعيها لكل زوج (key, value) من البيانات المدخلة.
* المعاملات:

  * `key` → المفتاح الداخلي (إزاحة السطر عادة) — هنا مش بنستخدمه.
  * `value` → محتوى السطر (نوعه `Text`).
  * `context` → كائن يديك واجهة للتواصل مع إطار العمل (تكتب به نتايج الـ map أو تقرأ إعدادات).
* الميثود بتصرح إنها ممكن ترمي `IOException` أو `InterruptedException`.

```java
      StringTokenizer itr = new StringTokenizer(value.toString());
```

* بنحول الـ `Text` إلى `String` باستعمال `value.toString()` وبعدها بننشئ `StringTokenizer` عشان يفصل السطر لكلمات مفصولة بالمسافات (والـ default delimiters).

```java
      while (itr.hasMoreTokens()) {
```

* بدء حلقة علشان نكرر على كل كلمة في السطر.

```java
        word.set(itr.nextToken());
```

* بنجيب الكلمة الجاية من الـ tokenizer ونحطها في كائن `Text` اللي اسمه `word`. `set()` يعيد استخدام الذاكرة الداخلية بدل إنشاء كائن جديد.

```java
        context.write(word, one);
```

* هنا بنبعت زوج المفتاح والقيمة (key = `word`, value = `one`) للخارج. الcontext بيستلم الأزواج دي ويبعتها للـ framework عشان تعمل shuffle & sort بعدين تتجمع حسب الـ key.

```java
      }
    }
  }
```

* نهاية الحلقة، نهاية ميثود الـ map، نهاية كلاس الـ Mapper.

---

**الـ Reducer:**

```java
  public static class IntSumReducer
       extends Reducer<Text,IntWritable,Text,IntWritable> {
```

* تعريف كلاس Reducer اسمه `IntSumReducer`.
* التوقيع `Reducer<Text, IntWritable, Text, IntWritable>` معناه:

  * `KEYIN` = `Text` (الكلمة اللي خرجت من الـ Mapper).
  * `VALUEIN` = `IntWritable` (القيم المرتبطة بالكلمة — أي مجموعة من `1`s).
  * `KEYOUT` = `Text` (الكلمة النهائية).
  * `VALUEOUT` = `IntWritable` (مجموع التكرار النهائي).
* `public static` علشان يكون سهل إنشاؤه من قبل Hadoop runtime.

```java
    private IntWritable result = new IntWritable();
```

* كائن `IntWritable` بنعيد استخدامه لتخزين نتيجة الجمع لكل مفتاح قبل كتابتها. تاني تحسين للأداء بتقليل إنشاء كائنات.

```java
    public void reduce(Text key, Iterable<IntWritable> values,
                       Context context
                       ) throws IOException, InterruptedException {
```

* تعريف ميثود `reduce` اللي Hadoop هيستدعيها لكل مفتاح مع كل القيم المرتبطة به.
* المعاملات:

  * `key` → الكلمة الحالية.
  * `values` → iterable لكل القيم (عادة مجموعة من `IntWritable(1)`).
  * `context` → نفس فكرة الـ context في الـ mapper، بنستخدمه لكتابة المخرجات.
* تصرح بإنها قد ترمي `IOException` أو `InterruptedException`.

```java
      int sum = 0;
```

* متغير محلي `sum` لتجميع قيم `values`.

```java
      for (IntWritable val : values) {
```

* حلقة for-each علشان نمر على كل قيمة في الـ iterable.

```java
        sum += val.get();
```

* `val.get()` بيجيب القيمة الكامنة داخل `IntWritable` (الـ primitive `int`). بنجمعها في `sum`.

```java
      }
```

* نهاية حلقة التجميع.

```java
      result.set(sum);
```

* بنحط قيمة الناتج الإجمالية في `result` (كائن `IntWritable`).

```java
      context.write(key, result);
```

* نكتب زوج المفتاح والقيمة (الكلمة + عدد التكرارات) كمخرج نهائي للـ Reducer. Hadoop هيكتب الأزواج دي في ملف(ات) الإخراج (`part-r-xxxxx`).

```java
    }
  }
```

* نهاية ميثود reduce ونهاية كلاس الـ Reducer.

---

**دالة main (إعداد وتشغيل الـ Job):**

```java
  public static void main(String[] args) throws Exception {
```

* دالة البداية للبرنامج. بتصرح `throws Exception` لتبسيط (ممكن ترمي استثناءات مختلفة زي `IOException`, `ClassNotFoundException`, الخ).

```java
    if (args.length != 2) {
```

* بنشيك إن المستخدم مرر بالضبط معطيين: مسار الإدخال ومسار الإخراج.

```java
      System.err.println("Usage: FunctionsS <input path> <output path>");
```

* لو المستخدم ميدخلش المعطيات المطلوبة، نطبع رسالة طريقة الاستخدام على الـ stderr.

```java
      System.exit(-1);
```

* ونخرج من البرنامج بكود فشل (-1).

```java
    }
```

* نهاية شرط التحقق.

```java
    Configuration conf = new Configuration();
```

* نخلق كائن `Configuration` جديد. ده يحمل إعدادات الـ Hadoop (من ملفات config لو متوفرة) ويستخدم لتهيئة الـ Job.

```java
    Job job = Job.getInstance(conf, "word count");
```

* ننشئ كائن `Job` مرتبط بالإعدادات `conf` ونعطيه اسم `'word count'` — الاسم ده بس للتبويب واللوغز، يساعد في تتبع الـ job.

```java
    job.setJarByClass(FunctionsS.class);
```

* بنخبر Hadoop أي كلاس فيها `main` — Hadoop بيستخدم ده لتحديد أي JAR هيُرفع للـ cluster. مهم إنه يبقى الاسم مظبوط (ليه الكود الأصلي كان بيستخدم `WordCount.class` وده كان سبب الخطأ لو مش موجود).

```java
    job.setMapperClass(TokenizerMapper.class);
```

* نسجل كلاس الـ Mapper اللي عايزين الـ Job يستخدمه (الكلاس اللي كتبناه فوق).

```java
    job.setCombinerClass(IntSumReducer.class);
```

* بنحدد `Combiner` اختياريًا. الـ Combiner هو Reducer صغير يتنفذ محليًا على نوات الـ Mapper قبل ما الداتا تمشي على الشبكة. هنا بنستخدم نفس منطق الـ Reducer (`IntSumReducer`) كـ Combiner لأن الجمع قابل للتجميع (associative/commutative) — ده يقلل كمية البيانات اللي بتمر في الشبكة.

```java
    job.setReducerClass(IntSumReducer.class);
```

* نسجل كلاس الـ Reducer النهائي.

```java
    job.setOutputKeyClass(Text.class);
```

* نحدد نوع المفاتيح اللي هيطلعها الـ Reducer (هنا `Text`).

```java
    job.setOutputValueClass(IntWritable.class);
```

* نحدد نوع القيم اللي هيطلعها الـ Reducer (هنا `IntWritable`).

> ملاحظة: في بعض الحالات بنستخدم `job.setMapOutputKeyClass(...)` و `job.setMapOutputValueClass(...)` لو أنواع مخرجات الـ Mapper مختلفة عن مخرجات الـ Reducer. هنا هما متطابقين فمش محتاجين.

```java
    FileInputFormat.addInputPath(job, new Path(args[0]));
```

* بنضيف مسار الإدخال للـ job. `args[0]` المفروض يكون المسار (ممكن HDFS path أو local path بحسب إعدادات `fs.defaultFS`). `FileInputFormat` هي المسؤولة عن تقسيم الملفات إلى أجزاء (splits).

```java
    FileOutputFormat.setOutputPath(job, new Path(args[1]));
```

* نحدد مسار الإخراج. Hadoop هيكتب ملفات زي `part-r-00000` في المجلد ده. مهم: المجلد ده لازم **ميكونش موجود مسبقًا** وإلا الـ job هيفشل (لو موجود إحذفه قبل التشغيل).

```java
    System.exit(job.waitForCompletion(true) ? 0 : 1);
```

* بنشغّل الـ job فعليًا بـ `job.waitForCompletion(true)`:

  * المعامل `true` معناه إننا نطلب من Hadoop يطبع حالة التقدم (progress) في الكونصول.
  * `waitForCompletion()` بترجع `true` لو الـ job نجح، `false` لو فشل.
* ثم بنخرج من البرنامج بكود 0 لو نجح أو 1 لو فشل.

```java
  }
}
```

* نهاية `main()`، ونهاية الكلاس `FunctionsS`.

---

# شوية ملاحظات مهمة وكدا (ملحوظات تقنية وممارسات جيدة)

1. **`job.setJarByClass(...)` لازم يبقى مظبوط**

   * لو كتبت `Job.setJarByClass(WordCount.class)` بس الكلاس `WordCount` مش موجود هتقابل `ClassNotFoundException`. خلي الاسم هو نفس اسم الكلاس اللي فيه الـ `main`.

2. **الـ Combiner مفيد لكن مش دايمًا ينفع**

   * Combiner يشتغل كويس لما عملية الـ reduce تكون **قابلة للتجميع** (associative & commutative) زي الجمع. هنا جمع الأعداد مناسب.

3. **أداء: إعادة استخدام الكائنات**

   * استخدام `private static final IntWritable one` و `private Text word` و `private IntWritable result` ده لتحسين الأداء ومنع إنشاء ملايين الكائنات في الـ map/reduce loops.

4. **تجزئة الكلمات (tokenization)**

   * `StringTokenizer` بسيط ومقبول في الأمثلة التعليمية، لكنه مش بيتعامل كويس مع علامات الترقيم أو الـ unicode المتقدم. لو هتعالج نص عربي أو نص فيه علامات ترقيم كتير: اعمل تنظيف `value.toString().toLowerCase().replaceAll("[^\\p{L}\\p{Nd}]+"," ")` بعدين `split("\\s+")`.

5. **التحقق من وجود مجلد الإخراج**

   * Hadoop ما بيكتبش فوق مجلد موجود. لازم تشيل `output` قبل التشغيل:

     * على HDFS: `hdfs dfs -rm -r /path/to/output`
     * لو محلي: `rm -rf output`

6. **تشغيل محلي للاختبار**

   * لو مش عايز تنشر على cluster، تقدر تعدل config أو تستخدم `LocalJobRunner` لتشغيل الـ job محليًا أثناء التطوير.

7. **ترميز UTF-8**

   * لو تتعامل مع عربي تأكد إن الملفات محفوظة utf-8 علشان الـ `Text` يقراها صح.


---


حلو — هاشرح كل سطر من اللي كتبته **سطر بسطر وبالعامية المصرية**، مع توضيح الصيغة الصحيحة، أمثلة، وملاحظات مهمة أو اختصارات لو احتجت. يلا نبدأ:

---

1. `pwd`

* الوظيفة: بيطبع المسار الكامل للدليل اللي انت واقف فيه دلوقتي (present working directory).
* مثال:

  * لو انت في مجلد البيت: `/home/bashar` هتطبع `pwd` نفس السطر ده.
* ملحوظة: مفيد لما تتوه ومش فاكر مكانك في الشجره بتاعت الفولدرات.

---

2. `whoami`

* الوظيفة: بيطبع اسم المستخدم الحالي اللي شغال بيه الـ shell.
* مثال: `whoami` => `basharrizq`
* ملحوظة: مفيد لو شغال على جهاز فيه أكتر من يوزر أو عندك sudo.

---

3. `mkdir name_of_folder`

* الوظيفة: ينشئ مجلد/دليل جديد بالاسم اللي انت حاطه.
* مثال: `mkdir projects` هينشئ فولدر اسمه `projects`.
* نصيحة: لو عايز تنشئ مجلدات متداخلة مرة واحدة استخدم `mkdir -p a/b/c`.

---

4. `cd name_of_directory`

* الوظيفة: يغير الدليل الحالي للدليل اللي محدده.
* مثال: `cd projects` => تنقلك لمجلد projects داخل الدليل الحالي.
* ملاحظة: لو حطيت مسار كامل زي `cd /home/bashar/projects` ينقلك مباشرة.

---

5. `cd ..`

* الوظيفة: يطلعك دليل واحد فوق (parent directory).
* مثال: لو أنت في `/home/bashar/projects` وكتبت `cd ..` هتبقى في `/home/bashar`.

---

6. `touch name-of-file.txt`

* الوظيفة: ينشئ ملف فاضي لو مش موجود، أو يحدث تاريخ التعديل (timestamp) لو الملف موجود.
* مثال: `touch notes.txt` => لو الملف مش موجود هيتعمل؛ لو موجود هيتغير تاريخ التعديل.
* ملاحظة: مش بيفتح الملف للكتابة، بس يخلّيه ظاهر.

---

7. `vi name-of-file`

* الوظيفة: يفتح محرر النص `vi` (أو `vim` لو مثبت) علشان تكتب في الملف.
* طريقة الكتابة والخروج الأساسية:

  * لما يفتح: إضغط `i` علشان تدخل **insert mode** وتبدأ تكتب.
  * بعد ما تخلص تكتب: إضغط `Esc` للخروج من insert mode.
  * اكتب `:wq` واضغط Enter علشان تحفظ وتخرج. لو عايز تجبر الحفظ (مثلاً لو read-only) استخدم `:wq!`.
* ملاحظة: في أوامر تانية زي `:q!` للخروج من غير حفظ.

---

8. `cat name-of-file`

* الوظيفة: يعرض محتوى الملف على شاشة الترمينال.
* مثال: `cat notes.txt` => يطبع النص كله.
* ملاحظة: لو الملف كبير استخدم `less notes.txt` أو `more notes.txt` للتصفح الصفحة بصفحة.

---

9. `cp first-directory-of-file-you-want-to-copy  second-directory-to-copy-to`

* الصيغة الصحيحة: `cp <source> <destination>`
* الوظيفة: ينسخ ملف/ملفات من مكان لمكان تاني.
* مثال: `cp file.txt /home/user/backup/` أو `cp file.txt file-copy.txt`.
* لو عايز تنسخ مجلد كامل: `cp -r folder1/ folder2/` (`-r` = recursive).

---

10. `mv first-directory-of-file-you-want-to-move  second-directory-to-move-to`

* الصيغة: `mv <source> <destination>`
* الوظيفة: ينقل الملف أو يعيد تسميته.
* مثال: `mv file.txt ../archive/` ينقله لمجلد archive، أو `mv oldname.txt newname.txt` يعيد تسميته.
* ملاحظة: مش بيعمل نسخة، بينقل فعلاً.

---

11. `rm directory-of-file-you-want-to-remove`

* الوظيفة: يحذف ملف أو فولدر (حسب الصيغة).
* أمثلة مهمة:

  * لحذف ملف: `rm file.txt`
  * لحذف مجلد ومحتواه: `rm -r foldername/` أو `rm -rf foldername/` (حذر: `-f` يجبر الحذف بدون تأكيد).
* تحذير: أمر خطير — خصوصًا `rm -rf /some/path` ممكن يحذف بيانات للأبد.

---

12. `ls`

* الوظيفة: يعرض قائمة الملفات والمجلدات في الدليل الحالي.
* أمثلة مفيدة:

  * `ls -l` => تفاصيل (أذونات، مالك، حجم، تاريخ).
  * `ls -a` => يعرض الملفات المخفية (اللي بيبتدوا بـ `.`).
  * `ls -la` أو `ls -al` => الاتنين مع بعض.

---

13. `hdfs dfs -ls /directory`

* الوظيفة: يعرض ملفات ومجلدات موجودة على **HDFS** (مش النظام المحلي).
* مثال: `hdfs dfs -ls /user/bashar/input` هيرجعلك لستة ملفات/folders في المسار ده على HDFS.
* ملاحظة: استخدم `hdfs dfs -ls -R /path` لو عايز لقائمة متفرعة recursively.

---

14. `hdfs dfs -mkdir name-of-directory`

* الوظيفة: ينشئ مجلد على HDFS.
* مثال: `hdfs dfs -mkdir /user/bashar/input`
* نصيحة: لو عايز تعمل كل المجلدات اللي مش موجودة استخدم `-p`: `hdfs dfs -mkdir -p /user/bashar/input/data`.

---

15. `hdfs dfs -put directory-of-file-on-your-local-to-copy-to-hdfs directory-to-copy-to-in-hdfs`

* الصيغة: `hdfs dfs -put <local_path> <hdfs_path>`
* الوظيفة: تنقل/تنسخ ملف من النظام المحلي إلى HDFS.
* مثال: `hdfs dfs -put localfile.txt /user/bashar/input/`
* ملاحظة: في أوامر تانية شبهها: `-copyFromLocal` (نفس الشغل تقريبًا).

---

16. `hdfs dfs -cat /path/to/file-on-hdfs`

* الوظيفة: يطبع محتوى ملف موجود على HDFS في الترمينال.
* الصيغة الصحيحة لازم تبدأ بـ `hdfs dfs -cat` مش مجرد `cat`.
* مثال: `hdfs dfs -cat /user/bashar/input/data.txt`
* ملاحظة: مفيد للتأكد من محتوى الملف اللي رفعته.

---

17. `hdfs dfs -get directory-of-file-on-your-hdfs-to-copy-to-local directory-to-copy-to-in-local`

* الصيغة: `hdfs dfs -get <hdfs_path> <local_path>`
* الوظيفة: يحمل ملف/مجلد من HDFS وينسخه للنظام المحلي.
* مثال: `hdfs dfs -get /user/bashar/output/part-r-00000 ./local-output/`
* ملاحظة: في برضو `-copyToLocal` اللي بتعمل نفس الشئ.

---

18. `hdfs dfs -touchz name-of-file.txt`

* الوظيفة: ينشئ ملف فاضي على HDFS (زي touch في اللينكس لكن بيخلق صفر بايت).
* مثال: `hdfs dfs -touchz /user/bashar/input/empty.txt`
* ملاحظة: مفيد لو محتاج تضمن وجود الملف قبل تشغيل job أو لتمثيل وجود ملف بدون محتوى.

---

19. `hdfs dfs -cp first-directory-of-file-you-want-to-copy  second-directory-to-copy-to`

* الصيغة: `hdfs dfs -cp <src> <dst>`
* الوظيفة: ينسخ ملف/مجلد داخل HDFS من مكان لمكان تاني.
* مثال: `hdfs dfs -cp /user/bashar/input/data.txt /user/bashar/backup/data.txt`
* ملاحظة: مش بيأثر على الملف الأصلي، بس بينسخه.

---

20. `hdfs dfs -rm first-directory-of-file-you-want-to-remove_on-hdfs`

* الصيغة: `hdfs dfs -rm <hdfs_path>` لحذف ملف. لحذف مجلد استخدم `-rm -r` أو `-rm -skipTrash -r` حسب النسخة.
* مثال: `hdfs dfs -rm /user/bashar/input/data.txt` أو `hdfs dfs -rm -r /user/bashar/tempdir`
* تحذير: زي الأمر المحلي، الحذف على HDFS ممكن يكون نهائي لو مافيش trash مفعل.

---

21. `hdfs dfs -help command-that-you-want-to-know-what-it-do`

* الصيغة: `hdfs dfs -help` أو `hdfs dfs -help -command` أو `hdfs help -cmd` حسب نسخة الـ Hadoop.
* الوظيفة: يديك مساعدة عن أوامر `hdfs dfs` أو تفاصيل عن كل option.
* مثال: `hdfs dfs -help -put` أو لو مش واضح، شغّل `hdfs dfs -help` وشوف اللستة الكاملة.

---

### شوية نصايح سريعة عامة:

* دايمًا خليك حذر مع أوامر الحذف (`rm -rf` / `hdfs dfs -rm -r`)، وخصوصًا لو شغال بصلاحيات root أو على HDFS production.
* لو مش متأكد من نتيجة `cp` أو `mv` جرب أول على ملف صغير أو مجلد تجريبي.
* استخدم `ls -la` و`hdfs dfs -ls -h` (لو متاح) علشان تشوف الحجم والبيانات بشكل أوضح.
* شغّل `hdfs dfs -du -h /path` لو عايز تعرف حجم الملفات على HDFS.


