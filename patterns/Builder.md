**وقتی بخواهیم یک آبجکت پیچیده را مرحله به مرحله بسازیم از این الگو استفاده می کنیم.**

---
 ساختن آبجکت‌ها همیشه به سادگی یک new کردن نیست. گاهی وقت‌ها ساختن یک آبجکت و آماده کردن آن برای استفاده، به پارامترها و ورودی‌های زیادی بستگی دارد. برای مثال در بیشتر زبان‌ها از Query Builder برای ساختن کوئری‌های SQL استفاده می شود.

 ```sh
$user = $db->users()->find(...);
// or
$post = $db->posts()->where(...)->select(...)->find(...);
 ```

 همانطور که می‌بینیم آبجکت‌های user و post به ورودی‌های مختلفی نیاز دارند و طی چندین مرحله ساخته میشوند.

پس اگه مثل کد بالا آبجکتی داریم که باید طی چند مرحله ساخته شود، از الگوی Builder استفاده می‌کنیم.‍

## پیاده‌سازی الگوی Builder:
فرض کنیم می‌خوایم یک Query Builder برای ساختن درخواست‌های SQL بسازیم. چیزی که می‌سازیم باید بتواند برای انواع دیتابیس‌ها مثل MySQL و MongoDB کارایی داشته باشه. به عبارت دیگر، برای کلاینت (قسمتی از برنامه که از این ویژگی استفاده می‌کند) نباید مهم باشد که چه نوع دیتابیسی مورد استفاده قرار گرفته است.

اولین راه پیاده‌سازی الگوی Builder این است که مراحل مشترک ساختن آبجکت‌ها را مشخص و تعریف کنیم.

ابتدا یک اینترفیس می‌سازیم و این مراحل را به صورت متد در آن قرار می‌دهیم:

```sh
interface  QueryBuilder {
    public function table($table):QueryBuilder;
    public function select($cols):QueryBuilder;
    public function limit($limits):QueryBuilder;
    public function where($wheres):QueryBuilder;
    public function getQuery():string;

    /* Other SQL Methods  */
}
```

در Query Builder ها معمولاً متدهایی مثل where و limit و چندین دستور SQL داریم که اینجا به طور خلاصه ۴ عدد از آنها رو تعریف کردیم.

همچنین نوع خروجی همه متدها رو برابر با خودِ اینترفیس یعنی QueryBuilder قرار دادیم. این کار برای این هست که به متدهای زیرکلاس‌ها بگوییم با return this، خودِ کلاس را return کنند تا بتوانیم متدهای زنجیره‌ای (Method Chaining) داشته باشیم.

 یعنی:

 ```sh
$obj->table()->where()->select()->getQuery()
 ```

 مرحله بعد باید برای هر نوع دیتابیس یک کلاس اختصاصی بسازیم که از این اینترفیس تبعیت می‌کنند. ابتدا برای MySQL می‌نویسیم:

 ```sh
class MySqlQueryBuilder implements QueryBuilder
{
    private string $query;
    private string $table_name;

    public function __construct()
    {
        $this->query = '';
    }

    public function table($table): QueryBuilder
    {
        $this->table_name = $table;
        return $this;
    }

    public function select($cols): QueryBuilder
    {
        // TODO: Implement select() method.
        return $this;
    }

    public function limit($limits): QueryBuilder
    {
        // TODO: Implement limit() method.
        return $this;
    }

    public function where($wheres): QueryBuilder
    {
        // TODO: Implement where() method.
        return $this;
    }

    public function getQuery(): string
    {
        // TODO: Implement getQuery() method.
        return "This is a MySQL query";
    }
}
 ```

 متدهای کلاس MySqlQueryBuilder باید دستورات خام MySQL را تولید کنند. مثلاً متد select برای MySQL باید چنین چیزی تولید کند:

 ```sh
SELECT col1, col2 FROM table_name
 ```

 و به همین صورت می‌تونیم برای بقیه انواع دیتابیس هم چنین کلاسی بسازیم.


 به عنوان مثال برای mongodb:

 ```sh
class MongoDbQueryBuilder implements QueryBuilder {
    // ***
}
 ```

 متدهای کلاس MongoDbQueryBuilder هم مسئول تولید کردن کوئری‌های خام MongoDB هستند.

 خب حالا وقت این هست که از این کد استفاده کنیم.<br>ابتدا قسمت Client (قسمت اصلی برنامه‌ی ما که از این الگو استفاده می‌کند) را می‌نویسیم:

```sh
function client(QueryBuilder $builder)
{
    $obj = new $builder;
    $query = $obj->table('posts')->where('id', 429)->limit(10)->select(['id', 'title'])->getQuery();

    print $query;
}
```

