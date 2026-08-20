# CORS do Storage — como aplicar

O arquivo `cors.json` **não faz nada sozinho** dentro do repositório. Ele
precisa ser aplicado ao bucket do Firebase Storage por um comando. Enquanto
não for aplicado (ou se for aplicado incompleto), o navegador bloqueia as
chamadas e o painel mostra erros assim:

```
Access to XMLHttpRequest at 'https://firebasestorage.googleapis.com/...'
from origin 'https://lorettoscarpa-icr.github.io' has been blocked by
CORS policy: Response to preflight request doesn't pass access control
check: It does not have HTTP ok status.
```

## O que quebra sem isso

| Ação no painel | Método HTTP | Sem CORS |
|---|---|---|
| Ver a lista de arquivos | — (Firestore) | funciona |
| Copiar link | — | funciona |
| Subir catálogo ou foto | `POST` / `PUT` | **falha** |
| Excluir | `DELETE` | **falha** |
| Baixar (com o nome certo) | `GET` | **falha** |
| Copiar foto pro WhatsApp | `GET` | **falha** |
| Baixar tudo (.zip) | `GET` | **falha** |

Por isso o `method` do `cors.json` precisa listar **todos** eles, não só
`GET` e `HEAD`.

## Como aplicar

O jeito mais simples é pelo Cloud Shell, que roda no navegador e não exige
instalar nada.

1. Abra <https://console.cloud.google.com/> e selecione o projeto
   **loretto-scarpa-l-icr** no seletor do topo.
2. Clique no ícone de terminal (**Ativar o Cloud Shell**), no canto
   superior direito. Espere abrir o terminal na parte de baixo.
3. Cole o bloco abaixo inteiro e dê Enter:

```bash
cat > cors.json <<'FIM'
[
  {
    "origin": ["https://lorettoscarpa-icr.github.io"],
    "method": ["GET", "HEAD", "PUT", "POST", "DELETE", "PATCH"],
    "responseHeader": [
      "Content-Type", "Content-Length", "Content-Range",
      "Content-Disposition", "Content-Encoding", "Range", "ETag",
      "X-Goog-Upload-URL", "X-Goog-Upload-Status",
      "X-Goog-Upload-Size-Received", "X-Goog-Upload-Chunk-Granularity",
      "X-Goog-Upload-Control-URL", "X-Firebase-Storage-Version"
    ],
    "maxAgeSeconds": 3600
  }
]
FIM

gcloud storage buckets update gs://loretto-scarpa-l-icr.firebasestorage.app --cors-file=cors.json
```

4. Confira se gravou:

```bash
gcloud storage buckets describe gs://loretto-scarpa-l-icr.firebasestorage.app --format="default(cors_config)"
```

Tem que aparecer a lista com `POST`, `PUT` e `DELETE`.

> Se o `gcloud storage` reclamar, o comando antigo equivalente é
> `gsutil cors set cors.json gs://loretto-scarpa-l-icr.firebasestorage.app`
> e o de leitura é `gsutil cors get gs://loretto-scarpa-l-icr.firebasestorage.app`.

5. No painel, recarregue com **Ctrl+Shift+R** e tente subir de novo.

## Atenção

- O nome do bucket é `loretto-scarpa-l-icr.firebasestorage.app` — com
  `.firebasestorage.app`, não `.appspot.com`. Ele aparece na própria URL do
  erro, entre `/v0/b/` e `/o`.
- `origin` é a origem do **site**, não do Storage. Se um dia o painel passar
  a ser servido de outro endereço (domínio próprio, por exemplo), esse novo
  endereço precisa entrar na lista.
- CORS é configuração do bucket, não do código. Publicar o site de novo não
  resolve; só o comando acima resolve.
- Isso é independente do `storage.rules`. As regras decidem **quem pode**;
  o CORS decide se o navegador **deixa a chamada sair**. Um erro de regra
  aparece como `storage/unauthorized` ("Sem permissão"); um erro de CORS
  aparece como bloqueio de preflight, igual ao do topo deste arquivo.
