【注】默认使用homestead环境开发
# 一 项目环境配置和用户表设计
1. 创建新项目zhihu_app
```
composer create-project --prefer-dist laravel/laravel zhihu_app
```

2. homestead.yaml中，sites和databases中添加新映射
```
sites:
   - map: zhihu.test
      to: /home/vagrant/code/zhihu_app/public

databases:
    - zhihudb
```

3. 修改Windows的C:\Windows\System32\drivers\etc下的hosts文件
```
192.168.10.10 zhihu.test
```

3. 用vagrant provision重导homestead设置

4. 修改/zhihu_app/.env中和数据库有关项
```
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=zhihudb
DB_USERNAME=root
DB_PASSWORD=serect
```
5. 修改/zhihu_app/database/migrations下的2014_10_12_000000_create_users_table.php
设置数据库中的users表
```
class CreateUsersTable extends Migration
{
    /**
     * Run the migrations.
     *
     * @return void
     */
    public function up()
    {
        Schema::create('users', function (Blueprint $table) {
            $table->increments('id');
            $table->string('name')->unique();
            $table->string('email')->unique();
            $table->string('password');
            $table->string('avater');
            $table->string('confirmation_token');
            $table->smallInteger('is_active')->default(0);
            $table->integer('questions_count')->default(0);
            $table->integer('answers_count')->default(0);
            $table->integer('comments_count')->default(0);
            $table->integer('favorites_count')->default(0);
            $table->integer('likes_count')->default(0);
            $table->integer('followers_count')->default(0);
            $table->integer('followings_count')->default(0);
            $table->json('settings')->nullable();
            $table->rememberToken();
            $table->timestamps();
        });
    }

    /**
     * Reverse the migrations.
     *
     * @return void
     */
    public function down()
    {
        Schema::dropIfExists('users');
    }
}
```

6. vagrant ssh进入虚拟环境后，在/zhihu_app下进行migrate
```
php artisan migrate
```
这样在数据库zhihudb下就按2014_10_12_000000_create_users_table.php中设置的一样生成了users表

# 二 用户注册
1. 新增 sendcloud 服务
在 /zhihu_app 下执行 `composer require naux/sendcloud`
打开 /zhihu_app/config/app.php，顺手将 timezone 设置为 PRC，即调整时区。
在  `providers` 中添加 `Naux\Mail\SendCloudServiceProvider::class,`，即：
```
/*
 * Package Service Providers...
 */

Naux\Mail\SendCloudServiceProvider::class,
```
修改/zhihu_app/.env
```
MAIL_DRIVER=sendcloud

SEND_CLOUD_USER=   # 创建的 api_user
SEND_CLOUD_KEY=    # 分配的 api_key
```
注意上面的 api_user 和 api_key 是要在 https://www.sendcloud.net 下注册得到的
我感觉这个部分以后可以被别的方案取代，毕竟sendcloud要用手机号注册，很烦

2. 在 /zhihu_app 下执行`php artisan make:auth`
这一步完成后，浏览器访问 zhihu.test，页面右上角就会有注册/登陆按钮出现了。
而项目中则会出现或改变下列文件:
/zhihu_app/resources/views/auth/login.blade.php
/zhihu_app/resources/views/auth/register.blade.php
/zhihu_app/resources/views/auth/passwords/email.blade.php
/zhihu_app/resources/views/auth/passwords/reset.blade.php
/zhihu_app/resources/views/layouts/app.blade.php
/zhihu_app/resources/views/home.blade.php

3. 修改 /zhihu_app/app/User.php
```
class User extends Authenticatable
{
    use Notifiable;

    /**
     * The attributes that are mass assignable.
     *
     * @var array
     */
    protected $fillable = [
        'name', 'email', 'password', 'avater', 'confirmation_token'
    ];

    /**
     * The attributes that should be hidden for arrays.
     *
     * @var array
     */
    protected $hidden = [
        'password', 'remember_token',
    ];
}
```
往 fillable 中添加 avater 等

