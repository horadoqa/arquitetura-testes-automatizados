# Arquitetura - Playwright

A ideia é montar uma arquitetura de testes automatizados com **Playwright** que não vire um conjunto de scripts frágeis depois de alguns meses, eu pensaria em duas dimensões: **fundação imediata** e **evolução de longo prazo**.

 ## 1\. O que precisa ser pensado de imediato

 Antes de escrever muitos testes, defina uma estrutura mínima consistente:

```
tests/
├── e2e/
│   ├── login/
│   │   ├── login.spec.ts
│   │   └── login.data.ts
│   ├── checkout/
│   │   └── checkout.spec.ts
│   └── ...
│
├── fixtures/
│   ├── auth.fixture.ts
│   └── test.fixture.ts
│
├── pages/
│   ├── LoginPage.ts
│   ├── CheckoutPage.ts
│   └── ...
│
├── components/
│   ├── Header.ts
│   ├── Modal.ts
│   └── ...
│
├── api/
│   ├── users.api.ts
│   └── orders.api.ts
│
├── data/
│   ├── users.ts
│   └── products.ts
│
├── utils/
│   ├── dates.ts
│   └── generators.ts
│
├── config/
│   └── ...
│
└── playwright.config.ts
```

 A estrutura exata pode mudar, mas alguns conceitos são importantes.

 ### 2\. Não transforme Page Object em "depósito de código"

 Um erro comum é criar:

```
LoginPage
CheckoutPage
ProductPage
...
```

 e colocar absolutamente tudo dentro dessas classes.

 Prefira separar responsabilidades:

```
Page Object
    ↓
interação com a página

Component
    ↓
elementos reutilizáveis

Fixture
    ↓
preparação/contexto do teste

API
    ↓
criação e manipulação de dados

Test
    ↓
regra/comportamento que está sendo validado
```

 Por exemplo:

```
test('cliente consegue finalizar uma compra', async ({
  loginPage,
  checkoutPage,
}) => {
  await loginPage.login(user);

  await checkoutPage.addProduct(product);
  await checkoutPage.finishPurchase();

  await expect(checkoutPage.successMessage)
    .toBeVisible();
});
```

 O teste deve contar **uma história**, não explicar como clicar em cada elemento.

---

 ## 3\. Pense em dados de teste desde o começo

 Esse ponto é muito mais importante do que parece.

 Evite:

```
await page.fill('#name', 'João');
await page.fill('#email', 'joao@gmail.com');
```

 espalhado por centenas de testes.

 Crie uma estratégia de dados:

```
const user = {
  name: 'Test User',
  email: `user-${Date.now()}@test.com`,
};
```

 Melhor ainda, quando o sistema permitir:

```
Teste
  ↓
Fixture
  ↓
API
  ↓
Criação do usuário
  ↓
Execução do cenário
  ↓
Limpeza
```

 Isso reduz enormemente a dependência de telas para preparar o estado do teste.

---

 # 4\. API + UI deve ser uma decisão arquitetural

 Eu recomendaria não usar Playwright exclusivamente como ferramenta de UI.

 Imagine um teste de pedido:

```
❌ Abordagem lenta

Login pela UI
↓
Navegar até produtos
↓
Criar cliente
↓
Adicionar produto
↓
Criar pedido
↓
Testar pagamento
```

 Em muitos casos, você pode fazer:

```
API → criar cliente
API → criar produto
API → criar pedido

UI → testar somente o comportamento que interessa
```

 Isso deixa o teste muito mais rápido e menos frágil.

 O Playwright também possui suporte para testes via API, então você pode usar a mesma stack para **preparação de dados + validação da interface**.

---

 # 5\. Autenticação merece atenção especial

 Não faça login pela UI em todos os testes.

 Se você tiver:

```
500 testes
×
login de 5 segundos
```

 você criou uma enorme fonte de lentidão.

 Use `storageState`/fixtures para compartilhar o estado autenticado quando isso fizer sentido.

 Algo conceitualmente assim:

```
setup
  ↓
login
  ↓
gera storage state
  ↓
testes utilizam sessão
```

 E mantenha testes específicos para validar o próprio fluxo de login.

---

 # 6\. Seletores precisam fazer parte da arquitetura

 Defina uma regra desde o primeiro dia.

 Prioridade típica:

```
getByRole()
getByLabel()
getByText()
getByTestId()
CSS/XPath extremamente específico
```

 Por exemplo:

```
page.getByRole('button', { name: 'Comprar' })
```

 é muito melhor que:

```
page.locator(
  '#app > div:nth-child(2) > div > button'
)
```

 Se a equipe controla o frontend, eu também criaria uma convenção para `data-testid` quando o elemento não tiver um identificador semântico adequado.

---

 # 7\. Evite `waitForTimeout`

 Um dos primeiros "code smells" que eu colocaria no radar:

```
await page.waitForTimeout(3000);
```

 Normalmente significa que o teste está esperando **tempo**, em vez de esperar **estado**.

 Prefira:

```
await expect(
  page.getByRole('heading', { name: 'Pedido realizado' })
).toBeVisible();
```

 ou:

```
await page.waitForResponse(...)
```

 ou outras esperas baseadas no comportamento real da aplicação.

---

 # 8\. Defina as camadas de teste

 Não coloque tudo em E2E.

 Uma arquitetura saudável normalmente fica mais parecida com:

```
                 ┌─────────────────┐
                 │      E2E        │
                 │ poucos, críticos│
                 └────────┬────────┘
                          │
                 ┌────────▼────────┐
                 │  Integration    │
                 │     / API       │
                 └────────┬────────┘
                          │
                 ┌────────▼────────┐
                 │      Unit       │
                 │ muitos, rápidos │
                 └─────────────────┘
```

 Playwright é excelente para E2E e também pode ser usado para testes de API e outros cenários, mas **não significa que todo teste deveria ser Playwright**.

---

 # 9\. CI/CD deve entrar no desenho desde o início

 Não espere ter 300 testes para pensar nisso.

 Desde cedo defina:

```
Pull Request
    ↓
Lint / Typecheck
    ↓
Testes rápidos
    ↓
Playwright smoke
    ↓
Merge
    ↓
Pipeline completo
```

 E posteriormente:

```
PR
 ├── smoke
 │
 └── testes críticos

Nightly
 └── suíte completa

Release
 └── regressão completa
```

 Também pense em:

 - paralelização;
- retries controlados;
- traces;
- screenshots;
- vídeos quando necessário;
- artifacts;
- relatórios;
- isolamento entre testes.

---

 # 10\. Flakiness precisa ser uma métrica

 Esse é um dos pontos mais importantes para o longo prazo.

 Não basta perguntar:

 > "Quantos testes temos?"

 Pergunte:

 > "Quantos testes falham de verdade?"

 Crie indicadores como:

```
Total de testes:       850
Passaram:              823
Falharam:               27
Flaky conhecidos:        9
Tempo médio:          18 min
```

 E acompanhe:

```
Flaky rate
Failure rate
Execution time
Retry rate
Defects encontrados
```

 Um teste que precisa de:

```
retries: 3
```

 constantemente não está "resolvido". Ele está **escondendo um problema**.

---

 # 11\. Pense em observabilidade do teste

 Quando um teste falha no CI às 3h da manhã, você quer conseguir responder:

 > "Por que falhou?"

 e não:

 > "Funciona na minha máquina."

 Eu habilitaria principalmente:

```
use: {
  trace: 'retain-on-failure',
  screenshot: 'only-on-failure',
  video: 'retain-on-failure',
}
```

 E faria o pipeline guardar esses artifacts.

 O **trace viewer** do Playwright é especialmente útil para investigar falhas.

---

 # 12\. No longo prazo: crie uma estratégia de manutenção

 Quando a suíte crescer, a arquitetura começa a mudar.

 Você provavelmente terá:

```
100 testes
   ↓
300 testes
   ↓
800 testes
   ↓
2.000+ testes
```

 O problema deixa de ser:

 > "Como automatizar?"

 e passa a ser:

 > "Como manter isso sustentável?"

 Aí entram conceitos como:

 ### Test ownership

 Cada domínio possui responsáveis:

```
Checkout
 └── Squad Payments

Cadastro
 └── Squad Identity

Produto
 └── Squad Catalog
```

 ### Tags

 Por exemplo:

```
test.describe('@smoke', () => {});
test.describe('@regression', () => {});
test.describe('@critical', () => {});
```

 Então o pipeline pode executar:

```
npx playwright test --grep @smoke
```

 ou:

```
npx playwright test --grep @critical
```

---

 # 13\. Tenha ambientes previsíveis

 Outro problema que aparece depois:

