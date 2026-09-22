# site/ — a página que faz o link do convite ser clicável

Isto **não** é publicado a partir deste repositório. O conteúdo daqui vai para
o repositório `lucasmello-dev/lucasmello-dev.github.io`, que serve
`https://lucasmello-dev.github.io/`.

## Por que num repositório separado

O Universal Link exige o arquivo `apple-app-site-association` na **raiz do
domínio**, em `/.well-known/`. Um repositório de projeto publica em
`lucasmello-dev.github.io/lembrei-ios/`, que é uma pasta — nunca a raiz. Só o
repositório de usuário (`<usuário>.github.io`) serve a raiz, e por isso é ele.

## O que cada arquivo faz

| Arquivo | Papel |
|---|---|
| `l/index.html` | A página do convite. Lê o pedido do fragmento (`#`), mostra o lembrete e oferece "Abrir no Lembrei". |
| `.well-known/apple-app-site-association` | Diz ao iOS que `/l/*` pertence ao app nas equipes de desenvolvimento (`7T2YFZ3JJM`) e distribuição paga (`92696QQ6C5`), com bundle `com.lucasmello.lembrei`. Sem ele o link abre no Safari em vez do app. |
| `.nojekyll` | O GitHub Pages roda Jekyll por padrão, e Jekyll **ignora toda pasta que começa com ponto**. Sem este arquivo o `.well-known/` simplesmente não é publicado, e o Universal Link nunca funciona. |

## Publicar

```bash
gh repo create lucasmello-dev/lucasmello-dev.github.io --public
cd /tmp && git clone https://github.com/lucasmello-dev/lucasmello-dev.github.io
cp -R ~/Work/lembrei-ios/site/. lucasmello-dev.github.io/
cd lucasmello-dev.github.io && git add -A && git commit -m "feat: página do convite do Lembrei" && git push
```

Depois, em *Settings → Pages*, a origem deve ser a branch `main`, pasta `/`.

## Conferir

O teste que vale não é o do nosso servidor — é o da CDN da Apple, que é quem
de fato busca e interpreta o arquivo:

```bash
curl -s https://app-site-association.cdn-apple.com/a/v1/lucasmello-dev.github.io
```

Se voltar o JSON, está aceito. **Verificado em 2026-08-05: voltou o JSON.**

Isso resolve, de passagem, a dúvida do cabeçalho: o GitHub Pages serve o
arquivo como `application/octet-stream`, e não `application/json`, porque ele
não tem extensão e não dá para configurar cabeçalho no Pages. A documentação
antiga da Apple pedia `application/json` — a CDN aceitou assim mesmo, e é ela
quem manda. Se um dia parar de aceitar, a saída é pôr um domínio próprio na
frente (Cloudflare, Vercel e Netlify deixam configurar cabeçalho; o Pages não).

O servidor cru, se precisar olhar:

```bash
# 200 e o JSON. Se vier 404, faltou o .nojekyll.
curl -sI https://lucasmello-dev.github.io/.well-known/apple-app-site-association
```

Duas coisas que costumam morder:

1. **Não pode haver redirecionamento** no caminho do `apple-app-site-association`.
   O GitHub Pages não redireciona esse caminho, mas confirme no `curl -I`.
2. A Apple **guarda o arquivo em cache**. Depois de mudar o arquivo, reinstalar
   o app no aparelho é o jeito mais rápido de forçar a releitura.

## Se o Universal Link não subir

Ele é o acabamento, não a fundação. Sem ele o link continua **clicável** no
WhatsApp (é `https`), abre esta página no Safari e o botão "Abrir no Lembrei"
entrega o pedido ao app em um toque. E o caminho de colar na aba Social segue
funcionando mesmo com o app instalado por conta gratuita, que não assina
`associated-domains`.