4. 修改/zhihu_app/app/Http/Controllers/Auth/RegisterController.php，并在/zhihu_app/public下新建/images/avaters，放入default.png
```
protected function create(array $data)
  {
    $user = User::create([
      'name' => $data['name'],
      'email' => $data['email'],
      'avater' => '/images/avaters/default.png',
      'confirmation_token' => str_random(40),
      'password' => bcrypt($data['password']),
    ]);
    $this->sendVerifyEmailTo($user);

    return $user;
  }
private function sendVerifyEmailTo($user)
  {
    $bind_data = [
      'url' => route('email.verify', ['token' => $user->confirmation_token]),
      'name' => $user->name
    ];
    $template = new SendCloudTemplate('zhihu_app_register', $bind_data);

    Mail::raw($template, function ($message) use ($user) {
      $message->from('rjs@1VPrrhrvkbw602A2b2TOFJaRGQBZYEJO.sendcloud.org', 'rsj');
      $message->to($user->email);
    });
  }
```
并在文件开头添加
```
use Mail;
use Naux\Mail\SendCloudTemplate;
```

5. 在 sendcloud 中创建 zhihu_app_register 模板
  见图
  ![zhihu_app_register](C:\Users\Luke\Desktop\zhihu_app_register.png)
  此时应该能使用 register 功能注册了，并且邮箱会收到确认邮箱的邮件。确认地址大概长这样：
  http://zhihu.test/email/verify/IROySiQPV4rKYHqz5tpzi5Y8S8rlLaLdFNGRj4TL

6. 下面完成确认邮箱的步骤。修改 /zhihu_app/routes/web.php
添加:
```
Route::get('email/verify/{token}', ['as' => 'email.verify', 'uses' => 'EmailController@verify']);
```

7. 在 /zhihu_app 下执行`php artisan make:controller EmailControlle`，新增EmailController

8. 修改/zhihu_app/app/Http/Controllers/EmailController.php
```
class EmailController extends Controller
{
    public function verify($token)
    {
        $user = User::where('confirmation_token', $token)->first();

        if (is_null($user)) {
            return redirect('/');
        }

        $user->is_active = 1;
        $user->confirmation_token = str_random(40);
        $user->save();

        Auth::login($user);
        return redirect('/home');
    }
}
```
并在文件开头添加
```
use Auth;
use App\User;
```
此时点击邮箱确认链接，应该就能完成 is_active 的修改，完成激活，重置 confirmation_token ,并登陆

# 三 用户登录，细化登陆和注册的提示显示
1. 新增 flash 服务（github link）
在 /zhihu_app 下执行 `composer require laracasts/flash`
打开 /zhihu_app/config/app.php，在  `providers` 中添加 `Naux\Mail\SendCloudServiceProvider::class,`，即
```
/*
 * Package Service Providers...
 */
Laracasts\Flash\FlashServiceProvider::class,
```

2. 在 /zhihu_app/resources/views/layouts/app.blade.php 中添加
```
<div class="container">
  @include('flash::message')
</div>

<script>
  $('#flash-overlay-modal').modal();
</script>
```
用于显示 flash 生成的提示，下面那个 script 好像其实也不用加

3. 添加验证邮箱时的提示
修改 /zhihu_app/app/Http/Controllers/EmailController.php
```
public function verify($token)
  {
    $user = User::where('confirmation_token', $token)->first();

    if (is_null($user)) {
      flash('邮箱验证失败','danger');
      return redirect('/');
    }

    $user->is_active = 1;
    $user->confirmation_token = str_random(40);
    $user->save();

    Auth::login($user);
    flash('邮箱验证成功','success');
    return redirect('/home');
  }
```

