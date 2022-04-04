## 数据库迁移 (13 提示)


### 无符号整型

作为迁移的外键，请使用 `unsignedInteger()`  类型或 `integer()->unsigned()` 来替代 `integer()` ，否则你会得到 SQL 错误。

```php
Schema::create('employees', function (Blueprint $table) {
    $table->unsignedInteger('company_id');
    $table->foreign('company_id')->references('id')->on('companies');
    // ...
});
```

同样，你可以用 `unsignedBigInteger()` 如果外键对应的是 `bigInteger()` 类型。

```php
Schema::create('employees', function (Blueprint $table) {
    $table->unsignedBigInteger('company_id');
});
```

### 迁移顺序

如果你想改变数据库迁移的顺序，只需要将文件按时间戳记命名， 就像 `2018_08_04_070443_create_posts_table.php` 改为 `2018_07_04_070443_create_posts_table.php` (从 `2018_08_04` 改成了 `2018_07_04`).

迁移是以字母顺序执行。

### 带时区的迁移字段

你知不知道在迁移中不止有 `timestamps()` 还有带时区的 `timestampsTz()` 。

```php
Schema::create('employees', function (Blueprint $table) {
    $table->increments('id');
    $table->string('name');
    $table->string('email');
    $table->timestampsTz();
});
```

同样，还有这么些列 `dateTimeTz()` ，` timeTz()` ， `timestampTz()` ， `softDeletesTz()`。

### 数据库迁移字段类型

迁移中有一些有趣的字段类型，下面是一些示例。

```php
$table->geometry('positions');
$table->ipAddress('visitor');
$table->macAddress('device');
$table->point('position');
$table->uuid('id');
```

在 [官方文档](https://laravel.com/docs/master/migrations#creating-columns) 中你可以找到全部的字段类型列表.

### 默认时间戳

当创建迁移文件时，你可以使用带
`useCurrent()` 和 `useCurrentOnUpdate()` 可选项的 `timestamp()` 类型，这将会设置使相应字段以 `CURRENT_TIMESTAMP` 作为默认值。

```php
$table->timestamp('created_at')->useCurrent();
$table->timestamp('updated_at')->useCurrentOnUpdate();
```

### 迁移状态

如果你想知道迁移是不是已经运行过了，不需要查看 "migrations" 表，你可以运行 `php artisan migrate:status` 命令来查看。

结果示例:

```
+------+------------------------------------------------+-------+
| Ran? | Migration                                      | Batch |
+------+------------------------------------------------+-------+
| Yes  | 2014_10_12_000000_create_users_table           | 1     |
| Yes  | 2014_10_12_100000_create_password_resets_table | 1     |
| No   | 2019_08_19_000000_create_failed_jobs_table     |       |
+------+------------------------------------------------+-------+
```

### 创建带空格的迁移

当你打入 `make:migration` 命令，你不 “必须” 在不同部分间使用下划线 _ 进行分隔，比如 `create_transactions_table` 。你可以把名字用引号引起来并把下划线换成空格。

```php
// This works
php artisan make:migration create_transactions_table

// But this works too
php artisan make:migration "create transactions table"
```

Source: [Steve O on Twitter](https://twitter.com/stephenoldham/status/1353647972991578120)

### 在另一列的后面创建一列

注意： 仅 MySQL 可用
如果你要在已经存在的表里增加一个新列，这个列不 “必须” 成为最后一列，你可以指定这个列创建在哪个列的后面

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('phone')->after('email');
});
```

如果你要在已经存在的表里增加一个新列，这个列不 “必须” 成为最后一列，你也可以指定这个列创建在哪个列的前面：

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('phone')->before('created_at');
});
```

如果你让创建的列排在表中的第一个，那么可以使用 `first` 方法。

```php
Schema::table('users', function (Blueprint $table) {
    $table->string('uuid')->first();
});
```

### 为已经存在的表生成迁移文件

如果你要为已经存的表生成迁移文件，而且你想让 Lavarel 来为你生成 Schema::table ()  代码，那么，在命令后面加入  "_in_xxxxx_table" 或"_to_xxxxx_table"，或者指明 "--table" 参数。
`php artisan make:migration change_fields_products_table` generates empty class

```php
class ChangeFieldsProductsTable extends Migration
{
    public function up()
    {
        //
    }
}
```

但是，加入 `in_xxxxx_table` `php artisan make:migration change_fields_in_products_table` 生成了填好了 `Schemma::table()` 的类。

```php
class ChangeFieldsProductsTable extends Migration
{
    public function up()
    {
        Schema::table('products', function (Blueprint $table) {
            //
        })
    };
}
```

同样，你可以指明 `--table` 参数 `php artisan make:migration whatever_you_want --table=products`

```php
class WhateverYouWant extends Migration
{
    public function up()
    {
        Schema::table('products', function (Blueprint $table) {
            //
        })
    };
}
```

### 执行迁移前先输出 SQL 语句

当输入 migrate --pretend 命令，你可以得到将在终端中执行的 SQL 查询。如果有需要的话调试 SQL 的方法，这是个很有趣的方法。

```php
// Artisan command
php artisan migrate --pretend
```

 [@zarpelon](https://github.com/zarpelon) 提供

### 匿名迁移

`Laravel`团队发布了`Laravel 8.37`版本 支持匿名迁移，解决了迁移命名冲突的问题。

这个问题的核心是 如果多个迁移有相同的类名 当尝试重新创建数据库时可能会导致问题

以下是一个来自 `pr `的例子

```php
use Illuminate\Database\Migrations\Migration;
use Illuminate\Database\Schema\Blueprint;
use Illuminate\Support\Facades\Schema;

return new class extends Migration {
    public function up(
    {
        Schema::table('people', function (Blueprint $table) {
            $table->string('first_name')->nullable();
        });
    }

    public function down()
    {
        Schema::table('people', function (Blueprint $table) {
            $table->dropColumn('first_name');
        });
    }
};
```

由 [@nicksdot](https://twitter.com/nicksdot/status/1432340806275198978)提供

### 给迁移添加注释

在迁移中你可以给字段添加 `comment` 提供有用的信息。

如果数据库被其他开发者管理,他们在任何操作之前可以看这些表结构的注释。

```php
$table->unsignedInteger('interval')
    ->index()
    ->comment('This column is used for indexing.')   
```

由 [@Zubairmohsin33](https://twitter.com/Zubairmohsin33/status/1442345998790107137)提供

### 表名表字段检测

你可以使用 `hasTable` 和 `hasColumn` 方法检测表或字段是否存在。

```php
if (Schema::hasTable('users')) {
    // The "users" table exists...
}
if (Schema::hasColumn('users', 'email')) {
    // The "users" table exists and has an "email" column...
}
```

 [@dipeshsukhia](https://github.com/dipeshsukhia)提供

