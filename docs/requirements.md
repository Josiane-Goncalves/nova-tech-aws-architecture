# Nova Tech — Requisitos e Premissas da Arquitetura

## 1. Contexto

A Nova Tech é uma plataforma fictícia de e-commerce utilizada como estudo de arquitetura de soluções na AWS.

O projeto tem como objetivo planejar uma infraestrutura para distribuição de conteúdo estático e processamento de operações dinâmicas, considerando segurança, escalabilidade, disponibilidade e custos.

A proposta é conceitual e não contempla a implementação da aplicação ou o provisionamento de recursos na AWS.

## 2. Objetivo da arquitetura

Projetar uma solução baseada em serviços gerenciados da AWS, capaz de atender ao volume de acessos estimado, proteger informações dos clientes e permitir crescimento de demanda sem exigir a administração direta de servidores.

A arquitetura também deve respeitar o limite financeiro estabelecido para o cenário.

## 3. Premissas de negócio e dimensionamento

| Indicador | Valor considerado |
|---|---|
| Visitantes únicos mensais | 30.000 |
| Visitas médias por usuário | 5 |
| Visitas mensais estimadas | 150.000 |
| Transferência estática por visita | 3 MB |
| Transferência estática mensal | 450 GB |
| Requisições estáticas por visita | 20 |
| Requisições estáticas mensais | 3.000.000 |
| Requisições à API por visita | 10 |
| Requisições mensais à API | 1.500.000 |
| Distribuição geográfica | 90% Brasil/América do Sul e 10% exterior |
| Orçamento mensal máximo | US$ 500 |
| Meta de custo mensal | Até US$ 400 |
| Região principal | sa-east-1 — São Paulo |

Os valores representam hipóteses de planejamento e não métricas coletadas de uma aplicação real.

O comportamento de picos, a simultaneidade de usuários e a distribuição das requisições ao longo do dia não foram medidos.

## 4. Requisitos funcionais

**RF01 — Catálogo de produtos**

A aplicação deverá permitir a consulta de produtos e suas informações.

**RF02 — Consulta por categoria**

A aplicação deverá permitir a consulta de produtos por categoria.

**RF03 — Autenticação de clientes**

A aplicação deverá oferecer um mecanismo de autenticação para acesso a operações protegidas.

**RF04 — Carrinho de compras**

A aplicação deverá permitir que o cliente organize temporariamente os itens que pretende comprar.

Para o escopo proposto, o carrinho será mantido no navegador, sem necessidade de persistência dedicada no backend.

**RF05 — Registro de pedidos**

A aplicação deverá permitir que um cliente autenticado registre um pedido.

**RF06 — Consulta de pedidos**

A aplicação deverá permitir que o cliente autenticado consulte seus próprios pedidos.

**RF07 — Proteção de dados dos clientes**

A aplicação deverá impedir que um cliente consulte ou altere pedidos pertencentes a outros usuários.

**RF08 — Entrega do frontend**

A aplicação deverá disponibilizar os arquivos estáticos da interface web por HTTPS.

**Observação:** esses requisitos definem funcionalidades previstas para o cenário fictício. Não representam funcionalidades implementadas.

## 5. Requisitos não funcionais

**RNF01 — Desempenho**

A solução deverá priorizar a entrega de conteúdo estático com baixa latência, especialmente para usuários no Brasil e na América do Sul.

Não foram definidos ou medidos limites quantitativos de tempo de resposta.

**RNF02 — Escalabilidade**

A arquitetura deverá permitir adaptação ao crescimento da demanda, respeitando limites, cotas e configurações dos serviços utilizados.

**RNF03 — Segurança**

A solução deverá utilizar HTTPS, controle de acesso, proteção contra ameaças comuns e permissões baseadas no princípio do menor privilégio.

**RNF04 — Disponibilidade**

A arquitetura deverá priorizar serviços gerenciados e recursos de alta disponibilidade.

Uma eventual meta numérica de disponibilidade deverá ser validada considerando o comportamento da aplicação completa e as dependências entre serviços.

**RNF05 — Observabilidade**

A solução deverá permitir acompanhamento de métricas, logs operacionais e alarmes relevantes.

**RNF06 — Eficiência financeira**

O planejamento deverá considerar um orçamento máximo de US$ 500 mensais, com meta de até US$ 400.

A conformidade com esse orçamento dependerá da validação das estimativas e do consumo real.

**RNF07 — Manutenibilidade**

A arquitetura deverá evitar componentes desnecessários e favorecer responsabilidades bem definidas entre os serviços.

**RNF08 — Integridade dos pedidos**

O processamento de pedidos deverá considerar mecanismos de idempotência, consistência das operações e prevenção de registros duplicados.

## 6. Restrições

- Preferência por componentes serverless e serviços gerenciados.
- Região principal definida como AWS South America (São Paulo).
- Conteúdo estático hospedado em bucket S3 privado.
- Back-end baseado em API Gateway, Lambda e DynamoDB.
- Ausência de servidores EC2 ou banco relacional no escopo atual.
- Orçamento mensal máximo de US$ 500.
- Projeto limitado ao planejamento e à documentação arquitetural.

## 7. Fora do escopo

- Desenvolvimento e publicação de uma aplicação funcional.
- Provisionamento de infraestrutura na AWS.
- Integração real com gateways de pagamento.
- Operações de pagamento, estorno e faturamento.
- Testes reais de carga e desempenho.
- Comprovação de disponibilidade em produção.
- Operação contínua e suporte da aplicação.

## 8. Critérios de avaliação do estudo

A proposta arquitetural será avaliada considerando:

1. Correspondência entre requisitos e serviços selecionados.
2. Clareza dos fluxos de acesso, autenticação e persistência.
3. Justificativas para as principais decisões arquiteturais.
4. Identificação das responsabilidades de segurança.
5. Coerência do dimensionamento e das estimativas financeiras.
6. Registro transparente das limitações do estudo.

Esses critérios avaliam a qualidade do planejamento, não substituem testes de uma infraestrutura implantada.