```
Local
Dev
QA
Staging
Production
```

 Os testes precisam saber claramente:

```
qual ambiente?
qual API?
qual usuário?
qual configuração?
qual feature flag?
```

 Use configuração por ambiente, sem colocar secrets no código:

```
.env
.env.qa
.env.staging
```

 e secrets no CI/CD.

---

 # 14\. Pense em paralelização desde cedo

 Playwright permite executar testes em paralelo.

 Mas isso só funciona bem se os testes forem independentes.

 Um teste não deveria depender de:

```
Teste A
  ↓
cria usuário "joao"
  ↓
Teste B
  ↓
usa usuário "joao"
```

 Prefira:

```
Teste A → seus próprios dados
Teste B → seus próprios dados
Teste C → seus próprios dados
```

 Isso permite:

```
Worker 1 → A
Worker 2 → B
Worker 3 → C
Worker 4 → D
```

 e reduz drasticamente o tempo da suíte.

---

 # 15\. Eu evitaria este tipo de arquitetura

```
tests/
  ├── login.spec.ts
  ├── checkout.spec.ts
  ├── helpers.ts
  ├── utils.ts
  ├── common.ts
  ├── common2.ts
  ├── testHelper.ts
  └── testHelperFinal.ts
```

 É muito fácil chegar nisso.

 Também evitaria:

```
Test
 ↓
Page Object
 ↓
Helper
 ↓
HelperHelper
 ↓
Utils
 ↓
API
```

 com abstrações demais.

 **Abstração deve existir porque existe um comportamento reutilizável, não simplesmente porque podemos criar uma classe.**

---

 # 16\. Uma arquitetura que eu consideraria um bom ponto de partida

```
                    ┌───────────────────────┐
                    │       CI / CD         │
                    └───────────┬───────────┘
                                │
                    ┌───────────▼───────────┐
                    │   Playwright Runner   │
                    └───────────┬───────────┘
                                │
              ┌─────────────────┼─────────────────┐
              │                 │                 │
        ┌─────▼─────┐     ┌─────▼─────┐    ┌─────▼─────┐
        │   Tests   │     │ Fixtures  │    │ Test Data │
        └─────┬─────┘     └─────┬─────┘    └─────┬─────┘
              │                 │                 │
        ┌─────▼─────────────────▼─────────────────▼─────┐
        │                  Domain Layer                  │
        │                                                │
        │ Page Objects │ Components │ API Clients        │
        └─────────────────────────┬──────────────────────┘
                                  │
                    ┌─────────────▼─────────────┐
                    │       Application         │
                    │       Web / APIs          │
                    └───────────────────────────┘
```

 E ao redor disso:

```
Observabilidade
    ├── Trace
    ├── Screenshot
    ├── Video
    └── Reports

Governança
    ├── Tags
    ├── Ownership
    ├── Flaky tests
    └── Quality gates
```

 ## 17\. O que eu decidiria em uma reunião de arquitetura

 Se você estiver começando esse projeto dentro de uma empresa, eu colocaria **estas decisões na mesa imediatamente**:

 | Tema | Decisão inicial |
| --- | --- |
| Linguagem | TypeScript |
| Runner | Playwright Test |
| Arquitetura | Tests + Fixtures + Page Objects/Components + API |
| Seletores | Role/Label/TestId |
| Dados | API/factories sempre que possível |
| Auth | `storageState` \+ fixtures |
| Execução | Paralela e testes independentes |
| Evidências | Trace + screenshot em falha |
| CI | Smoke no PR + regressão em pipeline dedicado |
| Ambientes | Configuração externa |
| Secrets | Secret manager/CI |
| Tags | smoke / regression / critical |
| Flaky | Métrica e processo de tratamento |
| Ownership | Por domínio/squad |

 **A principal ideia é:** não comece pensando "como criar Page Objects". Comece pensando **"como essa suíte vai continuar rápida, confiável e fácil de diagnosticar quando tiver 1.000 testes?"**.

 Se você me disser **qual é o tipo da aplicação (React/Angular/Vue, microfrontend etc.), quantidade de squads e aproximadamente quantos fluxos precisam ser automatizados**, posso montar uma **arquitetura de projeto Playwright completa**, incluindo estrutura de pastas, `playwright.config.ts`, fixtures, autenticação, API clients, estratégia de dados, CI/CD e uma estratégia de execução para PR/nightly/release.