# Nova Tech — Arquitetura AWS Serverless para E-commerce

**Estudo de arquitetura em nuvem | Amazon Web Services (AWS)**

Projeto individual de estudo e portfólio voltado ao planejamento de uma arquitetura AWS para um e-commerce fictício, priorizando distribuição de conteúdo, segurança, escalabilidade e eficiência financeira.

> **Escopo:** arquitetura conceitual, documentada e estimada na AWS Pricing Calculator. Os recursos não foram provisionados na AWS.

## Sobre o projeto

A Nova Tech surgiu como uma evolução individual do projeto acadêmico [Sechs Network — Escola da Nuvem](https://github.com/Josiane-Goncalves/Escola_da_Nuvem).

A proposta foi revisitar a arquitetura original, avaliar alternativas de serviços, reduzir complexidade e elaborar uma nova estimativa de custos, aplicando os conhecimentos adquiridos em AWS.

O desenvolvimento também faz parte do aprofundamento dos meus estudos durante a preparação para a certificação **AWS Certified Solutions Architect – Associate**.

Mais do que selecionar serviços, o objetivo foi compreender as decisões arquiteturais, suas justificativas, seus custos e suas limitações.

## Objetivos e requisitos

A arquitetura foi planejada considerando os seguintes requisitos:

- Entrega eficiente de conteúdo estático para usuários no Brasil e no exterior.
- Back-end escalável para operações dinâmicas do e-commerce.
- Autenticação de usuários e controle de acesso às operações protegidas.
- Armazenamento gerenciado de produtos, clientes e pedidos.
- Proteção contra ameaças comuns a aplicações web.
- Observabilidade e acompanhamento de custos.
- Preferência por serviços gerenciados e componentes serverless.
- Orçamento máximo de **US$ 500 mensais**, com meta operacional de até US$ 400.

### Premissas de dimensionamento

| Indicador | Premissa |
|---|---|
| Visitantes únicos por mês | 30.000 |
| Visitas médias por visitante | 5 |
| Visitas mensais estimadas | 150.000 |
| Conteúdo estático por visita | 3 MB |
| Transferência estática estimada | 450 GB/mês |
| Requisições estáticas | 3 milhões/mês |
| Requisições à API | 1,5 milhão/mês |
| Público principal | Brasil e América do Sul |
| Região principal | South America (São Paulo) — `sa-east-1` |

Esses números representam hipóteses utilizadas no planejamento, não medições de tráfego real.

## Arquitetura proposta

![Diagrama da arquitetura AWS Nova Tech](diagrams/architecture.svg)

*Diagrama elaborado no draw.io, representando serviços globais, componentes regionais e principais fluxos da aplicação.*

O arquivo editável está disponível em [`diagrams/architecture.drawio`](diagrams/architecture.drawio).

### Principais componentes

| Serviço AWS | Responsabilidade |
|---|---|
| Amazon Route 53 | Resolução DNS dos domínios |
| Amazon CloudFront | CDN para entrega de conteúdo estático |
| Amazon S3 | Armazenamento privado do frontend |
| AWS WAF | Proteção do CloudFront e da API |
| Amazon Cognito | Autenticação de usuários |
| Amazon API Gateway | Exposição da API REST Regional |
| AWS Lambda | Execução da lógica de negócio |
| Amazon DynamoDB | Persistência de produtos, clientes e pedidos |
| Amazon CloudWatch | Logs, métricas e alarmes operacionais |

### Fluxo de acesso ao site

1. O navegador resolve o domínio por meio do serviço DNS.
2. O cliente estabelece uma conexão HTTPS com o CloudFront.
3. Quando o conteúdo está disponível no cache, o CloudFront pode entregá-lo sem consultar a origem.
4. Em caso de cache MISS, o CloudFront busca o recurso no bucket S3 privado, utilizando Origin Access Control (OAC).
5. O AWS WAF associado à distribuição aplica as regras de proteção configuradas.

### Fluxo das operações dinâmicas

1. O navegador realiza requisições HTTPS diretamente ao API Gateway Regional.
2. O AWS WAF Regional aplica as regras de proteção da API.
3. Nas rotas protegidas, o API Gateway utiliza o autorizador do Cognito para validar o token.
4. O API Gateway invoca a função AWS Lambda.
5. A Lambda executa as regras de negócio e acessa o DynamoDB conforme as permissões IAM.
6. A aplicação devolve a resposta ao cliente.

A validação do token autentica o usuário, mas não substitui as verificações de autorização necessárias na lógica de negócio, como a confirmação de que determinado pedido pertence ao cliente autenticado.

## Decisões arquiteturais

### Região AWS

A região `sa-east-1` (São Paulo) foi escolhida para hospedar os componentes regionais da aplicação, considerando a predominância de usuários brasileiros e a proximidade geográfica.

Essa escolha envolve um equilíbrio entre latência, localização dos dados, disponibilidade de serviços e custos.

### Arquitetura serverless

A combinação de API Gateway, Lambda e DynamoDB foi escolhida para evitar a administração direta de servidores e permitir que o consumo de recursos acompanhe a demanda, respeitando os limites e as configurações de cada serviço.

### Distribuição de conteúdo

O CloudFront foi utilizado para reduzir a necessidade de consultas recorrentes ao S3 e melhorar a entrega de conteúdo estático.

O bucket S3 permanece privado, com acesso à origem controlado por OAC.

### Segurança

A proposta contempla:

- HTTPS com certificados gerenciados pelo AWS Certificate Manager (ACM).
- AWS WAF com proteções distintas para CloudFront e API Gateway.
- Cognito User Pools para autenticação.
- IAM com princípio do menor privilégio.
- Verificações de autorização na lógica de negócio.
- Restrições de acesso ao bucket S3.

### Observabilidade e governança

O Amazon CloudWatch foi considerado para centralizar logs operacionais, métricas e alarmes.

O planejamento também contempla AWS Budgets para acompanhamento financeiro e CloudTrail para consulta de eventos de auditoria.

## Estimativa de custos

A estimativa foi elaborada com a **AWS Pricing Calculator**, utilizando as premissas de dimensionamento definidas para o projeto.

| Indicador | Valor |
|---|---:|
| Custo inicial estimado | US$ 0,00 |
| Estimativa mensal registrada | US$ 85,49 |
| Projeção para 12 meses | US$ 1.025,88 |
| Orçamento máximo mensal | US$ 500,00 |

![Resumo da estimativa AWS Pricing Calculator](assets/cost-estimation.png)

**Importante:** os valores apresentados correspondem à simulação realizada, não a custos medidos em produção.

A estimativa ainda apresenta ressalvas relacionadas ao tráfego internacional do CloudFront, à aplicação das franquias gratuitas e a possíveis custos adicionais de operação.

Por isso, o valor de US$ 85,49 não deve ser interpretado como o custo definitivo da solução.

## Status e limitações

**Concluído:**
- Levantamento das premissas e dos requisitos.
- Seleção e revisão dos principais serviços AWS.
- Definição da arquitetura conceitual.
- Criação do diagrama no draw.io.
- Simulação inicial de custos na AWS Pricing Calculator.

**Em desenvolvimento:**
- Documentação técnica detalhada.
- Registro das decisões arquiteturais (ADRs).
- Revisão das premissas financeiras.
- Animação dos fluxos da arquitetura.

**Fora do escopo atual:**
- Provisionamento de infraestrutura na AWS.
- Implementação do frontend e do backend.
- Testes reais de desempenho e disponibilidade.
- Validação operacional em ambiente de produção.

## Referências

- [Documentação oficial da AWS](https://docs.aws.amazon.com/)
- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [AWS Pricing Calculator](https://calculator.aws/)
- [Projeto Sechs Network — Escola da Nuvem](https://github.com/Josiane-Goncalves/Escola_da_Nuvem)

---

**Autoria da evolução Nova Tech:** Josiane Cristina Gonçalves

*Projeto de estudo e portfólio, desenvolvido para aprofundamento de conhecimentos em arquitetura de soluções AWS.*