# روابط در Eloquent - لاراول

در لاراول، Eloquent ORM ابزاری برای کار با دیتابیس است. یکی از امکانات مهم آن تعریف روابط (Relationships) بین جداول است.
جداول پایگاه داده اغلب به یکدیگر مرتبط هستند. برای مثال، یک پست وبلاگ ممکن است نظرات زیادی داشته باشد یا یک سفارش می‌تواند به کاربری که آن را ثبت کرده است مربوط باشد.

## انواع روابط

- **One To One (یک به یک)**
- **One To Many (یک به چند)**
- **Many To Many (چند به چند)**
- **Has One Through / Has Many Through**
- **One To One (Polymorphic)**
- **One To Many (Polymorphic)**
- **Many To Many (Polymorphic)**

## تعریف روابط (Defining Relationships)

برای اینکه بین جداول مختلف بتونیم ارتباط برقرار کنیم، توی Eloquent این روابط به صورت **متد** داخل مدل‌ها تعریف میشن.
این کار دو تا مزیت بزرگ داره:

1.**کدی تمیز و خوانا** → راحت می‌فهمی چه ارتباطی بین جدول‌ها هست.  
2. **قدرت Query Building** → وقتی رابطه به صورت متد باشه، می‌تونی روش شرط‌های مختلف بزنی یا با متدهای دیگه ترکیب کنی.

مثال:

```php

$user->posts()->where('active', 1)->get();
// گرفتن همه پست‌های فعال کاربر
```

## ارتباط یک به یک (One To One)

رابطه یک به یک ساده ترین ارتباط در پایگاه داده هست و رابطه‌ای هست که در آن هر رکورد یک جدول دقیقاً به یک رکورد در جدول دیگر متصل است.

### مثال:

مدل `User` و مدل `Phone`.
برای تعریف این ارتباط ما متد `Phone` رو داخل مدل `User` قرار میدیم و متد `Phone` باید متد **hasOne** رو فراخوانی بکنه ونتیجه اون رو برگردونه.
اولین آرگومان داده شده به متد hasOne، نام کلاس مدل مرتبط هست.

**مدل User**

```php
class User extends Model {
    public function phone() {
        return $this->hasOne(Phone::class);
    }
}
```

> نکته:
> متد `hasOne` در واقع توسط کلاس پایه‌ی `Illuminate\Database\Eloquent\Model` در اختیار تمام مدل‌ها قرار داده میشه.
> یعنی هر مدلی که از `Model` ارث‌بری کنه می‌تونه خیلی راحت از این متد استفاده کنه.
> برای مثال:

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasOne;

class User extends Model
{
    /**
     * گرفتن شماره تلفنی که به کاربر مربوط است.
     */
    public function phone(): HasOne
    {
        return $this->hasOne(Phone::class);
    }
}
```

### استفاده از رابطه (Dynamic Properties)

بعد از اینکه رابطه رو تعریف کردیم، لاراول یک قابلیت جالب به ما میده به اسم **Dynamic Properties**.

ویژگی های پویا به ما اجازه میدن که متد رابطه رو مثل یک **property معمولی** صدا بزنیم، بدون اینکه پرانتز بگذاریم.

### مثال:

```php
$phone = User::find(1)->phone;
```

### کلید خارجی در روابط

لاراول به صورت پیش‌فرض کلید خارجی رو از روی نام مدل والد تشخیص میده.
برای مثال در رابطه‌ی `User` و `Phone`، لاراول فرض می‌کنه که جدول `phones` یک ستون به نام `user_id` داره.

اما اگر بخواید نام کلید خارجی رو تغییر بدید (مثلاً `userId` یا `owner_id`)، می‌تونی به متد `hasOne` یک آرگومان دوم بدید:

```php
return $this->hasOne(Phone::class, 'your_foreign_key');
```

برای مثال اگر ستون جدول phones به جای user_id، نام owner_id داشته باشه:

```php
class User extends Model {
    public function phone() {
        return $this->hasOne(Phone::class, 'owner_id');
    }
}
```

به این ترتیب لاراول به جای user_id، از owner_id برای پیدا کردن رکورد مرتبط استفاده می‌کنه.

### کلید اصلی (Local Key) در روابط

به طور پیش‌فرض، لاراول فرض می‌کنه که مقدار کلید خارجی (`user_id`) در جدول فرزند باید با مقدار ستون **id** (کلید اصلی) در جدول والد برابر باشه.

یعنی توی مثال `User` و `Phone`، لاراول مقدار ستون `id` کاربر رو با ستون `user_id` در جدول `phones` مقایسه می‌کنه.

اما اگه مدل والد کلید اصلی متفاوتی داشته باشه (مثلاً `uuid` یا `student_id`)، می‌تونی این رو هم مشخص کنی.
در این حالت باید به متد `hasOne` یک آرگومان سوم بدی:

```php
return $this->hasOne(Phone::class, 'foreign_key', 'local_key');
```

فرض کنید توی جدول users به جای ستون id، یک ستون به نام user_code به عنوان کلید اصلی دارید،
و توی جدول phones هم ستون owner_code به عنوان کلید خارجی استفاده میشه:

```php
class User extends Model {
    protected $primaryKey = 'user_code';

    public function phone() {
        return $this->hasOne(Phone::class, 'owner_code', 'user_code');
    }
}
```

به این شکل، لاراول ستون `owner_code` در جدول `phones` رو با مقدار ستون `user_code` در جدول `users` مقایسه می‌کنه.

### تعریف معکوس رابطه

تا اینجا فهمیدیم که میتونیم از مدل کاربر خود به مدل تلفن دسترسی داشته باشیم. در مرحله بعد، بیایید رابطه‌ای رو در مدل تلفن تعریف بکنیم که به ما امکان دسترسی به کاربری که صاحب تلفن است را می‌دهد.

### فرق blongsTo با hasOne

قبل از ادامه دادن ما با یک متد به اسم `blongsTo` روبه رو میشیم شاید بگید چه فرقی با `hasOne` داره در ادامه این رو بررسی می کنیم .

متدهای `hasOne` و `belongsTo` در لاراول هر دو برای تعریف روابط **یک به یک (One To One)** استفاده می‌شوند، ولی تفاوت اصلی در این است که **کدام مدل مالک کلید خارجی (foreign key)** است.

**hasOne**

وقتی در مدلی از `hasOne` استفاده می‌کنی یعنی این مدل **دارای یک رکورد مرتبط** در مدل دیگر است.کلید خارجی (مثلاً `user_id`) در جدول **مدل مقابل** قرار دارد.

```php
public function phone()
{
    return $this->hasOne(Phone::class);
}
```

اینجا می‌گیم هر کاربر یک شماره تلفن دارد.
پس ستون user_id در جدول phones ذخیره می‌شود.

**belongsTo**

وقتی از belongsTo استفاده می‌کنیم یعنی این مدل متعلق به یک رکورد دیگر است.
کلید خارجی در همین جدول فعلی قرار دارد.

مثال:

```php
// Phone.php
public function user()
{
    return $this->belongsTo(User::class);
}
```

اینجا می‌گیم هر شماره تلفن متعلق به یک کاربر است.
پس ستون user_id در جدول phones قرار دارد

### طرف مقابل رابطه (belongsTo)

خب حالا که فرق این دوتا رو فهمیدیم بریم تا رابطه‌ای را در مدل تلفن تعریف کنیم که به ما امکان دسترسی به کاربری که صاحب تلفن است را می‌دهد.

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Phone extends Model
{
    /**
     * Get the user that owns the phone.
     */
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}

```

وقتی از سمت جدول فرزند (مثل `Phone`) بخوایم به جدول والد (مثل `User`) دسترسی داشته باشیم،  
باید از متد **belongsTo** استفاده کنیم.

لاراول به صورت پیش‌فرض وقتی متد `user()` رو صدا می‌زنیم، دنبال مدلی می‌گرده که ستون `id` اون با ستون `user_id` در جدول `phones` برابر باشه:

```php
$phone = Phone::find(1);
$user = $phone->user;
```

لاراول نام کلید خارجی رو به شکل زیر حدس می‌زنه:

1- اسم متدی که در مدل فرزند تعریف کردی (که اینجا مدل فرزند `Phone` هست و اسم متد هم user گذاشتیم)

2- اضافه کردن پسوند \_id به اون

پس به طور پیش‌فرض انتظار داره در جدول phones ستونی به نام user_id وجود داشته باشه.

اما اگه اسم کلید خارجی چیز دیگه‌ای باشه، می‌تونی به متد belongsTo آرگومان دوم بدی:

```php
/**
 * Get the user that owns the phone.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class, 'your_foreign_key');
}
```

به طور پیش‌فرض لاراول فرض می‌کنه ستون id در جدول والد (مثلاً users) کلید اصلی هست.
اما اگه مدل والد از یک کلید اصلی دیگه (مثل user_code) استفاده کنه یا بخوای رابطه روی ستون دیگه‌ای ساخته بشه،
می‌تونی یک آرگومان سوم هم به belongsTo بدی:

```php
/**
 * Get the user that owns the phone.
 */
public function user(): BelongsTo
{
    return $this->belongsTo(User::class, 'foreign_key', 'owner_key');
}
```

فرض کنید در جدول phones ستونی به اسم owner_code داریم و در جدول users هم ستون user_code کلید اصلی هست.

```php
class Phone extends Model {
    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class, 'owner_code', 'user_code');
    }
}
```

به این ترتیب لاراول مقدار owner_code از جدول phones رو با مقدار user_code از جدول users مقایسه می‌کنه.

## رابطه یک به چند (One To Many)

