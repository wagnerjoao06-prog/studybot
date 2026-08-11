# Como usar o `study` (Studybot)

Este guia descreve como instalar, configurar e executar localmente o projeto "studybot" (comando/serviço referido aqui como "study"). Está em português e foca em Node/TypeScript, execução em desenvolvimento, build, e testes.

## Resumo rápido (comandos principais)
- Instalar dependências: npm install
- Rodar em desenvolvimento (macOS/Linux): NODE_ENV=dev npx tsx --trace-warnings ./src/main.ts
- Rodar em desenvolvimento (PowerShell): $env:NODE_ENV='dev'; npx tsx --trace-warnings ./src/main.ts
- Build: npm run build
- Iniciar produção (pm2): pm2 start ecosystem.config.cjs
- Rodar testes (usa Docker): npm run test  (inicia/para um container mongo temporário)

> Observação: o package.json define `dev` como `NODE_ENV=dev tsx --trace-warnings ./src/main.ts`. Em Windows o `NODE_ENV=...` não funciona nativamente — use as alternativas acima ou instale `cross-env`.

---

## Requisitos
- Node.js (recomendado: Node 18 LTS ou superior)
- npm (vem com Node) ou pnpm/yarn (os scripts usam npm)
- Docker (apenas para executar a suíte de testes localmente)
- pm2 (opcional, para `npm start` — ou qualquer process manager de sua preferência)
- Git (para clonar/abrir PRs)

Versões exatas não estão especificadas no repositório; recomenda-se Node >= 18.

---

## Variáveis de ambiente necessárias
O projeto usa dotenv (arquivo .env). As variáveis lidas em `src/config/env.ts` são:
- BOT_TOKEN / DEV_BOT_TOKEN  (token do bot Discord — use DEV_* em NODE_ENV=dev)
- APP_ID / DEV_APP_ID (ID da aplicação Discord)
- MONGODB_URI / DEV_MONGODB_URI (URI do MongoDB para produção/dev)
- PUBLIC_KEY / DEV_PUBLIC_KEY (public key para interações — opcional dependendo do uso)
- DEV_GUILD (ID da guild de desenvolvimento, usado para registrar comandos em um servidor)
- PORT (opcional, default 3000)

Crie um arquivo `.env` na raiz do projeto com as variáveis necessárias. Exemplo mínimo (.env):

BOT_TOKEN=your_production_token_here
DEV_BOT_TOKEN=your_dev_token_here
APP_ID=your_app_id_here
DEV_APP_ID=your_dev_app_id_here
MONGODB_URI=mongodb://user:pass@host:port/db
DEV_MONGODB_URI=mongodb://localhost:27027/devdb
PUBLIC_KEY=...
DEV_PUBLIC_KEY=...
DEV_GUILD=...
PORT=3000

Obs: o repositório está a ignorar `.env` (ver .gitignore) — nunca comitar tokens sensíveis.

---

## Passos de instalação (por SO)

1) Clonar repositório

    git clone <repo-url>
    cd studybot

2) Instalar dependências

Windows / macOS / Linux:

    npm install

(Recomenda-se usar a versão de Node indicada; se usar nvm, selecione a versão apropriada.)

3) (Opcional) Instalar pm2 globalmente para o script `npm start`:

    npm install -g pm2

---

## Executando localmente
Opções:

- Executar em desenvolvimento (hot-run via tsx)

  macOS / Linux (bash/zsh):

      NODE_ENV=dev npx tsx --trace-warnings ./src/main.ts

  Windows PowerShell:

      $env:NODE_ENV='dev'; npx tsx --trace-warnings ./src/main.ts

  Windows CMD:

      set NODE_ENV=dev && npx tsx --trace-warnings .\src\main.ts

  Observação: `npm run dev` usa `NODE_ENV=dev tsx ...` — em Windows esse trecho pode falhar; usar `npx tsx` com a forma acima ou instalar `cross-env` e alterar o script.

- Build e executar em produção

    npm run build
    pm2 start ecosystem.config.cjs   # requer pm2 instalado

- Comandos úteis (do package.json):
  - npm run build — compila via swc para `dist`
  - npm run eslint — roda eslint

---

## Rodando os testes
Os testes usam um container Mongo temporário. Os scripts do package.json:

- npm run test:db:start — inicia um container mongo em background (porta 27027 local -> 27017 no container)
- npm run test:db:stop — stopa o container
- npm test — inicia o DB, executa testes (com NODE_ENV=test) e para o DB
- npm run test:ci — roda somente os testes (assume ambiente de CI com Mongo disponível)

Exemplo (local):

    npm run test

Se preferir controlar manualmente:

    docker run -d --rm --name studybot-test -p 27027:27017 mongo
    NODE_ENV=test npx tsx --test "src/tests/**/*.test.ts" --test-concurrency=1
    docker stop studybot-test

---

## Saída esperada
- Em desenvolvimento, o processo loga mensagens como `express running on port <PORT>` e `Tracking Study Time` no console (ver `src/events/client.ts` e `src/App.ts`).
- Ao registrar comandos Discord (deploy), `src/deployCommands.ts` usa as variáveis APP_ID/BOT_TOKEN.

---

## Solução de problemas comuns
- "NODE_ENV=dev" não funciona no Windows: use as alternativas de PowerShell/CMD acima ou instale `cross-env` e ajuste o script em package.json.
- Testes falham por conexão com Mongo: verifique se Docker está rodando e porta 27027 livre; ou configure DEV_MONGODB_URI/MONGODB_URI adequadamente.
- Tokens/IDs faltando: crie `.env` com as variáveis listadas; sem BOT_TOKEN o bot não fará login.
- `pm2 start` falha: instale pm2 globalmente (`npm i -g pm2`) ou use `node dist/main.js` após build.
- Erros de TypeScript/Compilação: rodar `npm run build` para ver mensagens do swc/ts.

---

## Arquivos e scripts relevantes encontrados
- package.json — scripts importantes: `dev`, `start` (pm2), `build`, `test`, `test:db:start`, `test:db:stop`
- src/config/env.ts — lista as variáveis de ambiente usadas
- ecosystem.config.cjs — configuração usada por `npm start` (pm2)

---

## Itens em falta / incertezas
1. Não existe `.env.example` no repositório — seria útil para documentar valores mínimos. Recomenda-se adicionar um arquivo `env.example` com as chaves necessárias (sem valores sensíveis).
2. Versão mínima de Node não especificada; recomenda-se documentar oficialmente (sugestão: Node 18+).

---

Se desejar, posso:
- Adicionar um `.env.example` ao repositório (com placeholders não sensíveis);
- Atualizar o `package.json` para usar `cross-env` nos scripts `dev` e `test` para compatibilidade com Windows;
- Incluir instruções para deploy (ex.: configurar pm2/systemd).

---

Fim do guia.
