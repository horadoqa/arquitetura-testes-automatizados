# Arquitetura de Testes Automatizados

Como organizar uma arquitetura de testes automatizados de forma sustentável, escalável e fácil de manter.

Uma **arquitetura de testes automatizados** é a forma como você organiza, estrutura e conecta os testes automatizados de um sistema para que sejam fáceis de criar, executar, manter e evoluir.

Em outras palavras, não se trata apenas de escrever testes. É necessário definir **como os testes serão construídos, organizados, executados e integrados ao processo de desenvolvimento**.

Uma boa arquitetura de testes define, entre outras coisas:

- Camadas e tipos de testes.
- Organização dos arquivos e diretórios.
- Responsabilidade de cada componente.
- Estratégia de dados de teste.
- Fixtures, mocks e stubs.
- Clientes e abstrações para APIs.
- Utilitários compartilhados.
- Relatórios e evidências.
- Execução local e em CI/CD.
- Estratégias para evitar testes instáveis (_flaky tests_).

---

 ## Exemplo simples

Imagine uma aplicação de e-commerce.

Você poderia ter diferentes tipos de testes:

- **Testes unitários** → verificam funções, métodos e classes isoladamente.
- **Testes de integração** → verificam a comunicação entre componentes e dependências, como banco de dados, filas e serviços.
- **Testes de API** → validam endpoints, respostas, regras de negócio e contratos das APIs.
- **Testes E2E/UI** → simulam o comportamento de um usuário utilizando a aplicação.
- **Fixtures** → preparam dados e contexto necessários para a execução dos testes.
- **Mocks e stubs** → simulam dependências externas ou comportamentos controlados.
- **Clients** → encapsulam chamadas para APIs ou outros serviços.
- **Relatórios** → apresentam resultados, falhas, evidências e métricas.
- **CI/CD** → executa automaticamente os testes durante o processo de entrega do software.

---

## Pirâmide de testes

Uma forma comum de visualizar a estratégia de testes é através da **pirâmide de testes**.

A ideia geral é possuir uma quantidade maior de testes rápidos e baratos nas camadas inferiores e uma quantidade menor de testes mais complexos e caros nas camadas superiores.

Mermaid flowchart: Testes unitários, Testes de integração, Testes de API, Testes E2E / UI

### Testes unitários

São a base da estratégia.

Validam pequenas unidades de código de maneira isolada, como:

- Funções.
- Métodos.
- Classes.
- Regras de negócio.

Normalmente são rápidos de executar e fáceis de diagnosticar quando falham.

Exemplo:

```
calcular_total(produtos)
        ↓
   Teste unitário
        ↓
Resultado esperado
```

---

### Testes de integração

Validam a comunicação entre diferentes componentes.

Podem envolver:

- Banco de dados.
- APIs.
- Serviços.
- Filas.
- Sistemas externos.
- Componentes internos da aplicação.

 O objetivo é verificar se as partes do sistema funcionam corretamente quando utilizadas em conjunto.

---

### Testes de API

Validam uma aplicação através de suas interfaces de API.

Podem verificar:

- Status HTTP.
- Headers.
- Payloads.
- Schemas.
- Contratos.
- Autenticação.
- Autorização.
- Regras de negócio.
- Mensagens de erro.
- Tempo de resposta.

É importante observar que **"teste de API" e "teste de integração" não são necessariamente categorias exclusivas**.

Um teste de API pode ser, por exemplo, um teste de integração quando valida a comunicação entre a aplicação e um banco de dados ou serviço externo.

Nesse sentido:

- **API** descreve principalmente a interface utilizada pelo teste.
- **Integração** descreve principalmente a interação entre componentes ou dependências.

---

### Testes E2E / UI

São testes que validam o sistema de maneira mais próxima da experiência real do usuário.

Podem simular fluxos como:

```
Usuário
   ↓
Login
   ↓
Busca de produto
   ↓
Carrinho
   ↓
Checkout
   ↓
Pagamento
   ↓
Pedido criado
```

Esses testes são importantes para validar fluxos críticos, mas normalmente são mais lentos e mais suscetíveis a problemas de ambiente.