رابطه یک به چند برای تعریف روابطی استفاده میشه که یک مدل والد یک یا چند مدل فررند باشد .

مثلا پست یک وبلاگ ممکنه تعداد نامحدودی کامنت داشته باشه
.  
مثل بقیه مدل های Eloquent روابط یک به چند با تعریف یک متد در مدل Eloquent شما تعریف می شوند :

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class Post extends Model
{
    /**
     * Get the comments for the blog post.
     */
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class);
    }
}
```

> باز هم Eloquent اتوماتیک کلید خارجی مناسب رو برای مدل Comment انتخاب میکنه طبق قرار داد نام `snake case` مدل والد رو میگیره و پسوند \_id به اون اضافه می کنه . پس توی این مثال Eloquent فرض می کنه که ستون کلید خارجی توی مدل Comment، اسم مدل والد با پسوند \_id هست یعنی post_id.

پس از تعریف متد رابطه، می تونیم با دسترسی به ویژگی comments به مجموعه‌ای از نظرات مرتبط دسترسی پیدا بکنیم:

```php
use App\Models\Post;

$comments = Post::find(1)->comments;

foreach ($comments as $comment) {
    // ...
}
```

از اونجایی که تمام روابط در Eloquent در واقع **Query Builder** هم هستند،
می‌تونیم روی اون‌ها شرط‌های مختلف بزنیم.

به طور پیش‌فرض وقتی `comments` رو صدا بزنیم همه‌ی کامنت‌های مرتبط برمی‌گردن،  
اما اگه بخوایم فیلتر یا محدودیتی بذاریم، می‌تونیم متد رو به شکل تابعی (`comments()`) صدا بزنیم  
و شرط دلخواه رو زنجیره کنیم:

```php
$comment = Post::find(1)->comments()
    ->where('title', 'foo')
    ->first();
```

مثل متد `hasOne` میتونیم با ارسال آرگومان های اضافی به متد `hasMany` کلید های خارجی و محلی رو دوباره تعریف بکنیم :

```php
return $this->hasMany(Comment::class, 'foreign_key');

return $this->hasMany(Comment::class, 'foreign_key', 'local_key');
```

### Eager Loading

یکی از مشکلات رایج هنگام کار با روابط در دیتابیس، مشکل **N+1 Query** هست.  
فرض کنین ما می‌خوایم همه‌ی کاربران و پست‌های اون‌ها رو بگیریم:

```php
$users = User::all();

foreach ($users as $user) {
    foreach ($user->posts as $post) {
        echo $post->title;
    }
}
```

در این حالت:

یک کوئری برای گرفتن همه‌ی کاربران زده میشه.

برای هر کاربر، یک کوئری جدا برای گرفتن پست‌هاش اجرا میشه.

اگر 100 کاربر داشته باشیم 101 کوئری اجرا میشه!
این دقیقاً مشکل N+1 Query Problem هست.

### راه‌حل: Eager Loading

لاراول قابلیتی به اسم Eager Loading داره. با استفاده از این قابلیت می‌تونیم روابط رو همزمان با کوئری اصلی لود کنیم:

```php
$users = User::with('posts')->get();

foreach ($users as $user) {
    foreach ($user->posts as $post) {
        echo $post->title;
    }
}
```

حالا فقط ۲ کوئری اجرا میشه:

1- گرفتن همه‌ی کاربران

2- گرفتن همه‌ی پست‌های مرتبط با کاربران

این باعث میشه کوئری‌ها بهینه‌تر و سریع‌تر اجرا بشن.

### مشکل N+1 حتی با Eager Loading

گاهی اوقات حتی با وجود Eager Loading هم مشکل N+1 می‌تونه به‌وجود بیاد.برای مثال:

```php
$posts = Post::with('comments')->get();

foreach ($posts as $post) {
    foreach ($post->comments as $comment) {
        echo $comment->post->title;
    }
}
```

در این مثال، ما کامنت‌ها رو Eager Load کردیم، اما وقتی از داخل هر کامنت به post دسترسی پیدا می‌کنیم، دوباره کوئری جدا برای هر بار اجرا میشه. این یعنی همچنان مشکل N+1 باقی می‌مونه.

### راه‌حل: Hydrating Parent Models

لاراول قابلیتی داره به اسم chaperone داره که اجازه میده مدل والد (مثلاً Post) به صورت خودکار روی فرزندان (مثلاً Comment) هیدریت بشه. اینطوری حتی توی چنین شرایطی هم از مشکل N+1 جلوگیری می‌کنیم:

```php
class Post extends Model
{
    public function comments(): HasMany
    {
        return $this->hasMany(Comment::class)->chaperone();
    }
}
```

یا در زمان Eager Loading:

```php
$posts = Post::with([
    'comments' => fn ($comments) => $comments->chaperone(),
])->get();
```

## One to Many (Inverse) / Belongs To

حالا که می‌تونیم به همه‌ی کامنت‌های یک پست دسترسی داشته باشیم، لازمه رابطه معکوس هم تعریف بشه تا از سمت کامنت به پست والد برسیم.

تعریف رابطه در مدل فرزند (Comment)

```php
namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsTo;

class Comment extends Model
{
    /**
     * Get the post that owns the comment.
     */
    public function post(): BelongsTo
    {
        return $this->belongsTo(Post::class);
    }
}
```

دسترسی به پست والد از روی کامنت

```php
use App\Models\Comment;

$comment = Comment::find(1);
return $comment->post->title;
```

> اینجا Eloquent به‌طور پیش‌فرض فرض می‌کنه ستون post_id در جدول comments وجود داره.در این حالت مقدار ستون id از جدول posts با ستون post_id در جدول comments مقایسه می‌شه.

### سفارشی‌سازی کلید خارجی و کلید والد

اگر کلید خارجی در جدول فرزند (مثلاً comments) اسم متفاوتی داشته باشه، می‌تونیم به عنوان آرگومان دوم به `belongsTo` پاس بدیم :

```php
/**
 * Get the post that owns the comment.
 */
public function post(): BelongsTo
{
    return $this->belongsTo(Post::class,'foreign_key');
}
```

همچنین اگر کلید اصلی جدول والد چیزی غیر از `id` باشه یا بخوایم از یک ستون دیگه به عنوان کلید والد استفاده کنیم، می‌تونیم آرگومان سوم رو هم پاس بدیم:

```php
/**
 * Get the post that owns the comment.
 */
public function post(): BelongsTo
{
    return $this->belongsTo(Post::class, 'foreign_key','owner_key');
}
```

### Default Models

رابطه‌های belongsTo، hasOne، hasOneThrough و morphOne اجازه می‌دهند که یک مدل پیش‌فرض تعیین کنید تا در صورت خالی بودن رابطه، بازگردانده شود. این الگو به عنوان Null Object pattern شناخته می‌شود:

```php
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault();
}
```

می‌توانید مقادیر پیش‌فرض را با آرایه یا Closure پر کنید:

```php
public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault([
        'name' => 'Guest Author',
    ]);
}

public function user(): BelongsTo
{
    return $this->belongsTo(User::class)->withDefault(function (User $user, Post $post) {
        $user->name = 'Guest Author';
    });
}
```

### Querying Belongs To Relationships

برای گرفتن فرزندان یک رابطه `belongsTo` می‌توان شرط where را دستی ساخت:

```php
use App\Models\Post;

$posts = Post::where('user_id', $user->id)->get();
```

اما راحت‌تر است از متد whereBelongsTo استفاده کنیم که به‌طور خودکار رابطه و کلید خارجی مناسب را تشخیص می‌دهد:

```php
$posts = Post::whereBelongsTo($user)->get();
```

می‌توانید یک مجموعه (Collection) را هم به whereBelongsTo بدهید تا مدل‌هایی که به هر کدام از والدین تعلق دارند بازیابی شوند:

```php
$users = User::where('vip', true)->get();
$posts = Post::whereBelongsTo($users)->get();
```

لاراول به‌طور پیش‌فرض نام رابطه مرتبط با مدل داده‌شده را بر اساس نام کلاس مدل تعیین می‌کند، اما می تونید نام رابطه را به صورت دستی به عنوان آرگومان دوم مشخص کنید:

```php
$posts = Post::whereBelongsTo($user, 'author ')->get();
```

## Has One of Many

گاهی یک مدل می‌تواند چندین مدل مرتبط داشته باشد، اما شما می‌خواهید راحت‌ترین راه برای گرفتن جدیدترین یا قدیمی‌ترین مدل مرتبط داشته باشید. مثلا یک User می‌تواند چندین Order داشته باشد، اما شما می‌خواهید آخرین سفارش او را بگیرید.

```php
/**
 * Get the user's most recent order.
 */
public function latestOrder(): HasOne
{
    return $this->hasOne(Order::class)->latestOfMany();
}
```

همچنین برای قدیمی ترین رکورد هم اینطوری عمل می کنیم :

```php
/**
 * Get the user's oldest order.
 */
public function oldestOrder(): HasOne
{
    return $this->hasOne(Order::class)->oldestOfMany();
}
```

### مرتب‌سازی سفارشی

به‌طور پیش‌فرض، `latestOfMany` و `oldestOfMany` براساس کلید اصلی (primary key) مرتب‌سازی می‌کنند. اما می‌توانید معیار دیگری هم مشخص کنید.

مثلاً برای گرفتن سفارش با بیشترین قیمت:

```php
/**
 * Get the user's largest order.
 */