4. 添加登陆时的提示
修改 /zhihu_app/app/Http/Controllers/Auth/LoginController.php
在末尾添加两个函数
```
public function login(Request $request)
{
  $this->validateLogin($request);

  // If the class is using the ThrottlesLogins trait, we can automatically throttle
  // the login attempts for this application. We'll key this by the username and
  // the IP address of the client making these requests into this application.
  if ($this->hasTooManyLoginAttempts($request)) {
    $this->fireLockoutEvent($request);
    return $this->sendLockoutResponse($request);
  }

  if ($this->attemptLogin($request)) {
    flash('欢迎回来', 'success');
    return $this->sendLoginResponse($request);
  }

  // If the login attempt was unsuccessful we will increment the number of attempts
  // to login and redirect the user back to the login form. Of course, when this
  // user surpasses their maximum number of attempts they will get locked out.
  $this->incrementLoginAttempts($request);

  return $this->sendFailedLoginResponse($request);
}

protected function attemptLogin(Request $request)
{
  $credentials = array_merge($this->credentials($request),['is_active' =>1]);
  return $this->guard()->attempt(
    $credentials, $request->filled('remember')
  );
}
```
实际这两个函数都是从 \zhihu_app\vendor\laravel\framework\src\Illuminate\Foundation\Auth\AuthenticatesUsers.php 里拷出来的
随后再修改一下 \zhihu_app\resources\lang\en\auth.php 中的 failed 值，这是因为 sendFailedLoginResponse 函数会调用这个值作为登陆失败的提示
```
'failed' => '密码错误或者邮箱未验证',
```


# 四 本地化和自定义消息
本地化略
自定义消息通过修改 \zhihu_app\resources\lang\en 下的文件来完成
例如 validation.php
```
'custom' => [
    'email' => [
        'unique' => '邮箱已注册',
    ],
    'password' => [
        'comfirmed' => '密码输入不相符',
    ],
],
```
可以自定义验证时的提示信息

# 五 实现找回密码功能
1. 本地化，更改消息提示，通过更改 \zhihu_app\resources\lang\en\password 来实现，略
2. 打开 \zhihu_app\app\user.php 来修改点击重置按钮后的邮件发送函数，需要将其改写为用 SendCloud 发送
在 User 这一 class 下，添加函数 sendPasswordResetNotification ，用来覆盖之前存在的方法
```
class User extends Authenticatable
{
    public function sendPasswordResetNotification($token)
    {
        $bind_data = ['url' => url('password/reset', $token)];
        $template = new SendCloudTemplate('zhihu_app_password_reset', $bind_data);

        Mail::raw($template, function ($message) {
            $message->from('rjs@1VPrrhrvkbw602A2b2TOFJaRGQBZYEJO.sendcloud.org', 'rsj');
            $message->to($this->email);
        });
    }
}
```
并在这一文件开头添加
```
use Mail;
use Naux\Mail\SendCloudTemplate;
```
3. 在 sendcloud 中添加模板 zhihu_app_password_reset
见图
![zhihu_app_password_reset](C:\Users\Luke\Desktop\zhihu_app_password_reset.png)
此时便可实现密码重置了

# 六 设计问题表
1. 在 /zhihu_app 下执行 `php artisan make:model Question -m`，会生成两个文件，分别在 \zhihu_app\database\migrations 和 \zhihu_app\app 下
2. 修改 \zhihu_app\database\migrations 下的 2018_03_08_094030_create_questions_table.php
其中 2018_03_08_094030 为创建时间，具体情形下会不同
```
public function up()
    {
        Schema::create('questions', function (Blueprint $table) {
            $table->increments('id');
            $table->string('title');
            $table->text('body');
            $table->integer('user_id')->unsigned();
            $table->integer('comments_count')->default(0);
            $table->integer('followers_count')->default(1);
            $table->integer('answers_count')->default(0);
            $table->string('close_comment',8)->default('F');
            $table->string('is_hidden',8)->default('F');
            $table->timestamps();
        });
    }
```
3. 在 /zhihu_app 下执行 `php artisan migrate`

