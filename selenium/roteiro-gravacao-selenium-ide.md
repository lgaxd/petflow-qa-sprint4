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

| # | Command | Target | Value |
|---|---|---|---|
| 1 | open | `/tutor/pets` | |
| 2 | click | `xpath=//button[contains(.,'Novo pet')]` | |
| 3 | type | `xpath=(//form//input)[1]` | `Thor QA` |
| 4 | select | `xpath=//form//select` | `label=Cachorro` |
| 5 | type | `xpath=//form//input[@type='date']` | `2020-05-10` |
| 6 | type | `xpath=//form//input[@type='number']` | `28.5` |
| 7 | click | `xpath=//button[contains(.,'Salvar')]` | |
| 8 | wait for element visible | `xpath=//*[contains(text(),'Pet cadastrado com sucesso.')]` | `10000` |
| 9 | assert element present | `xpath=//*[contains(text(),'Thor QA')]` | |

## AUTO-GUI-06 — Pet com data de nascimento futura (CT-S4-09)

| # | Command | Target | Value |
|---|---|---|---|
| 1 | open | `/tutor/pets` | |
| 2 | click | `xpath=//button[contains(.,'Novo pet')]` | |
| 3 | type | `xpath=(//form//input)[1]` | `Pet Futuro QA` |
| 4 | select | `xpath=//form//select` | `label=Cachorro` |
| 5 | type | `xpath=//form//input[@type='date']` | `2030-01-01` |
| 6 | click | `xpath=//button[contains(.,'Salvar')]` | |
| 7 | wait for element visible | `xpath=//*[contains(text(),'A data de nascimento deve ser uma data passada')]` | `10000` |

---

## O que mostrar no vídeo (requisito: preparação/configuração, execução e resultados)

1. **Preparação/configuração:** abrir o Selenium IDE, mostrar o projeto, a URL base e os comandos de cada teste (inclusive os *assert/wait for*). Abrir o Postman e mostrar a coleção, o environment (sem exibir a senha do admin) e os scripts da aba *Tests*.
2. **Execução:** rodar a suíte no Selenium IDE (**Run all tests**) com o navegador visível; rodar a coleção no Postman Runner ou `npx newman run ...` no terminal.
3. **Resultados:** mostrar o log verde do Selenium IDE por teste e o resumo do Runner/newman (requisições, asserções, falhas = 0).
4. Citar, para cada teste automatizado, o caso manual correspondente (CT-S4-xx) no Azure Boards.