public function largestOrder(): HasOne
{
    return $this->hasOne(Order::class)->ofMany('price', 'max');
}
```

> متد ofMany ستون قابل مرتب‌سازی را به عنوان اولین آرگومان خود می‌پذیرد و اینکه کدام تابع تجمیعی (min یا max) را هنگام پرس‌وجو برای مدل مرتبط اعمال کند به عنوان آرگومان دوم می پذیرد.

### Converting "Many" Relationships to Has One Relationships

وقتی بخواهید یک مدل واحد با استفاده از latestOfMany, oldestOfMany یا ofMany دریافت کنید، ممکن است قبلاً رابطه hasMany برای همان مدل تعریف شده باشد. لاراول این امکان را می‌دهد که این رابطه را به راحتی به یک رابطه hasOne تبدیل کنید:

```php
/**
 * Get the user's orders.
 */
public function orders(): HasMany
{
    return $this->hasMany(Order::class);
}

/**
 * Get the user's largest order.
 */
public function largestOrder(): HasOne
{
    return $this->orders()->one()->ofMany('price', 'max');
}
```

همچنین می‌توانید از متد one برای تبدیل روابط HasManyThrough به روابط HasOneThrough استفاده کنید:

```php
public function latestDeployment(): HasOneThrough
{
    return $this->deployments()->one()->latestOfMany();
}
```

### Advanced Has One of Many Relationships

گاهی اوقات ممکن است بخواهید یک رابطه "Has One of Many" پیشرفته‌تر بسازید که معیارهای بیشتری برای انتخاب مدل مرتبط داشته باشد. برای مثال، یک مدل Product ممکن است چندین مدل Price مرتبط داشته باشد که حتی پس از انتشار قیمت جدید نیز در سیستم باقی می‌مانند. همچنین ممکن است اطلاعات قیمت جدید برای محصول بتواند از قبل منتشر شود و در تاریخ آینده اثرگذار باشد، که این توسط ستون published_at مدیریت می‌شود.

در این حالت، هدف ما دریافت آخرین قیمت منتشر شده است که تاریخ انتشار آن در آینده نباشد. علاوه بر این، اگر دو قیمت تاریخ انتشار مشابهی داشته باشند، قیمت با بیشترین شناسه (ID) انتخاب می‌شود. برای انجام این کار، باید یک آرایه به متد ofMany بدهیم که ستون‌های قابل مرتب‌سازی برای تعیین آخرین قیمت را مشخص کند. همچنین یک closure به عنوان آرگومان دوم به ofMany داده می‌شود تا شرایط اضافی مربوط به تاریخ انتشار را به کوئری اضافه کند:

```php
use Illuminate\Database\Eloquent\Builder;
use Illuminate\Database\Eloquent\Relations\HasOne;

/**
 * Get the current pricing for the product.
 */
public function currentPricing(): HasOne
{
    return $this->hasOne(Price::class)->ofMany([
        'published_at' => 'max',
        'id' => 'max',
    ], function (Builder $query) {
        $query->where('published_at', '<', now());
    });
}
```

توضیح جزئیات

ستون published_at مشخص می‌کند که جدیدترین قیمت منتشر شده را پیدا کنیم و ستون id برای جلوگیری از تساوی‌ها استفاده می‌شود.

داخل closure می‌توان شرط‌های پیچیده‌ای به کوئری اضافه کرد، مانند محدود کردن تاریخ انتشار به گذشته.

نتیجه نهایی : متد currentPricing تنها یک مدل Price بازمی‌گرداند که آخرین قیمت منتشر شده و معتبر برای محصول است.

این روش به شما امکان می‌دهد تا روابط پیچیده "Has One of Many" با معیارهای سفارشی بسازید و از انعطاف بالای Eloquent برای مدیریت روابط پیچیده بهره ببرید.

## Has One Through

رابطه "has-one-through" یک ارتباط یک به یک تعریف می‌کند که از طریق یک مدل میانی به مدل نهایی دسترسی پیدا می‌کند.

برای مثال، در یک اپلیکیشن تعمیرگاه خودرو:

هر مکانیک (Mechanic) با یک خودرو (Car) مرتبط است.

هر خودرو با یک مالک (Owner) مرتبط است.

مکانیک و مالک به طور مستقیم در جدول‌ها ارتباطی ندارند، اما مکانیک می‌تواند مالک خودرو را از طریق مدل Car دسترسی داشته باشد.  
بیاید اول به ساختار جداول یک نگاهی بکنیم :

**mechanics**

1- id

2- name

**cars**

1- id

2- model

mechanic_id

**owners**

1- id

2- name

3- car_id

حالا که ساختار جدول مربوط به رابطه را بررسی کردیم، بیاید رابطه را در مدل Mechanic تعریف کنیم:

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasOneThrough;

class Mechanic extends Model
{
    /**
     * Get the car's owner.
     */
    public function carOwner(): HasOneThrough
    {
        return $this->hasOneThrough(Owner::class, Car::class);
    }
}
```

> اولین آرگومان ارسالی به متد hasOneThrough نام مدل نهایی است که می‌خواهیم به آن دسترسی داشته باشیم، در حالی که آرگومان دوم نام مدل میانی است.

### تعریف رابطه با استفاده از روابط موجود روی مدل‌ها

اگر روابط قبلاً روی مدل‌ها تعریف شده باشند، می‌توان به صورت Fluent نوشت:

```php
// استفاده از نام رابطه به صورت رشته
return $this->through('cars')->has('owner');

// استفاده از نام تابع داینامیک
return $this->throughCars()->hasOwner();
```

این روش به شما امکان می‌دهد بدون استفاده مستقیم از کلیدها، رابطه پیچیده‌ی یک به یک از طریق مدل میانی را تعریف کنید.

### Key Conventions

اگر می‌خواهید کلیدهای رابطه را سفارشی کنید، می‌توانید آنها را به عنوان آرگومان‌های سوم و چهارم به متد `hasOneThrough` ارسال کنید. آرگومان سوم نام کلید خارجی در مدل `میانی` است. آرگومان چهارم نام کلید خارجی در مدل `نهایی` است. آرگومان پنجم `کلید محلی` است، در حالی که آرگومان ششم` کلید محلی مدل میانی` است:

```php
class Mechanic extends Model
{
    /**
     * Get the car's owner.
     */
    public function carOwner(): HasOneThrough
    {
        return $this->hasOneThrough(
            Owner::class,
            Car::class,
            'mechanic_id', // Foreign key on the cars table...
            'car_id', // Foreign key on the owners table...
            'id', // Local key on the mechanics table...
            'id' // Local key on the cars table...
        );
    }
}
```

اگر روابط لازم قبلاً در همه مدل‌ها تعریف شده باشد، می‌توانیم با روش Fluent رابطه HasOneThrough را تعریف کنیم و از کلیدهای قبلی استفاده کنیم:

```php
// String based syntax...
return $this->through('cars')->has('owner');

// Dynamic syntax...
return $this->throughCars()->hasOwner();
```
## Has Many Through  
رابطه‌ی hasManyThrough اجازه می‌دهد به رکوردهای دورتر از طریق یک رابطه‌ی واسطه دسترسی پیدا کنیم.  
مثلا فرض کنید یک Application چندین Environment دارد و هر Environment چندین Deployment.  
بیاید نگاهی به ساختار جداول این مثال نگاهی بکنیم  :    

**applications**  
    1- id - integer  
    2- name - string

**environments**  
    1- id - integer  
    2- application_id - integer  
    3- name - string  

**deployments**  
    1- id - integer  
    2- environment_id - integer  
    3- commit_hash - string    
      
حالا که ساختار جدول مربوط به رابطه را بررسی کردیم، بیایید رابطه را در مدل Application تعریف کنیم:  
  
```php  
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasManyThrough;

class Application extends Model
{
    /**
     * Get all of the deployments for the application.
     */
    public function deployments(): HasManyThrough
    {
        return $this->hasManyThrough(Deployment::class, Environment::class);
    }
}
```  
>اولین آرگومان ارسالی به متد hasManyThrough نام مدل نهایی است که می‌خواهیم به آن دسترسی داشته باشیم، در حالی که آرگومان دوم نام مدل میانی است.  
  
اگر رابطه‌های واسطه از قبل تعریف شده باشند میتوانیم به صورت زیر عمل کنیم :
 ```php  
 // String based syntax...
return $this->through('environments')->has('deployments');

// Dynamic syntax...
return $this->throughEnvironments()->hasDeployments();
```    
حتی اگر جدول deployments ستونی به نام application_id نداشته باشد، لاراول از طریق جدول واسطه (environments.application_id) این اتصال را برقرار می‌کند.  
  
### Key Conventions  
لاراول به‌طور پیش‌فرض از Conventionها برای تشخیص کلیدهای خارجی استفاده می‌کند. اما اگر ساختار جدول شما خاص باشد، می‌توانید کلیدها را به صورت دستی مشخص کنید.

آرگومان‌های متد hasManyThrough به ترتیب :

1- مدل نهایی (Deployment)

2- مدل واسط (Environment)

3- کلید خارجی روی مدل واسط (مثلاً application_id در جدول environments)

4- کلید خارجی روی مدل نهایی (مثلاً environment_id در جدول deployments)

5- کلید محلی در مدل اصلی (مثلاً id در جدول applications)

6- کلید محلی در مدل واسط (مثلاً id در جدول environments)  
  
```php  
class Application extends Model
{
    public function deployments(): HasManyThrough
    {
        return $this->hasManyThrough(
            Deployment::class,
            Environment::class,
            'application_id', // Foreign key on the environments table...
            'environment_id', // Foreign key on the deployments table...
            'id', // Local key on the applications table...
            'id' // Local key on the environments table...
        );
    }
}  
```  
یا همانطور که قبلاً بحث شد، اگر روابط مربوطه قبلاً در تمام مدل‌های درگیر در رابطه تعریف شده باشند، می‌توانید با فراخوانی متد through و ارائه نام آن روابط، یک رابطه "has-many-through" را به راحتی تعریف کنید. این رویکرد مزیت استفاده مجدد از قراردادهای کلیدی که قبلاً در روابط موجود تعریف شده‌اند را ارائه می‌دهد:  

