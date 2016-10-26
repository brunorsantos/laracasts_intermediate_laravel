# laracasts_Intermediate Laravel

Lição feita com laravel 5.3

## Criando comandos(Bonus)

[Documentacao Laravel sobre tópico](https://laravel.com/docs/master/artisan)

Para criar comandos do artisan, basta executar(sem nenhuma opcao):
```sh
php artisan make:command ShowGreeting
```
Adiconará um arquivo em 'app\console\command\' em que a logica do comando ficara no medoto handle.

Obs: '$this->info()', retorna na linha de comando para o usuário

Deve se registrar o comando em 'app\Console\Kernel.php' no array commands:

```php
protected $commands = [
    \App\Console\Commands\ShowGretting::class,
];
```
No arquivo criado para o commando em 'app\Console\command' deve se criar uma assinatura para ser executada via php artisan []

Comando sem parametros
```php
protected $signature = 'laracasts:greet';
```

Comando com parametro obrigatório
```php
protected $signature = 'laracasts:greet {name}';
```

Usando parametro no handle:
```php
public function handle()
{
    $this->info('Hello man, ' . $this->argument('name'));
}
```

Comando com parametro opcional
```php
protected $signature = 'laracasts:greet {name?}';
```
Comando com parametro com valor default
```php
protected $signature = 'laracasts:greet {name=Joe}';
```

Comando com passando uma option
```php
protected $signature = 'laracasts:greet {name=Joe} {--greeting=Hi}';
```

Usando a option no handle:
```php
public function handle()
{
    $this->info($this->option('greeting')  .' ' . $this->argument('name'));
}
```
Poderiamos executar da forma:

```sh
php artisan laracasts:greet bruno --greeting=Oi
```


## Scheduling Commands and Tasks

[Documentacao Laravel sobre tópico](https://laravel.com/docs/master/scheduling)

Para agendar comandos, devemos ir em app\Console\Commands\Kernel.php

Utilizar a funcao Schedule, que registrará os agendamentos que devem ocorrer.
Os agendamentos podem executar comandos puros do bash ou entao comandos do artisan.

Exemplos de comandos para registrar Schedules:

Executando comandos de bash:

```php
protected function schedule(Schedule $schedule)
{
    $schedule->exec('touch foo.txt')->everyFiveMinutes();
}
```

Executando comandos do artisan:

```php
protected function schedule(Schedule $schedule)
{
     $schedule->command('laracasts:greet')->monthly()->sendOutputTo('path/to/file')->emailOutputTo('mail@mail.com');
     $schedule->command('laracasts:daily-report')->dailyAt('23:55')->thenPing('url');
}
```

Para funcionar deve se configurar o Cron para executar o seguinte comando artisan da seguinte forma, de minuto em minuto.

```sh
* * * * * php /path/to/artisan schedule:run >> /dev/null 2>&1
```

Ele verificará se pussui algo para ser executado no lista do seu projeto.

## The power of eventing

O arquivo que controla os evento está em /app/Providers/EventServiceProvider.php.
Em devem sem incluídos os mapeamentos eventos e um array de listener para cada eventos. Devemos registra no array listen:

```php
protected $listen = [
    'App\Events\UserWasBanned' => [
        'App\Listeners\EmailBanNotification',
    ],
```

O evento é a situacão que ocorre passivel de ter alumas ou uma ação. E os listeners são cada ação que esse evento causa, em que deve se manter cada um com uma unica responsabilidade.

Existe um comando artisan que gera eventos e listeners baseado no esta registrado no EventServiceProvider. (Gera apenas se já nao existir)

```sh
php artisan event:generate
```

Na classe criada em app\Events, pode ser adicionar no construtor, como o exemplo abaixo utilizando automatic injection do laravel com a classe a instancia de User.

```php
public function __construct(User $user)
{
    $this->user = $user;
}
```

No listener, criado em App\Listeners, também pode se utilizar  automatic injection no construtor para passar classes como Mailer, Logger, etc.

E no metodo hanlde, deve se efetuar a ação do listener. Em que é possivel utilizar os atributos do Event como: 

```php
public function handle(UserWasBanned $event)
{
    var_dump('Notify '. $event->user->name . ' foi banido do site.');
}
```

Para disparar um evento, usa se:

```php
event(new UserWasBanned($user));
```

## Containers, Aliases, and Contracts

Laravel utiliza contratos do illuminate para funcionalidades do framework como autenticacao, mail, hash.
Eles estao no caminho: vendor/laravel/framework/src/Illuminate/Contracts

Suas implementações tambem estao presentes no projeto em /vendor/laravel/framework/src/Illuminate

Cada componente do lavavel possui um service provider que é responsavel por associar o componente ao conteiner da aplicacao, utilizando o metodo 'register()'

Por exemplo, o service provider do componete de hash esta em: /vendor/laravel/framework/src/Illuminate/Hashing/HashServiceProvider.php

No medoto register, é chamando um metodo que adiciona a um array de bindings do container a assocciação entre a chave e o objeto referente a este componente.

Através do metodo app->make() (Da classe da aplicacao que extende o classe de container). 

Esses bindings são localizados mesmo passando o nome da classe ou contrato do illuminate completo. Isto é feito baseado em alias da classe Aplication.

Sendo assim, podemos chamar de várias formas um componente do illuminate,

Exemplo o metodo make da classe hash do Illuminate


```php
dd(Hash::make('password')); // Usa o facade associado em /config/app.php através do array de aliases
dd(bcrypt('password'));  // Usa o helper definido em vendor/laravel/framework/src/Illuminate/Foundation/helper.php que por fim chama app('hash')->make()
dd(app('hash')->make('password')); Chama diretamente o bind efetuado pelo service provider do componente
dd(app()['hash']->make('password')); Como a classe container (pai da aplication) implementa ArrayAccess pode se chamar via array os metodos
dd(app('Illuminate\Hashing\BcryptHasher')->make('password')); funciona pois procura na lista de alias pelo bind feito no conteiner.


```




## Http Middleware

As middlewares são definidas em /app/Http/Kernel.php.
A middlewares que sempre devem ser executadas (independentes da rota) ficam no array $middleware.
```php

protected $middleware = [
    \Illuminate\Foundation\Http\Middleware\CheckForMaintenanceMode::class,
];
```

Tambem são definidas middlewares que sempre são executadas dependendo do grupo que sua rota está. Elas são definidas no array $middlewareGroups.
```php
protected $middlewareGroups = [
    'web' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \App\Http\Middleware\VerifyCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],

    'api' => [
        'throttle:60,1',
        'bindings',
    ],
];
```

As middlewares que são executadas sob demanda(dependendo da rota) são definidas em $routeMiddleware, sendo nomeadas pela chamada em que será feita para ela.

```php
protected $routeMiddleware = [
    'auth' => \Illuminate\Auth\Middleware\Authenticate::class,
    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
    'can' => \Illuminate\Auth\Middleware\Authorize::class,
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
];
```

Assim, com sua middleware definida, você pode optar por executar a mesma chamando o metodo middleware nas rotas.
```php
Route::get('admin/profile', function () {
    //
})->middleware('auth');
```

Podemos criar um middleware utilizando o artisan

```sh
php artisan make:middleware MustBeSubscribed
```

Um exemplo de logica para este middleware seria

```php
public function handle($request, Closure $next)
{
    $user = $request->user();
    
    if ($user && $user->isSubscribed()){
        return $next($request);
    }
    return redirect('/');
}
```

É possivel passar parametros para o middleware utlizando :paramtero
```php
Route::get('admin/profile', function () {
    //
})->middleware('subscribed:yearly');
```

Em que ele é recebido normalmente na função 'handle'

public function handle($request, Closure $next, $plan = null)
{
    $user = $request->user();
    
    if ($user && $user->isSubscribed($plan)){
        return $next($request);
    }
    return redirect('/nao_e_seu_plano');
}
```
