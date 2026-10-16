# Nova Tech — Arquitetura AWS para E-commerce

**Estudo de arquitetura em nuvem | CDN · Serverless · Segurança · Otimização de custos**

A Nova Tech é um estudo individual de arquitetura AWS para uma plataforma fictícia de e-commerce, com foco em **distribuição de conteúdo por CDN**, segurança, escalabilidade e eficiência financeira.

O projeto surgiu como evolução do trabalho acadêmico [Sechs Network — Escola da Nuvem](https://github.com/Josiane-Goncalves/Escola_da_Nuvem), durante meu aprofundamento em AWS e preparação para a certificação AWS Certified Solutions Architect – Associate.

> **Escopo:** arquitetura conceitual documentada e estimada na AWS Pricing Calculator. A infraestrutura não foi provisionada na AWS.

## Arquitetura proposta

![Arquitetura AWS Nova Tech](diagrams/architecture.svg)

*Arquitetura elaborada no draw.io. [Acessar diagrama editável](diagrams/architecture.drawio).*

A proposta separa a entrega de conteúdo estático das operações dinâmicas do e-commerce.

O **Amazon CloudFront** distribui os arquivos hospedados em um bucket **Amazon S3 privado**, utilizando Origin Access Control (OAC).

As operações dinâmicas são atendidas por uma arquitetura serverless com Amazon API Gateway, AWS Lambda e Amazon DynamoDB.

A solução também contempla proteção com AWS WAF, autenticação com Amazon Cognito e observabilidade com Amazon CloudWatch.

## Principais decisões arquiteturais

| Decisão | Justificativa |
|---|---|
| CloudFront + S3 | Distribuição global de conteúdo estático com cache e origem privada |
| Região `sa-east-1` | Proximidade com o público principal, concentrado no Brasil |
| API Gateway + Lambda | Processamento dinâmico sem administração direta de servidores |
| DynamoDB On-Demand | Persistência gerenciada com cobrança baseada no consumo |
| AWS WAF | Proteção planejada para a CDN e para a API |

As justificativas, integrações e limitações estão detalhadas na [documentação da arquitetura](docs/architecture.md).

## Dimensionamento e custos

O estudo considera:

| Indicador | Premissa |
|---|---:|
| Visitas mensais | 150.000 |
| Transferência estática | 450 GB/mês |
| Requisições ao CloudFront | 3 milhões/mês |
| Público Brasil/América do Sul | 90% |
| Orçamento máximo | US$ 500/mês |
| Estimativa inicial registrada | US$ 85,49/mês |

A estimativa foi produzida na AWS Pricing Calculator. O valor é **preliminar**, pois ainda existem ressalvas relacionadas ao tráfego internacional, às franquias gratuitas e a possíveis custos complementares.

[Consultar a análise financeira e a captura da calculadora](docs/cost-estimation.md).

## Documentação técnica

| Documento | Conteúdo |
|---|---|
| [Requisitos e premissas](docs/requirements.md) | Cenário, volumes estimados, restrições e objetivos |
| [Arquitetura AWS](docs/architecture.md) | Serviços, fluxos, cache e decisões técnicas |
| [Segurança](docs/security.md) | WAF, Cognito, IAM, HTTPS e proteção do S3 |
| [Estimativa de custos](docs/cost-estimation.md) | Dimensionamento, custos e limitações da simulação |

## Status do projeto

**Concluído:** definição dos requisitos, revisão da arquitetura, diagrama no draw.io e documentação dos controles de segurança e dos custos estimados.

**Próxima melhoria:** animação explicativa dos fluxos de entrega de conteúdo, com destaque para cache HIT e cache MISS.

**Fora do escopo:** implementação do e-commerce, provisionamento de recursos AWS e testes reais de desempenho ou disponibilidade.

## Referências

- [Documentação oficial da AWS](https://docs.aws.amazon.com/)
- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)
- [AWS Pricing Calculator](https://calculator.aws/)
- [Sechs Network — Escola da Nuvem](https://github.com/Josiane-Goncalves/Escola_da_Nuvem)

---

**Autoria:** Josiane Cristina Gonçalves

*Estudo individual de arquitetura AWS para aprendizado e portfólio profissional.*