# PetFlow — Plano de Testes Manuais (Sprint 4)

Disciplina: **Compliance, Quality Assurance & Tests — Parte A (Teste Manual de Software)**
Escopo: validação **em nível de sistema** das funcionalidades principais do PetFlow (Web React + API Java publicada no Azure + app mobile).
Rastreabilidade: cada caso corresponde a uma Task do Azure Boards (IDs 155 a 178), já vinculada por tag ao PBI que valida.

> **Como usar este arquivo (decisão de implementação):** o requisito oficial pede que, **no Azure Boards**, cada teste traga (1) teste planejado e objetivo, (2) dados de entrada, (3) dados de saída e (4) procedimento/passos. O CSV exportado só mostra título, estado e tags, então não deu para confirmar se as descrições das Tasks já têm esses quatro blocos. Copie cada bloco abaixo para o campo **Description** da Task correspondente (ou para o campo **Acceptance Criteria**, se preferir separar a saída esperada). Os textos foram escritos a partir do código atual da API (`lgaxd/petflow-api-java`, branch `develop`) e do front (`lgaxd/petflow-front-api-java`). Se a especificação da Task 130 já tiver outro texto, use este arquivo apenas como conferência.

## Ambiente e massa de dados controlada

| Item | Valor |
|---|---|
| Front (Web) | https://561413-petflow-front.azurewebsites.net |
| API / Swagger | https://561413-petflow-webapp.azurewebsites.net/swagger |
| Tutor A (criado no CT-S4-04) | Nome `Tutor QA A` · e-mail `qa.tutor.a@petflow.test` · telefone `(11) 91111-1111` · senha `QaPetflow2026` |
| Tutor B (para isolamento) | Nome `Tutor QA B` · e-mail `qa.tutor.b@petflow.test` · telefone `(11) 94444-4444` · senha `QaPetflow2026` |
| Administrador | Conta ADMIN do ambiente (`admin@petflow.com`). **Não registre a senha no Boards.** |
| Pet 1 do tutor A | `Thor QA` · Cachorro · Labrador · nascimento `10/05/2020` · peso `28.5` |
| Pet 2 do tutor A | `Mia QA` · Gato · Siamês · nascimento `01/08/2021` · peso `4.2` |
| Evento de saúde | Tipo `Vacinação` · descrição `Vacina V10 - QA Sprint 4` · data = **ontem** |
| Cupom de teste | Código `QA-S4-001` · status `DISPONIVEL` · validade `31/12/2099` · template `15% OFF em ração` (**20 pontos**) |
| Regras de pontuação (seed) | Cadastro de pet = **5** · evento de saúde concluído (REALIZADO) = **10** · resgate debita o custo do cupom |

**Ordem de execução:** os casos foram ordenados para que a saída de um seja a pré-condição do seguinte (saldo do tutor A: 0 → 5 → 15 → 20 → 0). Para reexecutar do zero, troque o sufixo dos e-mails (ex.: `qa.tutor.a2@petflow.test`) e o código do cupom (`QA-S4-002`).

**Registro da execução (recomendação):** ao executar, anote na Task o **resultado obtido** (Passou/Falhou), a data e anexe um print; depois mova a Task para **Done**. Falhas viram um item **Bug** vinculado à Task.

---

## CT-S4-01 — Login válido do tutor  (Task 155 · PBI-73)

**Objetivo:** verificar que um tutor cadastrado entra no sistema com e-mail e senha válidos e é direcionado à área do tutor.
**Pré-condições:** CT-S4-04 executado (tutor A existe). Navegador sem sessão ativa.
**Dados de entrada:** e-mail `qa.tutor.a@petflow.test` · senha `QaPetflow2026`.
**Passos:**
1. Acessar `/login` no front.
2. Informar o e-mail no campo **E-mail**.
3. Informar a senha no campo **Senha**.
4. Clicar em **Entrar**.
**Dados de saída esperados:**
- Mensagem `Bem-vindo(a), Tutor!`.
- Redirecionamento para `/tutor`, com o título `Olá, Tutor 👋`.
- (API) `POST /auth/login` → **200**, corpo com `token` JWT e `role = TUTOR`.

## CT-S4-02 — Login com senha inválida  (Task 156 · PBI-73)

