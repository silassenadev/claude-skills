# Blade — Padrões Avançados

## Componentes Blade

### Criar um componente
```bash
php artisan make:component Card
# Gera: app/View/Components/Card.php + resources/views/components/card.blade.php
```

### Usando o componente na view
```html
<x-card title="Meu Card" class="mt-4">
    <p>Conteúdo do card aqui.</p>
</x-card>
```

### Definindo o componente (card.blade.php)
```html
@props(['title' => 'Título padrão'])

<div {{ $attributes->merge(['class' => 'card shadow-sm']) }}>
    <div class="card-header">
        <h5 class="mb-0">{{ $title }}</h5>
    </div>
    <div class="card-body">
        {{ $slot }}
    </div>
</div>
```

---

## Slots Nomeados

```html
{{-- Componente com múltiplos slots --}}
<x-modal title="Confirmar">
    <x-slot:body>
        <p>Tem certeza?</p>
    </x-slot:body>
    <x-slot:footer>
        <button class="btn btn-danger">Confirmar</button>
    </x-slot:footer>
</x-modal>
```

```html
{{-- modal.blade.php --}}
@props(['title'])
<div class="modal-dialog">
    <div class="modal-header">{{ $title }}</div>
    <div class="modal-body">{{ $body }}</div>
    <div class="modal-footer">{{ $footer }}</div>
</div>
```

---

## Diretivas Úteis

```html
{{-- Condicionais --}}
@if($user->isAdmin())
    <span class="badge bg-danger">Admin</span>
@elseif($user->isModerator())
    <span class="badge bg-warning">Moderador</span>
@else
    <span class="badge bg-secondary">Usuário</span>
@endif

{{-- @auth e @guest --}}
@auth
    <a href="{{ route('dashboard') }}">Painel</a>
@endauth

@guest
    <a href="{{ route('login') }}">Entrar</a>
@endguest

{{-- Loop com $loop --}}
@foreach($itens as $item)
    <tr class="{{ $loop->even ? 'table-light' : '' }}">
        <td>{{ $loop->iteration }}</td>  {{-- 1, 2, 3... --}}
        <td>{{ $item->nome }}</td>
        @if($loop->last)
            <td><strong>Último item</strong></td>
        @endif
    </tr>
@endforeach

{{-- @forelse (com fallback para lista vazia) --}}
@forelse($itens as $item)
    <li>{{ $item->nome }}</li>
@empty
    <li class="text-muted">Nenhum item encontrado.</li>
@endforelse
```

---

## @stack e @push

Útil para injetar scripts/estilos específicos de uma view no layout:

```html
{{-- No layout (app.blade.php) --}}
@stack('styles')   {{-- dentro do <head> --}}
@stack('scripts')  {{-- antes do </body> --}}

{{-- Na view filha --}}
@push('styles')
    <link rel="stylesheet" href="/css/pagina-especifica.css">
@endpush

@push('scripts')
    <script src="/js/pagina-especifica.js"></script>
@endpush
```

---

## Herança de Layout

```html
{{-- view filha --}}
@extends('layouts.app')

@section('title', 'Título da Página')

@section('content')
    <h1>Conteúdo aqui</h1>
@endsection
```

---

## Diretiva @include vs @component

| Situação                              | Usar           |
|---------------------------------------|----------------|
| Partial simples, sem lógica           | `@include`     |
| Reuso com props/variáveis             | `<x-component>`|
| Partial com dados do contexto atual   | `@include`     |
| UI isolada e reutilizável             | `<x-component>`|

---

## Blade Directives Customizadas

Registre em `App\Providers\AppServiceProvider`:

```php
use Illuminate\Support\Facades\Blade;

Blade::directive('money', function ($expression) {
    return "<?php echo 'R$ ' . number_format($expression, 2, ',', '.'); ?>";
});
```

Uso na view:
```html
<td>@money($produto->preco)</td>
```
