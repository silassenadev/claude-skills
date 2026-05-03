# jQuery — Padrões para Laravel

## Configuração Global (sempre no layout)

```html
{{-- No <head> --}}
<meta name="csrf-token" content="{{ csrf_token() }}">

{{-- Antes do </body>, após carregar jQuery --}}
<script>
    $.ajaxSetup({
        headers: { 'X-CSRF-TOKEN': $('meta[name="csrf-token"]').attr('content') }
    });
</script>
```

---

## AJAX — GET (buscar dados)

```javascript
$(function () {
    $('#btn-buscar').on('click', function () {
        const termo = $('#campo-busca').val();

        $.get('/api/buscar', { q: termo }, function (data) {
            const lista = $('#resultado').empty();
            $.each(data, function (i, item) {
                lista.append(`<li>${item.nome}</li>`);
            });
        }).fail(function () {
            alert('Erro ao buscar.');
        });
    });
});
```

---

## AJAX — POST (enviar formulário sem recarregar)

```javascript
$('#form-cadastro').on('submit', function (e) {
    e.preventDefault();

    const dados = $(this).serialize(); // captura todos os campos

    $.post($(this).attr('action'), dados)
        .done(function (resposta) {
            $('#mensagem').html('<div class="alert alert-success">Salvo!</div>');
        })
        .fail(function (xhr) {
            if (xhr.status === 422) {
                // Erros de validação do Laravel
                const erros = xhr.responseJSON.errors;
                let html = '<ul>';
                $.each(erros, function (campo, msgs) {
                    $.each(msgs, function (i, msg) {
                        html += `<li>${msg}</li>`;
                    });
                });
                html += '</ul>';
                $('#erros').html(html).show();
            }
        });
});
```

---

## AJAX — DELETE (com spoofing de método)

O Laravel precisa de `_method: 'DELETE'` porque formulários HTML só aceitam GET/POST:

```javascript
$('.btn-deletar').on('click', function () {
    if (!confirm('Confirmar exclusão?')) return;

    const url = $(this).data('url');
    const linha = $(this).closest('tr');

    $.ajax({
        url: url,
        type: 'POST',
        data: { _method: 'DELETE' },
    })
    .done(function () {
        linha.fadeOut(300, function () { $(this).remove(); });
    })
    .fail(function () {
        alert('Não foi possível deletar.');
    });
});
```

---

## AJAX — PUT/PATCH (atualização parcial)

```javascript
$.ajax({
    url: '/nomes/' + id,
    type: 'POST',
    data: {
        _method: 'PATCH',
        campo: novoValor
    },
    success: function (resposta) {
        console.log('Atualizado:', resposta);
    }
});
```

---

## Upload de Arquivo com AJAX

```javascript
$('#form-upload').on('submit', function (e) {
    e.preventDefault();

    const formData = new FormData(this); // FormData suporta arquivos

    $.ajax({
        url: $(this).attr('action'),
        type: 'POST',
        data: formData,
        processData: false,  // IMPORTANTE: não processar FormData
        contentType: false,  // IMPORTANTE: deixar o browser definir o boundary
        xhr: function () {
            const xhr = new window.XMLHttpRequest();
            xhr.upload.addEventListener('progress', function (e) {
                if (e.lengthComputable) {
                    const pct = Math.round((e.loaded / e.total) * 100);
                    $('#barra-progresso').css('width', pct + '%').text(pct + '%');
                }
            });
            return xhr;
        },
        success: function (resposta) {
            alert('Upload concluído!');
        }
    });
});
```

---

## Validação de Formulário com jQuery

```javascript
$(function () {
    $('#form').on('submit', function (e) {
        let valido = true;

        // Limpar erros anteriores
        $('.is-invalid').removeClass('is-invalid');
        $('.invalid-feedback').text('');

        // Validar campo nome
        const nome = $('#nome').val().trim();
        if (!nome) {
            $('#nome').addClass('is-invalid');
            $('#nome-erro').text('Nome é obrigatório.');
            valido = false;
        }

        // Validar e-mail
        const email = $('#email').val().trim();
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        if (!emailRegex.test(email)) {
            $('#email').addClass('is-invalid');
            $('#email-erro').text('E-mail inválido.');
            valido = false;
        }

        if (!valido) e.preventDefault();
    });
});
```

No campo HTML do Blade:
```html
<div class="mb-3">
    <label for="nome">Nome</label>
    <input type="text" id="nome" name="nome" class="form-control @error('nome') is-invalid @enderror"
           value="{{ old('nome') }}">
    <div class="invalid-feedback" id="nome-erro">
        @error('nome') {{ $message }} @enderror
    </div>
</div>
```

---

## Polling (atualização automática)

```javascript
function atualizarStatus() {
    $.get('/status', function (data) {
        $('#status-badge')
            .text(data.status)
            .removeClass()
            .addClass('badge bg-' + (data.status === 'ativo' ? 'success' : 'danger'));
    });
}

// Atualiza a cada 5 segundos
setInterval(atualizarStatus, 5000);
atualizarStatus(); // chama imediatamente na carga
```

---

## DataTables (tabelas com paginação/busca automática)

Adicione no layout:
```html
<link rel="stylesheet" href="https://cdn.datatables.net/1.13.6/css/dataTables.bootstrap5.min.css">
<script src="https://cdn.datatables.net/1.13.6/js/jquery.dataTables.min.js"></script>
<script src="https://cdn.datatables.net/1.13.6/js/dataTables.bootstrap5.min.js"></script>
```

Inicialize na view:
```javascript
$(function () {
    $('#tabela').DataTable({
        language: {
            url: '//cdn.datatables.net/plug-ins/1.13.6/i18n/pt-BR.json'
        },
        order: [[0, 'desc']],
        pageLength: 25,
    });
});
```