**Objetivo:** garantir que credenciais incorretas são recusadas sem revelar se o e-mail existe.
**Pré-condições:** tutor A existe.
**Dados de entrada:** e-mail `qa.tutor.a@petflow.test` · senha `SenhaErrada99`.
**Passos:**
1. Acessar `/login`.
2. Informar o e-mail e a senha incorreta.
3. Clicar em **Entrar**.
**Dados de saída esperados:**
- Mensagem de erro `E-mail ou senha inválidos`.
- Usuário permanece em `/login`; nenhum token é gravado.
- (API) `POST /auth/login` → **401**.

## CT-S4-03 — Controle de acesso por autenticação e perfil  (Task 157 · PBI-73)

**Objetivo:** verificar que rotas protegidas exigem login e que o perfil TUTOR não acessa funções de ADMIN.
**Pré-condições:** tutor A existe.
**Dados de entrada:** (a) sem login: URL `/tutor/pets`; (b) logado como tutor A: URL `/admin/coupons`; (c) API: `GET /tutors` com token do tutor A.
**Passos:**
1. Sem estar logado, digitar na barra a URL `/tutor/pets`.
2. Fazer login como tutor A.
3. Digitar na barra a URL `/admin/coupons`.
4. No Swagger, autorizar com o token do tutor A e executar `GET /tutors`.
**Dados de saída esperados:**
- Passo 1: redirecionamento para `/login`.
- Passo 3: redirecionamento para `/tutor` (a tela de cupons do admin não abre).
- Passo 4: **403** com mensagem `Você não tem permissão para acessar este recurso.`

## CT-S4-04 — Cadastro de tutor válido  (Task 158 · PBI-72)

**Objetivo:** verificar que um novo tutor cria a conta pela tela pública e entra automaticamente.
**Pré-condições:** e-mail `qa.tutor.a@petflow.test` ainda não cadastrado.
**Dados de entrada:** nome `Tutor QA A` · e-mail `qa.tutor.a@petflow.test` · telefone `(11) 91111-1111` · senha e confirmação `QaPetflow2026`.
**Passos:**
1. Acessar `/login` e clicar em **Criar conta** (`/register`).
2. Preencher nome, e-mail, telefone, senha e confirmação.
3. Clicar no botão de envio.
**Dados de saída esperados:**
- Mensagem `Conta criada! Bem-vindo(a), Tutor!`.
- Redirecionamento para `/tutor`.
- (API) `POST /tutors` → **201** com `id` gerado e `role = TUTOR`.

## CT-S4-05 — Cadastro de tutor com e-mail duplicado  (Task 159 · PBI-72)

**Objetivo:** garantir que o e-mail é único no cadastro.
**Pré-condições:** CT-S4-04 executado. Sair da sessão.
**Dados de entrada:** nome `Tutor QA Duplicado` · e-mail `qa.tutor.a@petflow.test` · telefone `(11) 92222-2222` · senha `QaPetflow2026`.
**Passos:**
1. Acessar `/register`.
2. Preencher os dados acima.
3. Enviar o formulário.
**Dados de saída esperados:**
- Mensagem `Tutor já cadastrado(a) com email: qa.tutor.a@petflow.test`; o usuário permanece em `/register`.
- (API) `POST /tutors` → **409** (`Recurso duplicado`).
- Nenhum segundo tutor é criado com o mesmo e-mail.

## CT-S4-06 — Cadastro de tutor com senha fora da regra  (Task 160 · PBI-72)

**Objetivo:** verificar a regra de senha (mínimo 8 caracteres, com pelo menos uma letra e um número).
**Pré-condições:** nenhuma.
**Dados de entrada:** nome `Tutor QA Senha Fraca` · e-mail `qa.fraca@petflow.test` · senha e confirmação `abcdefgh` (8 letras, sem número).
**Passos:**
1. Acessar `/register`.
2. Preencher os dados acima.
3. Enviar o formulário.
**Dados de saída esperados:**
- Mensagem `A senha deve ter no mínimo 8 caracteres, contendo pelo menos uma letra e um número.`
- Conta não criada.
- (API) `POST /tutors` com a mesma senha → **400** com erro de validação no campo `password`.
- Observação: senhas com menos de 8 caracteres são barradas antes pelo próprio navegador (atributo `minLength`), por isso o dado de entrada usa 8 letras sem número.

## CT-S4-07 — Administrador cadastra cupom de teste  (Task 161 · PBI-86, PBI-87)

