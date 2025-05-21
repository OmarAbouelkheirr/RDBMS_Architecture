# شرح معمارية الـ RDBMS

## إيه هو الـ RDBMS؟

الـ **RDBMS** (Relational Database Management System) هو نظام إدارة قواعد بيانات بيخزن البيانات في **Tables** (جداول) مرتبطة ببعض عن طريق **Primary Keys (PK)** و**Foreign Keys (FK)**. بيستخدم لغة **SQL** عشان نكتب أوامر زي:
**SELECT**, **INSERT**, **UPDATE**, **DELETE**.

الـ **RDBMS** بيتبع قواعد الـ **ACID** (Atomicity, Consistency, Isolation, Durability) عشان يضمن إن الـ **Transactions** تتم بشكل آمن.

### مثال واقعي:

تخيل إنك بتشتغل على نظام مكتبة. عندك جدول **Books** (رقم الكتاب، العنوان، المؤلف) وجدول **Borrowers** (رقم العميل، الاسم، تاريخ الاستعارة).
لو عايز تعرف مين استعار كتاب معين، هتعمل **JOIN** بين الجدولين باستخدام الـ **PK** و**FK**.

---

## RDBMS Architecture

الـ **RDBMS** مكوّن من:

1. **Instance**: الجزء اللي فيه **Memory Structures** (زي **SGA** و**PGA**) و**Background Processes**.
2. **Database**: الجزء اللي فيه البيانات، وبيتقسم لـ:

   * **Physical Structures** (زي **Data Files**).
   * **Logical Structures** (زي **Tables**).

---

### الفرق بين Physical و Logical:

* **Physical Structures**: الملفات الفعلية اللي على الـ Disk، زي ملف **.mdf** في SQL Server.
* **Logical Structures**: الطريقة اللي بنشوف بيها البيانات جوه الـ Database، زي الجداول **Tables** في SSMS.

#### مثال واقعي:

في **SQL Server**، لما تعمل:

```sql
CREATE DATABASE MyLibrary;
```

* الـ **Instance** هي خدمة **SQL Server** نفسها.
* الـ **Database** هي ملفات زي:

  * **MyLibrary.mdf** (بيانات)
  * **MyLibrary.ldf** (لوج التعديلات)

---

## Memory Structures

الـ **RDBMS** بيستخدم الذاكرة لتحسين الأداء، وبتتقسم لـ:

### 1. System Global Area (SGA)

الـ **SGA** هي ذاكرة مشتركة بين كل الـ **Processes**، وبتحتوي على:

* **Database Buffer Cache**: بيخزن الـ **Data Blocks**.

  * لو البيانات موجودة (Cache Hit)، الـ Query تشتغل بسرعة.
  * لو مش موجودة (Cache Miss)، يجيبها من الـ Disk.

* **Redo Log Buffer**: بيخزن تغييرات البيانات زي **INSERT** و**UPDATE** مؤقتًا قبل ما تتكتب في ملفات الـ **Redo Logs**.

* **Shared Pool**: وده بيشمل:

  * **Library Cache**: بيحتفظ بـ SQL Queries اللي تم تحليلها مسبقًا.
  * **Data Dictionary Cache**: معلومات عن الـ Tables وColumns.
  * **Control Structures**.

* **Large Pool**: للعمليات الكبيرة زي الـ Backup.

* **Java Pool**: لو فيه كود Java شغال جوه الـ DBMS.

* **Streams Pool**: خاص بـ Oracle Streams (مش شائع في SQL Server).

#### مثال واقعي:

في **SQL Server**، لما تعمل:

```sql
SELECT * FROM Books;
```

الـ **Buffer Cache** بيحاول يجيب البيانات من الـ RAM. ولو الـ Query تم تحليلها قبل كده، بيتجاب الـ Execution Plan من الـ **Plan Cache**.

### 2. Program Global Area (PGA)