```php  
// String based syntax...
return $this->through('environments')->has('deployments');

// Dynamic syntax...
return $this->throughEnvironments()->hasDeployments();  
```  
مزیت اصلی HasManyThrough

1- بدون نیاز به نوشتن Queryهای پیچیده Join می‌تونیم مستقیماً به داده‌های نهایی دسترسی داشته باشیم.

2- وقتی رابطه‌ها درست تعریف بشن، کد تمیزتر و خواناتر خواهد بود.

3- خیلی برای گزارش‌گیری و کوئری‌های آماری کاربرد داره.  
  
  
## Scoped Relationships

گاهی اوقات نیاز داریم روابطی که قبلاً تعریف کرده‌ایم را محدودتر کنیم. به عنوان مثال، فرض کنید یک کاربر پست‌های زیادی دارد، اما ما می‌خواهیم فقط پست‌های ویژه (featured) او را داشته باشیم. برای این کار می‌توانیم یک متد جدید در مدل تعریف کنیم که همان رابطه‌ی اصلی را با شرط اضافه برگرداند.  
```php  
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Model
{
    /**
     * Get the user's posts.
     */
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class)->latest();
    }

    /**
     * Get the user's featured posts.
     */
    public function featuredPosts(): HasMany
    {
        return $this->posts()->where('featured', true);
    }
}  
```  
اگر بخواهید از طریق متد featuredPosts یک مدل جدید بسازید، مقدار featured به صورت خودکار روی true ست نمی‌شود. چون فقط شرط where اضافه شده است.  
 
برای حل این مشکل لاراول متدی به نام withAttributes ارائه کرده که علاوه بر اضافه کردن شرط، مقادیر مشخص‌شده را به‌طور خودکار هنگام ایجاد مدل جدید هم اعمال می‌کند:  
```php  
/**
 * Get the user's featured posts.
 */
public function featuredPosts(): HasMany
{
    return $this->posts()->withAttributes(['featured' => true]);
}  
```  
**مزیت withAttributes**

1- شرط where روی رابطه اعمال می‌شود.

2- هنگام ساختن مدل جدید، مقدار featured به صورت خودکار برابر true قرار می‌گیرد.  
  
اگر بخواهید مقدار پیش‌فرض روی مدل‌های جدید ست شود ولی در کوئری where اعمال نشود، می‌توانید پارامتر دوم withAttributes(asConditions) را برابر false بگذارید:    
```php  
return $this->posts()->withAttributes(['featured' => true], asConditions: false); 
``` 
  
## Many to Many Relationships  
ارتباط چند به چند (Many to Many) نسبت به hasOne و hasMany کمی پیچیده‌تر است. در این نوع رابطه، یک رکورد می‌تواند به چند رکورد دیگر وابسته باشد و برعکس.  
  
فرض کنید یک کاربر (User) می‌تواند چند نقش (Role) داشته باشد. برای نمونه یک کاربر ممکن است نقش نویسنده و ویرایشگر داشته باشد. همچنین هر نقش هم می‌تواند به چند کاربر اختصاص داده شود. مثلاً نقش نویسنده می‌تواند برای چندین کاربر تعریف شده باشد.

پس داریم:

یک کاربر می‌تواند چند نقش داشته باشد.

یک نقش می‌تواند برای چند کاربر تعریف شود.    

  
برای پیاده‌سازی این رابطه در دیتابیس، به سه جدول نیاز داریم:

جدول users برای کاربران

جدول roles برای نقش‌ها

جدول میانی role_user که ارتباط بین کاربر و نقش را نگه‌داری می‌کند.

این جدول میانی معمولاً از ترکیب نام دو جدول اصلی به‌صورت الفبایی ساخته می‌شود. ستون‌های آن شامل:

user_id برای شناسه کاربر

role_id برای شناسه نقش  
  
اگر می‌خواستیم تنها یک ستون user_id در جدول roles قرار دهیم، در این صورت هر نقش فقط می‌توانست به یک کاربر اختصاص یابد. اما هدف این است که یک نقش بتواند بین چند کاربر مشترک باشد. بنابراین استفاده از جدول میانی الزامی است.  
  
بیایید نگاهی به ساختار جدول ها بکنیم :      

**users**  
    1- id - integer  
    2- name - string  

**roles**  
    1- id - integer  
    2- name - string  

**role_user**  
    1- user_id - integer  
    2- role_id - integer
      
---
### Model Structure  
برای پیاده‌سازی رابطه‌ی Many-to-Many در مدل‌ها، از متد belongsToMany استفاده می‌کنیم. این متد به‌صورت پیش‌فرض در کلاس پایه‌ی Illuminate\Database\Eloquent\Model که همه‌ی مدل‌های Eloquent از آن ارث‌بری می‌کنند، وجود دارد.

به عنوان مثال، می‌خواهیم در مدل User متدی برای ارتباط با Role تعریف کنیم:  
  
```php  
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class User extends Model
{
    /**
     * نقش‌هایی که به کاربر تعلق دارند.
     */
    public function roles(): BelongsToMany
    {
        return $this->belongsToMany(Role::class);
    }
}  
```  
وقتی این رابطه تعریف شد، می‌توانیم با استفاده از property داینامیک roles به نقش‌های کاربر دسترسی داشته باشیم:  
```php  
use App\Models\User;

$user = User::find(1);

foreach ($user->roles as $role) {
    echo $role->name;
}  
```  
از آنجایی که تمام روابط Eloquent در واقع query builder هم هستند، می‌توانیم روی آن‌ها شرط و کوئری‌های اضافی قرار دهیم. مثلاً مرتب‌سازی نقش‌های کاربر بر اساس نام:  
```php  
$roles = User::find(1)
            ->roles()
            ->orderBy('name')
            ->get();  
```  
  
### تعیین نام جدول میانی و کلیدها  
به صورت پیش‌فرض، لاراول نام جدول میانی  را با ترکیب نام دو مدل مرتبط و بر اساس ترتیب حروف الفبا تعیین می‌کند. مثلاً در رابطه‌ی بین User و Role، جدول میانی به صورت role_user نام‌گذاری می‌شود.

اما شما می‌توانید این نام را به‌صورت دستی هم تغییر دهید. برای این کار کافیست نام جدول میانی را به‌عنوان آرگومان دوم به متد belongsToMany بدهید:  
```php  
return $this->belongsToMany(Role::class, 'role_user');  
```  
همچنین می‌توانید نام ستون‌های کلید خارجی در جدول میانی را نیز تغییر دهید. آرگومان سوم نام کلید خارجی مربوط به مدلی است که در آن رابطه تعریف شده و آرگومان چهارم نام کلید خارجی مربوط به مدل مقابل است:  
```php  
return $this->belongsToMany(Role::class, 'role_user', 'user_id', 'role_id');  
```  
### تعریف رابطه معکوس  
برای تعریف رابطه‌ی معکوس در ارتباط‌های Many-to-Many، باید در مدل مرتبط نیز متدی تعریف کنیم که خروجی آن متد belongsToMany باشد.

برای تکمیل مثال User / Role، این بار در مدل Role متدی به نام users تعریف می‌کنیم:  
```php  
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Role extends Model
{
    /**
     * The users that belong to the role.
     */
    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class);
    }
}  
```  
همانطور که مشاهده می‌کنید، رابطه دقیقاً مشابه نمونه‌ای است که در مدل User تعریف کردیم، با این تفاوت که اینجا به مدل App\Models\User اشاره کرده‌ایم.

از آنجایی که همچنان از متد belongsToMany استفاده می‌کنیم، تمام قابلیت‌های مربوط به سفارشی‌سازی نام جدول میانی و کلیدهای خارجی نیز برای تعریف رابطه معکوس در دسترس هستند.  
  
  
## بازیابی ستون‌های جدول میانی  
همان‌طور که می‌دانیم، پیاده‌سازی رابطه‌ی Many-to-Many نیازمند وجود یک جدول میانی است. لاراول امکاناتی برای کار با این جدول فراهم می‌کند. به عنوان مثال، اگر مدل User با مدل‌های Role رابطه‌ی چند‌به‌چند داشته باشد، می‌توانیم پس از دسترسی به این رابطه، از طریق ویژگی pivot به جدول میانی دسترسی پیدا کنیم:  
  
```php  
use App\Models\User;

$user = User::find(1);

foreach ($user->roles as $role) {
    echo $role->pivot->created_at;
}  
```
>همان‌طور که می‌بینید، هر مدل Role که بازیابی می‌شود به‌صورت خودکار دارای ویژگی pivot خواهد بود. این ویژگی در واقع نمایانگر رکورد مربوطه در جدول میانی است.

به‌طور پیش‌فرض، فقط کلیدهای اصلی مدل‌ها در ویژگی pivot موجود هستند. اما اگر جدول میانی شامل ستون‌های اضافی باشد، باید هنگام تعریف رابطه آن‌ها را مشخص کنیم  
```php  
return $this->belongsToMany(Role::class)->withPivot('active', 'created_by');  
```  
>در این حالت ستون‌های active و created_by نیز به ویژگی pivot اضافه می‌شوند و قابل دسترسی خواهند بود.  
  
اگر بخواهیم جدول میانی دارای ستون‌های created_at و updated_at باشد و لاراول به‌طور خودکار آن‌ها را مدیریت کند، می‌توانیم از متد withTimestamps استفاده کنیم:  
```php  
return $this->belongsToMany(Role::class)->withTimestamps();  
```  
> برای استفاده از این قابلیت، جدول میانی باید حتماً هر دو ستون created_at و updated_at را داشته باشد.  
  
