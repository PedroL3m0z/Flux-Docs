---
title: Contribuindo
description: Como configurar o repo da Flux API, criar branch e abrir um PR que entra limpo.
---

Contribuições são bem-vindas. Esta página espelha os canônicos
[`CONTRIBUTING.md`](https://github.com/PedroL3m0z/Flux-Api/blob/main/.github/CONTRIBUTING.md)
e [`BRANCHING.md`](https://github.com/PedroL3m0z/Flux-Api/blob/main/.github/BRANCHING.md)
do repo da API — leia-os para o detalhe completo; aqui é a orientação rápida.

## Configure o ambiente de dev

Rode a infra no Docker e o app no host com hot-reload:

```bash
git clone https://github.com/PedroL3m0z/Flux-Api.git
cd Flux-Api
cp .env.example .env          # opcional — o app sobe sem configuração
yarn install
yarn prisma:generate
docker compose -f docker-compose.dev.yml up -d   # só Postgres + Redis
yarn start:dev
```

A API sobe em `http://localhost:3000`. Para o gateway em si rodando, veja
[Primeiros passos](/Flux-Docs/pt-br/getting-started/).

## Fluxo de trabalho

A Flux usa **GitHub Flow** (trunk-based): uma `main` protegida, branches de feature
de vida curta, PRs com squash-merge. **Não** existe branch `develop`.

1. Crie a branch a partir da `main` mais recente — nunca de outra branch de feature:
   ```bash
   git checkout main && git pull origin main
   git checkout -b feat/minha-feature
   ```
2. Mantenha cada PR focado — **um assunto por branch**, pequeno e revisável.
3. Garanta que builda, passa no lint e nos testes (abaixo).
4. Abra um PR **para a `main`** usando o template; linke issues relacionadas.
5. Após o merge, **delete a branch**. Feche PRs substituídos/duplicados.

:::caution[Duas coisas que mordem]
**Não** empilhe PRs (base = outra branch `feat/*`), e **não** faça merge de dois
PRs para o mesmo trabalho — feche o obsoleto primeiro. Ambos confundem a
automação do release-please.
:::

## Nomes de branch

Use os prefixos do Conventional Commits como nome da branch:

| Prefixo | Para |
| --- | --- |
| `feat/<desc>` | nova funcionalidade |
| `fix/<desc>` | correção de bug |
| `chore/<desc>` | tooling, deps, CI |
| `docs/<desc>` | só documentação |
| `refactor/<desc>` | mudança interna, sem diferença de comportamento |

Exemplos: `feat/zero-config`, `fix/auth-register-api-key`. Não reuse o nome de uma
branch depois que o PR dela mergeou — crie uma nova.

## Portões de qualidade

Estes rodam no CI em todo PR; rode-os localmente antes:

```bash
yarn lint        # ESLint (--fix)
yarn build       # tsc / nest build
yarn test        # testes unitários
yarn test:e2e    # testes e2e
```

Faça rebase na `main` antes de abrir ou atualizar um PR, para o CI rodar num diff
limpo:

```bash
git fetch origin
git rebase origin/main
git push --force-with-lease
```

## Mensagens de commit

Siga [Conventional Commits](https://www.conventionalcommits.org/) — elas movem o
versionamento automatizado e o changelog:

```
feat(auth): add API key authentication strategy
fix(prisma): close pool on shutdown
docs(readme): document /health endpoint
```

Habilite o template compartilhado localmente:

```bash
git config commit.template .github/.gitmessage.txt
```

## Mudanças no banco

O schema fica em `src/core/prisma/schema.prisma`. Depois de editá-lo:

```bash
yarn prisma:migrate     # cria uma migration + regenera o client
```

Commite os arquivos de migration gerados. **Não** commite
`src/core/prisma/generated`.

## Releases

Releases são automatizadas com **release-please** — não edite números de versão
ou o `CHANGELOG.md` à mão. Commits `feat:` / `fix:` mergeados se acumulam num PR
de release (`chore(main): release X.Y.Z`); mergear esse PR cria a tag da versão e
publica o GitHub Release.

## Reportando bugs e segurança

Use os templates de issue do GitHub para bugs e pedidos de feature. Para questões
de **segurança**, siga o
[`SECURITY.md`](https://github.com/PedroL3m0z/Flux-Api/blob/main/.github/SECURITY.md)
— não abra uma issue pública. O projeto segue o
[Contributor Covenant](https://github.com/PedroL3m0z/Flux-Api/blob/main/.github/CODE_OF_CONDUCT.md).
