# Authentication

- [Introduction (введение)](#introduction)
    - [Database Considerations (Рекомендации по базам данных)](#introduction-database-considerations)
- [Authentication Quickstart (Быстрый старт аутентификации)](#authentication-quickstart)
    - [Routing(Маршрутизация)](#included-routing)
    - [Views](#included-views)
    - [Authenticating](#included-authenticating)
    - [Retrieving The Authenticated User](#retrieving-the-authenticated-user)
    - [Protecting Routes](#protecting-routes)
    - [Login Throttling](#login-throttling)
- [Manually Authenticating Users](#authenticating-users)
    - [Remembering Users](#remembering-users)
    - [Other Authentication Methods](#other-authentication-methods)
- [HTTP Basic Authentication](#http-basic-authentication)
    - [Stateless HTTP Basic Authentication](#stateless-http-basic-authentication)
- [Social Authentication](https://github.com/laravel/socialite)
- [Adding Custom Guards](#adding-custom-guards)
- [Adding Custom User Providers](#adding-custom-user-providers)
    - [The User Provider Contract](#the-user-provider-contract)
    - [The Authenticatable Contract](#the-authenticatable-contract)
- [Events](#events)

<a name="introduction"></a>
## Introduction (введение)

> {tip} **Хотите быстро начать работу?** Просто выполните `php artisan make:auth` и `php artisan migrate` в новом приложении Laravel. Затем перейдите в свой браузер, по адресу `http://your-app.dev/register` или любой другой URL-адрес, назначенный вашему приложению. Эти две команды позаботятся о создании всей системы аутентификации!

Laravel делает внедрение аутентификации очень простым. Фактически, почти все настроено для вас из коробки. Файл конфигурации аутентификации находится по адресу `config/auth.php`, который содержит несколько хорошо документированных вариантов настройки поведения служб аутентификации.

По своей сути, средства аутентификации Laravel состоят из «охранников» и «провайдеров». Гвардейцы определяют, как пользователи аутентифицируются для каждого запроса. Например, Laravel отправляется с `session` который поддерживает состояние с использованием хранилища сеансов и файлов cookie.

Поставщики определяют, как пользователи извлекаются из вашего постоянного хранилища. Laravel поставляется с поддержкой для извлечения пользователей с использованием Eloquent и построителя запросов базы данных. Тем не менее, вы можете свободно определять дополнительных поставщиков по мере необходимости для своего приложения.

Не волнуйтесь, если это все сейчас сбивает с толку! Многим приложениям никогда не потребуется изменять конфигурацию аутентификации по умолчанию.

<a name="introduction-database-considerations"></a>
### Database Considerations (Рекомендации по базам данных)

По умолчанию, Laravel включает `App\User` [Eloquent model](/docs/{{version}}/eloquent) в вашей `app` каталог. Эта модель может использоваться с драйвером проверки подлинности Eloquent по умолчанию. Если ваше приложение не использует Eloquent, вы можете использовать `database` который использует построитель запросов Laravel.

При построении схемы базы данных для `App\User`, убедитесь, что длина столбца паролей не менее 60 символов. Поддержание длины столбца строки по умолчанию из 255 символов было бы хорошим выбором.

Кроме того, вы должны убедиться, что ваш `users` (or equivalent) таблица содержит нулевую, строку `remember_token` столбец из 100 символов. Этот столбец будет использоваться для хранения токена для пользователей, которые выбирают опцию «запомнить меня» при входе в ваше приложение.

<a name="authentication-quickstart"></a>
## Authentication Quickstart (Быстрый старт аутентификации)

Laravel поставляется с несколькими предустановленными контроллерами аутентификации, которые расположены в `App\Http\Controllers\Auth` namespace. Контроллер `RegisterController` обрабатывает новую регистрацию пользователя, `LoginController` обрабатывает аутентификацию, `ForgotPasswordController` обрабатывает ссылки электронной почты для сброса паролей и `ResetPasswordController` содержит логику сброса паролей. Каждый из этих контроллеров использует черту, чтобы включить их необходимые методы. Для многих приложений вам не потребуется изменять эти контроллеры вообще.

<a name="included-routing"></a>
### Routing (маршрутизация)

Laravel обеспечивает быстрый способ поднять все маршруты и представления, необходимые для аутентификации, с помощью одной простой команды:

    php artisan make:auth

Эта команда должна использоваться в новых приложениях и будет устанавливать виды макета, регистрации и входа в систему, а также маршруты для всех конечных точек аутентификации. Контроллер `HomeController` также будет сгенерирован для обработки запросов после входа в панель приложений вашего приложения.
<a name="included-views"></a>
### Views (Просмотры)

Как упоминалось в предыдущем разделе, `php artisan make:auth` команда создаст все виды, необходимые для аутентификации, и поместит их в `resources/views/auth` каталог.

Команда `make: auth` также создаст каталог `resources/views/auth`, содержащий базовый макет для вашего приложения. Все эти представления используют фреймворк Bootstrap CSS, но вы можете их настроить, как пожелаете.

<a name="included-authenticating"></a>
### Authenticating (Проверка подлинности)

Теперь, когда у вас установлены маршруты и представления для включенных контроллеров проверки подлинности, вы готовы регистрировать и проверять подлинность новых пользователей для своего приложения! Вы можете просто получить доступ к своему приложению в браузере, поскольку контроллеры проверки подлинности уже содержат логику (через свои черты) для аутентификации существующих пользователей и хранения новых пользователей в базе данных.

#### Path Customization (Настройка пути)

Когда пользователь успешно аутентифицирован, они будут перенаправлены на `/home` URI. Вы можете настроить местоположение перенаправления после аутентификации, указав `redirectTo` собственности на `LoginController`, `RegisterController`, и `ResetPasswordController`:

    protected $redirectTo = '/';

Если для пути перенаправления требуется логика пользовательского генерации, вы можете определить метод `redirectTo` вместо свойства `redirectTo`:

    protected function redirectTo()
    {
        return '/path';
    }

> {tip} Метод `redirectTo` будет иметь приоритет над атрибутом` redirectTo`.

#### Username Customization (Пользовательская настройка)

По умолчанию Laravel использует поле `email` для аутентификации. Если вы хотите настроить это, вы можете определить метод `username` на вашем` LoginController`:

    public function username()
    {
        return 'username';
    }

#### Guard Customization (Настройка)

Вы также можете настроить «guard», который используется для аутентификации и регистрации пользователей. Чтобы начать работу, определите метод `guard` на ваших` LoginController`, `RegisterController` и` ResetPasswordController`. Метод должен возвращать экземпляр защиты:

    use Illuminate\Support\Facades\Auth;

    protected function guard()
    {
        return Auth::guard('guard-name');
    }

#### Validation / Storage Customization (Настройка валидации / Хранения)

Чтобы изменить поля формы, которые требуются, когда новый пользователь регистрируется с вашим приложением или настроить, как новые пользователи будут сохранены в вашей базе данных, вы можете изменить класс «RegisterController». Этот класс отвечает за проверку и создание новых пользователей вашего приложения.

Метод `validator`` RegisterController` содержит правила проверки для новых пользователей приложения. Вы можете изменить этот метод, как хотите.

Метод `create` `RegisterController` отвечает за создание новых записей `App\User` в вашей базе данных с помощью [Eloquent ORM] (/docs/{{version}}/eloquent). Вы можете изменять этот метод в соответствии с потребностями вашей базы данных.

<a name="retrieving-the-authenticated-user"></a>
### Retrieving The Authenticated User

You may access the authenticated user via the `Auth` facade:

    use Illuminate\Support\Facades\Auth;

    // Get the currently authenticated user...
    $user = Auth::user();

    // Get the currently authenticated user's ID...
    $id = Auth::id();

Alternatively, once a user is authenticated, you may access the authenticated user via an `Illuminate\Http\Request` instance. Remember, type-hinted classes will automatically be injected into your controller methods:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Http\Request;

    class ProfileController extends Controller
    {
        /**
         * Update the user's profile.
         *
         * @param  Request  $request
         * @return Response
         */
        public function update(Request $request)
        {
            // $request->user() returns an instance of the authenticated user...
        }
    }

#### Determining If The Current User Is Authenticated

To determine if the user is already logged into your application, you may use the `check` method on the `Auth` facade, which will return `true` if the user is authenticated:

    use Illuminate\Support\Facades\Auth;

    if (Auth::check()) {
        // The user is logged in...
    }

> {tip} Even though it is possible to determine if a user is authenticated using the `check` method, you will typically use a middleware to verify that the user is authenticated before allowing the user access to certain routes / controllers. To learn more about this, check out the documentation on [protecting routes](/docs/{{version}}/authentication#protecting-routes).

<a name="protecting-routes"></a>
### Protecting Routes

[Route middleware](/docs/{{version}}/middleware) can be used to only allow authenticated users to access a given route. Laravel ships with an `auth` middleware, which is defined at `Illuminate\Auth\Middleware\Authenticate`. Since this middleware is already registered in your HTTP kernel, all you need to do is attach the middleware to a route definition:

    Route::get('profile', function () {
        // Only authenticated users may enter...
    })->middleware('auth');

Of course, if you are using [controllers](/docs/{{version}}/controllers), you may call the `middleware` method from the controller's constructor instead of attaching it in the route definition directly:

    public function __construct()
    {
        $this->middleware('auth');
    }

#### Specifying A Guard

When attaching the `auth` middleware to a route, you may also specify which guard should be used to authenticate the user. The guard specified should correspond to one of the keys in the `guards` array of your `auth.php` configuration file:

    public function __construct()
    {
        $this->middleware('auth:api');
    }

<a name="login-throttling"></a>
### Login Throttling

If you are using Laravel's built-in `LoginController` class, the `Illuminate\Foundation\Auth\ThrottlesLogins` trait will already be included in your controller. By default, the user will not be able to login for one minute if they fail to provide the correct credentials after several attempts. The throttling is unique to the user's username / e-mail address and their IP address.

<a name="authenticating-users"></a>
## Manually Authenticating Users

Of course, you are not required to use the authentication controllers included with Laravel. If you choose to remove these controllers, you will need to manage user authentication using the Laravel authentication classes directly. Don't worry, it's a cinch!

We will access Laravel's authentication services via the `Auth` [facade](/docs/{{version}}/facades), so we'll need to make sure to import the `Auth` facade at the top of the class. Next, let's check out the `attempt` method:

    <?php

    namespace App\Http\Controllers;

    use Illuminate\Support\Facades\Auth;

    class LoginController extends Controller
    {
        /**
         * Handle an authentication attempt.
         *
         * @return Response
         */
        public function authenticate()
        {
            if (Auth::attempt(['email' => $email, 'password' => $password])) {
                // Authentication passed...
                return redirect()->intended('dashboard');
            }
        }
    }

The `attempt` method accepts an array of key / value pairs as its first argument. The values in the array will be used to find the user in your database table. So, in the example above, the user will be retrieved by the value of the `email` column. If the user is found, the hashed password stored in the database will be compared with the hashed `password` value passed to the method via the array. If the two hashed passwords match an authenticated session will be started for the user.

The `attempt` method will return `true` if authentication was successful. Otherwise, `false` will be returned.

The `intended` method on the redirector will redirect the user to the URL they were attempting to access before being intercepted by the authentication middleware. A fallback URI may be given to this method in case the intended destination is not available.

#### Specifying Additional Conditions

If you wish, you may also add extra conditions to the authentication query in addition to the user's e-mail and password. For example, we may verify that user is marked as "active":

    if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
        // The user is active, not suspended, and exists.
    }

> {note} In these examples, `email` is not a required option, it is merely used as an example. You should use whatever column name corresponds to a "username" in your database.

#### Accessing Specific Guard Instances

You may specify which guard instance you would like to utilize using the `guard` method on the `Auth` facade. This allows you to manage authentication for separate parts of your application using entirely separate authenticatable models or user tables.

The guard name passed to the `guard` method should correspond to one of the guards configured in your `auth.php` configuration file:

    if (Auth::guard('admin')->attempt($credentials)) {
        //
    }

#### Logging Out

To log users out of your application, you may use the `logout` method on the `Auth` facade. This will clear the authentication information in the user's session:

    Auth::logout();

<a name="remembering-users"></a>
### Remembering Users

If you would like to provide "remember me" functionality in your application, you may pass a boolean value as the second argument to the `attempt` method, which will keep the user authenticated indefinitely, or until they manually logout. Of course, your `users` table must include the string `remember_token` column, which will be used to store the "remember me" token.

    if (Auth::attempt(['email' => $email, 'password' => $password], $remember)) {
        // The user is being remembered...
    }

> {tip} If you are using the built-in `LoginController` that is shipped with Laravel, the proper logic to "remember" users is already implemented by the traits used by the controller.

If you are "remembering" users, you may use the `viaRemember` method to determine if the user was authenticated using the "remember me" cookie:

    if (Auth::viaRemember()) {
        //
    }

<a name="other-authentication-methods"></a>
### Other Authentication Methods

#### Authenticate A User Instance

If you need to log an existing user instance into your application, you may call the `login` method with the user instance. The given object must be an implementation of the `Illuminate\Contracts\Auth\Authenticatable` [contract](/docs/{{version}}/contracts). Of course, the `App\User` model included with Laravel already implements this interface:

    Auth::login($user);

    // Login and "remember" the given user...
    Auth::login($user, true);

Of course, you may specify the guard instance you would like to use:

    Auth::guard('admin')->login($user);

#### Authenticate A User By ID

To log a user into the application by their ID, you may use the `loginUsingId` method. This method simply accepts the primary key of the user you wish to authenticate:

    Auth::loginUsingId(1);

    // Login and "remember" the given user...
    Auth::loginUsingId(1, true);

#### Authenticate A User Once

You may use the `once` method to log a user into the application for a single request. No sessions or cookies will be utilized, which means this method may be helpful when building a stateless API:

    if (Auth::once($credentials)) {
        //
    }

<a name="http-basic-authentication"></a>
## HTTP Basic Authentication

[HTTP Basic Authentication](https://en.wikipedia.org/wiki/Basic_access_authentication) provides a quick way to authenticate users of your application without setting up a dedicated "login" page. To get started, attach the `auth.basic` [middleware](/docs/{{version}}/middleware) to your route. The `auth.basic` middleware is included with the Laravel framework, so you do not need to define it:

    Route::get('profile', function () {
        // Only authenticated users may enter...
    })->middleware('auth.basic');

Once the middleware has been attached to the route, you will automatically be prompted for credentials when accessing the route in your browser. By default, the `auth.basic` middleware will use the `email` column on the user record as the "username".

#### A Note On FastCGI

If you are using PHP FastCGI, HTTP Basic authentication may not work correctly out of the box. The following lines should be added to your `.htaccess` file:

    RewriteCond %{HTTP:Authorization} ^(.+)$
    RewriteRule .* - [E=HTTP_AUTHORIZATION:%{HTTP:Authorization}]

<a name="stateless-http-basic-authentication"></a>
### Stateless HTTP Basic Authentication

You may also use HTTP Basic Authentication without setting a user identifier cookie in the session, which is particularly useful for API authentication. To do so, [define a middleware](/docs/{{version}}/middleware) that calls the `onceBasic` method. If no response is returned by the `onceBasic` method, the request may be passed further into the application:

    <?php

    namespace Illuminate\Auth\Middleware;

    use Illuminate\Support\Facades\Auth;

    class AuthenticateOnceWithBasicAuth
    {
        /**
         * Handle an incoming request.
         *
         * @param  \Illuminate\Http\Request  $request
         * @param  \Closure  $next
         * @return mixed
         */
        public function handle($request, $next)
        {
            return Auth::onceBasic() ?: $next($request);
        }

    }

Next, [register the route middleware](/docs/{{version}}/middleware#registering-middleware) and attach it to a route:

    Route::get('api/user', function () {
        // Only authenticated users may enter...
    })->middleware('auth.basic.once');

<a name="adding-custom-guards"></a>
## Adding Custom Guards

You may define your own authentication guards using the `extend` method on the `Auth` facade. You should place this call to `provider` within a [service provider](/docs/{{version}}/providers). Since Laravel already ships with an `AuthServiceProvider`, we can place the code in that provider:

    <?php

    namespace App\Providers;

    use App\Services\Auth\JwtGuard;
    use Illuminate\Support\Facades\Auth;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::extend('jwt', function ($app, $name, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\Guard...

                return new JwtGuard(Auth::createUserProvider($config['provider']));
            });
        }
    }

As you can see in the example above, the callback passed to the `extend` method should return an implementation of `Illuminate\Contracts\Auth\Guard`. This interface contains a few methods you will need to implement to define a custom guard. Once your custom guard has been defined, you may use this guard in the `guards` configuration of your `auth.php` configuration file:

    'guards' => [
        'api' => [
            'driver' => 'jwt',
            'provider' => 'users',
        ],
    ],

<a name="adding-custom-user-providers"></a>
## Adding Custom User Providers

If you are not using a traditional relational database to store your users, you will need to extend Laravel with your own authentication user provider. We will use the `provider` method on the `Auth` facade to define a custom user provider:

    <?php

    namespace App\Providers;

    use Illuminate\Support\Facades\Auth;
    use App\Extensions\RiakUserProvider;
    use Illuminate\Foundation\Support\Providers\AuthServiceProvider as ServiceProvider;

    class AuthServiceProvider extends ServiceProvider
    {
        /**
         * Register any application authentication / authorization services.
         *
         * @return void
         */
        public function boot()
        {
            $this->registerPolicies();

            Auth::provider('riak', function ($app, array $config) {
                // Return an instance of Illuminate\Contracts\Auth\UserProvider...

                return new RiakUserProvider($app->make('riak.connection'));
            });
        }
    }

After you have registered the provider using the `provider` method, you may switch to the new user provider in your `auth.php` configuration file. First, define a `provider` that uses your new driver:

    'providers' => [
        'users' => [
            'driver' => 'riak',
        ],
    ],

Finally, you may use this provider in your `guards` configuration:

    'guards' => [
        'web' => [
            'driver' => 'session',
            'provider' => 'users',
        ],
    ],

<a name="the-user-provider-contract"></a>
### The User Provider Contract

The `Illuminate\Contracts\Auth\UserProvider` implementations are only responsible for fetching a `Illuminate\Contracts\Auth\Authenticatable` implementation out of a persistent storage system, such as MySQL, Riak, etc. These two interfaces allow the Laravel authentication mechanisms to continue functioning regardless of how the user data is stored or what type of class is used to represent it.

Let's take a look at the `Illuminate\Contracts\Auth\UserProvider` contract:

    <?php

    namespace Illuminate\Contracts\Auth;

    interface UserProvider {

        public function retrieveById($identifier);
        public function retrieveByToken($identifier, $token);
        public function updateRememberToken(Authenticatable $user, $token);
        public function retrieveByCredentials(array $credentials);
        public function validateCredentials(Authenticatable $user, array $credentials);

    }

The `retrieveById` function typically receives a key representing the user, such as an auto-incrementing ID from a MySQL database. The `Authenticatable` implementation matching the ID should be retrieved and returned by the method.

The `retrieveByToken` function retrieves a user by their unique `$identifier` and "remember me" `$token`, stored in a field `remember_token`. As with the previous method, the `Authenticatable` implementation should be returned.

The `updateRememberToken` method updates the `$user` field `remember_token` with the new `$token`. The new token can be either a fresh token, assigned on a successful "remember me" login attempt, or when the user is logging out.

The `retrieveByCredentials` method receives the array of credentials passed to the `Auth::attempt` method when attempting to sign into an application. The method should then "query" the underlying persistent storage for the user matching those credentials. Typically, this method will run a query with a "where" condition on `$credentials['username']`. The method should then return an implementation of `Authenticatable`. **This method should not attempt to do any password validation or authentication.**

The `validateCredentials` method should compare the given `$user` with the `$credentials` to authenticate the user. For example, this method should probably use `Hash::check` to compare the value of `$user->getAuthPassword()` to the value of `$credentials['password']`. This method should return `true` or `false` indicating on whether the password is valid.

<a name="the-authenticatable-contract"></a>
### The Authenticatable Contract

Now that we have explored each of the methods on the `UserProvider`, let's take a look at the `Authenticatable` contract. Remember, the provider should return implementations of this interface from the `retrieveById` and `retrieveByCredentials` methods:

    <?php

    namespace Illuminate\Contracts\Auth;

    interface Authenticatable {

        public function getAuthIdentifierName();
        public function getAuthIdentifier();
        public function getAuthPassword();
        public function getRememberToken();
        public function setRememberToken($value);
        public function getRememberTokenName();

    }

This interface is simple. The `getAuthIdentifierName` method should return the name of the "primary key" field of the user and the `getAuthIdentifier` method should return the "primary key" of the user. In a MySQL back-end, again, this would be the auto-incrementing primary key. The `getAuthPassword` should return the user's hashed password. This interface allows the authentication system to work with any User class, regardless of what ORM or storage abstraction layer you are using. By default, Laravel includes a `User` class in the `app` directory which implements this interface, so you may consult this class for an implementation example.

<a name="events"></a>
## Events

Laravel raises a variety of [events](/docs/{{version}}/events) during the authentication process. You may attach listeners to these events in your `EventServiceProvider`:

    /**
     * The event listener mappings for the application.
     *
     * @var array
     */
    protected $listen = [
        'Illuminate\Auth\Events\Registered' => [
            'App\Listeners\LogRegisteredUser',
        ],

        'Illuminate\Auth\Events\Attempting' => [
            'App\Listeners\LogAuthenticationAttempt',
        ],

        'Illuminate\Auth\Events\Authenticated' => [
            'App\Listeners\LogAuthenticated',
        ],

        'Illuminate\Auth\Events\Login' => [
            'App\Listeners\LogSuccessfulLogin',
        ],

        'Illuminate\Auth\Events\Failed' => [
            'App\Listeners\LogFailedLogin',
        ],

        'Illuminate\Auth\Events\Logout' => [
            'App\Listeners\LogSuccessfulLogout',
        ],

        'Illuminate\Auth\Events\Lockout' => [
            'App\Listeners\LogLockout',
        ],
    ];