### سفارشی‌سازی نام ویژگی pivot   
همانطور که قبلاً توضیح داده شد، برای دسترسی به داده‌های جدول میانی در روابط Many-to-Many از ویژگی pivot استفاده می‌کنیم. اما در برخی موارد، ممکن است بخواهیم نام این ویژگی را تغییر دهیم تا متناسب‌تر با مفهوم پروژه باشد.

برای مثال، فرض کنید کاربران می‌توانند در پادکست‌ها عضو شوند. در این صورت رابطه‌ی many-to-many بین User و Podcast برقرار می‌شود. به جای استفاده از pivot، شاید بخواهیم نام ویژگی میانی را به subscription تغییر دهیم. این کار با متد as انجام می‌شود:  
```php  
  
return $this->belongsToMany(Podcast::class)
    ->as('subscription')
    ->withTimestamps();
```
اکنون به‌جای pivot، داده‌های جدول میانی با نام جدید قابل دسترسی هستند:  
```php  
$users = User::with('podcasts')->get();

foreach ($users->flatMap->podcasts as $podcast) {
    echo $podcast->subscription->created_at;
}
```
 این قابلیت باعث می‌شود کد خواناتر و متناسب‌تر با دامنه‌ی پروژه باشد؛ مخصوصاً وقتی داده‌های جدول میانی معنا و نقش مشخصی در سیستم دارند.  
   
## فیلتر کردن کوئری‌ها بر اساس ستون‌های جدول میانی   
  
یکی از قابلیت‌های مهم روابط Many-to-Many در لاراول این است که می‌توانید نتایج بازگشتی را بر اساس ستون‌های جدول میانی فیلتر کنید. این کار با استفاده از متدهای کمکی مانند wherePivot، wherePivotIn، wherePivotNotIn، wherePivotBetween، wherePivotNotBetween، wherePivotNull و wherePivotNotNull انجام می‌شود.

به چند نمونه توجه کنید:  
  
```php  
return $this->belongsToMany(Role::class)
    ->wherePivot('approved', 1);

return $this->belongsToMany(Role::class)
    ->wherePivotIn('priority', [1, 2]);

return $this->belongsToMany(Role::class)
    ->wherePivotNotIn('priority', [1, 2]);

return $this->belongsToMany(Podcast::class)
    ->as('subscriptions')
    ->wherePivotBetween('created_at', ['2020-01-01 00:00:00', '2020-12-31 00:00:00']);

return $this->belongsToMany(Podcast::class)
    ->as('subscriptions')
    ->wherePivotNotBetween('created_at', ['2020-01-01 00:00:00', '2020-12-31 00:00:00']);

return $this->belongsToMany(Podcast::class)
    ->as('subscriptions')
    ->wherePivotNull('expired_at');

return $this->belongsToMany(Podcast::class)
    ->as('subscriptions')
    ->wherePivotNotNull('expired_at');  
```  
همان‌طور که می‌بینید، این متدها امکان اعمال شرط‌های مختلف روی داده‌های جدول میانی را فراهم می‌کنند. به طور مثال، wherePivot مشابه یک where ساده عمل می‌کند اما روی ستون‌های جدول میانی.  
  
**نکته مهم:** استفاده از wherePivot فقط شرط روی کوئری اضافه می‌کند و هنگام ایجاد رکورد جدید مقدار مشخص‌شده را در جدول میانی ذخیره نمی‌کند. اگر بخواهید هم هنگام ایجاد و هم هنگام کوئری گرفتن یک مقدار خاص برای ستون جدول میانی تنظیم شود، باید از متد withPivotValue استفاده کنید:  
 
```php  
return $this->belongsToMany(Role::class)
    ->withPivotValue('approved', 1);  
```  
به این ترتیب، هم رکوردهای فیلترشده برگردانده می‌شوند و هم اگر رابطه‌ی جدیدی ایجاد شود، مقدار ستون approved به صورت خودکار روی 1 قرار می‌گیرد.  
  
## مرتب‌سازی کوئری‌ها از طریق ستون‌های جدول میانی  
گاهی اوقات نیاز داریم نتایج روابط Many-to-Many را بر اساس یکی از ستون‌های جدول میانی مرتب کنیم. برای این کار می‌توان از متد orderByPivot استفاده کرد.

به عنوان مثال، فرض کنید کاربر تعدادی Badge دارد و ما می‌خواهیم آخرین Badgeهای کاربر را که رتبه‌ی آن‌ها "gold" است بازیابی کنیم:  
```php  
return $this->belongsToMany(Badge::class)
    ->where('rank', 'gold')
    ->orderByPivot('created_at', 'desc');  
```  
## تعریف مدل سفارشی برای جدول میانی   
در برخی موارد نیاز داریم برای جدول میانی یک مدل سفارشی تعریف کنیم تا رفتارهای خاصی روی آن پیاده‌سازی کنیم. لاراول این امکان را با متد using فراهم می‌کند.

برای مثال، فرض کنید یک مدل Role داریم که به وسیله جدول میانی به مدل User متصل است و می‌خواهیم از مدل سفارشی RoleUser برای این جدول میانی استفاده کنیم:  
 
```php  
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\BelongsToMany;

class Role extends Model
{
    /**
     * The users that belong to the role.
     */
    public function users(): BelongsToMany
    {
        return $this->belongsToMany(User::class)->using(RoleUser::class);
    }
}  
```  
در این حالت باید یک مدل جدید برای جدول میانی ایجاد کنیم که از کلاس Pivot ارث‌بری کند:  
```php  
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Relations\Pivot;

class RoleUser extends Pivot
{
    // ...
}  
```  
>نکته: مدل‌های Pivot از ویژگی SoftDeletes پشتیبانی نمی‌کنند. اگر نیاز به حذف نرم (Soft Delete) دارید، باید جدول میانی را به یک مدل معمولی Eloquent تبدیل کنید.  
  
اگر جدول میانی شما کلید اصلی افزایشی (Auto Increment) دارد، باید داخل مدل Pivot سفارشی ویژگی incrementing را فعال کنید:  
```php  
/**
 * Indicates if the IDs are auto-incrementing.
 *
 * @var bool
 */
public $incrementing = true;  
```  
  
## Polymorphic Relationships  
یک رابطه چندریختی (Polymorphic Relationship) این امکان را فراهم می‌کند که یک مدل فرزند به بیش از یک نوع مدل والد وابسته باشد. به عبارت دیگر، می‌توان از یک ارتباط واحد برای چندین مدل مختلف استفاده کرد.

به عنوان مثال، فرض کنید در حال ساخت اپلیکیشنی هستید که کاربران می‌توانند پست‌های وبلاگ و ویدئوها را به اشتراک بگذارند. در چنین سیستمی، ممکن است مدل Comment هم به مدل Post و هم به مدل Video تعلق داشته باشد.  
  
### One to One (Polymorphic)  
#### Table Structure  
یک رابطه یک‌به‌یک چندریختی مشابه یک رابطه یک‌به‌یک معمولی است؛ با این تفاوت که مدل فرزند می‌تواند به بیش از یک نوع مدل والد وابسته باشد.

به عنوان مثال، فرض کنید یک Post و یک User هر دو می‌توانند یک تصویر (Image) داشته باشند. در این حالت می‌توان از یک رابطه چندریختی استفاده کرد تا یک جدول تصاویر داشته باشیم که هم به پست‌ها و هم به کاربران متصل شود.  
بیایید نگاهی به ساختار جداول بیندازیم :  
  
**posts**  
    1- id - integer  
    2- name - string  

**users**  
    1- id - integer  
    2- name - string  

**images**  
    1- id - integer  
    2- url - string  
    3- imageable_id - integer  
    4- imageable_type - string  
        

ستون imageable_id شامل شناسه (ID) مدل والد (مثل Post یا User) است.

ستون imageable_type شامل کلاس مدل والد خواهد بود (مثلاً App\Models\Post یا App\Models\User).

لاراول از ستون imageable_type برای تشخیص نوع والد استفاده می‌کند و بر اساس آن، رکورد درست را برمی‌گرداند.

به این ترتیب می‌توان تنها با یک جدول images چند مدل مختلف را مدیریت کرد.  
  
#### Model Structure  
برای پیاده‌سازی رابطه‌ی One-to-One Polymorphic، باید مدل‌های مربوطه را به شکل زیر تعریف کنیم:  
```php  
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class Image extends Model
{
    /**
     * Get the parent imageable model (user or post).
     */
    public function imageable(): MorphTo
    {
        return $this->morphTo();
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphOne;

class Post extends Model
{
    /**
     * Get the post's image.
     */
    public function image(): MorphOne
    {
        return $this->morphOne(Image::class, 'imageable');
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphOne;

class User extends Model
{
    /**
     * Get the user's image.
     */
    public function image(): MorphOne
    {
        return $this->morphOne(Image::class, 'imageable');
    }
}  
```  
##### Retrieving the Relationship  
بعد از ایجاد جداول و مدل‌ها، می‌توانیم داده‌های رابطه‌ای را به شکل زیر دریافت کنیم:

برای دریافت تصویر مربوط به یک پست:  
```php  
use App\Models\Post;

$post = Post::find(1);

$image = $post->image;  
```  
برای دریافت مدل والد از سمت جدول images (یعنی پیدا کردن اینکه این تصویر متعلق به Post است یا User):  
```php  
use App\Models\Image;

$image = Image::find(1);

$imageable = $image->imageable;  
```  
>متد imageable روی مدل Image یک رابطه‌ی morphTo است.
خروجی آن بسته به نوع رکورد می‌تواند یک نمونه‌ی Post یا User باشد.  