Por isso, não é recomendado utilizar testes E2E para validar tudo o que poderia ser validado em uma camada inferior.

---

# Arquitetura versus execução no CI/CD

É importante diferenciar a **arquitetura dos testes** da **ordem de execução no pipeline**.

A pirâmide representa uma estratégia de distribuição dos testes. Ela não determina obrigatoriamente como o pipeline deve executar cada etapa.

Por exemplo, um pipeline pode executar os testes nesta ordem:

Mermaid flowchart: Pipeline CI/CD, Testes unitários, Testes de integração, Testes de API, Testes E2E / UI

Essa estratégia permite falhar rapidamente caso exista um problema básico no código antes de executar testes mais lentos.

Em um projeto real, também é possível executar algumas etapas em paralelo.

Por exemplo:

Mermaid flowchart: Pipeline CI/CD, Testes unitários, Testes de integração, Testes de API, Testes E2E / UI, Relatórios

A estratégia ideal depende do projeto, do tempo disponível no pipeline e dos recursos de infraestrutura.

---

# O que uma boa arquitetura busca

Uma boa arquitetura de testes deve buscar principalmente os seguintes objetivos.

## 1\. Manutenibilidade

Se a aplicação mudar, não devemos precisar alterar centenas de testes manualmente.

Para isso, é importante:

- Evitar duplicação.
- Centralizar comportamentos comuns.
- Utilizar abstrações adequadas.
- Separar regras de negócio da implementação dos testes.
- Manter responsabilidades bem definidas.

---

## 2\. Reutilização

Componentes comuns podem ser reutilizados por diversos testes.

Por exemplo:

```
Autenticação
      ↓
 ┌────┴────┐
 ↓         ↓
API       E2E
```

Uma funcionalidade de autenticação pode ser encapsulada para evitar que cada teste implemente novamente o mesmo fluxo.

---

## 3\. Independência

Um teste não deveria depender desnecessariamente de outro teste para funcionar.

Evite situações como:

```
Teste A → cria usuário
             ↓
Teste B → utiliza usuário criado pelo Teste A
```

O ideal é que cada teste prepare o próprio estado necessário ou utilize uma fixture responsável por isso.

Assim:

```
Teste A → prepara seus dados → executa → valida

Teste B → prepara seus dados → executa → valida
```

Isso facilita a execução individual, paralela e repetida dos testes.

---

## 4\. Velocidade

Testes rápidos podem ser executados com maior frequência.

Por isso, normalmente:

- Testes unitários devem ser rápidos.
- Testes de integração devem ter escopo controlado.
- Testes de API devem validar os principais cenários.
- Testes E2E devem concentrar-se nos fluxos críticos.

 Quanto mais alto o nível do teste, maior tende a ser o custo de execução.

---

## 5\. Confiabilidade

Uma suíte de testes precisa produzir resultados confiáveis.

Um problema comum são os **flaky tests**.

Um _flaky test_ é um teste que pode passar ou falhar de maneira inconsistente sem que exista uma alteração relevante no código.

Exemplo:

```
Execução 1 → PASSOU
Execução 2 → PASSOU
Execução 3 → FALHOU
Execução 4 → PASSOU
```

Possíveis causas:

- Condições de corrida.
- Dependência de tempo.
- Dados compartilhados.
- Serviços externos instáveis.
- Problemas de sincronização.
- Dependência da ordem de execução.
- Ambiente inconsistente.

Uma boa arquitetura deve reduzir essas fontes de instabilidade.

---

## 6\. Escalabilidade

A estrutura deve continuar funcionando conforme a quantidade de testes cresce.

Uma estrutura que funciona com 100 testes pode se tornar difícil de manter com 10.000 testes.

Por isso, desde o início é importante estabelecer padrões para:

- Nomenclatura.
- Organização de diretórios.
- Fixtures.
- Clientes.
- Dados.
- Configurações.
- Relatórios.
- Execução.

---

# Estrutura de um projeto de testes

 ma possível estrutura para um projeto de automação seria:

