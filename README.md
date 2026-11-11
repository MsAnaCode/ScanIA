# ScanIA

Aplicativo mobile que lê notas fiscais com inteligência artificial: fotografe o cupom, e o app extrai estabelecimento, data, valor, categoria e itens. Você revisa, corrige e acompanha seus gastos por categoria.

## Funcionalidades

- Cadastro, login e sessão persistente (Supabase Auth)
- Captura por câmera ou galeria, com redução da imagem antes do envio
- Leitura da nota por IA em uma Edge Function (a chave da IA nunca vai para o app)
- Revisão e correção dos dados extraídos (humano no circuito)
- Resumo mensal com gráfico por categoria
- Fotos em Storage privado, com acesso restrito ao dono (RLS)

## Tecnologias

React Native · Expo · Expo Router · TypeScript · Supabase (Auth, Postgres, Storage, Edge Functions) · Claude API (visão) · TanStack Query · React Hook Form · Zod · Jest

## Como executar

1. Instale as dependências: `npm install`
2. Crie um projeto no [Supabase](https://supabase.com) e execute `supabase/schema.sql`.
3. Copie `.env.example` para `.env` e preencha a URL e a chave pública.
4. Publique a função e configure os segredos:
   ```bash
   npx supabase login
   npx supabase link --project-ref SEU-PROJECT-REF
   npx supabase secrets set MOCK_AI=true
   npx supabase functions deploy scan-receipt --use-api
   ```
   Para usar a IA de verdade, defina `ANTHROPIC_API_KEY` e `MOCK_AI=false`.
5. Inicie o app: `npx expo start` e abra no Expo Go.

## Arquitetura

```
app/                        rotas (Expo Router)
src/                        componentes, hooks, serviços, schemas e utilitários
supabase/
  schema.sql                tabelas, RLS e bucket privado
  functions/scan-receipt    Edge Function (Deno) que chama a IA
__tests__/                  testes unitários (Jest)
```

## Scripts

| Comando | O que faz |
| --- | --- |
| `npm start` | Inicia o servidor do Expo |
| `npm test` | Executa os testes automatizados |
| `npm run typecheck` | Verifica os tipos do TypeScript |
| `npm run format` | Formata o código (Prettier) |

## Autor

Ana Clara · [GitHub](https://github.com/MsAnaCode)

