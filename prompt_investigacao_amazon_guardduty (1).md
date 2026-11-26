# Do Alerta à Resposta: Investigação de Achado do Amazon GuardDuty

## Objetivo

Orientar a investigação de um achado do Amazon GuardDuty de forma
estruturada, separando fatos confirmados, hipóteses e lacunas de
informação. A análise deve apoiar a decisão de contenção sem inventar
dados nem recomendar ações disruptivas sem explicitar seus impactos.

## Prompt final

``` text
Você é um analista de segurança em nuvem especializado em AWS. Ajude-me a investigar o achado do Amazon GuardDuty descrito abaixo. Sua análise será utilizada pela equipe de infraestrutura durante o atendimento de um alerta.

O ACHADO

- Tipo: UnauthorizedAccess:EC2/SSHBruteForce
- Recurso afetado: instância EC2 em uma sub-rede pública, responsável por hospedar a API de homologação de um e-commerce.
- Severidade: 8.0 — alta.
- Quando: entre 02h10 e 03h40 de uma terça-feira, no horário de Brasília. A data exata ainda não foi informada.
- Descrição do GuardDuty: tentativas repetidas de autenticação SSH na porta 22, originadas de um único endereço IP externo, durante aproximadamente noventa minutos.

PÚBLICO-ALVO

A resposta será lida pela pessoa de plantão da equipe de infraestrutura. Ela tem conhecimento sólido de Linux e está começando a atuar com segurança na AWS.

DECISÃO ESPERADA

Determinar, com base nas evidências disponíveis, se é necessário restringir imediatamente o acesso à instância ou se há condições seguras para continuar a investigação antes de aplicar medidas adicionais.

CONTEXTO DO AMBIENTE — DADOS CONFIRMADOS ATÉ O MOMENTO

- A instância hospeda a API de homologação. Se ficar indisponível, a equipe de QA terá suas atividades interrompidas, mas nenhum cliente de produção será afetado.
- O acesso administrativo é realizado exclusivamente pela equipe de desenvolvimento, por SSH, a partir do escritório e da VPN corporativa.
- O security group permite conexões na porta 22 a partir de qualquer origem. Essa configuração foi herdada da criação do ambiente.
- O AWS CloudTrail está habilitado na região.
- Os VPC Flow Logs estão ativos na VPC da instância.
- A instância utiliza autenticação por par de chaves e não permite autenticação SSH por senha.
- Ainda não foi confirmado se alguma tentativa de autenticação teve sucesso.
- Ainda não foi verificado se o endereço IP de origem aparece em outros achados do GuardDuty.

FORMATO OBRIGATÓRIO DA RESPOSTA

Organize a análise rigorosamente nesta ordem:

1. O QUE O ACHADO INDICA
   Explique em linguagem simples o significado do achado, o comportamento detectado e o que ele permite afirmar. Diferencie tentativa de conexão, tentativa de autenticação e autenticação bem-sucedida. Não trate o alerta, isoladamente, como prova de comprometimento.

2. HIPÓTESES
   Apresente as hipóteses em ordem decrescente de plausibilidade. Identifique explicitamente cada uma como hipótese, nunca como fato confirmado. Considere, quando aplicável:
   - Tentativa externa não autorizada de força bruta ou descoberta de credenciais.
   - Varredura automatizada ou atividade oportunista da Internet.
   - Atividade legítima não reconhecida, como acesso de origem corporativa não documentada.
   - Comprometimento da instância, somente se houver evidências que sustentem essa possibilidade.

3. EVIDÊNCIAS PARA CONFIRMAR OU DESCARTAR CADA HIPÓTESE
   Para cada hipótese, informe:
   - Evidência que a confirmaria.
   - Evidência que a enfraqueceria ou descartaria.
   - Serviço AWS, sistema operacional ou fonte de log em que a evidência pode ser encontrada.
   - Campo, evento ou informação específica a procurar.
   - Limitação da fonte, quando houver.

   Considere as fontes abaixo sem atribuir a elas capacidades que não possuem:
   - Amazon GuardDuty: detalhes do finding, endereço IP de origem quando disponibilizado, recurso afetado, portas, horários, contagem e histórico de achados relacionados.
   - Amazon VPC Flow Logs: registros de fluxo de rede, endereços de origem e destino, portas, protocolo, ação de tráfego e intervalo de agregação, conforme o formato configurado. Os Flow Logs demonstram tráfego de rede, mas não comprovam, por si só, sucesso ou falha de autenticação SSH.
   - Logs de autenticação do Linux: /var/log/auth.log em distribuições baseadas em Debian/Ubuntu ou /var/log/secure em distribuições que utilizam esse arquivo, além do journal do serviço SSH quando aplicável. Procurar eventos como tentativas malsucedidas, autenticações aceitas, usuário utilizado, método de autenticação e horário. Não presuma qual distribuição está instalada.
   - Amazon EC2: verificar estado da instância, metadados disponíveis, configuração de rede, interfaces, security groups associados e alterações relevantes.
   - AWS CloudTrail: verificar chamadas de API relacionadas à instância, à rede e às configurações AWS, incluindo alterações de security group e operações administrativas. Não use o CloudTrail como se fosse um registro das autenticações SSH realizadas dentro do sistema operacional.
   - AWS IAM: verificar alterações de identidade e permissões que sejam pertinentes ao caso, sem presumir que o acesso SSH por chave da instância esteja necessariamente registrado como evento IAM.
   - Outros achados do GuardDuty: verificar recorrência do mesmo IP, recursos relacionados, tipos de achado e períodos de ocorrência.

   Se uma fonte não estiver habilitada, não estiver disponível ou não registrar o evento procurado, declare essa limitação. Não invente nomes de eventos, campos, IDs ou resultados.

4. CONTENÇÃO SUGERIDA
   Separe as medidas em:
   A. Reversíveis e de menor impacto operacional.
   B. Disruptivas ou de maior impacto, que exigem avaliação e autorização.

   Para cada medida, informe objetivo, impacto operacional, condição que justifica sua aplicação e possibilidade de reversão.

   Avalie prioritariamente a restrição do acesso SSH no security group para as origens corporativas autorizadas, desde que os endereços ou intervalos oficiais do escritório e da VPN sejam confirmados antes da alteração. Não invente esses endereços.

   Antes de recomendar qualquer ação que possa interromper acesso, indisponibilizar a API, encerrar a instância, apagar recursos ou revogar credenciais, informe explicitamente o impacto em uma linha.

   Não recomende encerrar ou excluir a instância como primeira medida sem evidência e justificativa proporcionais. Preserve evidências relevantes antes de ações que possam alterá-las ou eliminá-las, quando isso for viável e compatível com a urgência do incidente.

5. INFORMAÇÕES PENDENTES PARA ENCERRAR A INVESTIGAÇÃO
   Liste objetivamente os dados ainda necessários, indicando por que cada um é importante e onde obtê-lo. Diferencie o que impede uma conclusão do que apenas aumenta a confiança da análise.

REGRAS OBRIGATÓRIAS

- Utilize somente os dados fornecidos e as evidências efetivamente apresentadas.
- Nunca invente ID de instância, ID de finding, endereço IP, conta AWS, região, data, horário, usuário, nome de security group, regra, endereço corporativo ou resultado de log.
- Identifique como FATO CONFIRMADO apenas aquilo que foi explicitamente fornecido ou demonstrado por evidência.
- Identifique como HIPÓTESE toda explicação ainda não comprovada.
- Identifique como NÃO VERIFICADO todo ponto cuja evidência ainda não foi consultada.
- Nunca declare falso positivo sem indicar a evidência objetiva que sustentaria essa conclusão.
- Não confunda tráfego de rede registrado em VPC Flow Logs com autenticação SSH bem-sucedida.
- Não confunda eventos de API do CloudTrail com logs de autenticação do sistema operacional.
- Não afirme que a ausência de um registro prova que uma atividade não ocorreu sem avaliar a cobertura, retenção, configuração e disponibilidade da fonte.
- Antes de recomendar ação disruptiva, informe o impacto operacional e a justificativa.
- Se faltar informação essencial, declare precisamente o dado ausente e faça perguntas objetivas antes de emitir uma conclusão definitiva.
- Não transforme possibilidade em certeza. Não atribua país, identidade, intenção ou reputação ao IP sem fonte verificável.
- Encerre com uma avaliação explícita: CONTENÇÃO IMEDIATA JUSTIFICADA, CONTENÇÃO PREVENTIVA RECOMENDADA ou INVESTIGAÇÃO ADICIONAL NECESSÁRIA. Explique quais fatos sustentam a classificação e quais limitações permanecem.
```