**Objetivo:** verificar que o administrador cadastra um cupom que passa a ficar disponível no catálogo.
**Pré-condições:** acesso à conta ADMIN.
**Dados de entrada:** código `QA-S4-001` · status `DISPONIVEL` · validade `31/12/2099` · template `15% OFF em ração` (20 pontos).
**Passos:**
1. Fazer login como administrador.
2. Abrir **Cupons** (`/admin/coupons`) e clicar em **Novo cupom**.
3. Preencher os dados e salvar.
**Dados de saída esperados:**
- Mensagem `Cupom criado com sucesso.`
- Cupom `QA-S4-001` listado com status `DISPONIVEL`.
- (API) `POST /coupons` → **201**.

## CT-S4-08 — Cadastro de pet válido  (Task 162 · PBI-75)

**Objetivo:** verificar que o tutor cadastra um pet e recebe a pontuação do cadastro.
**Pré-condições:** logado como tutor A, saldo **0**.
**Dados de entrada:** `Thor QA` · Cachorro · Labrador · `10/05/2020` · `28.5` kg.
**Passos:**
1. Abrir **Meus pets** (`/tutor/pets`) e clicar em **Novo pet**.
2. Preencher os dados e clicar em **Salvar**.
3. Voltar ao painel do tutor e conferir o saldo.
**Dados de saída esperados:**
- Mensagem `Pet cadastrado com sucesso.` e pet `Thor QA` na lista.
- Saldo de pontos = **5**.
- (API) `POST /pets` → **201**.

## CT-S4-09 — Cadastro de pet com data de nascimento futura  (Task 163 · PBI-75)

**Objetivo:** garantir que a data de nascimento precisa ser passada.
**Pré-condições:** logado como tutor A.
**Dados de entrada:** `Pet Futuro QA` · Cachorro · SRD · nascimento = hoje + 30 dias · `5` kg.
**Passos:**
1. Em **Meus pets**, clicar em **Novo pet**.
2. Preencher os dados com a data futura e salvar.
**Dados de saída esperados:**
- Mensagem `A data de nascimento deve ser uma data passada`; o pet **não** aparece na lista.
- Saldo continua **5**.
- (API) `POST /pets` → **400** com erro no campo `birthDate` (`A data de nascimento deve ser uma data passada`).

## CT-S4-10 — Isolamento de dados entre tutores  (Task 164 · PBI-75, PBI-76)

**Objetivo:** garantir que um tutor não vê nem acessa pets de outro tutor.
**Pré-condições:** tutor A com o pet `Thor QA` (anotar o `id`). Tutor B cadastrado.
**Dados de entrada:** login do tutor B; `GET /pets/{id do Thor QA}` com token do tutor B.
**Passos:**
1. Fazer login como tutor B e abrir **Meus pets**.
2. No Swagger, autorizar com o token do tutor B e executar `GET /pets/{id}`.
**Dados de saída esperados:**
- Passo 1: lista vazia (o `Thor QA` não aparece).
- Passo 2: **403** `Acesso negado`.

## CT-S4-11 — Registrar evento de saúde realizado  (Task 165 · PBI-79, PBI-82)

**Objetivo:** verificar que concluir um evento de saúde gera pontos para o tutor.
**Pré-condições:** evento do CT-S4-12 (passo 1) agendado para o `Thor QA`; saldo **5**.
**Dados de entrada:** evento `Vacina V10 - QA Sprint 4` · data = ontem · status `REALIZADO`.
**Passos:**
1. Em **Eventos de saúde**, editar o evento e mudar o status para `REALIZADO`.
2. Salvar e conferir o saldo no painel.
**Dados de saída esperados:**
- Evento com status `REALIZADO`.
- Saldo = **15** (+10).
- (API) `PUT /health-events/{id}` → **200**.

## CT-S4-12 — Evento agendado só pontua ao ser concluído, uma única vez  (Task 166 · PBI-82)

**Objetivo:** verificar que evento `AGENDADO` não pontua e que a conclusão pontua uma vez só.
**Pré-condições:** tutor A com `Thor QA`; saldo **5**.
**Dados de entrada:** tipo `Vacinação` · `Vacina V10 - QA Sprint 4` · data = ontem · status inicial `AGENDADO`.
**Passos:**
1. Em **Eventos de saúde**, criar o evento com status `AGENDADO`; conferir o saldo.
2. Executar o CT-S4-11 (mudar para `REALIZADO`); conferir o saldo.
3. Salvar o mesmo evento novamente como `REALIZADO` (ou reenviar `PUT /health-events/{id}` com o mesmo corpo); conferir o saldo.
**Dados de saída esperados:**
- Após o passo 1: saldo **5** (sem pontuação).
- Após o passo 2: saldo **15**.
- Após o passo 3: saldo continua **15**; o extrato tem um único lançamento do evento.

