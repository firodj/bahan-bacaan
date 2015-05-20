XHP Template Engine pada Laravel 4
==================================

Limitations
-----------

Semenjak rilis HHVM dan XHP yang baru, agar bisa memakai fitur XHP component
dengan cara composition, pada file view harus diawali dengan open tag hhvm
**&lt;?hh** dan tidak boleh ditutup.

Pada Blade compiler hal tersebut tidak dimungkinkan sebab secara default tidak
ada open tag hhvm di awal file. Selain itu setiap syntax blade seperti
`@include`, `@section`, `@if`, `@foreach`, dst., akan diubah dan mengandung tag
**&lt;?php** dan **?&gt;**.

Sementara itu proses migrasi View template akan dilakukan secara perlahan dengan
masih menggunakan fitur pada Blade compiler.

<!--more-->

Examples
--------

File source: `index.blade.php`.
```
@extends('layout')

@section('main')
  @include('header')
  <p>Hello My Pren {$name}</p>
@stop
```

File compiled (*actual result*): `storage/.../index.php`
```
<?php $__env->startSection('main', ...); ?>
  <?php echo $__env->make('header', ...)->render(); ?>
  <p>Hello My Pren <?php echo $name ?></p>
<?php $__env->stopSection('main', ...); ?>

<?php echo $__env->make('layout')->render(); ?>
```

File template Blade adalah file HTML yang disisipi tag PHP.
Sedangkan pada XHP, file template adalah file script PHP dimana
`echo` berfungsi untuk menghasilkan HTML.

*Expected result* dari template Blade di atas seharusnya:

```
<?hh
$__env->startSection('main', ...);
  echo $__env->make('header', ...)->render();
  echo <p>Hello My Pren {$name}</p>;
$__env->stopSection();

echo $__env->make('layout')->render();
```

Dengan membandingkan *actual result* dan *expected result*, maka
konversi dari Blade menjadi XHP pada `XhpBladeCompiler`
(extends dari `Illuminate\View\Compilers\BladeCompiler`)
dengan *snippet* sebagai berikut:

1\. Tambah tag **&lt;?hh** pada awal file.

```
public function compile(...) {
  ...
  $this>files->put(..., "<?hh" . PHP_EOL . $contents);
  ...
}
```

2\. Buang tag **&lt;?php** dan **?&gt;** pada syntax `@extends`, `@section`,
`@stop`, dan `@include`.

```
protected function compileExtends(...) {...}
protected function compileSection(...) {...}
protected function compileStop(...) {...}
protected function compileInclude(...) {...}
```

3\. Ubah syntax pada tempate dari Blade menjadi XHP-Blade:
File source: `index.xhp.php`.

```
@extends('layout')

@section('main')
  @include('header')
  echo <p>Hello My Pren {$name}</p>;
@stop
```

4\. Register `XhpBladeCompiler` untuk file ber-ekstensi 'xhp.php'.

```
use Illuminate\\View\\Engines\\CompilerEngine;
use Illuminate\\Support\\ServiceProvider;

// @see: Illuminate\\View\\ViewServiceProvider
class XhpServiceProvider extends ServiceProvider {

  public function register() {
    $resolver = $this->app['view.engine.resolver'];

    $this->app->bindShared('xhp.compiler', function($app) {
      $cache = $app['path.storage'].'/views';
      return new XhpBladeCompiler($app['files'], $cache);
    });

    $resolver->register('xhp.engine', function() use($app) {
      return new CompilerEngine($app['xhp.compiler'], $app['files']);
    });

    View::addExtension('xhp.php', 'xhp.engine');
  }
}
```