##### Key Conventions  
گاهی اوقات نیاز داریم نام ستون‌های مربوط به id و type که در مدل فرزند پلی‌مورفیک استفاده می‌شوند را تغییر دهیم. در این حالت باید هنگام تعریف متد morphTo این نام‌ها را مشخص کنیم. همچنین دقت داشته باشید که آرگومان اول متد morphTo باید نام رابطه باشد. معمولاً این مقدار برابر با نام متد است، بنابراین می‌توان از ثابت __FUNCTION__ در PHP استفاده کرد.  
```php  
/**
 * Get the model that the image belongs to.
 */
public function imageable(): MorphTo
{
    return $this->morphTo(__FUNCTION__, 'imageable_type', 'imageable_id');
}  
```  
اولین آرگومان که `__FUNCTION__` هست نام متد جاری (imageable) را به‌عنوان نام رابطه به morphTo پاس می‌دهد.  
ستون `imageable_type` مشخص می‌کند که تصویر به کدام مدل (مثل Post یا User) تعلق دارد.  
ستون `imageable_id` شناسه‌ی رکورد مربوط به آن مدل را نگه‌داری می‌کند.   

این قابلیت زمانی کاربرد دارد که بخواهید نام ستون‌های جدول را سفارشی کنید و همچنان رابطه‌ی پلی‌مورفیک به‌درستی کار کند.  
    
  
### One to Many (Polymorphic)  
  
#### Table Structure  
رابطه یک‌به‌چند پلی‌مورفیک مشابه رابطه‌ی معمولی یک‌به‌چند است؛ با این تفاوت که مدل فرزند می‌تواند به بیش از یک نوع مدل تعلق داشته باشد.

به عنوان مثال، تصور کنید کاربران برنامه شما می‌توانند روی پست‌ها (Posts) و ویدیوها (Videos) نظر بدهند. در این حالت، با استفاده از رابطه‌های پلی‌مورفیک می‌توانیم یک جدول comments داشته باشیم که کامنت‌های هر دو نوع (پست و ویدیو) را در خود نگه دارد.  
  
بیایید نگاهی به ساختار جداول بیندازیم :  
 
**posts**  
    1- id - integer  
    2- title - string  
    3- body - text

**videos**  
    1- id - integer  
    2- title - string  
    3- url - string

**comments**  
    1- id - integer  
    2- body - text  
    3- commentable_id - integer  
    4- commentable_type - string  
      
ستون commentable_id شناسه‌ی رکورد والد (پست یا ویدیو) را ذخیره می‌کند.

ستون commentable_type نام کلاس مدل والد (مثل App\Models\Post یا App\Models\Video) را نگه می‌دارد.

به این ترتیب لاراول با استفاده از این دو ستون متوجه می‌شود که هر کامنت به کدام مدل و رکورد تعلق دارد.  
  
#### Model Structure  
پس از آماده‌سازی جداول دیتابیس، باید مدل‌ها را تعریف کنیم تا بتوانیم ارتباط چند به یک چندریختی (One to Many Polymorphic) را پیاده‌سازی کنیم.  
```php  
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphTo;

class Comment extends Model
{
    /**
     * Get the parent commentable model (post or video).
     */
    public function commentable(): MorphTo
    {
        return $this->morphTo();
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphMany;

class Post extends Model
{
    /**
     * Get all of the post's comments.
     */
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphMany;

class Video extends Model
{
    /**
     * Get all of the video's comments.
     */
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable');
    }
}  
```  
##### Retrieving the Relationship  
حالا می‌توانیم داده‌ها را از طریق این روابط بازیابی کنیم. برای مثال، دسترسی به کامنت‌های یک پست:  
```php
use App\Models\Post;

$post = Post::find(1);

foreach ($post->comments as $comment) {
    // ...
}
```
و اگر بخواهیم به مدل والد یک کامنت (Post یا Video) دسترسی پیدا کنیم:  
```php  
use App\Models\Comment;

$comment = Comment::find(1);

$commentable = $comment->commentable;  
```
>در این حالت، $commentable می‌تواند یک نمونه از Post یا Video باشد، بسته به اینکه کامنت متعلق به کدام مدل است.  

#### Automatically Hydrating Parent Models on Children
حتی هنگام استفاده از eager loading در Eloquent، ممکن است مشکل کوئری "N + 1" ایجاد شود اگر بخواهید به مدل والد از مدل فرزند در هنگام پیمایش مجموعه فرزندان دسترسی پیدا کنید:  
```php 
$posts = Post::with('comments')->get();

foreach ($posts as $post) {
    foreach ($post->comments as $comment) {
        echo $comment->commentable->title;
    }
}  
```  
در مثال بالا، مشکل "N + 1" به وجود آمده است، زیرا حتی اگر کامنت‌ها برای هر مدل Post به صورت eager load شده باشند، Eloquent به طور خودکار مدل والد Post را در هر مدل Comment هیدراته نمی‌کند.

اگر می‌خواهید Eloquent والدین را به طور خودکار روی فرزندان هیدراته کند، می‌توانید از متد chaperone هنگام تعریف رابطه morphMany استفاده کنید:  
  
```php  
class Post extends Model
{
    /**
     * Get all of the post's comments.
     */
    public function comments(): MorphMany
    {
        return $this->morphMany(Comment::class, 'commentable')->chaperone();
    }
}  
```  
همچنین، اگر بخواهید به صورت runtime گزینه هیدراته کردن والدین را انتخاب کنید، می‌توانید هنگام eager loading رابطه از chaperone استفاده کنید:  
```php  
use App\Models\Post;

$posts = Post::with([
    'comments' => fn ($comments) => $comments->chaperone(),
])->get();  
```  
  
### One of Many (Polymorphic)  
گاهی اوقات یک مدل ممکن است چندین مدل مرتبط داشته باشد، اما شما می‌خواهید به راحتی به "جدیدترین" یا "قدیمی‌ترین" مدل مرتبط دسترسی پیدا کنید. به عنوان مثال، یک مدل User ممکن است به چند مدل Image مرتبط باشد، اما می‌خواهید یک روش راحت برای تعامل با آخرین تصویری که کاربر آپلود کرده، تعریف کنید. می‌توان این کار را با استفاده از رابطه morphOne و متدهای ofMany انجام داد:  
```php  
/**
 * Get the user's most recent image.
 */
public function latestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->latestOfMany();
}  
```  
به همین ترتیب، می‌توانید روشی برای دریافت "قدیمی‌ترین" یا اولین مدل مرتبط از رابطه تعریف کنید:  
```php  
/**
 * Get the user's oldest image.
 */
public function oldestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->oldestOfMany();
}  
```
به طور پیش‌فرض، متدهای latestOfMany و oldestOfMany مدل مرتبط را بر اساس کلید اصلی مدل که باید قابل مرتب‌سازی باشد، بازیابی می‌کنند. با این حال، گاهی ممکن است بخواهید یک مدل منفرد از یک رابطه بزرگتر را با استفاده از معیار مرتب‌سازی متفاوت دریافت کنید.

به عنوان مثال، با استفاده از متد ofMany، می‌توانید محبوب‌ترین تصویر کاربر را بازیابی کنید. متد ofMany ستون قابل مرتب‌سازی را به عنوان اولین آرگومان و تابع تجمیعی (min یا max) را هنگام پرس‌وجو برای مدل مرتبط می‌پذیرد:  
```php  
/**
 * Get the user's most popular image.
 */
public function bestImage(): MorphOne
{
    return $this->morphOne(Image::class, 'imageable')->ofMany('likes', 'max');
}  
```  
### Many to Many (Polymorphic)  

#### Table Structure  
روابط چند به چند پلی‌مورفیک کمی پیچیده‌تر از روابط morph one و morph many هستند. برای مثال، ممکن است مدل‌های Post و Video یک رابطه پلی‌مورفیک مشترک با مدل Tag داشته باشند. استفاده از رابطه چند به چند پلی‌مورفیک در این حالت اجازه می‌دهد که یک جدول یکتا برای تگ‌ها داشته باشیم که می‌تواند به پست‌ها یا ویدئوها مرتبط باشد.

ساختار جدول‌های لازم برای این رابطه به شرح زیر است:  
  
**posts**  
    1- id - integer  
    2- name - string  

**videos**  
    1- id - integer  
    2- name - string  

**tags**  
    1- id - integer  
    2- name - string  

**taggables**  
    1- tag_id - integer  
    2- taggable_id - integer  
    3- taggable_type - string  

  
#### Model Structure
بعد از تعریف جدول‌ها، باید روابط را در مدل‌ها مشخص کنیم. مدل‌های Post و Video هر دو یک متد tags خواهند داشت که متد morphToMany ارائه‌شده توسط کلاس پایه Eloquent را فراخوانی می‌کند.

متد morphToMany نام مدل مرتبط و همچنین نام رابطه را می‌پذیرد. بر اساس نامی که به جدول میانی اختصاص داده‌ایم و ستون‌های آن، نام رابطه را "taggable" قرار می‌دهیم:  
```php  
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphToMany;

class Post extends Model
{
    /**
     * Get all of the tags for the post.
     */
    public function tags(): MorphToMany
    {
        return $this->morphToMany(Tag::class, 'taggable');
    }
}  
```  
##### Defining the Inverse of the Relationship  
در مرحله بعد، باید روی مدل Tag برای هر مدل والد ممکن یک متد تعریف کنیم. در این مثال، دو متد posts و videos تعریف می‌کنیم. هر دو متد باید نتیجه‌ی فراخوانی morphedByMany را برگردانند.