```
tests/
├── unit/
│   ├── login/
│   ├── usuarios/
│   └── pedidos/
│
├── integration/
│   ├── login/
│   ├── usuarios/
│   └── pedidos/
│
├── api/
│   ├── login/
│   ├── usuarios/
│   └── pedidos/
│
├── e2e/
│   ├── login/
│   ├── usuarios/
│   └── pedidos/
│
├── fixtures/
│   └── dados_usuario.json
│
├── clients/
│   ├── auth_client.py
│   ├── user_client.py
│   └── order_client.py
│
├── utils/
│   ├── gerador_dados.py
│   └── configuracao.py
│
└── reports/
```

A estrutura pode variar de acordo com a linguagem, framework e características do projeto.

O importante é que cada diretório tenha uma responsabilidade clara.

---

# Responsabilidade de cada camada

## `unit/`

Contém os testes unitários.

Exemplo:

```
unit/
└── usuarios/
    └── test_usuario.py
```

Esses testes devem evitar dependências externas sempre que possível.

---

## `integration/`

Contém testes que verificam a integração entre componentes.

Exemplo:

```
integration/
└── usuarios/
    └── test_usuario_repository.py
```

Podem utilizar recursos como banco de dados, filas ou outros serviços controlados.

---

## `api/`

Contém testes que acessam diretamente as APIs.

Exemplo:

```
api/
└── usuarios/
    ├── test_criar_usuario.py
    ├── test_consultar_usuario.py
    └── test_excluir_usuario.py
```

---

## `e2e/`

Contém os testes de ponta a ponta.

Exemplo:

```
e2e/
└── pedidos/
    └── test_criar_pedido.py
```

Normalmente são utilizados para validar os principais fluxos de negócio.

---

# Fixtures

Fixtures são recursos utilizados para preparar o contexto necessário para os testes.

Podem fornecer:

- Dados.
- Usuários.
- Tokens.
- Conexões.
- Configurações.
- Objetos.
- Estado inicial.

Exemplo:

```
fixtures/
├── dados_usuario.json
├── dados_pedido.json
└── configuracao.json
```

Uma fixture pode, por exemplo, criar um usuário antes do teste e disponibilizar seus dados para utilização.

---

# Mocks e Stubs

Mocks e stubs permitem controlar dependências durante os testes.

Por exemplo, imagine que a aplicação consulte um serviço externo de pagamentos.

Em vez de depender do serviço real durante um teste unitário, podemos simular sua resposta.

```
Teste
  │
  ▼
Aplicação
  │
  ▼
Mock do serviço de pagamento
  │
  ▼
Resposta controlada
```

Isso torna o teste:

- Mais rápido.
- Mais previsível.
- Independente de serviços externos.

 Mocks não devem ser utilizados indiscriminadamente. Em alguns cenários, validar a integração real é justamente o objetivo do teste.

---

# API Clients

Uma boa prática em projetos de automação de API é encapsular as chamadas HTTP em clientes.

Por exemplo:

```
clients/
├── auth_client.py
├── user_client.py
└── order_client.py
```

 Em vez de espalhar detalhes HTTP pelos testes:

```
Teste
 ├── URL
 ├── Headers
 ├── Token
 ├── Payload
 └── Request
```

podemos utilizar uma abstração:

```
Teste
  │
  ▼
UserClient
  │
  ▼
API
```

Assim, o teste fica mais focado no comportamento que está sendo validado.

---

# Utilitários

A pasta `utils/` pode conter funcionalidades compartilhadas que não pertencem diretamente a uma camada específica de teste.

Exemplo:

```
utils/
├── gerador_dados.py
├── configuracao.py
└── datas.py
```

Possíveis responsabilidades:

- Geração de dados.
- Manipulação de datas.
- Configurações.
- Conversões.
- Funções auxiliares.

É importante evitar transformar `utils/` em uma pasta que contenha "qualquer coisa".

Se uma funcionalidade possuir uma responsabilidade específica, pode ser melhor criar uma estrutura própria para ela.

---

# Dados de teste

Os dados utilizados pelos testes devem ser tratados como parte importante da arquitetura.

Uma estratégia ruim é fazer todos os testes dependerem dos mesmos dados compartilhados:

