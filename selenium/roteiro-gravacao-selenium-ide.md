# Roteiro de gravação — Selenium IDE (automação GUI, Record & Playback)

Ferramenta: **Selenium IDE** (extensão do Chrome/Firefox). Projeto: `PetFlow QA Sprint 4`. URL base: `https://561413-petflow-front.azurewebsites.net`.

Os textos esperados abaixo foram conferidos rodando o front e a API atuais (branch `develop` da API e `main` do front) em ambiente local. Grave cada teste com o botão **Rec**, execute o fluxo e depois adicione os comandos de verificação (clique direito no elemento → *Selenium IDE → Assert / Wait For*) ou digite-os na tabela de comandos.

Ao final: **File → Save project** gera `PetFlow-QA-Sprint4.side`. Salve esse arquivo nesta pasta (`selenium/`) do repositório.

## Variável de e-mail único (evita conflito ao reexecutar)

Comando usado no AUTO-GUI-01:

| Command | Target | Value |
|---|---|---|
| execute script | `return 'qa.gui.' + Date.now() + '@petflow.test'` | `email` |

Use `${email}` nos campos de e-mail. Coloque os seis testes em uma suíte, nesta ordem, e nas configurações da suíte marque **Persist session** (mantém o login entre os testes). Rode com **Run all tests in suite**: o AUTO-GUI-01 cria o usuário que os demais usam. Se `${email}` chegar vazio aos testes seguintes, troque-o pelo e-mail literal que aparece no log do AUTO-GUI-01.

---

## Esperas antes das verificações

O painel do tutor e a lista de pets mostram um *spinner* enquanto a API responde (de 1 a 7 s no Azure) e só então renderizam o título e os itens; as mensagens (toasts) também dependem da resposta da API. No Selenium IDE, `assert element present` não espera esse carregamento. Por isso toda verificação de conteúdo vindo da API tem antes um `wait for element present` (15 s); o `assert`/`wait for element visible` continua logo depois.

## AUTO-GUI-01 — Cadastro de tutor válido (CT-S4-04)

> **Navegador sem sessão (passos 1 a 4, repetidos no início do 02, 03 e 04):** com usuário logado, `/register` e `/login` redirecionam para `/tutor` e os campos não existem. O front guarda a sessão só no `localStorage` (`petflow_token` e `petflow_user`); os passos removem essas chaves e reabrem a página. Assim a suíte não depende de uma execução anterior ter terminado com **Sair**.

| # | Command | Target | Value |
|---|---|---|---|
| 1 | open | `/register` |  |
| 2 | execute script | `localStorage.removeItem('petflow_token'); localStorage.removeItem('petflow_user');` |  |
| 3 | open | `/register` |  |
| 4 | wait for element present | `xpath=//input[@type='email']` | `15000` |
| 5 | execute script | `return 'qa.gui.' + Date.now() + '@petflow.test'` | `email` |
| 6 | echo | `${email}` |  |
| 7 | type | `xpath=//input[@type='text']` | `Tutor QA GUI` |
| 8 | type | `xpath=//input[@type='email']` | `${email}` |
| 9 | type | `xpath=//input[@type='tel']` | `(11) 91111-1111` |
| 10 | type | `xpath=(//input[@type='password'])[1]` | `QaPetflow2026` |
| 11 | type | `xpath=(//input[@type='password'])[2]` | `QaPetflow2026` |
| 12 | click | `css=button[type=submit]` |  |
| 13 | wait for element present | `xpath=//*[contains(text(),'Conta criada!')]` | `15000` |
| 14 | wait for element visible | `xpath=//*[contains(text(),'Conta criada!')]` | `15000` |
| 15 | wait for element present | `xpath=//h1[contains(.,'Olá, Tutor')]` | `15000` |
| 16 | assert element present | `xpath=//h1[contains(.,'Olá, Tutor')]` |  |
| 17 | click | `xpath=//button[contains(.,'Sair')]` |  |
| 18 | wait for element present | `css=input[type=email]` | `15000` |

## AUTO-GUI-02 — Cadastro com e-mail duplicado (CT-S4-05)