## Testes de validação do prompt

Execute os três testes abaixo separadamente e avalie se a IA respeita as
regras.

### Teste 1 --- Cenário principal

Use o achado e o contexto preenchidos no prompt final.

**Critérios de aprovação:** - Mantém a estrutura obrigatória de cinco
seções. - Separa fatos confirmados, hipóteses e informações não
verificadas. - Não afirma que houve acesso bem-sucedido sem consultar os
logs de autenticação do Linux. - Indica fontes específicas para cada
evidência. - Diferencia corretamente VPC Flow Logs, CloudTrail e logs de
autenticação SSH. - Separa contenção reversível de ação disruptiva e
explicita o impacto operacional.

### Teste 2 --- Informação que não foi fornecida

Após a primeira resposta, pergunte:

``` text
De qual país partiu o acesso e qual é o ID da instância EC2?
```

**Critério de aprovação:** a IA deve informar que esses dados não foram
fornecidos e que precisam ser consultados nos detalhes do finding ou nos
recursos correspondentes. Não deve estimar, deduzir ou inventar país,
endereço IP ou ID.

### Teste 3 --- Achado ambíguo

Substitua o achado principal por:

``` text
Tipo: Policy:IAMUser/RootCredentialUsage
Contexto: o alerta ocorreu durante uma migração planejada. Ainda não foi informado quem executou a atividade, qual credencial foi utilizada nem se o uso estava previsto no plano de mudança.
```

Pergunte se o evento deve ser considerado incidente ou atividade
legítima.

**Critério de aprovação:** a IA não deve concluir que é incidente
confirmado nem falso positivo. Deve solicitar dados como identidade
envolvida, horário, evento correspondente no CloudTrail, contexto e
autorização da mudança. Deve manter a distinção entre evidência e
hipótese.

## Checklist de revisão antes da publicação

-   [ ] O prompt apresenta o tipo, recurso, severidade, período e
    descrição do achado.
-   [ ] O público-alvo e a decisão esperada estão definidos.
-   [ ] O contexto operacional e os controles existentes estão
    descritos.
-   [ ] A resposta exige cinco seções em ordem fixa.
-   [ ] Fatos, hipóteses e informações não verificadas são
    diferenciados.
-   [ ] As fontes de evidência são identificadas com suas limitações.
-   [ ] As ações de contenção estão classificadas por impacto e
    reversibilidade.
-   [ ] Os três testes de validação foram executados.
-   [ ] O arquivo publicado está acessível sem autenticação.

## Sugestão de nome do arquivo

`prompt_investigacao_amazon_guardduty.md`

## Descrição curta para o repositório

Prompt estruturado para investigação de achados do Amazon GuardDuty, com
classificação de hipóteses, validação de evidências em serviços AWS e
logs Linux, critérios de contenção e testes de robustez contra respostas
sem fundamentação.
