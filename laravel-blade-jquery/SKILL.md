---
name: laravel-blade-jquery
description: >
  Use esta skill para criar projetos Laravel com Blade templates e jQuery.
  Ative sempre que o usuário mencionar: "criar projeto Laravel", "novo projeto
  Laravel", "scaffold Laravel", "projeto com Blade", "Laravel + jQuery",
  "estrutura Laravel MVC", ou qualquer variação de setup/inicialização de
  projeto Laravel. Cobre desde a instalação até a estrutura de arquivos,
  configuração de rotas, controllers, models, migrations, layouts Blade e
  integração com jQuery via CDN ou npm.
---

# Laravel + Blade + jQuery — Skill de Criação de Projeto

Esta skill guia a criação completa de um projeto Laravel usando Blade como
motor de templates e jQuery para interatividade no frontend.

---

## 1. Pré-requisitos

Antes de começar, verifique se o ambiente tem:

- PHP >= 8.2
- Composer >= 2.x
- Node.js >= 18 + npm (para assets)
- Banco de dados configurado (MySQL, SQLite, etc.)

Comando de verificação rápida:
```bash
php -v && composer -V && node -v && npm -v
```

---

## 2. Criação do Projeto

```bash
composer create-project laravel/laravel nome-do-projeto
cd nome-do-projeto
```

Configure o `.env`:
```env
APP_NAME="Meu Projeto"
APP_URL=http://localhost:8000
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=nome_do_banco
DB_USERNAME=root
DB_PASSWORD=
```

---

## 3. Estrutura de Diretórios Recomendada

```
nome-do-projeto/
├── app/
│   ├── Http/Controllers/     ← Controllers da aplicação
│   └── Models/               ← Eloquent Models
├── database/
│   ├── migrations/           ← Migrations de tabelas
│   └── seeders/              ← Dados iniciais
├── resources/
│   ├── views/
│   │   ├── layouts/
│   │   │   └── app.blade.php ← Layout principal
│   │   ├── components/       ← Componentes Blade reutilizáveis
│   │   └── [modulo]/         ← Views por módulo (ex: users/, posts/)
│   ├── css/
│   │   └── app.css
│   └── js/
│       └── app.js            ← jQuery e scripts globais
├── routes/
│   └── web.php               ← Rotas web
└── public/
    └── (assets compilados)
```

---

## 4. Layout Principal com Blade

Crie `resources/views/layouts/app.blade.php`:

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta name="csrf-token" content="{{ csrf_token() }}">
    <title>@yield('title', config('app.name'))</title>

    {{-- CSS --}}
    <link rel="stylesheet" href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/css/bootstrap.min.css">
    @stack('styles')
</head>
<body>

    @include('layouts.partials.navbar')

    <main class="container py-4">
        @include('layouts.partials.alerts')
        @yield('content')
    </main>

    @include('layouts.partials.footer')

    {{-- Scripts: jQuery primeiro, depois Bootstrap, depois scripts da página --}}
    <script src="https://code.jquery.com/jquery-3.7.1.min.js"></script>
    <script src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.0/dist/js/bootstrap.bundle.min.js"></script>

    {{-- CSRF para requisições Ajax com jQuery --}}
    <script>
        $.ajaxSetup({
            headers: { 'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content') }
        });
    </script>

    @stack('scripts')
</body>
</html>
```

> ℹ️ Veja `references/blade-patterns.md` para padrões avançados de Blade
> (componentes, slots, diretivas customizadas).

---

## 5. Alertas Flash (partial reutilizável)

Crie `resources/views/layouts/partials/alerts.blade.php`:

```html
@if(session('success'))
    <div class="alert alert-success alert-dismissible fade show" role="alert">
        {{ session('success') }}
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
@endif

@if(session('error'))
    <div class="alert alert-danger alert-dismissible fade show" role="alert">
        {{ session('error') }}
        <button type="button" class="btn-close" data-bs-dismiss="alert"></button>
    </div>
@endif