| # | Command | Target | Value |
|---|---|---|---|
| 1 | open | `/register` |  |
| 2 | execute script | `localStorage.removeItem('petflow_token'); localStorage.removeItem('petflow_user');` |  |
| 3 | open | `/register` |  |
| 4 | wait for element present | `xpath=//input[@type='email']` | `15000` |
| 5 | type | `xpath=//input[@type='text']` | `Tutor QA Duplicado` |
| 6 | type | `xpath=//input[@type='email']` | `${email}` |
| 7 | type | `xpath=(//input[@type='password'])[1]` | `QaPetflow2026` |
| 8 | type | `xpath=(//input[@type='password'])[2]` | `QaPetflow2026` |
| 9 | click | `css=button[type=submit]` |  |
| 10 | wait for element present | `xpath=//*[contains(text(),'Tutor já cadastrado')]` | `15000` |
| 11 | wait for element visible | `xpath=//*[contains(text(),'Tutor já cadastrado')]` | `15000` |
| 12 | assert element present | `xpath=//h1[contains(.,'Criar conta')]` |  |

> Se rodar este teste isolado, troque `${email}` por um e-mail que já exista (ex.: o do tutor A do plano manual).

## AUTO-GUI-03 — Login com senha inválida (CT-S4-02)

| # | Command | Target | Value |
|---|---|---|---|
| 1 | open | `/login` |  |
| 2 | execute script | `localStorage.removeItem('petflow_token'); localStorage.removeItem('petflow_user');` |  |
| 3 | open | `/login` |  |
| 4 | wait for element present | `css=input[type=email]` | `15000` |
| 5 | type | `css=input[type=email]` | `${email}` |
| 6 | type | `css=input[type=password]` | `SenhaErrada99` |
| 7 | click | `css=button[type=submit]` |  |
| 8 | wait for element present | `xpath=//*[contains(text(),'E-mail ou senha inválidos')]` | `15000` |
| 9 | wait for element visible | `xpath=//*[contains(text(),'E-mail ou senha inválidos')]` | `15000` |

## AUTO-GUI-04 — Login válido do tutor (CT-S4-01)

| # | Command | Target | Value |
|---|---|---|---|
| 1 | open | `/login` |  |
| 2 | execute script | `localStorage.removeItem('petflow_token'); localStorage.removeItem('petflow_user');` |  |
| 3 | open | `/login` |  |
| 4 | wait for element present | `css=input[type=email]` | `15000` |
| 5 | type | `css=input[type=email]` | `${email}` |
| 6 | type | `css=input[type=password]` | `QaPetflow2026` |
| 7 | click | `css=button[type=submit]` |  |
| 8 | wait for element present | `xpath=//h1[contains(.,'Olá, Tutor')]` | `15000` |
| 9 | wait for element visible | `xpath=//h1[contains(.,'Olá, Tutor')]` | `15000` |
| 10 | assert element present | `xpath=//*[contains(text(),'Seu saldo de pontos')]` |  |

## AUTO-GUI-05 — Cadastro de pet válido (CT-S4-08)

Pré-condição: AUTO-GUI-04 executado (usuário logado).

> **Abrir, cancelar e reabrir o modal (passos 1 a 5):** o formulário de pet nasce com a primeira espécie *já carregada* de `GET /species`. Se o modal abre antes dessa resposta, o estado fica `speciesId = 0`; a lista mostra "Cachorro", mas selecionar a opção já exibida não dispara `onChange`, e a API responde 409 "Registro relacionado não encontrado". Na reprodução automática isso acontece sempre. Os passos esperam as espécies e reabrem o modal. Comportamento do front publicado, observado em 03/11/2026.
>
> **Data em ISO (`aaaa-mm-dd`):** o Selenium IDE grava e reproduz o valor do `input type=date` em ISO (`2020-05-10`). Digitar `10/05/2020` no IDE deixa o campo vazio e o pet é salvo sem data de nascimento.

| # | Command | Target | Value |
|---|---|---|---|
| 1 | open | `/tutor/pets` |  |
| 2 | click | `xpath=//button[contains(.,'Novo pet')]` |  |
| 3 | wait for element present | `xpath=//form//select/option[.='Cachorro']` | `30000` |
| 4 | click | `xpath=//button[contains(.,'Cancelar')]` |  |
| 5 | click | `xpath=//button[contains(.,'Novo pet')]` |  |
| 6 | type | `xpath=(//form//input)[1]` | `Thor QA` |
| 7 | select | `xpath=//form//select` | `label=Cachorro` |
| 8 | type | `xpath=//form//input[@type='date']` | `2020-05-10` |
| 9 | type | `xpath=//form//input[@type='number']` | `28.5` |
| 10 | click | `xpath=//button[contains(.,'Salvar')]` |  |
| 11 | wait for element present | `xpath=//*[contains(text(),'Pet cadastrado com sucesso.')]` | `15000` |
| 12 | wait for element visible | `xpath=//*[contains(text(),'Pet cadastrado com sucesso.')]` | `15000` |
| 13 | wait for element present | `xpath=//*[contains(text(),'Thor QA')]` | `15000` |
| 14 | assert element present | `xpath=//*[contains(text(),'Thor QA')]` |  |

