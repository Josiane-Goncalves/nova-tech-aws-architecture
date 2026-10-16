# Nova Tech — Segurança da Arquitetura AWS

## 1. Objetivo

Este documento descreve os controles de segurança planejados para a arquitetura conceitual da Nova Tech.

A proposta considera proteção em múltiplas camadas, seguindo o modelo de responsabilidade compartilhada da AWS e o princípio do menor privilégio.

**Status:** controles projetados e documentados, ainda não implementados ou testados em uma conta AWS.

## 2. Modelo de responsabilidade compartilhada

A AWS é responsável pela segurança da infraestrutura que opera seus serviços.

A responsabilidade pela configuração segura dos recursos, proteção dos dados, gerenciamento de identidades e definição de permissões varia conforme o serviço utilizado e permanece, em parte, com o cliente.

Para a Nova Tech, isso inclui:

- Configurar corretamente as políticas de acesso ao S3.
- Definir permissões IAM restritas.
- Proteger os endpoints públicos.
- Configurar a autenticação e a autorização.
- Evitar exposição de informações sensíveis.
- Monitorar eventos e configurar alertas.

Utilizar serviços gerenciados reduz determinadas responsabilidades operacionais, mas não elimina a necessidade de configuração e manutenção de segurança.

## 3. Proteção do frontend — CloudFront e S3

O frontend estático será armazenado em um bucket Amazon S3 privado e distribuído pelo Amazon CloudFront.

### Controles planejados

- S3 Block Public Access habilitado.
- Bucket sem permissões públicas de leitura.
- Origin Access Control (OAC) associado à distribuição CloudFront.
- OAC configurado para assinar sempre as requisições (`SigningBehavior: always`).
- Política de bucket permitindo acesso de leitura exclusivamente à distribuição CloudFront autorizada.
- Comunicação HTTPS para os usuários e entre CloudFront e a origem S3.
- Configuração de cache adequada aos arquivos publicados.

A política do bucket deverá conceder ao principal de serviço `cloudfront.amazonaws.com` a permissão `s3:GetObject`, restringindo o acesso ao ARN da distribuição correspondente por meio da condição `AWS:SourceArn`.

O CloudFront utilizará a origem S3 convencional, e não o endpoint de hospedagem de site estático do S3, pois o OAC não é compatível com origens configuradas como S3 static website endpoint.

O objetivo é impedir que usuários acessem diretamente os objetos por um endpoint público do S3, mantendo a distribuição como ponto de entrada autorizado.

Essas configurações serão necessárias em uma eventual implantação e ainda não foram aplicadas a recursos reais.

## 4. Proteção de aplicações — AWS WAF

A arquitetura contempla duas Web ACLs independentes.

### Web ACL global — CloudFront

Associada à distribuição CloudFront, com regras planejadas para identificar e bloquear ou limitar requisições indesejadas.

### Web ACL Regional — API Gateway

Associada ao estágio da REST API na região `sa-east-1`.

### Regras previstas

Para cada Web ACL, a estimativa considera:

- Dois grupos de regras gerenciadas.
- Uma regra personalizada baseada em taxa de requisições.

Os grupos gerenciados específicos, os limites das regras de taxa e as ações de bloqueio deverão ser definidos e avaliados antes de uma implantação.

O WAF reduz a exposição a determinadas ameaças, mas não garante a eliminação de todos os ataques.

Também não substitui validação de entrada, autenticação ou autorização na aplicação.

## 5. Autenticação de clientes — Amazon Cognito

A Nova Tech utiliza conceitualmente Amazon Cognito User Pools para gerenciar a autenticação de clientes.

O fluxo proposto para a aplicação web é OAuth 2.0 Authorization Code com PKCE.

### Fluxo previsto

1. O cliente inicia o processo de autenticação.
2. O navegador interage com o serviço de autenticação do Cognito.
3. Após a autenticação, a aplicação recebe os tokens conforme o fluxo configurado.
4. O cliente utiliza um token apropriado ao acessar endpoints protegidos.
5. O API Gateway aplica o autorizador configurado ao método solicitado.

O projeto não contempla a utilização de Cognito Identity Pools nem a entrega direta de credenciais AWS aos clientes.

### Proteção dos tokens