الـ **PGA** هي ذاكرة خاصة بكل Session أو Server Process، وبتحتوي على:

* **Private SQL Area**: بتخزن بيانات الـ Session زي المتغيرات والـ Cursors.
* **Sort Area**: بيستخدمها أثناء عمليات الفرز زي **ORDER BY** أو **GROUP BY**.

#### مثال واقعي:

في **SQL Server**، لو كتبت:

```sql
SELECT * FROM Books ORDER BY Title;
```

الفرز بيتم باستخدام الـ **Sort Area** داخل الـ **PGA**.

---

## Process Structures

الـ **RDBMS** بيشتغل من خلال نوعين من الـ Processes:

### 1. Server Processes

دي المسؤولة عن تنفيذ الـ Queries اللي بيبعتها المستخدم. بتقوم بـ:

* Parse للـ Query.
* استدعاء البيانات من الذاكرة أو القرص.
* إرسال النتيجة للمستخدم.

#### مثال واقعي:

في تطبيق ويب خاص بمكتبة، لما المستخدم يعمل Search عن كتاب، السيرفر بيرسل Query للـ DB، و**Server Process** بتجيب النتيجة.

### 2. Background Processes

عمليات بتشتغل في الخلفية عشان تحافظ على استقرار وأداء الـ Database:

* **DBWn (Database Writer)**: بيكتب التغييرات من الـ Buffer Cache إلى ملفات البيانات.

* **LGWR (Log Writer)**: بيكتب Redo Entries على ملفات اللوج عند:

  * تنفيذ **COMMIT**.
  * امتلاء **Redo Log Buffer**.
  * قبل ما يكتب **DBWn**.

* **SMON (System Monitor)**: بيعمل Recovery لو الـ Instance وقعت.

* **PMON (Process Monitor)**: بينضف بعد فشل أي User Process.

* **CKPT (Checkpoint Process)**: بيعمل Checkpoint لحفظ الحالة الحالية.

* **ARCn (Archiver)**: بيأرشف ملفات اللوج بعد ما يتم Switch.

* **RECO (Recoverer)**: بيعالج الـ In-Doubt Transactions.

#### مثال واقعي:

في **SQL Server**، بعد تنفيذ:

```sql
UPDATE Books SET Title = 'New Title' WHERE BookID = 1;
COMMIT;
```

الـ **LGWR** بيكتب التغييرات في ملف **.ldf**، ولو حصل Crash، الـ **SMON** بيرجع البيانات لحالتها الصحيحة.

---

## إزاي SQL Server بيطبق الـ Architecture؟

* **SGA** ⇨ زي **Buffer Pool** في SQL Server.
* **Redo Log Buffer** ⇨ زي **Transaction Log Buffer**.
* **Shared Pool** ⇨ زي **Plan Cache**.
* **Background Processes** ⇨ زي **Lazy Writer** و **Log Writer**.
* **PGA** ⇨ زي **Memory Grants** لكل Session.

---

## مثال تطبيقي كبير:

في نظام إدارة شركة شحن:

1. العميل يعمل **Order**.
2. السيرفر يرسل **INSERT** Query.
3. **Server Process** تبدأ تنفذ.
4. Execution Plan يتم تخزينه في **Shared Pool**.
5. البيانات يتم تحميلها أو إضافتها للـ **Database Buffer Cache**.
6. **LGWR** يكتب التغييرات في الـ Transaction Log.
7. **DBWn** يكتب على ملف **.mdf**.
8. في حالة حدوث Crash، **SMON** بيرجع كل حاجة لحالتها الأخيرة المضمونة.


---
---

📌 **By Omar Abouelkhier**: [@Dev3mora](https://t.me/dev3mora)

🎥 **Lecture 10**: [Press Here](https://youtube.com/watch?v=123456789)

📚 **Playlist**: [Press Here](https://youtube.com/playlist?list=PLabcde12345)