## AUTO-GUI-06 — Pet com data de nascimento futura (CT-S4-09)

Mesma preparação do AUTO-GUI-05 (passos 1 a 5). Com a data em ISO, a API recusa a data futura e o front mostra a mensagem de erro.

| # | Command | Target | Value |
|---|---|---|---|
| 1 | open | `/tutor/pets` |  |
| 2 | click | `xpath=//button[contains(.,'Novo pet')]` |  |
| 3 | wait for element present | `xpath=//form//select/option[.='Cachorro']` | `30000` |
| 4 | click | `xpath=//button[contains(.,'Cancelar')]` |  |
| 5 | click | `xpath=//button[contains(.,'Novo pet')]` |  |
| 6 | type | `xpath=(//form//input)[1]` | `Pet Futuro QA` |
| 7 | select | `xpath=//form//select` | `label=Cachorro` |
| 8 | type | `xpath=//form//input[@type='date']` | `2030-01-01` |
| 9 | click | `xpath=//button[contains(.,'Salvar')]` |  |
| 10 | wait for element present | `xpath=//*[contains(text(),'A data de nascimento deve ser uma data passada')]` | `15000` |
| 11 | wait for element visible | `xpath=//*[contains(text(),'A data de nascimento deve ser uma data passada')]` | `15000` |

---

## Projeto pronto e validação por linha de comando

`PetFlow-QA-Sprint4.side` (nesta pasta) contém os seis testes acima, prontos para abrir no Selenium IDE (*Open project*) e servir de referência durante a gravação.

Validado em 03/11/2026 (23:47) com `selenium-side-runner` 4.0.13 (Microsoft Edge, idioma pt-BR) contra o front publicado: 6/6 testes passaram, veja [`evidencias/selenium-side-runner.txt`](../evidencias/selenium-side-runner.txt). O runner 4.0.13 trava quando a suíte usa *Persist session* (depois do primeiro teste a sessão é fechada e os demais esperam até o timeout). Por isso a validação rodou em duas fases: AUTO-GUI-01 e, em seguida, AUTO-GUI-02 a 06 com o e-mail criado pelo 01 e um login antes do 05 e 06, que é o que o *Persist session* faz no Selenium IDE. Como o WebDriver digita a data tecla a tecla, nessa validação as datas ISO do projeto são convertidas para a ordem do pt-BR (`10/05/2020`); no Selenium IDE vale o ISO.

**Primeira execução no Selenium IDE (03/11/2026, 23:28):** falhou por dois motivos já corrigidos neste arquivo: verificações sem espera pelo carregamento da API e datas em `dd/mm/aaaa`. A falha do AUTO-GUI-01 manteve a sessão aberta, o que derrubou em cascata os testes 02 a 04.

**Segunda execução no Selenium IDE (03/11/2026, 23:44):** o AUTO-GUI-05 gravou o pet com a data correta e o 06 teve a data futura recusada, mas 01 a 04 falharam porque o navegador ainda estava logado com o tutor da execução anterior. Os testes 01 a 04 agora limpam a sessão no início; a validação por linha de comando começou propositalmente logada para reproduzir esse cenário.

## O que mostrar no vídeo (requisito: preparação/configuração, execução e resultados)

1. **Preparação/configuração:** abrir o Selenium IDE, mostrar o projeto, a URL base e os comandos de cada teste (inclusive os *assert/wait for*). Abrir o Postman e mostrar a coleção, o environment (sem exibir a senha do admin) e os scripts da aba *Tests*.
2. **Execução:** rodar a suíte no Selenium IDE (**Run all tests**) com o navegador visível; rodar a coleção no Postman Runner ou `npx newman run ...` no terminal.
3. **Resultados:** mostrar o log verde do Selenium IDE por teste e o resumo do Runner/newman (requisições, asserções, falhas = 0).
4. Citar, para cada teste automatizado, o caso manual correspondente (CT-S4-xx) no Azure Boards.