همانطور که گفتیم قسمت Client لازم نیست بداند که چه نوع دیتابیسی مورد استفاده قرار گرفته است.<br>
 برای همین، مفهوم Abstraction اینجا به کمک ما می آید.<br>
 Client بدون اطلاع از نوع دیتابیس دارد کار خودش را انجام میدهد. اما به هر حال جایی از برنامه باید مشخص کنیم که چه نوع دیتابیسی می‌خواهیم. مشخص کردن نوع دیتابیس یا به طور کلی نوع Builder، قبل از اجرای واقعی قسمت Client اتفاق می‌افتد:

 ```sh
 $config = [
    'db1'=>'MySqlQueryBuilder',
    'db2'=>'MongoDbQueryBuilder',

];

client(new $config['db1']);
client(new $config['db2']);
 ```

 بدون استفاده از این الگو باید دست به دامن توابعی با تعداد پارامترهای زیاد می شدیم:

 ```sh
function makeMySqlQuery($table_name, $type, $constraints, $cols, $values, $limit = null, $offset = 0)
{
  $query = '';
  
  if (cols) {
    // ...  
  }

  // + ...

  return $query;
}

makeMySqlQuery('users', 'select', 'id = 429', null, null, null, 10);
makeMySqlQuery('post', 'update', 'id = 900', ['name'], ['hesam'], 15);
 ```

**مزایای الگوی Builder:**

۱. با تعریف کردن مراحل و ترکیب کردن آنها به طور دلخواه، بدون دستکاری کد می‌توانیم آبجکت‌های متنوع و با خروجی کاملاً متفاوتی داشته باشیم. برای مثال کد زیر را در نظر بگیرید:

```sh
builder->table('posts')->where(...)->select(...)->find(...);
```
بسته به نوع کانفیگ برنامه، خروجی این کد می‌تواند یک نمونه از کلاس MySql باشه یا MongoDb که با هم دیگر تفاوت خواهند داشت. بنابراین، مراحل یکسان، خروجی متفاوت.

۲. قسمت Client فقط درگیر انجام مسئولیت خودش می باشد، نه ساختن و کانفیگ کردن آبجکت.<br>
 مثلاً در قسمت Client تنها هدف ما باید این باشد که کاربران را از دیتابیس بخوانیم و نمایش بدهیم (اصل اول سالید: Single Responsibility)

۳. می‌توانیم بی‌نهایت Builder بسازیم بدون اینکه تغییری در کدها به وجود بیاوریم (اصل دوم سالید: Open/Closed)

۴. قسمت Client وابسته به Abstraction هست، نه کلاس‌های واقعی یا Concrete (اصل پنجم سالید: Dependency Inversion)

## کد کامل برنامه:
```sh
interface  QueryBuilder
{
    public function table($table): QueryBuilder;

    public function select($cols): QueryBuilder;

    public function limit($limits): QueryBuilder;

    public function where($wheres): QueryBuilder;

    public function getQuery(): string;

    /* Other SQL Methods  */
}

class MySqlQueryBuilder implements QueryBuilder
{
    private string $query;
    private string $table_name;

    public function __construct()
    {
        $this->query = '';
    }

    public function table($table): QueryBuilder
    {
        $this->table_name = $table;
        return $this;
    }

    public function select($cols): QueryBuilder
    {
        // TODO: Implement select() method.
        return $this;
    }

    public function limit($limits): QueryBuilder
    {
        // TODO: Implement limit() method.
        return $this;
    }

    public function where($wheres): QueryBuilder
    {
        // TODO: Implement where() method.
        return $this;
    }

    public function getQuery(): string
    {
        // TODO: Implement getQuery() method.
        return "This is a MySQL query";
    }
}

class MongoDbQueryBuilder implements QueryBuilder
{

    public function table($table): QueryBuilder
    {
        // TODO: Implement table() method.
        return $this;
    }

    public function select($cols): QueryBuilder
    {
        // TODO: Implement select() method.
        return $this;
    }

    public function limit($limits): QueryBuilder
    {
        // TODO: Implement limit() method.
        return $this;
    }

    public function where($wheres): QueryBuilder
    {
        // TODO: Implement where() method.
        return $this;
    }

    public function getQuery(): string
    {
        // TODO: Implement getQuery() method.
        return "This is a MongoDb query";
    }
}

function client(QueryBuilder $builder)
{
    $obj = new $builder;
    $query = $obj->table('posts')->where('id', 429)->limit(10)->select(['id', 'title'])->getQuery();

    print $query;
}

$config = [
    'db1'=>'MySqlQueryBuilder',
    'db2'=>'MongoDbQueryBuilder',

];

client(new $config['db1']);
client(new $config['db2']);
```