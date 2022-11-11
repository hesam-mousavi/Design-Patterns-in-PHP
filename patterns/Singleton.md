**این الگو به ما این اطمینان را می دهد که فقط یک نمونه از یک کلاس خاص در برنامه وجود دارد و راهی برای دسترسی به همان نمونه را ارائه می دهد.**

---
بعضی اوقات در برنامه، کلاس هایی داریم که نمونه سازی از آن هزینه بر هست و نمی خواهیم بیش از یک نمونه از آن تولید شود. 

قطعه کد زیر را در نظر بگیرید:

```sh
class Db
{
    private $conn;

    public function __construct()
    {
        $this->conn = "...";
    }

    public function query() { ... }
}
```

// User.file
```sh
$db = new DB();
$users = $db.query('...');
```
// Post.file
```sh
$db = new DB();
$posts = $db.query('...');
```
بر اساس این کد، به محض ساخته شدن یک نمونه از کلاس **DB**، ارتباط با دیتابیس(کانکشن) برقرار می شود (چون ارتباط با دیتابیس را در سازنده کلاس تعریف کرده ایم).
حالا اگر در یک چرخه ایجاد برنامه، 20 درخواست sql داشته باشیم (یعنی 20 نمونه بسازیم)، 20 کانکشن مجزا ایجاد می شود که اصلا بهینه نیست و باعث افت کارایی برنامه می شود.

اینجا پای الگوی سینگلتون وسط می آید تا به ما کمک بکند در سراسر این برنامه فقط یک نمونه از این کلاس ساخته شود(در صورت نیاز).

# پیاده‌سازی الگوی سینگلتون برای کلاس DB:

ابتدا یک پراپرتی private از نوع static برای نگهداری نمونه ساخته شده از کلاس ایجاد می کنیم.
```sh
class DB
{
    private static $instance = null;
 
    // ...
}
```
سپس باید متد سازنده کلاس را هم به صورت private تعریف کرد تا نتوان در بیرون از برنامه از روی کلاس DB نمونه ساخت.

```sh
class DB
{
    private static $instance = null;
 
    private function __construct()
    {
        // ...
    }

    // ...
}
```
حالا یک متد از نوع استاتیک می سازیم که مسئول تولید نمونه از کلاس می باشد. دلیل تعریف متد به صورت استاتیک این هست که بتوانیم بدون ساخت نمونه از کلاس به این متد دسترسی داشته باشیم.

```sh
class DB
{
    private static $instance = null;
 
    private function __construct()
    {
        // ...
    }

    public static function getInstance()
    {
        if (self::$instance == null)
            self::$instance = new DB();

        return self::$instance;
    }

    // ...
}
```
در متد getInstance ابتدا چک میکنیم که آیا از این کلاس نمونه ای ساخته شده است یا خیر. اگر ساخته نشده بود، یک نمونه ساخته می شود و سپس برگشت داده می شود.

نمونه کامل کد را ببینید:

```sh
class DB
{
    private $conn;
    private static $instance = null;

    private function __construct()
    {
        $this->conn = "this is a db connection";
    }

    public static function getInstance()
    {
        if (self::$instance == null)
            self::$instance = new DB();

        return self::$instance;
    }

    public function query()
    {
        return "this is a query ...";
    }
}

$db = DB::getInstance();
print ($db->query());
```
الان اگر بارها متد getInstance را صدا بزنیم فقط یکبار از کلاس DB نمونه ساخته می شود.