Uma implementação futura deverá considerar:

- Transporte exclusivo por HTTPS.
- Validação apropriada de tokens.
- Proteção contra exposição por scripts ou armazenamento inseguro.
- Políticas de expiração e renovação de sessões.
- Configuração segura dos endereços de redirecionamento OAuth.

## 6. Autorização no API Gateway

A API proposta é do tipo Amazon API Gateway REST Regional.

Nas operações protegidas, será utilizado um autorizador do tipo `COGNITO_USER_POOLS`.

### Comportamento do autorizador

No API Gateway REST, o tipo de token validado depende da configuração dos métodos:

- **Sem OAuth Authorization Scopes:** o autorizador utiliza o ID token para validar a identidade do usuário.
- **Com OAuth Authorization Scopes:** o autorizador utiliza o access token para verificar os escopos exigidos pelo método.

Para o escopo inicial da Nova Tech, a proposta considera métodos protegidos sem OAuth Authorization Scopes, utilizando ID token para validação de identidade.

A adoção futura de access tokens e escopos específicos poderá ser avaliada conforme as necessidades de autorização por permissões.

### Proteção dos endpoints

As operações públicas de consulta ao catálogo não exigirão autenticação.

As operações de registro e consulta de pedidos exigirão autenticação por meio do Cognito User Pools Authorizer.

Cada método protegido deverá estar explicitamente associado ao autorizador.

A validação de identidade pelo API Gateway não substitui as verificações de autorização de negócio executadas pela Lambda.

Essas definições representam configurações planejadas e ainda não foram implementadas.

## 7. Autorização de negócio — AWS Lambda

A validação da identidade pelo API Gateway não substitui as regras de autorização da aplicação.

Considere o cenário de consulta de pedidos:

Um cliente autenticado solicita um pedido pelo identificador informado na URL.

A Lambda deverá confirmar que o pedido pertence à identidade autenticada antes de disponibilizar os dados.

A aplicação não deve confiar exclusivamente no identificador do cliente enviado pelo navegador.

### Riscos que desejamos prevenir

- Consulta de pedidos de outros clientes.
- Alteração indevida de informações.
- Acesso horizontal não autorizado.
- Exposição de dados pessoais.

Os identificadores confiáveis da sessão autenticada deverão orientar as verificações de propriedade dos registros.

## 8. IAM — Princípio do menor privilégio

A função AWS Lambda deverá utilizar uma IAM Execution Role.

Essa função de execução concederá apenas as permissões necessárias para as operações previstas.

### Exemplo de permissões

| Permissão | Finalidade |
|---|---|
| `dynamodb:GetItem` | Consultar um registro |
| `dynamodb:Query` | Executar consultas por chave ou índice |
| `dynamodb:PutItem` | Registrar dados |
| `dynamodb:UpdateItem` | Atualizar dados autorizados |
| `dynamodb:TransactWriteItems` | Executar operações transacionais necessárias |
| Permissões de CloudWatch Logs | Registrar logs de execução |

Essa tabela é ilustrativa. Não representa uma política IAM pronta para implantação.

Cada permissão deverá ser associada aos recursos efetivamente utilizados.

Operações de leitura não deverão receber desnecessariamente permissões de gravação.

Se a aplicação for dividida em funções Lambda com responsabilidades distintas, suas permissões também deverão ser separadas quando apropriado.

Permissões IAM controlam o acesso aos recursos AWS, mas não substituem regras de autorização entre clientes.

## 9. Segurança do DynamoDB

O DynamoDB será acessado pela aplicação por meio de permissões IAM.

### Controles previstos

- Restringir as permissões às tabelas e índices necessários.
- Utilizar criptografia em repouso fornecida pelo serviço.
- Proteger os dados durante o transporte.
- Validar os dados antes de operações de escrita.
- Considerar operações condicionais e transacionais para integridade.
- Definir políticas de recuperação adequadas à importância dos dados.

A estimativa contempla recuperação para um ponto no tempo (PITR) conforme a configuração considerada na AWS Pricing Calculator.

A restauração, o processo de recuperação e seus tempos não foram testados.

## 10. Proteção da comunicação

O acesso público à aplicação e à API deverá utilizar HTTPS.

