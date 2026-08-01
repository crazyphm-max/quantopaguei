# 🚀 Guia: estrutura para vários micro-SaaS independentes

Arquitetura escolhida: **cada app é um sistema separado** — cadastro próprio,
banco de dados próprio. O que é compartilhado é só a infraestrutura de
administração: um domínio (com um subdomínio por app), uma hospedagem
(GitHub Pages, grátis) e um único painel (Firebase) onde cada app tem o
seu projeto isolado.

```
seusapps.com.br  (1 domínio registrado — Registro.br)
├── mercado.seusapps.com.br   → repositório quantopaguei  → projeto Firebase "quantopaguei"
├── calorias.seusapps.com.br  → repositório calorias      → projeto Firebase "calorias"
└── bingo.seusapps.com.br     → repositório bingo         → projeto Firebase "bingo"
```

- Criar conta no QuantoPaguei **não** cria conta nos outros apps — cada
  projeto Firebase tem sua própria base de usuários e seu próprio banco.
- Tudo no plano grátis: o Firebase permite vários projetos por conta Google,
  cada um com login ilimitado (e-mail/senha) e banco de dados generoso
  para começar.

## Passo 1 — Criar o projeto Firebase do QuantoPaguei

1. Acesse https://console.firebase.google.com (entre com sua conta Google)
2. **Criar um projeto** → nome: `quantopaguei`
3. Quando perguntar do Google Analytics, pode **desativar** (simplifica)
4. Aguarde criar e clique em **Continuar**

> Para cada app novo (calorias, bingo…), repita este guia criando outro
> projeto no mesmo painel. Cinco minutos por app.

## Passo 2 — Ativar o login por e-mail e senha

1. No menu lateral: **Criação** (Build) → **Authentication** → **Vamos começar**
2. Aba **Sign-in method** → clique em **E-mail/senha** → **Ativar** → Salvar

O telefone do usuário será pedido no formulário de cadastro do app e salvo
no banco (captura do lead), sem SMS de verificação (SMS é pago — não precisa).

## Passo 3 — Criar o banco de dados

1. Menu lateral: **Criação** → **Firestore Database** → **Criar banco de dados**
2. Local: **southamerica-east1 (São Paulo)**
3. Modo: **produção** (as regras de segurança vêm no passo 4)

## Passo 4 — Colar as regras de segurança

Ainda no Firestore: aba **Regras** → apague o que estiver lá, cole o texto
abaixo e clique em **Publicar**. Elas garantem que cada usuário só acessa
os próprios dados.

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // perfil do usuário (nome, telefone, plano free/premium, trial)
    match /perfis/{uid} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
    // dados do app, sempre debaixo do usuário dono
    match /usuarios/{uid}/{documento=**} {
      allow read, write: if request.auth != null && request.auth.uid == uid;
    }
  }
}
```

## Passo 5 — Registrar o app web e pegar a configuração

1. Engrenagem (canto superior esquerdo) → **Configurações do projeto**
2. Seção **Seus aplicativos** → ícone **`</>`** (Web)
3. Apelido: `quantopaguei-web` → **Registrar app** (NÃO marque Firebase Hosting)
4. Vai aparecer um bloco `const firebaseConfig = { apiKey: "...", ... }`
   — **copie esse bloco inteiro** e envie para o desenvolvedor

> Essa configuração **pode** ser compartilhada e fica no código do site — é
> feita para isso. A segurança real está nas regras do passo 4.

## Passo 6 — Domínio (pode fazer em paralelo)

1. https://registro.br → pesquise e registre o domínio (~R$ 40/ano, precisa de CPF/CNPJ)
2. Depois apontamos cada subdomínio para o GitHub Pages do respectivo app
   (grátis, com HTTPS)

## O que mandar para o desenvolvedor

Somente o bloco `firebaseConfig` do passo 5. Com ele o app ganha:

- Tela de **cadastro** (nome, e-mail, telefone) e **login**
- Dados sincronizados na nuvem (continuando a funcionar offline no mercado)
- Campo de **plano** (`free`/`premium`) e data de **trial** já estruturados,
  prontos para o modelo de cobrança que você definir depois

## Custos desta fase

| Item | Custo |
|---|---|
| Firebase (login + banco, 1 projeto por app) | R$ 0 |
| GitHub Pages (hospedagem de todos os apps) | R$ 0 |
| Domínio .com.br (um só, subdomínios ilimitados) | ~R$ 40/ano |
| **Total** | **~R$ 40/ano** |

Quando algum app crescer muito (dezenas de milhares de usuários ativos), o
Firebase cobra só o excedente daquele projeto — os outros continuam grátis.