```
Banco
  │
  ├── usuário 1
  ├── usuário 2
  └── usuário 3

Todos os testes dependem desses mesmos registros.
```

Isso aumenta o risco de interferência entre testes.

 ma abordagem mais robusta é criar ou preparar os dados necessários para cada cenário.

```
Teste A → seus dados
Teste B → seus dados
Teste C → seus dados
```

Sempre que possível, os testes devem ser independentes e reproduzíveis.

---

# Relatórios

Os relatórios ajudam a entender o resultado da execução da suíte.

Podem apresentar:

- Quantidade de testes executados.
- Testes aprovados.
- Testes falhos.
- Testes ignorados.
- Tempo de execução.
- Evidências.
- Logs.
- Screenshots.
- Histórico de execução.

Uma estrutura simples poderia ser:

```
reports/
├── execution/
├── screenshots/
└── logs/
```

Dependendo do framework, esses arquivos podem ser gerados automaticamente.

---

# Integração com CI/CD

A automação de testes ganha ainda mais valor quando integrada ao processo de CI/CD.

Um pipeline pode seguir uma estratégia como:

Mermaid flowchart: Commit / Pull Request, Build, Testes unitários, Testes de integração, Testes de API, Testes E2E / UI, Relatórios, Deploy

Uma estratégia mais avançada pode executar algumas etapas em paralelo:

Mermaid flowchart: Commit / Pull Request, Build, Testes unitários, Testes de integração, Testes de API, Testes E2E / UI, Relatórios, Deploy

A configuração real dependerá da ferramenta de CI/CD utilizada.

---

# Princípios importantes

Uma arquitetura de testes sustentável pode seguir alguns princípios simples:

### Teste uma responsabilidade por vez

Cada teste deve ter um objetivo claro.

### Evite duplicação

Código repetido aumenta o custo de manutenção.

### Prefira testes independentes

Um teste não deve depender do resultado de outro.

### Controle os dados

Dados compartilhados e mutáveis podem gerar efeitos colaterais.

### Evite abstrações excessivas

Nem todo código precisa ser transformado em uma camada ou framework.

### Mantenha os testes legíveis

Um teste deve ser fácil de entender por alguém que não o escreveu.

### Automatize o que faz sentido

Nem tudo precisa ser automatizado. Priorize cenários repetitivos, críticos e de alto valor.

### Monitore testes instáveis

Flaky tests devem ser identificados e corrigidos, e não simplesmente ignorados.

---

# Exemplo de fluxo completo

Uma arquitetura de automação pode ser visualizada da seguinte maneira:

Mermaid flowchart: Aplicação, Camada de Testes, Testes Unitários, Testes de Integração, Testes de API, Testes E2E / UI, Fixtures / Mocks, API Clients, Page Objects / Componentes, Dados de Teste, Relatórios, CI/CD

Essa representação demonstra que os testes não existem isoladamente. Eles fazem parte de um ecossistema que envolve código de teste, dados, dependências, infraestrutura, relatórios e execução automatizada.

---

# Conclusão

Arquitetura de testes automatizados é o **projeto estrutural do ecossistema de testes**.

Ela define como os testes serão:

- Organizados.
- Construídos.
- Executados.
- Mantidos.
- Integrados.
- Monitorados.
- Evoluídos.

Uma arquitetura bem estruturada não significa necessariamente ter muitas pastas, classes ou abstrações.

O objetivo é criar uma estrutura que permita que a suíte de testes continue sendo **rápida, confiável, legível e sustentável conforme o sistema cresce**.

Em resumo:

```
Boa arquitetura
      │
      ├── Testes bem organizados
      ├── Responsabilidades claras
      ├── Dados controlados
      ├── Reutilização
      ├── Independência
      ├── Execução confiável
      ├── Relatórios
      └── Integração com CI/CD
```

 A arquitetura ideal dependerá da tecnologia, do tamanho do sistema, do nível de criticidade da aplicação e dos objetivos do projeto. O mais importante é estabelecer uma estrutura consistente que facilite a evolução da automação ao longo do tempo.