متد morphedByMany نام مدل مرتبط و نام رابطه را دریافت می‌کند. بر اساس نام جدول میانی و کلیدهایی که تعریف کرده‌ایم، نام رابطه را "taggable" می‌گذاریم:  
```php 
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\MorphToMany;

class Tag extends Model
{
    /**
     * Get all of the posts that are assigned this tag.
     */
    public function posts(): MorphToMany
    {
        return $this->morphedByMany(Post::class, 'taggable');
    }

    /**
     * Get all of the videos that are assigned this tag.
     */
    public function videos(): MorphToMany
    {
        return $this->morphedByMany(Video::class, 'taggable');
    }
} 
```  
##### Retrieving the Relationship  
پس از تعریف جدول‌های پایگاه داده و مدل‌ها، می‌توانید از طریق مدل‌ها به روابط دسترسی پیدا کنید. برای مثال، برای دسترسی به تمام تگ‌های یک پست، می‌توان از ویژگی دینامیک tags استفاده کرد:  
```php  
use App\Models\Post;

$post = Post::find(1);

foreach ($post->tags as $tag) {
    // ...
}  
``` 
همچنین می‌توانید والد یک رابطه پلی‌مورفیک را از مدل فرزند پلی‌مورفیک دریافت کنید، با دسترسی به نام متدی که فراخوانی morphedByMany را انجام می‌دهد. در این مثال، متدهای posts و videos روی مدل Tag هستند:  
```php  
use App\Models\Tag;

$tag = Tag::find(1);

foreach ($tag->posts as $post) {
    // ...
}

foreach ($tag->videos as $video) {
    // ...
}  
```    
### Custom Polymorphic Types
به‌صورت پیش‌فرض، لاراول برای ذخیره‌ی نوع مدل مرتبط در ستون «type»، از نام کامل کلاس استفاده می‌کند. برای مثال، در رابطه‌ی One-to-Many Polymorphic که در آن یک Comment می‌تواند به یک Post یا Video تعلق داشته باشد، مقدار commentable_type به‌صورت پیش‌فرض App\Models\Post یا App\Models\Video خواهد بود.

اما ممکن است بخواهید این مقادیر را از ساختار داخلی اپلیکیشن جدا کنید. به عنوان مثال، به‌جای استفاده از نام کامل کلاس‌ها، می‌توانیم از رشته‌های ساده‌ای مثل post و video استفاده کنیم. با این کار، مقادیر ستون «type» در دیتابیس حتی در صورت تغییر نام یا جابجایی مدل‌ها معتبر باقی خواهند ماند.

برای این کار از متد enforceMorphMap استفاده می‌کنیم:  
  
```php  
use Illuminate\Database\Eloquent\Relations\Relation;

Relation::enforceMorphMap([
    'post' => 'App\Models\Post',
    'video' => 'App\Models\Video',
]);  
```
>شما می‌توانید این کد را در متد boot کلاس App\Providers\AppServiceProvider قرار دهید یا در صورت نیاز یک Service Provider مجزا برای آن ایجاد کنید.

#### دریافت نام مستعار (Alias) یا کلاس اصلی

برای دریافت alias مربوط به یک مدل در زمان اجرا، می‌توان از متد getMorphClass استفاده کرد.  
برای دریافت نام کامل کلاس مرتبط با یک alias نیز می‌توان از متد Relation::getMorphedModel استفاده کرد.  
```php  
use Illuminate\Database\Eloquent\Relations\Relation;

$alias = $post->getMorphClass();

$class = Relation::getMorphedModel($alias);  
```  
زمانی که یک morph map را به اپلیکیشن موجود اضافه می‌کنید، باید تمام مقادیر موجود در ستون‌های *_type که همچنان شامل نام کامل کلاس هستند را به مقادیر alias جدید تغییر دهید تا ناسازگاری پیش نیاید.  
  
## Dynamic Relationships
لاراول این امکان را فراهم می‌کند که به‌صورت داینامیک روابط بین مدل‌ها را در زمان اجرا تعریف کنید. این کار با استفاده از متد resolveRelationUsing انجام می‌شود. هرچند این روش معمولاً برای توسعه‌های عادی اپلیکیشن توصیه نمی‌شود، اما در مواقعی مثل توسعه پکیج‌ها می‌تواند بسیار مفید باشد.

متد resolveRelationUsing دو آرگومان می‌پذیرد:

آرگومان اول: نام رابطه‌ای که می‌خواهید تعریف کنید.

آرگومان دوم: یک Closure که مدل را دریافت کرده و یک تعریف رابطه Eloquent معتبر برمی‌گرداند.  
```php  
use App\Models\Order;
use App\Models\Customer;

Order::resolveRelationUsing('customer', function (Order $orderModel) {
    return $orderModel->belongsTo(Customer::class, 'customer_id');
});  
```  
>نکته: هنگام تعریف روابط داینامیک، همیشه نام کلیدهای خارجی (foreign keys) را به‌طور صریح مشخص کنید تا از خطاهای احتمالی جلوگیری شود.

این روش بیشتر برای زمانی مناسب است که بخواهید درون پکیج‌ها یا افزونه‌ها روابطی را بدون تغییر مستقیم در کد مدل‌ها ایجاد کنید.    
  
## Querying Relations  
از آنجایی که تمام روابط Eloquent از طریق متدها تعریف می‌شوند، شما می‌توانید این متدها را صدا بزنید تا یک نمونه از رابطه دریافت کنید، بدون اینکه فوراً کوئری اجرا شود. علاوه بر این، تمام انواع روابط Eloquent به عنوان query builder هم عمل می‌کنند، بنابراین می‌توانید محدودیت‌ها و شرط‌های بیشتری به کوئری اضافه کنید و در نهایت آن را روی دیتابیس اجرا کنید.

برای مثال، فرض کنید یک اپلیکیشن وبلاگ داریم که در آن مدل User دارای چندین مدل Post است:  
```php  
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;
use Illuminate\Database\Eloquent\Relations\HasMany;

class User extends Model
{
    /**
     * Get all of the posts for the user.
     */
    public function posts(): HasMany
    {
        return $this->hasMany(Post::class);
    }
}  
```  
اکنون می‌توانیم رابطه posts را کوئری کنیم و شرط‌های اضافی روی آن اعمال کنیم:  
```php  
use App\Models\User;

$user = User::find(1);

$user->posts()->where('active', 1)->get();  
```  
در این مثال، ابتدا رابطه posts بازیابی می‌شود، سپس با استفاده از query builder شرط where اضافه شده و در نهایت نتایج گرفته می‌شوند.

### Chaining orWhere Clauses After Relationships 
همانطور که در مثال قبل دیدیم، می‌توانیم هنگام کوئری گرفتن از روابط، شرط‌های اضافه اعمال کنیم. اما باید دقت کنیم وقتی که از orWhere استفاده می‌کنیم، چون این شرط‌ها هم‌سطح شرط رابطه قرار می‌گیرند و ممکن است منطق کوئری ما را تغییر دهند. 
```php  
$user->posts()
    ->where('active', 1)
    ->orWhere('votes', '>=', 100)
    ->get();  
```
این کد کوئری زیر را تولید می‌کند:  
```sql 
select *
from posts
where user_id = ? and active = 1 or votes >= 100  
```
همانطور که می‌بینید، شرط or باعث می‌شود که تمام پست‌هایی که بیش از 100 رأی دارند هم برگردند، حتی اگر متعلق به کاربر مورد نظر نباشند.   

راه‌حل درست استفاده از گروه‌بندی شرط‌ها با پرانتز است:  
```php  
use Illuminate\Database\Eloquent\Builder;

$user->posts()
    ->where(function (Builder $query) {
        return $query->where('active', 1)
            ->orWhere('votes', '>=', 100);
    })
    ->get();  
```
که این کوئری SQL زیر را تولید می‌کند:  
```sql 
select *
from posts
where user_id = ? and (active = 1 or votes >= 100)  
```
در این حالت، شرط orWhere به درستی در محدوده‌ی همان کاربر اعمال می‌شود و نتیجه‌ی صحیح بازگردانده خواهد شد.  
  
## Relationship Methods vs. Dynamic Properties  
در لاراول، وقتی یک رابطه (relationship) بین مدل‌ها تعریف می‌کنیم، دو روش برای دسترسی به داده‌های مرتبط داریم:

### استفاده از متد رابطه

اگر بخواهیم روی رابطه شرط یا کوئری اضافه کنیم (مثلاً where یا orderBy)، باید از متد رابطه استفاده کنیم: 
```php 
$user = User::find(1);

$activePosts = $user->posts()->where('active', 1)->get();
```
در این حالت posts() به عنوان یک query builder عمل می‌کند و می‌توانیم متدهای مختلف کوئری را زنجیره کنیم.

### استفاده از پراپرتی داینامیک
اگر نیازی به شرط اضافه نداشته باشیم، می‌توانیم مستقیماً رابطه را به صورت یک پراپرتی صدا بزنیم:
```php  
use App\Models\User;

$user = User::find(1);

foreach ($user->posts as $post) {
    // ...
}  
```
در این حالت لاراول رابطه را فقط زمانی که به آن دسترسی پیدا می‌کنیم Lazy Load می‌کند (یعنی داده‌ها را تنبل‌وار و فقط در زمان نیاز می‌آورد).


### Lazy Loading vs Eager Loading   

**Lazy Loading (بارگذاری تنبل):**   
وقتی اولین بار به رابطه دسترسی پیدا کنید، کوئری اجرا می‌شود. اگر در یک حلقه بارها این کار انجام شود، تعداد زیادی کوئری اجرا خواهد شد.

