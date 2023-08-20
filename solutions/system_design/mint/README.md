# تصميم Mint.com

*ملحوظة: يتصل هذا المستند مباشرة بالمجالات ذات الصلة الموجودة في [مواضيع تصميم النظام](https://github.com/donnemartin/system-design-primer#index-of-system-design-topics) لتجنب التكرار. يُرجى الرجوع إلى المحتوى المرتبط للحصول على نقاط النقاش العامة وميزات الاستبدال والبدائل.*

## الخطوة 1: توضيح حالات الاستخدام والقيود

> اجمع المتطلبات وحدد نطاق المشكلة.
> اطرح الأسئلة لتوضيح حالات الاستخدام والقيود.
> ناقش الافتراضات.

بدون مقابلة لتوضيح الأسئلة، سنحدد بعض حالات الاستخدام والقيود.

### حالات الاستخدام

#### سنحدد نطاق المشكلة للتعامل فقط مع الحالات الاستخدامية التالية

* **المستخدم** يتصل بحساب مالي
* **الخدمة** تستخرج المعاملات من الحساب
    * تحديث يومي
    * تصنيف المعاملات
        * يسمح بتجاوز التصنيف اليدوي من قبل المستخدم
        * لا إعادة تصنيف تلقائي
    * تحليل الإنفاق الشهري، حسب الفئة
* **الخدمة** توصي بميزانية
    * يسمح للمستخدمين بتعيين ميزانية يدويًا
    * يرسل إشعارات عندما يقتربون من الميزانية أو يتجاوزونها
* **الخدمة** لديها توفرًا عاليًا

#### خارج النطاق

* **الخدمة** تنفذ تسجيل وتحليل إضافي

### القيود والافتراضات

#### توضيح الافتراضات

* حركة المرور غير موزعة بالتساوي
* التحديث اليومي التلقائي للحسابات ينطبق فقط على المستخدمين النشطين في الـ 30 يومًا الماضية
* إضافة أو إزالة الحسابات المالية نادرة نسبيًا
* إشعارات الميزانية لا تحتاج إلى أن تكون فورية
* 10 ملايين مستخدم
    * 10 فئات ميزانية لكل مستخدم = 100 مليون عنصر ميزانية
    * أمثلة للفئات:
        * الإسكان = 1,000 دولار
        * الطعام = 200 دولار
        * الوقود = 100 دولار
    * يتم استخدام البائعين لتحديد فئة المعاملة
        * 50,000 بائع
* 30 مليون حساب مالي
* 5 مليار معاملة شهريًا
* 500 مليون طلب قراءة شهريًا
* نسبة كتابة إلى قراءة 10:1
    * تركيز على الكتابة، حيث يقوم المستخدمون بإجراء معاملات يوميًا، ولكن القليل منهم يزورون الموقع يوميًا

#### حساب الاستخدام

**توضيح مع المقابل ما إذا كنتم تريدون أداء حسابات استخدام سريعة.**

* الحجم لكل معاملة:
    * `user_id` - 8 بايت
    * `created_at` - 5 بايت
    * `seller` - 32 بايت
    * `amount` - 5 بايت
    * المجموع: حوالي 50 بايت
* 250 جيجابايت من محتوى المعاملات الجديدة شهريًا
    * 50 بايت لكل معاملة * 5 مليار معاملة شهريًا
    * 9 تيرابايت من محتوى المعاملات الجديدة في 3 سنوات
    * نفترض أن معظمها معاملات جديدة بدلاً من تحديثات للمعاملات الحالية
* 2,000 معاملة في الثانية في المتوسط
* 200 طلب قراءة في الثانية في المتوسط

دليل التحويل المفيد:

* 2.5 مليون ثانية في الشهر
* 1 طلب في الثانية = 2.5 مليون طلب في الشهر
* 40 طلب في الثانية = 100 مليون طلب في الشهر
* 400 طلب في الثانية = 1 مليار طلب في الشهر



## الخطوة 2: إنشاء تصميم عالي المستوى

> صور تصميمًا عالي المستوى مع جميع المكونات الهامة.

![Imgur](http://i.imgur.com/E8klrBh.png)

## الخطوة 3: تصميم المكونات الأساسية

> استمر في التفاصيل لكل مكون أساسي.

### حالة الاستخدام: المستخدم يتصل بحساب مالي

يمكننا تخزين معلومات حول 10 ملايين مستخدم في [قاعدة بيانات علاقية](https://github.com/donnemartin/system-design-primer#relational-database-management-system-rdbms). يجب أن نناقش [حالات الاستخدام والمقابلات بين اختيار SQL أو NoSQL](https://github.com/donnemartin/system-design-primer#sql-or-nosql).

* **العميل** يرسل طلبًا إلى **خادم الويب**، الذي يعمل كـ [وكيل عكسي](https://github.com/donnemartin/system-design-primer#reverse-proxy-web-server)
* يعيد **خادم الويب** الطلب إلى خادم **واجهة برمجة التطبيقات للحسابات**
* يحدث خادم **واجهة برمجة التطبيقات للحسابات** تحديثًا لجدول **قاعدة البيانات SQL** `accounts` بمعلومات الحساب الجديدة المُدخلة

**توضيح مع المقابل الخاص بك بمدى الكود الذي يُتوقع منك كتابته**.

يمكن أن يكون لدى جدول `accounts` الهيكل التالي:

```
id int NOT NULL AUTO_INCREMENT
created_at datetime NOT NULL
last_update datetime NOT NULL
account_url varchar(255) NOT NULL
account_login varchar(32) NOT NULL
account_password_hash char(64) NOT NULL
user_id int NOT NULL
PRIMARY KEY(id)
FOREIGN KEY(user_id) REFERENCES users(id)
```

سنقوم بإنشاء [فهرس](https://github.com/donnemartin/system-design-primer#use-good-indices) على `id` و `user_id` و `created_at` لتسريع عمليات البحث (بدلاً من فحص الجدول بأكمله) وللحفاظ على البيانات في الذاكرة. القراءة المتسلسلة لـ 1 ميغابايت من الذاكرة تستغرق حوالي 250 ميكروثانية، بينما تستغرق القراءة من وحدة التخزين SSD 4 مرات ومن القرص الصلب 80 مرة أطول.<sup><a href=https://github.com/donnemartin/system-design-primer#latency-numbers-every-programmer-should-know>1</a></sup>

سنستخدم [**REST API**](https://github.com/donnemartin/system-design-primer#representational-state-transfer-rest) العام:

```
$ curl -X POST --data '{ "user_id": "foo", "account_url": "bar", \
    "account_login": "baz", "account_password": "qux" }' \
    https://mint.com/api/v1/account
```

بالنسبة للاتصالات الداخلية، يمكننا استخدام [المكالمات الإجرائية عن بعد](https://github.com/donnemartin/system-design-primer#remote-procedure-call-rpc).

بعد ذلك، يقوم الخدمة بإخراج المعاملات من الحساب.

### حالة الاستخدام: الخدمة تستخرج المعاملات من الحساب

نريد استخراج المعلومات من حساب في هذه الحالات:

* يقوم المستخدم أول مرة بربط الحساب
* المستخدم يقوم يدويًا بتحديث الحساب
* تلقائيًا كل يوم للمستخدمين الذين كانوا نشطين في الـ 30 يومًا الماضية

تدفق البيانات:

* **العميل** يرسل طلبًا إلى **خادم الويب**
* يعيد **خادم الويب** الطلب إلى خادم **واجهة برمجة التطبيقات للحسابات**
* يضع خادم **واجهة برمجة التطبيقات للحسابات** وظيفة في **صف انتظار** مثل [Amazon SQS](https://aws.amazon.com/sqs/) أو [RabbitMQ](https://www.rabbitmq.com/)
    * قد يستغرق استخراج المعاملات وقتًا، ربما نريد القيام بذلك [بشكل غير متزامن باستخدام صف](https://github.com/donnemartin/system-design-primer#asynchronism)، وهذا يضيف تعقيدًا إضافيًا
* تقوم **خدمة استخراج المعاملات** بما يلي:
    * تستخر

#### Category service

For the **Category Service**, we can seed a seller-to-category dictionary with the most popular sellers.  If we estimate 50,000 sellers and estimate each entry to take less than 255 bytes, the dictionary would only take about 12 MB of memory.

**Clarify with your interviewer how much code you are expected to write**.

```python
class DefaultCategories(Enum):

    HOUSING = 0
    FOOD = 1
    GAS = 2
    SHOPPING = 3
    ...

seller_category_map = {}
seller_category_map['Exxon'] = DefaultCategories.GAS
seller_category_map['Target'] = DefaultCategories.SHOPPING
...
```

For sellers not initially seeded in the map, we could use a crowdsourcing effort by evaluating the manual category overrides our users provide.  We could use a heap to quickly lookup the top manual override per seller in O(1) time.

```python
class Categorizer(object):

    def __init__(self, seller_category_map, seller_category_crowd_overrides_map):
        self.seller_category_map = seller_category_map
        self.seller_category_crowd_overrides_map = \
            seller_category_crowd_overrides_map

    def categorize(self, transaction):
        if transaction.seller in self.seller_category_map:
            return self.seller_category_map[transaction.seller]
        elif transaction.seller in self.seller_category_crowd_overrides_map:
            self.seller_category_map[transaction.seller] = \
                self.seller_category_crowd_overrides_map[transaction.seller].peek_min()
            return self.seller_category_map[transaction.seller]
        return None
```

Transaction implementation:

```python
class Transaction(object):

    def __init__(self, created_at, seller, amount):
        self.created_at = created_at
        self.seller = seller
        self.amount = amount
```

### Use case: Service recommends a budget

To start, we could use a generic budget template that allocates category amounts based on income tiers.  Using this approach, we would not have to store the 100 million budget items identified in the constraints, only those that the user overrides.  If a user overrides a budget category, which we could store the override in the `TABLE budget_overrides`.

```python
class Budget(object):

    def __init__(self, income):
        self.income = income
        self.categories_to_budget_map = self.create_budget_template()

    def create_budget_template(self):
        return {
            DefaultCategories.HOUSING: self.income * .4,
            DefaultCategories.FOOD: self.income * .2,
            DefaultCategories.GAS: self.income * .1,
            DefaultCategories.SHOPPING: self.income * .2,
            ...
        }

    def override_category_budget(self, category, amount):
        self.categories_to_budget_map[category] = amount
```

For the **Budget Service**, we can potentially run SQL queries on the `transactions` table to generate the `monthly_spending` aggregate table.  The `monthly_spending` table would likely have much fewer rows than the total 5 billion transactions, since users typically have many transactions per month.

As an alternative, we can run **MapReduce** jobs on the raw transaction files to:

* Categorize each transaction
* Generate aggregate monthly spending by category

Running analyses on the transaction files could significantly reduce the load on the database.

We could call the **Budget Service** to re-run the analysis if the user updates a category.

**Clarify with your interviewer how much code you are expected to write**.

Sample log file format, tab delimited:

```
user_id   timestamp   seller  amount
```

**MapReduce** implementation:

```python
class SpendingByCategory(MRJob):

    def __init__(self, categorizer):
        self.categorizer = categorizer
        self.current_year_month = calc_current_year_month()
        ...

    def calc_current_year_month(self):
        """Return the current year and month."""
        ...

    def extract_year_month(self, timestamp):
        """Return the year and month portions of the timestamp."""
        ...

    def handle_budget_notifications(self, key, total):
        """Call notification API if nearing or exceeded budget."""
        ...

    def mapper(self, _, line):
        """Parse each log line, extract and transform relevant lines.

        Argument line will be of the form:

        user_id   timestamp   seller  amount

        Using the categorizer to convert seller to category,
        emit key value pairs of the form:

        (user_id, 2016-01, shopping), 25
        (user_id, 2016-01, shopping), 100
        (user_id, 2016-01, gas), 50
        """
        user_id, timestamp, seller, amount = line.split('\t')
        category = self.categorizer.categorize(seller)
        period = self.extract_year_month(timestamp)
        if period == self.current_year_month:
            yield (user_id, period, category), amount

    def reducer(self, key, value):
        """Sum values for each key.

        (user_id, 2016-01, shopping), 125
        (user_id, 2016-01, gas), 50
        """
        total = sum(values)
        yield key, sum(values)
```

## Step 4: Scale the design

> Identify and address bottlenecks, given the constraints.

![Imgur](http://i.imgur.com/V5q57vU.png)

**Important: Do not simply jump right into the final design from the initial design!**

State you would 1) **Benchmark/Load Test**, 2) **Profile** for bottlenecks 3) address bottlenecks while evaluating alternatives and trade-offs, and 4) repeat.  See [Design a system that scales to millions of users on AWS](../scaling_aws/README.md) as a sample on how to iteratively scale the initial design.

It's important to discuss what bottlenecks you might encounter with the initial design and how you might address each of them.  For example, what issues are addressed by adding a **Load Balancer** with multiple **Web Servers**?  **CDN**?  **Master-Slave Replicas**?  What are the alternatives and **Trade-Offs** for each?

We'll introduce some components to complete the design and to address scalability issues.  Internal load balancers are not shown to reduce clutter.

*To avoid repeating discussions*, refer to the following [system design topics](https://github.com/donnemartin/system-design-primer#index-of-system-design-topics) for main talking points, tradeoffs, and alternatives:

* [DNS](https://github.com/donnemartin/system-design-primer#domain-name-system)
* [CDN](https://github.com/donnemartin/system-design-primer#content-delivery-network)
* [Load balancer](https://github.com/donnemartin/system-design-primer#load-balancer)
* [Horizontal scaling](https://github.com/donnemartin/system-design-primer#horizontal-scaling)
* [Web server (reverse proxy)](https://github.com/donnemartin/system-design-primer#reverse-proxy-web-server)
* [API server (application layer)](https://github.com/donnemartin/system-design-primer#application-layer)
* [Cache](https://github.com/donnemartin/system-design-primer#cache)
* [Relational database management system (RDBMS)](https://github.com/donnemartin/system-design-primer#relational-database-management-system-rdbms)
* [SQL write master-slave failover](https://github.com/donnemartin/system-design-primer#fail-over)
* [Master-slave replication](https://github.com/donnemartin/system-design-primer#master-slave-replication)
* [Asynchronism](https://github.com/donnemartin/system-design-primer#asynchronism)
* [Consistency patterns](https://github.com/donnemartin/system-design-primer#consistency-patterns)
* [Availability patterns](https://github.com/donnemartin/system-design-primer#availability-patterns)

We'll add an additional use case: **User** accesses summaries and transactions.

User sessions, aggregate stats by category, and recent transactions could be placed in a **Memory Cache** such as Redis or Memcached.

* The **Client** sends a read request to the **Web Server**
* The **Web Server** forwards the request to the **Read API** server
    * Static content can be served from the **Object Store** such as S3, which is cached on the **CDN**
* The **Read API** server does the following:
    * Checks the **Memory Cache** for the content
        * If the url is in the **Memory Cache**, returns the cached contents
        * Else
            * If the url is in the **SQL Database**, fetches the contents
                * Updates the **Memory Cache** with the contents

Refer to [When to update the cache](https://github.com/donnemartin/system-design-primer#when-to-update-the-cache) for tradeoffs and alternatives.  The approach above describes [cache-aside](https://github.com/donnemartin/system-design-primer#cache-aside).

Instead of keeping the `monthly_spending` aggregate table in the **SQL Database**, we could create a separate **Analytics Database** using a data warehousing solution such as Amazon Redshift or Google BigQuery.

We might only want to store a month of `transactions` data in the database, while storing the rest in a data warehouse or in an **Object Store**.  An **Object Store** such as Amazon S3 can comfortably handle the constraint of 250 GB of new content per month.

To address the 200 *average* read requests per second (higher at peak), traffic for popular content should be handled by the **Memory Cache** instead of the database.  The **Memory Cache** is also useful for handling the unevenly distributed traffic and traffic spikes.  The **SQL Read Replicas** should be able to handle the cache misses, as long as the replicas are not bogged down with replicating writes.

2,000 *average* transaction writes per second (higher at peak) might be tough for a single **SQL Write Master-Slave**.  We might need to employ additional SQL scaling patterns:

* [Federation](https://github.com/donnemartin/system-design-primer#federation)
* [Sharding](https://github.com/donnemartin/system-design-primer#sharding)
* [Denormalization](https://github.com/donnemartin/system-design-primer#denormalization)
* [SQL Tuning](https://github.com/donnemartin/system-design-primer#sql-tuning)

We should also consider moving some data to a **NoSQL Database**.

## Additional talking points

> Additional topics to dive into, depending on the problem scope and time remaining.

#### NoSQL

* [Key-value store](https://github.com/donnemartin/system-design-primer#key-value-store)
* [Document store](https://github.com/donnemartin/system-design-primer#document-store)
* [Wide column store](https://github.com/donnemartin/system-design-primer#wide-column-store)
* [Graph database](https://github.com/donnemartin/system-design-primer#graph-database)
* [SQL vs NoSQL](https://github.com/donnemartin/system-design-primer#sql-or-nosql)

### Caching

* Where to cache
    * [Client caching](https://github.com/donnemartin/system-design-primer#client-caching)
    * [CDN caching](https://github.com/donnemartin/system-design-primer#cdn-caching)
    * [Web server caching](https://github.com/donnemartin/system-design-primer#web-server-caching)
    * [Database caching](https://github.com/donnemartin/system-design-primer#database-caching)
    * [Application caching](https://github.com/donnemartin/system-design-primer#application-caching)
* What to cache
    * [Caching at the database query level](https://github.com/donnemartin/system-design-primer#caching-at-the-database-query-level)
    * [Caching at the object level](https://github.com/donnemartin/system-design-primer#caching-at-the-object-level)
* When to update the cache
    * [Cache-aside](https://github.com/donnemartin/system-design-primer#cache-aside)
    * [Write-through](https://github.com/donnemartin/system-design-primer#write-through)
    * [Write-behind (write-back)](https://github.com/donnemartin/system-design-primer#write-behind-write-back)
    * [Refresh ahead](https://github.com/donnemartin/system-design-primer#refresh-ahead)

### Asynchronism and microservices

* [Message queues](https://github.com/donnemartin/system-design-primer#message-queues)
* [Task queues](https://github.com/donnemartin/system-design-primer#task-queues)
* [Back pressure](https://github.com/donnemartin/system-design-primer#back-pressure)
* [Microservices](https://github.com/donnemartin/system-design-primer#microservices)

### Communications

* Discuss tradeoffs:
    * External communication with clients - [HTTP APIs following REST](https://github.com/donnemartin/system-design-primer#representational-state-transfer-rest)
    * Internal communications - [RPC](https://github.com/donnemartin/system-design-primer#remote-procedure-call-rpc)
* [Service discovery](https://github.com/donnemartin/system-design-primer#service-discovery)

### Security

Refer to the [security section](https://github.com/donnemartin/system-design-primer#security).

### Latency numbers

See [Latency numbers every programmer should know](https://github.com/donnemartin/system-design-primer#latency-numbers-every-programmer-should-know).

### Ongoing

* Continue benchmarking and monitoring your system to address bottlenecks as they come up
* Scaling is an iterative process
