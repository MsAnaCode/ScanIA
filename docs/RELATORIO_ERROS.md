# Relatório Técnico de Erros e Resoluções — Projeto 5: ScanIA

Este documento consolida todos os erros, inconsistências e alertas encontrados durante o processo de configuração, desenvolvimento e implantação do aplicativo **ScanIA**, detalhando o sintoma, a causa raiz e a solução aplicada.

---

## 1. Confirmação de E-mail Pendente no Supabase Auth

- **Sintoma:** Ao cadastrar um novo usuário no aplicativo, o app exibia alerta solicitando confirmação de e-mail e impedia o acesso imediato. Ao tentar validar via link no celular, ocorria falha de redirecionamento.
- **Causa Raiz:** Por padrão, o Supabase Auth vem com a opção *Confirm email* habilitada. Em ambiente de desenvolvimento mobile com Expo Go sem esquema de *Deep Linking* pré-configurado para retorno de e-mail do navegador externo, o usuário ficava preso no estado `email_confirmed_at = null`.
- **Resolução:**
  1. Acessou-se o painel do Supabase em **Authentication** → **Providers** → **Email**.
  2. Desativou-se a opção **Confirm email**.
  3. Removeu-se o usuário anterior pendente em **Authentication** → **Users** e refez-se o cadastro no app, obtendo autenticação imediata e sessão persistente.

---

## 2. Tabelas Ausentes no Cache de Schema (`public.receipts`)

- **Sintoma:** Mensagem de erro ao abrir a tela inicial do aplicativo:
  ```text
  Could not find the table 'public.receipts' in the schema cache
  ```
- **Causa Raiz:** O banco de dados PostgreSQL do Supabase ainda não havia executado o script de DDL (`supabase/schema.sql`). A API PostgREST do Supabase não encontrou as tabelas expostas no schema público.
- **Resolução:**
  1. Executou-se o script completo via **SQL Editor** do Supabase.
  2. Foram criadas as tabelas `public.receipts`, `public.receipt_items`, os índices (`receipts_user_created_idx`, `receipt_items_receipt_idx`), as políticas de RLS e o bucket de armazenamento privado `receipts` no Supabase Storage.

---

## 3. Falha na Execução da IA / Edge Function (`non-2xx status code`)

- **Sintoma:** Ao fotografar uma nota e clicar em *Analisar com IA*, o app retornava:
  ```text
  A leitura falhou. Edge Function returned a non-2xx status code.
  A nota foi salva: você pode tentar de novo.
  ```
- **Causa Raiz:** A Edge Function `scan-receipt` em Deno ainda não estava publicada (deploy) no projeto Supabase, ou a variável de ambiente `MOCK_AI` não estava configurada nos segredos do servidor.
- **Resolução:**
  1. Login e vinculação do projeto pela CLI do Supabase:
     ```bash
     npx supabase login
     npx supabase link --project-ref iegemjpistdzrpjcrmcm
     ```
  2. Ativação do modo simulado:
     ```bash
     npx supabase secrets set MOCK_AI=true
     ```
  3. Deploy da função via API do Supabase:
     ```bash
     npx supabase functions deploy scan-receipt --use-api
     ```
  4. A função passou para o status `ACTIVE`, permitindo o retorno dos dados estruturados e itens da nota.

---

## 4. Incompatibilidade de Tipos TypeScript (`ZodEnum` e `RefreshControl`)

- **Sintomas:**
  - Erro no schema de validação: `Argument of type [...] is not assignable to parameter of type 'Readonly<Record<string, EnumValue>>'`.
  - Erro no componente de abas: `Type '____ColorValue_Internal' is not assignable to type 'string'`.
  - Erro na prop `onRefresh` do `RefreshControl`: tipagem retornando `Promise<QueryObserverResult>` em vez de `void | Promise<void>`.
- **Causa Raiz:**
  - `z.enum`: Atualizações da API do Zod alteraram o formato do parâmetro de mensagem customizada de `{ errorMap: ... }` para `{ message: ... }`.
  - `@react-navigation/bottom-tabs`: O tipo da propriedade `color` injetada na renderização de ícone pode ser nulo ou derivado do tema nativo.
  - `RefreshControl`: O retorno do `refetch()` do TanStack Query não combinava diretamente com a assinatura de callback síncrona/vazia esperada pelo React Native.
- **Resolução:**
  - Em `src/schemas/receipt.ts`: corrigido para `z.enum(CATEGORY_NAMES, { message: 'Escolha uma categoria' })`.
  - Em `app/(app)/(tabs)/_layout.tsx`: ajustado o callback `tabIcon` com fallback de cor padrão.
  - Em `app/(app)/(tabs)/index.tsx`: ajustado para `onRefresh={() => { refetch(); }}`.

---

## 5. Dependências Nativas Ausentes e Conflito de Versões Expo

- **Sintoma:** O diagnóstico do `expo-doctor` apontou:
  - Falta da dependência `expo-font` requerida pelo `@expo/vector-icons`.
  - Incompatibilidade de versão com o Expo SDK 58 para `@react-native-async-storage/async-storage` e `react-native-svg`.
- **Causa Raiz:** Dependências instaladas diretamente via `npm` sem resolução de compatibilidade do ecossistema Expo.
- **Resolução:**
  - Executou-se a instalação das versões homologadas pelo SDK:
    ```bash
    npx expo install expo-font @react-native-async-storage/async-storage@2.2.0 react-native-svg@15.16.0
    ```

---

## 6. Resultado Final e Validação

Após as correções:
- **Testes Automatizados:** 36 testes passando (100% de sucesso nas 4 suítes: `date`, `money`, `summary`, `parse`).
- **TypeScript:** `npm run typecheck` sem nenhum erro.
- **Versão em Produção:** Repositório sincronizado e versionado no GitHub [MsAnaCode/ScanIA](https://github.com/MsAnaCode/ScanIA).