## CT-S4-13 — Transição de status inválida do evento  (Task 167 · PBI-79)

**Objetivo:** garantir que um evento `REALIZADO` não volta para `AGENDADO`.
**Pré-condições:** evento do CT-S4-11 com status `REALIZADO`.
**Dados de entrada:** mesmo evento com status `AGENDADO`.
**Passos:**
1. Editar o evento e trocar o status para `AGENDADO` (ou `PUT /health-events/{id}` com `status = AGENDADO`).
2. Salvar.
**Dados de saída esperados:**
- Mensagem de erro de transição inválida; o evento permanece `REALIZADO`.
- (API) **422** `Transição de status inválida`.
- Saldo continua **15**.

## CT-S4-14 — Evento de saúde com data futura  (Task 168 · PBI-79)

**Objetivo:** garantir que um evento marcado como `REALIZADO` não pode ter data futura (agendado pode).
**Pré-condições:** tutor A com `Thor QA`.
**Dados de entrada:** `Vacinação` · data = hoje + 30 dias · status `REALIZADO`.
**Passos:**
1. Em **Eventos de saúde**, criar o evento com os dados acima.
2. Salvar.
**Dados de saída esperados:**
- Mensagem `Um evento realizado não pode ter data futura`; evento não criado.
- (API) `POST /health-events` → **422**, `code = FUTURE_DATE_NOT_ALLOWED`.
- Saldo inalterado.

## CT-S4-15 — Evento de saúde para pet inexistente  (Task 169 · PBI-79)

**Objetivo:** garantir que não é possível registrar evento para um pet que não existe.
**Pré-condições:** logado como tutor A (token no Swagger).
**Dados de entrada:** `POST /health-events` com `petId = 999999999`, `eventTypeId` = Vacinação, data = hoje + 30, status `AGENDADO`.
**Passos:**
1. No Swagger, autorizar com o token do tutor A.
2. Executar `POST /health-events` com o corpo acima.
**Dados de saída esperados:**
- **404** `Entidade não encontrada` (Pet 999999999).
- Nenhum evento criado.

## CT-S4-16 — Consultar saldo e extrato de pontos  (Task 170 · PBI-83, PBI-84)

**Objetivo:** verificar que o tutor consulta o saldo e o histórico de pontos coerentes com as ações realizadas.
**Pré-condições:** CT-S4-08 e CT-S4-11 executados.
**Dados de entrada:** login do tutor A.
**Passos:**
1. Abrir o painel do tutor (`/tutor`) e conferir **Seu saldo de pontos** e **Histórico recente de pontos**.
2. (API) `GET /gamification/points`.
**Dados de saída esperados:**
- Saldo **15**.
- Extrato com 2 lançamentos: cadastro de pet (**+5**) e evento de saúde realizado (**+10**).

## CT-S4-17 — Consultar catálogo de cupons  (Task 171 · PBI-87)

**Objetivo:** verificar que o tutor vê os cupons disponíveis com o custo em pontos.
**Pré-condições:** CT-S4-07 executado.
**Dados de entrada:** login do tutor A.
**Passos:**
1. Abrir **Resgatar cupons** (`/tutor/redeem`).
**Dados de saída esperados:**
- Cupom `15% OFF em ração` (`QA-S4-001`) listado com custo **20 pontos**.
- Com saldo 15, o botão do cupom mostra `Pontos insuficientes`.

## CT-S4-18 — Resgate de cupom com saldo suficiente  (Task 172 · PBI-88, PBI-89)

**Objetivo:** verificar o resgate com saldo suficiente e o débito dos pontos.
**Pré-condições:** saldo 15 → cadastrar o pet `Mia QA` (+5) para chegar a **20**.
**Dados de entrada:** cupom `QA-S4-001` (20 pontos).
**Passos:**
1. Cadastrar o pet `Mia QA`; conferir saldo **20**.
2. Abrir **Resgatar cupons**, clicar em **Resgatar** no cupom e confirmar.
3. Conferir o saldo e a lista de cupons resgatados.
**Dados de saída esperados:**
- Mensagem `Cupom "15% OFF em ração" resgatado com sucesso!`
- Saldo = **0**.
- Cupom aparece entre os resgatados do tutor A; deixa de estar disponível no catálogo.
- (API) `POST /gamification/redeem` → **200**.

