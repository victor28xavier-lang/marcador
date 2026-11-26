# Clube de leitura: como colocar no ar

São 4 passos: criar o banco de dados (Firebase), publicar o site, instalar no celular e chamar os amigos. Sem o passo 1 o app já funciona sozinho (leitor, meta, minutos e período do dia); o passo 1 só é necessário para o clube com amigos.

## 1. Firebase (grátis, ~10 min, melhor fazer no computador)

1. Entre em https://console.firebase.google.com com sua conta Google e clique em **Criar um projeto** (ex.: `clube-leitura`). Pode desativar o Google Analytics.
2. Na página do projeto, clique no ícone **</>** (Web) para registrar um app. Dê um nome e **não** marque Firebase Hosting.
3. Ele mostra um bloco `const firebaseConfig = { ... }`. Copie os valores para o arquivo **config.js** desta pasta (apiKey, authDomain, projectId etc.).
4. Menu **Criação > Authentication > Vamos começar > Anônimo > Ativar > Salvar**.
5. Menu **Criação > Firestore Database > Criar banco de dados**. Escolha o local `southamerica-east1 (São Paulo)` e o **modo de produção**.
6. Na aba **Regras** do Firestore, apague tudo, cole o texto abaixo e clique em **Publicar**:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /users/{uid} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
    match /clubs/{club} {
      function cpath() { return /databases/$(database)/documents/clubs/$(club); }
      function cdata() { return exists(cpath()) ? get(cpath()).data : {}; }
      function signed() { return request.auth != null; }
      function owner() { return cdata().get('owner', ''); }
      function isOwner() { return signed() && owner() == request.auth.uid; }
      function banned() { return signed() && request.auth.uid in cdata().get('banned', []); }
      function member() { return signed() && !banned() && (!cdata().get('closed', false) || isOwner() || request.auth.uid in cdata().get('members', [])); }
      function clubKeys() { return request.resource.data.keys().hasOnly(['name','owner','at','closed','members','banned','bnames','by']); }

      allow read: if signed();
      allow create: if signed() && clubKeys() && request.resource.data.owner == request.auth.uid;
      allow update: if signed() && clubKeys() && (
        resource.data.get('owner', '') == request.auth.uid ||
        (resource.data.get('owner', '') == '' && request.resource.data.owner == request.auth.uid));

      match /readers/{uid} {
        allow read: if member();
        allow create, update: if member() && request.auth.uid == uid
          && request.resource.data.keys().hasOnly(['books','days','goal','per','nick','updated','live','cheers','ev','photo']);
        allow delete: if signed() && (request.auth.uid == uid || isOwner());
      }
      match /shelves/{uid} {
        allow read: if member();
        allow create, update: if member() && request.auth.uid == uid
          && request.resource.data.keys().hasOnly(['books','updated']);
        allow delete: if signed() && (request.auth.uid == uid || isOwner());
      }
      match /requests/{uid} {
        allow read: if signed() && (request.auth.uid == uid || isOwner());
        allow create, update: if signed() && request.auth.uid == uid && !banned()
          && request.resource.data.keys().hasOnly(['nick','photo','at']);
        allow delete: if signed() && (request.auth.uid == uid || isOwner());
      }
    }
  }
}
```

Com essas regras, cada pessoa só altera os próprios dados e todo mundo do app consegue ler os clubes. Por isso, use um código de clube que não seja óbvio.

## 2. Publicar o site

Precisa ser um endereço https, porque sem isso o celular não instala o app.

**Opção fácil: Netlify**
1. Crie uma conta grátis em https://app.netlify.com.
2. Vá em **Add new site > Deploy manually** e arraste esta pasta inteira.
3. Ele cria um link tipo `https://nome-aleatorio.netlify.app` (dá para trocar o nome em Site settings).

**Opção alternativa: GitHub Pages**: crie um repositório, envie os arquivos e ative **Settings > Pages**.

Para atualizar depois, arraste a pasta de novo (Netlify: aba Deploys).

## 3. Instalar no celular

- **Android (Chrome):** abra o link e toque em **Instalar app** no topo, ou vá no menu ⋮ e escolha **Instalar app**.
- **iPhone (Safari):** abra o link, toque em **Compartilhar** e depois em **Adicionar à Tela de Início**.

Depois de instalado, ele abre como um app normal e funciona sem internet. O clube sincroniza quando a internet voltar.

## 4. Chamar os amigos

Mande o link para eles e combinem o mesmo **código do clube** (ex.: `turma-ifrj-2026`). Cada um coloca um apelido e o próprio EPUB.

Observações:
- O arquivo do livro nunca sai do aparelho. O clube recebe só o apelido, título, autor, % lido, minutos por dia (últimos 14 dias) e minutos por período (manhã/tarde/noite/madrugada, últimos 30 dias).
- Cada aparelho conta como um leitor separado. Se usar no celular e no PC, você aparece duas vezes.
- O plano grátis do Firebase (Spark) aguenta com folga um grupo de amigos.
