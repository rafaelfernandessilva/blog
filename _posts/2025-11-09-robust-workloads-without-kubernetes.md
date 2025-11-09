---
title: "Workloads Robustos Sem Kubernetes: Quando o K8s Não é Necessário"
date: 2025-11-09 12:00:00 +0000
categories: [devops]
tags: [kubernetes, devops, arquitetura, containers, workloads]
image: /assets/img/robust-workloads-without-kubernetes.png
---

Neste artigo, mostrarei que você pode ter um workload super robusto, com alta resiliência e disponibilidade, sem precisar do famigerado Kubernetes (nesse momento, começo a ter haters). Sim, o Kubernetes é uma das criações de maior impacto para quem trabalha com tecnologia nos últimos anos. Porém, houve um surto coletivo em seu hype que fez com que empresas achassem que tudo precisava ser em Kubernetes, e não é bem assim.

Eu conheci pessoas que atenderam clientes que tinham um site estático mega simples, quase uma landing page, em um cluster Kubernetes.

Eu mesmo peguei casos em que o cluster tinha aplicações super simples e de pouco tráfego, um banco de dados um backend pouquissimo utilizado, tudo criado sem o mínimo de organização, sem NetPol,healthcheck, estrategia de deployment, distinção por namespace, e diversas outras boas práticas de organização que deveriam ter.

Ou seja, acontece o que sempre digo, `não é porque está funcionando que está correto`.

Mesmo utilizando um Kubernetes gerenciado, como o EKS, o nível de expertise para configurar e manter um ambiente desse é algo que não se acha em qualquer esquina.

Todo esse contexto, toda essa "balela", é porque outro dia eu estava em um evento de tecnologia com foco na cultura DevOps, o DevOpsDaysBH, e comecei aleatoriamente um papo com um cara que é referência em Kubernetes, que conheci no evento. Papo vai, papo vem, eu falei com ele que fiz o caminho inverso de muitas empresas,pois eu realizei algumas migrações de Kubernetes para o ECS.

E caso você não conheça o ECS, você está vivendo errado( brincadeira 😅 ). 

Mas, vamos lá. Primeiro, vou falar rapidamente, de forma bem básica, o que é o ECS.

## O que é o Amazon ECS?

ECS é o Elastic Container Service, um serviço gerenciado pela AWS de orquestração de contêineres que ajuda a gerenciar e dimensionar facilmente aplicações conteinerizadas. Por ser um serviço gerenciado pela AWS, ele tem uma enorme facilidade de integração com serviços como ECR, CloudWatch, ELB, GuardDuty e diversos outros.

"Tá, mas de que é composto o ECS? Estou acostumado com o meu Kubernetes." Beleza, tudo bem, vamos lá.

No ECS, temos:

**Cluster:** É basicamente um conjunto de máquinas/instâncias/nós operando para a execução de contêineres. De forma bem básica, seria isso.

**Task Definition:** A Task Definition é como se fosse um blueprint de tudo que será criado. Fazendo um paralelo ao Kubernetes, é como se você estivesse escrevendo um YAML do seu Pod, no qual você define quais contêineres irão subir, define volume, recursos operacionais que podem ser utilizados, imagem a ser utilizada, e etc.

**Task:** A Task é como se fosse o output do que a Task Definition mandou para o Cluster. Ela será o resultado do que você definiu na Task Definition. É como se você desse um `kubectl apply -f meupod.yaml` o Pod subirá no seu Cluster. A ideia aqui é a mesma vai pegar a configuração definida no seu arquivo Task Definition e então criar, com base naquela "receita de bolo" feita por você, os volumes, contêineres, e etc.

**Service:** O Service é o conjunto de Tasks criadas com base no seu Task Definition. E, como vocês aqui são tudo kuberneiteiros, é como se fosse o seu deployment.yaml. Nele, você consegue definir estratégia de deployment, o número de Tasks a serem criadas e mais algumas coisinhas.

