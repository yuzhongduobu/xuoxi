# 本地

## 一 创建项目并修改 .env
1. 通过 `composer create-project --prefer-dist laravel/laravel function42_server` 下载项目

2. (optional) homestead.yaml 中， sites 和 databases 中添加新映射

```
sites:
   - map: function42.server
      to: /home/vagrant/code/function42_server/public

databases:
    - fun42db
```

3. (optional) 修改 Windows 的 C:\Windows\System32\drivers\etc 下的 hosts 文件

```
192.168.10.10 function42.server
```

4. (optional) 用 `vagrant provision` 重导 homestead 设置

5. 修改 /function42_server/.env

```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=fun42db
DB_USERNAME=root
DB_PASSWORD=happy
```

## 二 数据库

在 /function42_server 下执行 `php artisan make:model Count -m`

修改 /function42_server/database/migrations 中的 2018_03_11_090139_create_counts_table.php

```
public function up()
    {
        Schema::create('counts', function (Blueprint $table) {
            $table->increments('id');
            $table->bigInteger('count_num')->default(0);
            $table->timestamps();
        });
    }
```

进入 mysql 创建数据库 fun42db ，指令为 `create database fun42db;`

随后在 /function42_server 下执行 `php artisan migrate`

再次进入 mysql ，在数据库 fun42db 下为表 counts 创建一条数据，指令为 `insert into counts (count_num) values (0);`

## 三 创建 CountController

1. 在 /function42_server 下执行 `php artisan make:controller CountController --resource`

2. 修改 /function42_server/routes 下的 web.php ，新增如下：

```
Route::resource('count', 'CountController');
```

3. 修改 /function42_server/app/Http/Controllers 下的 CountController.php

```
use App\Count;

public function index()
{
    $count = Count::find(1)->count_num;
    return $count;

}

public function create()
{
    $count = Count::find(1)->count_num;
    $newcount = $count + 1;
    $res = Count::where('id', '=', 1) -> update(array('count_num' => $newcount));
    return $res;
}
```

## 四 新建 CrossHttp 用于跨域

1. 在 /function42_server 下执行 `php artisan make:middleware CrossHttp`

修改 /function42_server/app/Http/Middleware 下的 CrossHttp.php

```
public function handle($request, Closure $next)
{
  $response = $next($request);
  $response->header('Access-Control-Allow-Origin', '*');
  $response->header('Access-Control-Allow-Headers', 'Origin, Content-Type, Cookie, Accept, Authorization');
  $response->header('Access-Control-Allow-Methods', 'GET, POST, PATCH, PUT, OPTIONS');
  // $response->header('Access-Control-Allow-Credentials', 'true');
  return $response;
}
```

并在 /function42_server/app/Http 的 Kernel.php 中注册 CrossHttp

```
protected $middleware = [
    \Illuminate\Foundation\Http\Middleware\CheckForMaintenanceMode::class,
    \Illuminate\Foundation\Http\Middleware\ValidatePostSize::class,
    \App\Http\Middleware\TrimStrings::class,
    \Illuminate\Foundation\Http\Middleware\ConvertEmptyStringsToNull::class,
    \App\Http\Middleware\TrustProxies::class,

    \App\Http\Middleware\CrossHttp::class,
];
```