A arquitetura prevê certificados do AWS Certificate Manager:

| Endpoint | Região do certificado |
|---|---|
| CloudFront com domínio personalizado | `us-east-1` |
| API Gateway REST Regional com domínio personalizado | `sa-east-1` |

Os domínios do projeto são ilustrativos.

### Origem S3

O CloudFront deverá utilizar OAC com assinatura de requisições habilitada (`SigningBehavior: always`) para acessar o bucket S3 privado.

A política do bucket deverá restringir a leitura à distribuição autorizada, utilizando a condição `AWS:SourceArn`.

### Comunicação entre serviços

A integração API Gateway–Lambda e o acesso Lambda–DynamoDB utilizam os mecanismos de integração, autenticação e autorização disponibilizados pela AWS.

Essas comunicações não exigem a criação de uma VPC própria para o cenário proposto.

## 11. Segurança da API e CORS

O frontend e a API utilizam domínios distintos.

A implementação deverá configurar CORS com as origens, métodos e cabeçalhos necessários.

Não serão permitidas origens arbitrárias indiscriminadamente nas operações que exijam restrições específicas.

CORS não é uma ferramenta de autenticação.

Como medida adicional de redução da superfície de exposição, a proposta considera desabilitar o endpoint padrão `execute-api` quando a API estiver publicada exclusivamente por domínio personalizado.

Essa medida deverá ser validada com as configurações finais de implantação.

## 12. Logs, métricas e auditoria

### Amazon CloudWatch

A proposta contempla:

- Logs operacionais da Lambda.
- Métricas de execução.
- Alarmes para erros e outros indicadores relevantes.
- Retenção planejada de logs de 14 dias.

Os logs não deverão registrar senhas, tokens completos, informações de pagamento ou dados pessoais desnecessários.

### AWS CloudTrail

O histórico de eventos de gerenciamento será utilizado como referência para análise de ações realizadas nos recursos AWS.

A disponibilidade e retenção dos registros deverão ser consideradas conforme a modalidade de auditoria configurada.

### AWS Budgets

O orçamento máximo mensal previsto é de US$ 500.

Os alertas planejados são:

| Percentual | Valor |
|---|---:|
| 50% | US$ 250 |
| 80% | US$ 400 |
| 100% | US$ 500 |

Esses alertas auxiliam o acompanhamento financeiro, mas não desligam automaticamente os recursos nem impedem cobranças.

## 13. Riscos residuais e limitações

Mesmo com os controles planejados, permanecem riscos que exigiriam análise e validação em uma implementação real.

Entre eles:

- Configuração inadequada de IAM ou de políticas de recursos.
- Falhas na autorização de pedidos.
- Exposição indevida de tokens.
- Regras WAF insuficientes ou excessivamente restritivas.
- Falhas na configuração de CORS.
- Abuso de endpoints públicos.
- Crescimento inesperado do consumo de recursos.
- Tratamento inadequado de informações pessoais.

Não foram realizados testes de segurança, revisão de políticas efetivas ou auditorias em uma infraestrutura provisionada.

## 14. Considerações finais

A arquitetura Nova Tech adota uma abordagem de defesa em profundidade, na qual diferentes controles possuem responsabilidades complementares.

O objetivo não é afirmar que a aplicação já está segura, mas demonstrar como seus principais riscos foram considerados durante o planejamento.

A eficácia dessas medidas dependeria de implementação correta, configuração adequada e validação contínua.

## 15. Referências

- [AWS Shared Responsibility Model](https://aws.amazon.com/compliance/shared-responsibility-model/)
- [AWS IAM — Security Best Practices](https://docs.aws.amazon.com/IAM/latest/UserGuide/best-practices.html)
- [CloudFront — Restrict Access to an S3 Origin](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html)
- [API Gateway — Cognito User Pools Authorizer](https://docs.aws.amazon.com/apigateway/latest/developerguide/apigateway-integrate-with-cognito.html)
- [AWS WAF Documentation](https://docs.aws.amazon.com/waf/)
- [AWS Well-Architected Framework](https://docs.aws.amazon.com/wellarchitected/latest/framework/welcome.html)

---

Nova Tech — Estudo individual de arquitetura AWS, desenvolvido para aprendizado e portfólio profissional.