@if($errors->any())
    <div class="alert alert-danger">
        <ul class="mb-0">
            @foreach($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif
```

---

## 6. Controller CRUD padrão

```bash
php artisan make:controller NomeController --resource
php artisan make:model Nome -m    # Model + Migration juntos
```

Estrutura padrão do controller:

```php
<?php

namespace App\Http\Controllers;

use App\Models\Nome;
use Illuminate\Http\Request;

class NomeController extends Controller
{
    public function index()
    {
        $itens = Nome::latest()->paginate(10);
        return view('nome.index', compact('itens'));
    }

    public function create()
    {
        return view('nome.create');
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'campo' => 'required|string|max:255',
        ]);

        Nome::create($validated);

        return redirect()->route('nome.index')
                         ->with('success', 'Criado com sucesso!');
    }

    public function edit(Nome $nome)
    {
        return view('nome.edit', compact('nome'));
    }

    public function update(Request $request, Nome $nome)
    {
        $validated = $request->validate([
            'campo' => 'required|string|max:255',
        ]);

        $nome->update($validated);

        return redirect()->route('nome.index')
                         ->with('success', 'Atualizado com sucesso!');
    }

    public function destroy(Nome $nome)
    {
        $nome->delete();
        return redirect()->route('nome.index')
                         ->with('success', 'Removido com sucesso!');
    }
}
```

---

## 7. Rotas (routes/web.php)

```php
<?php

use Illuminate\Support\Facades\Route;
use App\Http\Controllers\NomeController;

Route::get('/', fn() => view('welcome'));

// Rotas de recurso (gera index, create, store, show, edit, update, destroy)
Route::resource('nomes', NomeController::class);

// Rota AJAX de exemplo
Route::post('/nomes/ajax-acao', [NomeController::class, 'ajaxAcao'])
     ->name('nomes.ajax');
```

---

## 8. jQuery + AJAX no Blade

Exemplo de uma view que usa jQuery para chamadas assíncronas:

```html
@extends('layouts.app')

@section('title', 'Lista de Nomes')

@section('content')
<div class="d-flex justify-content-between mb-3">
    <h1>Nomes</h1>
    <a href="{{ route('nomes.create') }}" class="btn btn-primary">+ Novo</a>
</div>

<table class="table table-striped" id="tabela-nomes">
    <thead>
        <tr><th>ID</th><th>Campo</th><th>Ações</th></tr>
    </thead>
    <tbody>
        @foreach($itens as $item)
        <tr id="linha-{{ $item->id }}">
            <td>{{ $item->id }}</td>
            <td>{{ $item->campo }}</td>
            <td>
                <a href="{{ route('nomes.edit', $item) }}" class="btn btn-sm btn-warning">Editar</a>
                <button class="btn btn-sm btn-danger btn-deletar"
                        data-id="{{ $item->id }}"
                        data-url="{{ route('nomes.destroy', $item) }}">
                    Deletar
                </button>
            </td>
        </tr>
        @endforeach
    </tbody>
</table>

{{ $itens->links() }}
@endsection

@push('scripts')
<script>
$(function () {

    // Delete via AJAX sem recarregar a página
    $('.btn-deletar').on('click', function () {
        const id  = $(this).data('id');
        const url = $(this).data('url');

        if (!confirm('Confirmar exclusão?')) return;

        $.ajax({
            url: url,
            type: 'POST',
            data: { _method: 'DELETE' },
            success: function () {
                $('#linha-' + id).fadeOut(300, function () { $(this).remove(); });
            },
            error: function () {
                alert('Erro ao deletar. Tente novamente.');
            }
        });
    });

});
</script>
@endpush
```

> ℹ️ Veja `references/jquery-patterns.md` para padrões avançados de jQuery
> (validação de formulários, upload de arquivos, polling, etc.).

---

## 9. Migration de exemplo

```php
Schema::create('nomes', function (Blueprint $table) {
    $table->id();
    $table->string('campo');
    $table->text('descricao')->nullable();
    $table->boolean('ativo')->default(true);
    $table->timestamps();
    $table->softDeletes(); // opcional: exclusão lógica
});
```

```bash
php artisan migrate
```

---

## 10. Comandos úteis de referência rápida

| Ação                        | Comando                                      |
|-----------------------------|----------------------------------------------|
| Criar controller resource   | `php artisan make:controller X --resource`   |
| Criar model + migration     | `php artisan make:model X -m`                |
| Criar componente Blade      | `php artisan make:component NomeComponente`  |
| Rodar migrations            | `php artisan migrate`                        |
| Seedar banco                | `php artisan db:seed`                        |
| Limpar cache de views       | `php artisan view:clear`                     |
| Iniciar servidor local      | `php artisan serve`                          |
| Listar todas as rotas       | `php artisan route:list`                     |

---

## Referências adicionais

- `references/blade-patterns.md` — Componentes, slots, diretivas, herança de layout
- `references/jquery-patterns.md` — AJAX, validação, DataTables, upload assíncrono