## Por que escolher o ECS em vez do Kubernetes?

Beleza, mas o fato de ser algo que lembra muito o Kubernetes, por que eu iria escolher ele ao invés do Kubernetes? Vamos a alguns pontos:

#### Complexidade e Curva de Aprendizado

O Kubernetes exige uma curva de aprendizado infinitamente maior que o ECS. Sua equipe precisará aprender conceitos de rede, segurança, escalabilidade, observabilidade, entre outros, que são abstraídos no ECS, fazendo dele uma solução simples de orquestração de containers.

#### Gerenciamento da Infraestrutura

Você terá muito pouco esforço no que diz respeito à sobrecarga operacional, pois a AWS gerencia a maior parte dos requisitos, tudo por debaixo dos panos, e suas integrações. Você pode, por exemplo, utilizar o Fargate e basicamente só se preocupar com suas Tasks/contêineres.

#### Velocidade de Entrega

Com o ECS o famoso Time-to-Market que é o período de tempo que leva desde "start" de uma ideia para um produto ou serviço até o seu lançamento ser bem menor, e isso para startups, equipes de produtos ou projetos que precisam ser lançados rapidamente é essencial, dessa forma a simplicidade do ECS pode ser um grande aliado. Isso ocorre porque haverá menos tempo gasto tanto na curva de aprendizado quanto nas melhores práticas de configuração a serem adotadas.

#### Custo

Para muitas cargas de trabalho, o ECS pode ser muito mais econômico que o EKS, visto que não é cobrado o gerenciamento do control plane. É cobrado apenas as instâncias usadas, no caso de EC2, e o custo operacional de uma equipe para manter um cluster ECS é bem menor do que um custo operacional de uma equipe para manter um cluster kubernetes.

#### Integração

Por ser um serviço da AWS, a sua integração com outras ferramentas do ecossistema da AWS é excelente fazendo com que não precise utilizar ferramentas de terceiros, ou configurações mirabolantes para resolver determinados problemas, hoje com a evolução de serviços como a stack Developer Tools no qual você pode utilizar os serviços:

- CodeCommit
- CodeArtifact
- CodeBuild
- CodeDeploy
- CodePipeline

Faz que você possa criar toda uma esteira para entrega para deploy ao ECS.

Tem o ECR o registry que já tem a sua integração ao criar uma taskdefinition, que no console você pode visualmente escolher qual versão da imagem inserir no seu taskdefinition.

Você tem o cloudwatch no qual tem uma ótima stack de observabilidade, com:

- CloudWatch Metrics
- CloudWatch Logs
- CloudWatch Logs Insights
- CloudWatch Alarms
- CloudWatch Container Insights

E diversos outros recursos que te dão o poder de pegar métricas do seu service, task e seus conteiners dentro da task.

Temos também o parameter-store, que para os kuberneteiros de plantão seria como se fosse um configmap, no qual podemos guardar os env da aplicação e compartilhar o mesmo entre os services (caso o ambiente exija).

E diversos outros serviços complementares que podemos bater um papo depois.

## Rollout e Rede no ECS

Bom agora nos já vimos que temos vários pontos positivos para se usar ECS em alguns workload ao invés do Kubernetes, mas um dos pontos fortes do kubernetes é sua maneira de realizar o rollout e também o seu "serviço de rede" que seria a forma que com os serviços de comunicação dentro do cluster entre outras coisas.

Hoje no ECS temos vários tipos de rollout no qual podemos utilizar, são eles:

- Rolling Update
- Blue/Green
- Linear Deployment
- Canary Deployment

Mas no ECS nos também temos algumas configurações de rede no qual pode deixar a comunicação entre serviços internamente no cluster algo muito bom, e para isso podemos usar service conect e também temos o cloudmaps, no qual eu vou poder por exemplo fazer chamades entre minhas aplicações utilizando dns interno como `meuapp.dns.local`, aumentando inclusive a segurança na comunicação entre apps dentro do cluster.