**Eager Loading (بارگذاری پیش‌دستانه):**     
  
  اگر می‌دانید که بعد از گرفتن مدل، قرار است روابطش را هم نیاز داشته باشید، می‌توانید از eager loading استفاده کنید تا همه داده‌ها با یک یا چند کوئری کمتر از قبل گرفته شوند:
  
  

```php
$users = User::with('posts')->get();
```
این کار باعث می‌شود همه پست‌های کاربران همراه با خود کاربران واکشی شوند و در نتیجه تعداد کوئری‌ها به شدت کاهش پیدا کند.
  
## Querying Relationship Existence  
گاهی اوقات لازم است هنگام واکشی رکوردهای مدل، نتایج را بر اساس وجود داشتن یا نداشتن یک رابطه محدود کنید. برای مثال، فرض کنید می‌خواهید تمام پست‌های وبلاگ را واکشی کنید که حداقل یک کامنت دارند. برای این کار می‌توانید از متدهای has یا orHas استفاده کنید:  
```php  
use App\Models\Post;

// Retrieve all posts that have at least one comment...
$posts = Post::has('comments')->get();
```
همچنین می‌توانید یک عملگر و مقدار مشخص کنید تا کوئری خود را دقیق‌تر کنید:  
```php  
// Retrieve all posts that have three or more comments...
$posts = Post::has('comments', '>=', 3)->get();
```
برای ساخت شرط‌های تو در تو، می‌توانید از نوتیشن `"dot"` استفاده کنید. مثلاً اگر بخواهید تمام پست‌هایی که حداقل یک کامنت دارند و آن کامنت‌ها نیز حداقل یک تصویر دارند را واکشی کنید:  
```php  
// Retrieve posts that have at least one comment with images...
$posts = Post::has('comments.images')->get();
```  
اگر نیاز به قدرت و انعطاف بیشتری داشتید، می‌توانید از متدهای `whereHas` و `orWhereHas` استفاده کنید تا محدودیت‌های بیشتری روی کوئری اعمال کنید؛ مثلاً بررسی محتوای کامنت‌ها:  
```php 
use Illuminate\Database\Eloquent\Builder;

// Retrieve posts with at least one comment containing words like code%...
$posts = Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'code%');
})->get();

// Retrieve posts with at least ten comments containing words like code%...
$posts = Post::whereHas('comments', function (Builder $query) {
    $query->where('content', 'like', 'code%');
}, '>=', 10)->get();
``` 
>نکته: در حال حاضر، Eloquent از کوئری زدن برای وجود رابطه‌ها در دیتابیس‌های متفاوت پشتیبانی نمی‌کند. همه‌ی روابط باید در یک دیتابیس وجود داشته باشند.  
 
### Many to Many Relationship Existence Queries  
متد whereAttachedTo در لاراول برای کوئری گرفتن روی مدل‌هایی استفاده می‌شود که از طریق یک رابطه‌ی Many to Many به یک مدل دیگر یا مجموعه‌ای از مدل‌ها متصل باشند.

به‌عنوان مثال، فرض کنید هر کاربر می‌تواند نقش‌های مختلفی داشته باشد. برای اینکه تمام کاربرانی را پیدا کنیم که به یک نقش خاص متصل هستند، می‌توانیم به شکل زیر عمل کنیم:
```php  
$users = User::whereAttachedTo($role)->get();  
```
همچنین می‌توانیم یک Collection از مدل‌ها را به متد whereAttachedTo بدهیم. در این صورت لاراول همه مدل‌هایی را برمی‌گرداند که به هرکدام از مدل‌های موجود در Collection متصل باشند:  
```php  
$tags = Tag::whereLike('name', '%laravel%')->get();

$posts = Post::whereAttachedTo($tags)->get();  
```
در مثال بالا، ابتدا همه‌ی تگ‌هایی که نامشان شامل عبارت «laravel» است را پیدا می‌کنیم، سپس همه‌ی پست‌هایی را می‌گیریم که به هرکدام از این تگ‌ها متصل باشند.

### Inline Relationship Existence Queries
گاهی اوقات می‌خواهید وجود یک رابطه را بررسی کنید اما تنها با یک شرط ساده روی همان رابطه. در چنین شرایطی، لاراول متدهای راحت‌تری مثل whereRelation ،orWhereRelation ،whereMorphRelation و orWhereMorphRelation را در اختیارتان قرار می‌دهد.

این متدها به شما امکان می‌دهند بدون نیاز به نوشتن whereHas طولانی، شرط روی رابطه را مستقیم در یک خط بنویسید.

مثال: فرض کنید می‌خواهیم همه‌ی پست‌هایی را بگیریم که کامنت تأیید نشده دارند: 
```php  
use App\Models\Post;

$posts = Post::whereRelation('comments', 'is_approved', false)->get();  
```
یا مثلا بخواهیم همه پست‌هایی را پیدا کنیم که یکی از کامنت‌هایشان در یک ساعت گذشته ایجاد شده باشد:  
```php  
$posts = Post::whereRelation(
    'comments', 'created_at', '>=', now()->subHour()
)->get();  
```  

## Querying Relationship Absence


گاهی اوقات ممکن است بخواهیم رکوردهایی از مدل‌ها را واکشی کنیم که هیچ رابطه‌ای با یک مدل دیگر ندارند. برای این کار می‌توانیم از متدهای doesntHave و orDoesntHave استفاده کنیم.

به عنوان مثال، فرض کنید می‌خواهیم همه‌ی پست‌هایی را بگیریم که هیچ کامنتی ندارند:
```php  
use App\Models\Post;

$posts = Post::doesntHave('comments')->get();  
```  
اگر بخواهیم شرط بیشتری روی رابطه‌هایی که وجود ندارند قرار دهیم، می‌توانیم از متدهای whereDoesntHave و orWhereDoesntHave استفاده کنیم. این متدها امکان بررسی محتوای رابطه‌ی ناموجود را می‌دهند:  
```php  
use Illuminate\Database\Eloquent\Builder;

$posts = Post::whereDoesntHave('comments', function (Builder $query) {
    $query->where('content', 'like', 'code%');
})->get();  
```
لاراول اجازه می‌دهد از "dot notation" برای بررسی روابط تو در تو استفاده کنیم. به عنوان مثال، می‌توانیم همه‌ی پست‌هایی را بگیریم که هیچ کامنتی ندارند یا کامنت‌هایشان مربوط به کاربرانی نیست که بن شده‌اند:  
```php  
use Illuminate\Database\Eloquent\Builder;

$posts = Post::whereDoesntHave('comments.author', function (Builder $query) {
    $query->where('banned', 1);
})->get();  
```
## Querying Morph To Relationships  
برای کوئری گرفتن از رابطه‌های morphTo، می‌توان از متدهای whereHasMorph و whereDoesntHaveMorph استفاده کرد. این متدها نام رابطه را به عنوان آرگومان اول دریافت می‌کنند. سپس نام مدل‌های مرتبطی که می‌خواهید در کوئری لحاظ شوند، به عنوان آرگومان دوم ارسال می‌شود. در نهایت می‌توانید یک closure برای سفارشی‌سازی کوئری رابطه ارائه دهید.  
```php  
use App\Models\Comment;
use App\Models\Post;
use App\Models\Video;
use Illuminate\Database\Eloquent\Builder;

// Retrieve comments associated to posts or videos with a title like code%...
$comments = Comment::whereHasMorph(
    'commentable',
    [Post::class, Video::class],
    function (Builder $query) {
        $query->where('title', 'like', 'code%');
    }
)->get();

// Retrieve comments associated to posts with a title not like code%...
$comments = Comment::whereDoesntHaveMorph(
    'commentable',
    Post::class,
    function (Builder $query) {
        $query->where('title', 'like', 'code%');
    }
)->get();  
```  
گاهی لازم است کوئری را بر اساس "نوع" مدل چندریختی فیلتر کنید. در این حالت، متد whereHasMorph می‌تواند یک پارامتر دوم به closure ارسال کند که نوع مدل مرتبط را مشخص می‌کند:  
```php  
use Illuminate\Database\Eloquent\Builder;

$comments = Comment::whereHasMorph(
    'commentable',
    [Post::class, Video::class],
    function (Builder $query, string $type) {
        $column = $type === Post::class ? 'content' : 'title';

        $query->where($column, 'like', 'code%');
    }
)->get();  
```
گاهی می‌خواهید فرزندان یک رابطه morphTo را بر اساس والدینشان کوئری کنید. برای این کار می‌توانید از متدهای whereMorphedTo و whereNotMorphedTo استفاده کنید. این متدها به طور خودکار نوع صحیح morph را برای مدل داده‌شده تشخیص می‌دهند:  
```php  
$comments = Comment::whereMorphedTo('commentable', $post)
    ->orWhereMorphedTo('commentable', $video)
    ->get();  
```

### Querying All Related Models
گاهی اوقات به جای اینکه یک آرایه از مدل‌های ممکن برای رابطه‌ی چندشکلی (morphTo) مشخص کنیم، می‌توانیم از کاراکتر وایلدکارت * استفاده کنیم. این کار به لاراول می‌گوید که همه‌ی مدل‌های ممکن برای آن رابطه را از دیتابیس واکشی کند. برای انجام این کار لاراول یک کوئری اضافه اجرا خواهد کرد.
```php  
use Illuminate\Database\Eloquent\Builder;

$comments = Comment::whereHasMorph('commentable', '*', function (Builder $query) {
    $query->where('title', 'like', 'foo%');
})->get();  
```  
  
## Aggregating Related Models  
### Counting Related Models  
