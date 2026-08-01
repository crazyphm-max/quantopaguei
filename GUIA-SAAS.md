# 🚀 Guia: transformando os micro-apps em micro-SaaS

Este guia prepara a base para **todos** os seus micro-SaaS (QuantoPaguei, calorias,
bingo e os próximos) com **uma conta única por usuário** — a pessoa cadastra
nome, e-mail e telefone uma vez e usa o mesmo login em todos os apps.

## Passo 1 — Criar a conta no Supabase (grátis)

O Supabase é o serviço que dará **login** e **banco de dados na nuvem** sem
precisar de servidor próprio.

1. Acesse https://supabase.com e clique em **Start your project**
2. Crie a conta (pode entrar com o GitHub, mesma conta que você já usa)
3. Clique em **New project**:
   - **Name**: `meus-apps` (um projeto só serve para todos os micro-SaaS)
   - **Database password**: crie uma senha forte e **guarde** (não vai me mandar essa!)
   - **Region**: `South America (São Paulo)` — mais rápido para usuários no Brasil
4. Aguarde 1–2 minutos até o projeto ficar pronto

## Passo 2 — Pegar as 2 chaves que o desenvolvedor precisa

No painel do projeto: **Settings** (engrenagem) → **API**. Copie:

- **Project URL** — algo como `https://abcdefgh.supabase.co`
- **anon public key** — um código longo começando com `eyJ...`

> Essas duas informações **podem** ser compartilhadas e ficam no código do site
> (a "anon key" é feita para isso — a segurança real fica nas regras do banco,
> que o script abaixo configura). A **Database password** e a **service_role key**
> NUNCA devem ser compartilhadas nem colocadas no site.

## Passo 3 — Rodar o script do banco de dados

No painel do Supabase: **SQL Editor** → **New query** → cole TODO o script
abaixo → botão **Run**.

```sql
-- ============================================================
-- BASE COMPARTILHADA: perfil único do usuário (todos os apps)
-- ============================================================
create table public.perfis (
  id uuid primary key references auth.users(id) on delete cascade,
  nome text,
  email text,
  telefone text,
  plano text not null default 'free',      -- free | premium
  trial_ate timestamptz,                    -- fim do período de teste (se usar)
  criado_em timestamptz not null default now()
);
alter table public.perfis enable row level security;
create policy "ler o proprio perfil" on public.perfis
  for select using (auth.uid() = id);
create policy "editar o proprio perfil" on public.perfis
  for update using (auth.uid() = id);

-- cria o perfil automaticamente quando alguém se cadastra,
-- guardando nome e telefone informados no formulário
create or replace function public.novo_usuario()
returns trigger language plpgsql security definer set search_path = public as $$
begin
  insert into public.perfis (id, email, nome, telefone)
  values (new.id, new.email,
          new.raw_user_meta_data->>'nome',
          new.raw_user_meta_data->>'telefone');
  return new;
end; $$;
create trigger ao_criar_usuario
  after insert on auth.users
  for each row execute function public.novo_usuario();

-- ============================================================
-- QUANTOPAGUEI: tabelas do app de preços de mercado (prefixo qp_)
-- Cada app novo ganha suas tabelas com um prefixo próprio
-- (ex.: cal_ para calorias), sempre no mesmo projeto.
-- ============================================================
create table public.qp_compras (
  id bigint generated always as identity primary key,
  usuario uuid not null default auth.uid() references auth.users(id) on delete cascade,
  chave text not null,
  estabelecimento text,
  cnpj text,
  data date,
  total numeric,
  criado_em timestamptz not null default now(),
  unique (usuario, chave)
);
create table public.qp_itens (
  id bigint generated always as identity primary key,
  usuario uuid not null default auth.uid() references auth.users(id) on delete cascade,
  compra_chave text,
  descricao_original text,
  nome_normalizado text,
  quantidade numeric,
  unidade text,
  valor_unitario numeric,
  valor_total numeric,
  sub_quantidade integer not null default 1,
  estabelecimento text,
  data date,
  eh_visto boolean not null default false,
  criado_em timestamptz not null default now()
);
create index qp_itens_usuario_nome on public.qp_itens (usuario, nome_normalizado);

-- segurança: cada usuário só enxerga e mexe nos PRÓPRIOS dados
alter table public.qp_compras enable row level security;
alter table public.qp_itens enable row level security;
create policy "qp_compras proprias" on public.qp_compras
  for all using (auth.uid() = usuario) with check (auth.uid() = usuario);
create policy "qp_itens proprios" on public.qp_itens
  for all using (auth.uid() = usuario) with check (auth.uid() = usuario);
```

## Passo 4 — Configurar o login por e-mail

No painel: **Authentication** → **Providers** → confira que **Email** está
habilitado. Em **Authentication → Settings**:

- Se quiser cadastro sem confirmação por e-mail (mais fácil para começar):
  desligue **"Confirm email"**. Dá para ligar depois.

## Passo 5 — Domínio (pode fazer em paralelo)

1. Acesse https://registro.br e pesquise o nome que você quer (ex.: `seusapps.com.br`)
2. Registre (~R$ 40/ano) — precisa de CPF/CNPJ
3. Depois a gente aponta os subdomínios: `mercado.seusapps.com.br`,
   `calorias.seusapps.com.br` etc. (cada um para o GitHub Pages do app, grátis)

## O que mandar para o desenvolvedor

Depois dos passos 1–4, envie apenas:

1. **Project URL** (ex.: `https://abcdefgh.supabase.co`)
2. **anon public key** (`eyJ...`)

Com isso o app ganha: tela de cadastro/login (nome, e-mail, telefone),
sincronização dos dados na nuvem (mantendo o uso offline no mercado) e o
campo de plano free/premium pronto para o modelo de cobrança que você escolher.

## Custos desta fase

| Item | Custo |
|---|---|
| Supabase (login + banco, até ~50 mil usuários ativos/mês) | R$ 0 |
| GitHub Pages (hospedagem dos apps) | R$ 0 |
| Domínio .com.br | ~R$ 40/ano |
| **Total** | **~R$ 40/ano** |
