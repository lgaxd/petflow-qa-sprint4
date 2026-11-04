# PetFlow — Compliance, Quality Assurance & Tests (Sprint 4)

Entrega da disciplina **Compliance, Quality Assurance & Tests** do Challenge FIAP 2026 — Sprint 4.

## Links da entrega

| Item | Link |
|---|---|
| **Azure Boards — plano de testes manuais (Parte A)** | https://dev.azure.com/RM562027/PetFlow |
| **Vídeo — configuração e execução dos testes automatizados (Parte B)** | `<COLE AQUI o link do vídeo>` |
| Aplicação web testada | https://561413-petflow-front.azurewebsites.net |
| API testada (Swagger) | https://561413-petflow-webapp.azurewebsites.net/swagger |

O professor está cadastrado na organização do Azure DevOps com perfil **Basic** e no projeto com acesso **Administrador**.

## Integrantes

| Nome | RM |
|---|---:|
| Lucas Grillo Alcântara | 561413 |
| Pietro Ferreira Gomes Abrahamian | 561469 |
| Pedro Peres Benitez | 561792 |
| Lucca Ramos Mussumecci | 562027 |

## Parte A — Testes manuais (Azure Boards)

24 casos de teste de sistema (CT-S4-01 a CT-S4-24), um por Task no Azure Boards, vinculados aos PBIs que validam. Cada Task traz objetivo, dados de entrada, passos e dados de saída esperados, com massa de dados controlada. Cópia da especificação: [docs/casos-de-teste-manuais.md](docs/casos-de-teste-manuais.md).

## Parte B — Testes automatizados

| ID | Ferramenta | Caso manual | O que valida |
|---|---|---|---|
| AUTO-GUI-01 | Selenium IDE | CT-S4-04 | Cadastro de tutor válido |
| AUTO-GUI-02 | Selenium IDE | CT-S4-05 | Cadastro com e-mail duplicado |
| AUTO-GUI-03 | Selenium IDE | CT-S4-02 | Login com senha inválida |
| AUTO-GUI-04 | Selenium IDE | CT-S4-01 | Login válido |
| AUTO-GUI-05 | Selenium IDE | CT-S4-08 | Cadastro de pet válido |
| AUTO-GUI-06 | Selenium IDE | CT-S4-09 | Pet com data de nascimento futura |
| Coleção Postman (28 requisições de teste) | Postman / newman | CT-S4-01 a 03, 05 a 23 | Autenticação, perfis, cadastro, pets, eventos de saúde, pontuação, catálogo e resgate de cupons, isolamento entre tutores |

### Selenium IDE (interface)

1. Instale a extensão Selenium IDE no Chrome.
2. **Open project** → `selenium/PetFlow-QA-Sprint4.side`.
3. Confira a URL base `https://561413-petflow-front.azurewebsites.net` e rode **Run all tests in suite**.

Roteiro dos comandos: [selenium/roteiro-gravacao-selenium-ide.md](selenium/roteiro-gravacao-selenium-ide.md).

### Postman / newman (API e serviço)

1. Importe `postman/PetFlow-QA-Sprint4.postman_collection.json`.
2. Copie `postman/PetFlow-QA-Sprint4.postman_environment.example.json` para `local.postman_environment.json` (arquivo ignorado pelo Git) e preencha `admin_password`.
3. Rode a coleção inteira, em ordem, no **Collection Runner**, ou no terminal:

```bash
npx newman run postman/PetFlow-QA-Sprint4.postman_collection.json -e local.postman_environment.json
```

A coleção cria a própria massa de dados a cada execução (tutores com e-mail único) e verifica o saldo de pontos em cada etapa. As regras de pontuação de cada ambiente ficam no environment e são conferidas no início da execução:

| Variável | Azure (Oracle FIAP) | Oracle local (seed V2) |
|---|---|---|
| `event_type_id` / `event_points` | `3` (Consulta de Retorno) / `40` | `1` (Vacinação) / `10` |
| `coupon_template_id` / `coupon_points` | `1` (15% OFF em produtos PataFeliz) / `50` | `1` (15% OFF em ração) / `20` |

Saldo esperado do tutor A no Azure: 0 → 5 → 45 → 50 → 0.

### Evidências

| Execução | Resultado | Arquivo |
|---|---|---|
| newman contra a API publicada no Azure | 35 requisições, 64 asserções, **0 falhas** | [evidencias/newman-azure.txt](evidencias/newman-azure.txt) · [JUnit](evidencias/newman-azure.xml) |
| selenium-side-runner contra o front publicado (Edge) | 6/6 testes passaram | [evidencias/selenium-side-runner.txt](evidencias/selenium-side-runner.txt) |

## Estrutura

```
docs/       especificação dos casos de teste manuais
postman/    coleção e environment de exemplo (sem credenciais)
selenium/   projeto .side do Selenium IDE e roteiro de gravação
evidencias/ saídas reais das execuções automatizadas (newman e selenium-side-runner)
```
