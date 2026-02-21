# Landing Page - Dolarização da Carteira (Apu Capital)



## Stack

Lovable + Supabase



## Objetivo: Captura de leads com rastreamento estruturado



Primeiramente foi criado o schema apu e a tabela t_leads no SQL Editor do Supabase:

```sql
create schema if not exists apu;

create table if not exists apu.t_leads (
    id uuid not null default gen_random_uuid(),
    nome character varying(255) not null,
    email character varying(255) null,
    telefone character varying(20) null,
    cpf character varying(14) null,
    atributos jsonb null default '{}'::jsonb,
    tracking jsonb null default '{}'::jsonb,
    created_at timestamp with time zone null default now(),
    constraint t_leads_pkey primary key (id)
);
```

A proteção dos dados foi implementada utilizando RLS (Row Level Security). Com o RLS ativado, nenhuma operação é permitida sem policy explícita.

```sql
alter table apu.t_leads enable row level security;
```

Permite que a landing page (role anon) insira registros. A policy autoriza inserções sem restrição de linha, permitindo que o formulário funcione publicamente.

```sql
create policy "Public insert"
on apu.t_leads
for insert
with check (true);
```

Criação da policy que permite SELECT apenas para os usuários autenticados no Supabase:

```sql
create policy "Authenticated select"
on apu.t_leads
for select
using (auth.role() = 'authenticated');
```

Controle de acesso (RLS + Grants) - Utiliza RLS para proteger os dados na tabela apu.t_leads

Permissão de Schema:

```sql
grant usage on schema apu to anon;
grant usage on schema apu to authenticated;
```

Permissões de tabela:

```sql
grant insert on apu.t_leads to anon;
grant select on apu.t_leads to authenticated;
```

Os trechos acima garantem que o formulário público consiga inserir dados, usuários autenticados possam visualizar os leads e que os dados não fiquem expostos publicamente.

**Campos atributos e Tracking:**

Atributos: JSON flexível com informações adicionais do lead, exemplo real do Supabase:

- mensagem: texto livre opcional do usuário
- faixa_patrimonio: faixa de patrimônio selecionada
- perfil_investidor: nível de risco do investidor, escolhido no formulário

```json
{
  "mensagem": "Testando Testando Testando",
  "faixa_patrimonio": "200k-500k",
  "perfil_investidor": "conservador"
}
```

Tracking: JSON que captura automaticamente parâmetros de rastreamento da URL e dados de navegação, exemplo real do Supabase:

- utm_source, utm_medium, utm_campaign: parâmetros UTM da campanha
- device: tipo de dispositivo do usuário (desktop, mobile, tablet)
- referrer: URL de origem do visitante; pode aparecer vazio em previews do Lovable ou quando o usuário acessa diretamente a LP

Observação: em tráfego real a partir de links externos o referrer será preenchido. Os parâmetros utm_source, utm_medium e utm_campaign são capturados quando estiverem presentes na URL; em acessos diretos ao link padrão, esses campos não aparecem, pois não fazem parte da navegação.

```json
{
  "device": "desktop",
  "referrer": "",
  "utm_medium": "cpc",
  "utm_source": "google",
  "utm_campaign": "test"
}
```

## Como rodar/testar o projeto:

O projeto está publicado e pode ser acessado pelo link:

[https://hello-apu-leads.lovable.app](https://hello-apu-leads.lovable.app/)

Para testar o envio de leads:

1. Acesse a URL publicada
2. Preencha o formulário com dados válidos
3. Envie o formulário

Para testar o tracking de campanha, utilize a URL com parâmetros na query string, por exemplo:

https://hello-apu-leads.lovable.app/?utm_source=google&utm_medium=cpc&utm_campaign=test

Ao enviar o formulário utilizando esse link, os campos utm_source, utm_medium e utm_campaign serão armazenados dentro do campo tracking, juntamente com informações de device e referrer (quando disponíveis).

## O que faria diferente se tivesse mais tempo:

Se houvesse mais tempo e mais créditos para evoluir o projeto, eu implementaria uma camada de logs estruturados para acompanhar todo o fluxo de envio do formulário. Isso permitiria monitorar falhas recorrentes, identificar possíveis quedas de integração com o Supabase, erros frequentes de validação ou até comportamentos suspeitos, como tentativas automatizadas.

Além disso, essa camada poderia ser integrada a alertas simples, via webhook ou notificação por e-mail, sempre que ocorressem falhas consecutivas de envio. Dessa forma, evitaria a perda silenciosa de leads e aumentaria a confiabilidade do projeto em um cenário mais próximo de produção.

Também implementaria mecanismos de proteção contra spam e abuso, algo bastante comum em landing pages públicas. Isso incluiria a adoção de um mecanismo anti-bot, como reCAPTCHA, além de limitação de requisições por IP e bloqueio de envios excessivos em curto intervalo de tempo. Essas medidas ajudariam a proteger a base de dados contra automações maliciosas e garantiriam maior qualidade e confiabilidade dos leads capturados.
