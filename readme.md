# laracasts_Intermediate Laravel

Lição feita com laravel 5.3

## Criando pacotes(Bonus)

[Documentacao Laravel sobre tópico](https://laravel.com/docs/master/artisan)

Para criar pacotes do artisan, basta executar(sem nenhuma opcao):
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