# 七 发布问题
1. 新增 ueditor 模块 github.com/overtrue/laravel-ueditor
在 \zhihu_app 下执行 `composer require "overtrue/laravel-ueditor:~1.0"`
修改 \zhihu_app\config\app.php，在 providers 下 添加
```
Overtrue\LaravelUEditor\UEditorServiceProvider::class,
```
在 \zhihu_app 下执行 `php artisan vendor:publish --provider='Overtrue\LaravelUEditor\UEditorServiceProvider'` 发布配置文件与资源，多出很多个文件，命令行里能看到的有：
/config/ueditor.php
/vendor/overtrue/laravel-ueditor/src/assets/ueditor] To [/public/vendor/ueditor
/vendor/overtrue/laravel-ueditor/src/views] To [/resources/views/vendor/ueditor
/vendor/overtrue/laravel-ueditor/src/translations] To [/resources/lang/vendor/ueditor

2. 在 \zhihu_app\resources\views\ 目录下新建目录 questions，在 questions 文件夹下创建文件 create.blade.php
内容如下：
```
@extends('layouts.app')
@section('content')
    @include('vendor.ueditor.assets')
<div class="container">
    <div class="row justify-content-center">
        <div class="col-md-8">
            <div class="card">
                <div class="card-header">发布问题</div>
                <div class="card-body">
                    <form action="/questions" method="post">
                        {!! csrf_field() !!}
                        <div class="form-group">
                            <label for="title">标题</label>
                            <input type="text" name="title" class="form-control" placeholder="标题" id="title">
                        </div>
                        <script id="container" name="body" type="text/plain"></script>
                        
                        <button class="btn btn-success pull-right" type="submit">提交问题</button>
                    </form>
                </div>
            </div>
        </div>
    </div>
</div>

<script type="text/javascript">
    var ue = UE.getEditor('container');
    ue.ready(function() {
        ue.execCommand('serverparam', '_token', '{{ csrf_token() }}'); // 设置 CSRF token.
    });
</script>

@endsection
```

3. 在 \zhihu_app 下执行 `php artisan make:controller QuestionsController --resource`
修改 \zhihu_app\app\Http\Controllers\QuestionsController.php
```
public function index()
    {
        return 'create question index.';
    }

public function create()
    {
        return view('questions.create');
    }

public function store(Request $request)
    {
        $data =[
            'title' => $request->get('title'),
            'body' => $request->get('body'),
            'user_id' => Auth::id()
        ];

        $question = Question::create($data);
        return redirect()->route('questions.show', [$question->id]);
    }

public function show($id)
    {
        $question = Question::find($id);

        return view('questions.show',compact('question'));
    }
```
并在开头添加
```
use Auth;
use App\Question;
```

4. 在 \zhihu_app\routes\web.php 中添加：
```
Route::resource('questions', 'QuestionsController', ['name' => [
  'create' => 'questions.create',
  'show' => 'questions.show',
]]);
```
此时已经可以访问 http://zhihu.test/questions/create 了

5. 创建并修改 \zhihu_app\resources\views\show.blade.php
```
<div class="card">
    <div class="card-header">发布问题</div>
    <div class="card-body">
        <form action="/questions" method="post">
            {!! csrf_field() !!}
            <div class="form-group">
                <label for="title">标题</label>
                <input type="text" name="title" class="form-control" placeholder="标题" id="title">
            </div>
            <script id="container" name="body" type="text/plain"></script>
            
            <button class="btn btn-success pull-right" type="submit">提交问题</button>
        </form>
    </div>
</div>
```

6. 修改 \zhihu_app\app\Question.php
```
class Question extends Model
{
    protected $fillable = ['title', 'body', 'user_id'];
}
```
综上可实现创建问题并自动跳转到展示界面


# 八 验证问题表单字段
修改 \zhihu_app\app\Question.php
```
public function store(Request $request)
    {
        $rules = [
            'title' => 'require|min:6|max:196',
            'body' => 'require|min:36'
        ];
        $this->validate($request,$rules);
        $data =[
            'title' => $request->get('title'),
            'body' => $request->get('body'),
            'user_id' => Auth::id()
        ];

        $question = Question::create($data);
        return redirect()->route('questions.show', [$question->id]);
    }
```
错误提示等修改略去 有好些验证方法啊啥的，我并不想看

# 九 