## CT-S4-19 — Resgate de cupom com saldo insuficiente  (Task 173 · PBI-88)

**Objetivo:** garantir que não é possível resgatar com saldo menor que o custo.
**Pré-condições:** saldo do tutor A = **15** (antes do passo 1 do CT-S4-18); cupom `QA-S4-001` disponível.
**Dados de entrada:** `POST /gamification/redeem` com `couponId` do `QA-S4-001`.
**Passos:**
1. Na tela **Resgatar cupons**, verificar o botão do cupom.
2. No Swagger, com o token do tutor A, executar o resgate.
**Dados de saída esperados:**
- Passo 1: botão desabilitado com o texto `Pontos insuficientes`.
- Passo 2: **422** `Pontos insuficientes`, com detalhe de saldo atual 15 e necessário 20.
- Saldo continua **15**.

## CT-S4-20 — Resgate de cupom já resgatado  (Task 174 · PBI-88)

**Objetivo:** garantir que um cupom não é resgatado duas vezes.
**Pré-condições:** CT-S4-18 executado.
**Dados de entrada:** `POST /gamification/redeem` com o mesmo `couponId`.
**Passos:**
1. Com o token do tutor A, executar novamente o resgate do `QA-S4-001`.
**Dados de saída esperados:**
- **422** (`Cupom não está disponível para resgate`).
- Nenhum novo resgate criado; saldo inalterado.

## CT-S4-21 — Resgate de cupom inexistente  (Task 175 · PBI-88)

**Objetivo:** garantir o tratamento de cupom que não existe.
**Pré-condições:** logado como tutor A.
**Dados de entrada:** `couponId = 999999999`.
**Passos:**
1. Executar `POST /gamification/redeem` com o corpo `{"couponId": 999999999}`.
**Dados de saída esperados:**
- **404** `Entidade não encontrada`.
- Saldo inalterado.

## CT-S4-22 — Resgate sem autenticação  (Task 176 · PBI-73, PBI-88)

**Objetivo:** garantir que o resgate exige usuário autenticado.
**Pré-condições:** nenhuma.
**Dados de entrada:** `POST /gamification/redeem` sem cabeçalho `Authorization`, corpo `{"couponId": <id do QA-S4-001>}`.
**Passos:**
1. No Swagger (sem autorizar) ou Postman (sem token), executar o resgate.
**Dados de saída esperados:**
- **401** `Credenciais ausentes ou inválidas. Faça login novamente.`

## CT-S4-23 — Cupons resgatados visíveis apenas ao dono  (Task 177 · PBI-89)

**Objetivo:** garantir que um tutor não vê os resgates de outro.
**Pré-condições:** CT-S4-18 executado (anotar o `id` do resgate). Tutor B cadastrado.
**Dados de entrada:** login do tutor B; `GET /redeems/{id do resgate do tutor A}` com token do tutor B.
**Passos:**
1. Logado como tutor B, abrir a lista de cupons resgatados.
2. No Swagger, com o token do tutor B, executar `GET /redeems/{id}`.
**Dados de saída esperados:**
- Passo 1: lista vazia.
- Passo 2: **403** `Acesso negado`.

## CT-S4-24 — App mobile: cadastro, login, pet e logout  (Task 178 · PBI-72, PBI-73, PBI-75)

**Objetivo:** validar o fluxo principal no app mobile publicado, integrado à mesma API.
**Pré-condições:** app instalado pela versão publicada no Firebase App Distribution.
**Dados de entrada:** nome `Tutor QA Mobile` · e-mail `qa.tutor.mobile@petflow.test` · senha `QaPetflow2026` · pet `Bob QA` · Cachorro · nascimento `15/03/2019`.
**Passos:**
1. Abrir o app e criar a conta com os dados acima.
2. Sair e entrar novamente com o mesmo e-mail e senha.
3. Cadastrar o pet `Bob QA`.
4. Fechar e reabrir a lista de pets; confirmar que o pet continua lá.
5. Fazer logout.
**Dados de saída esperados:**
- Conta criada e login realizado sem erro.
- Pet `Bob QA` listado (persistido na API; também aparece no front web ao entrar com o mesmo usuário).
- Após o logout, o app volta à tela de login e não abre telas internas sem novo login.

> Os nomes de telas e botões deste caso precisam ser conferidos no app mobile: o repositório do app não foi analisado.
