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

## AUTO-GUI-01 — Cadastro de tutor válido (CT-S4-04)

| # | Command | Target | Value |
|---|---|---|---|
| 1 | open | `/register` | |
| 2 | execute script | `return 'qa.gui.' + Date.now() + '@petflow.test'` | `email` |
| 2a | echo | `${email}` | |
| 3 | type | `xpath=//input[@type='text']` | `Tutor QA GUI` |
| 4 | type | `xpath=//input[@type='email']` | `${email}` |
| 5 | type | `xpath=//input[@type='tel']` | `(11) 91111-1111` |
| 6 | type | `xpath=(//input[@type='password'])[1]` | `QaPetflow2026` |
| 7 | type | `xpath=(//input[@type='password'])[2]` | `QaPetflow2026` |
| 8 | click | `css=button[type=submit]` | |
| 9 | wait for element visible | `xpath=//*[contains(text(),'Conta criada!')]` | `10000` |
| 10 | assert element present | `xpath=//h1[contains(.,'Olá, Tutor')]` | |
| 11 | click | `xpath=//button[contains(.,'Sair')]` | |

## AUTO-GUI-02 — Cadastro com e-mail duplicado (CT-S4-05)

| # | Command | Target | Value |
|---|---|---|---|
| 1 | open | `/register` | |
| 2 | type | `xpath=//input[@type='text']` | `Tutor QA Duplicado` |
| 3 | type | `xpath=//input[@type='email']` | `${email}` |
| 4 | type | `xpath=(//input[@type='password'])[1]` | `QaPetflow2026` |
| 5 | type | `xpath=(//input[@type='password'])[2]` | `QaPetflow2026` |
| 6 | click | `css=button[type=submit]` | |
| 7 | wait for element visible | `xpath=//*[contains(text(),'Tutor já cadastrado')]` | `10000` |
| 8 | assert element present | `xpath=//h1[contains(.,'Criar conta')]` | |

> Se rodar este teste isolado, troque `${email}` por um e-mail que já exista (ex.: o do tutor A do plano manual).

## AUTO-GUI-03 — Login com senha inválida (CT-S4-02)

| # | Command | Target | Value |
|---|---|---|---|
| 1 | open | `/login` | |
| 2 | type | `css=input[type=email]` | `${email}` |
| 3 | type | `css=input[type=password]` | `SenhaErrada99` |
| 4 | click | `css=button[type=submit]` | |
| 5 | wait for element visible | `xpath=//*[contains(text(),'E-mail ou senha inválidos')]` | `10000` |

## AUTO-GUI-04 — Login válido do tutor (CT-S4-01)

| # | Command | Target | Value |
|---|---|---|---|
| 1 | open | `/login` | |
| 2 | type | `css=input[type=email]` | `${email}` |
| 3 | type | `css=input[type=password]` | `QaPetflow2026` |
| 4 | click | `css=button[type=submit]` | |
| 5 | wait for element visible | `xpath=//h1[contains(.,'Olá, Tutor')]` | `10000` |
| 6 | assert element present | `xpath=//*[contains(text(),'Seu saldo de pontos')]` | |

## AUTO-GUI-05 — Cadastro de pet válido (CT-S4-08)

Pré-condição: AUTO-GUI-04 executado (usuário logado).

> **Por que abrir, cancelar e reabrir o modal (passos 1a–1e):** o formulário de pet nasce com a primeira espécie *já carregada* de `GET /species`. Se o modal abre antes dessa resposta, o estado fica `speciesId = 0`; a lista mostra "Cachorro", mas selecionar a opção já exibida não dispara `onChange`, e a API responde 409 "Registro relacionado não encontrado". Na reprodução automática isso acontece sempre. Os passos esperam as espécies e reabrem o modal. Observado no front publicado em 03/11/2026; o comportamento é do front, não do teste.
>
> **Data:** `type` em `input type=date` envia teclas na ordem do idioma do navegador. Com o navegador em **pt-BR** use `dd/mm/aaaa` (`10/05/2020`); em inglês (en-US) use `05/10/2020`. Digitar `2020-05-10` em pt-BR gera `0510-02-20`.

| # | Command | Target | Value |
|---|---|---|---|
| 1a | open | `/tutor/pets` | |
| 1b | click | `xpath=//button[contains(.,'Novo pet')]` | |
| 1c | wait for element present | `xpath=//form//select/option[.='Cachorro']` | `30000` |
| 1d | click | `xpath=//button[contains(.,'Cancelar')]` | |
| 1e | click | `xpath=//button[contains(.,'Novo pet')]` | |
| 3 | type | `xpath=(//form//input)[1]` | `Thor QA` |
| 4 | select | `xpath=//form//select` | `label=Cachorro` |
| 5 | type | `xpath=//form//input[@type='date']` | `10/05/2020` |
| 6 | type | `xpath=//form//input[@type='number']` | `28.5` |
| 7 | click | `xpath=//button[contains(.,'Salvar')]` | |
| 8 | wait for element visible | `xpath=//*[contains(text(),'Pet cadastrado com sucesso.')]` | `10000` |
| 9 | assert element present | `xpath=//*[contains(text(),'Thor QA')]` | |

## AUTO-GUI-06 — Pet com data de nascimento futura (CT-S4-09)

| # | Command | Target | Value |
|---|---|---|---|
| 1a–1e | | mesmos passos 1a–1e do AUTO-GUI-05 | |
| 3 | type | `xpath=(//form//input)[1]` | `Pet Futuro QA` |
| 4 | select | `xpath=//form//select` | `label=Cachorro` |
| 5 | type | `xpath=//form//input[@type='date']` | `01/01/2030` (pt-BR; en-US: `01/01/2030`) |
| 6 | click | `xpath=//button[contains(.,'Salvar')]` | |
| 7 | wait for element visible | `xpath=//*[contains(text(),'A data de nascimento deve ser uma data passada')]` | `10000` |

---

## Projeto pronto e validação por linha de comando

`PetFlow-QA-Sprint4.side` (nesta pasta) contém os seis testes acima, prontos para abrir no Selenium IDE (*Open project*) e servir de referência durante a gravação.

Validado em 03/11/2026 com `selenium-side-runner` 4.0.13 (Microsoft Edge, idioma pt-BR) contra o front publicado: 6/6 testes passaram, veja [`evidencias/selenium-side-runner.txt`](../evidencias/selenium-side-runner.txt). O runner 4.0.13 trava quando a suíte usa *Persist session* (depois do primeiro teste a sessão é fechada e os demais esperam até o timeout). Por isso a validação rodou em duas fases: AUTO-GUI-01 e, em seguida, AUTO-GUI-02 a 06 com o e-mail criado pelo 01 e um login antes do 05 e 06, que é o que o *Persist session* faz no Selenium IDE.

## O que mostrar no vídeo (requisito: preparação/configuração, execução e resultados)

1. **Preparação/configuração:** abrir o Selenium IDE, mostrar o projeto, a URL base e os comandos de cada teste (inclusive os *assert/wait for*). Abrir o Postman e mostrar a coleção, o environment (sem exibir a senha do admin) e os scripts da aba *Tests*.
2. **Execução:** rodar a suíte no Selenium IDE (**Run all tests**) com o navegador visível; rodar a coleção no Postman Runner ou `npx newman run ...` no terminal.
3. **Resultados:** mostrar o log verde do Selenium IDE por teste e o resumo do Runner/newman (requisições, asserções, falhas = 0).
4. Citar, para cada teste automatizado, o caso manual correspondente (CT-S4-xx) no Azure Boards.
