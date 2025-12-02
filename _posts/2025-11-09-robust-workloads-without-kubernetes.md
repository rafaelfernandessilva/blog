---
title: "Workloads Robustos Sem Kubernetes: Quando o K8s Não é Necessário."
date: 2025-12-02 12:00:00 +0000
categories: [devops]
tags: [kubernetes, devops, arquitetura, containers, workloads]
image: /assets/img/robust-workloads-without-kubernetes.png
---

## Por que migrei workloads de Kubernetes para ECS (e pode ser que também seja seu caminho).

Neste artigo, mostrarei que você pode ter um workload super robusto, com alta resiliência, disponibilidade e confiabilidade, sem precisar do famigerado Kubernetes (nesse momento, começo a ter haters). Sim, o Kubernetes é uma das criações de maior impacto para quem trabalha com tecnologia nos últimos anos. Porém, o hype foi tão grande que gerou um surto coletivo, fazendo com que empresas achassem que tudo precisava ser em Kubernetes, e não é bem assim.

Eu conheci pessoas que atenderam clientes que tinham um site estático mega simples, quase uma landing page, em um cluster Kubernetes.

Eu mesmo peguei casos em que o cluster tinha aplicações super simples e de pouco tráfego, um banco de dados, tudo criado sem o mínimo de organização, sem NetPol, sem definição de rollout, containers sem limites definidos, o deployment era feito manualmente alterando o arquivo yaml dentro do cluster, sem distinção por namespace, e diversas outras boas práticas de organização que deveriam ter.

Ou seja, acontece o que sempre digo, ***`não é porque está funcionando que está correto`***.

Mesmo utilizando um Kubernetes gerenciado, como o EKS, o nível de expertise para configurar e manter um ambiente desse é algo alto e caro, que não se acha em qualquer esquina.

Todo esse contexto, toda essa "provocação", é porque outro dia eu estava em um evento de tecnologia com foco na galera DevOps, o DevOpsDaysBH, e comecei aleatoriamente um papo com um cara que é referência em Kubernetes, que conheci no evento. Papo vai, papo vem, eu falei com ele que fiz o caminho inverso de muitas empresas,pois eu realizei algumas migrações de Kubernetes para o ECS.

E caso você não conheça o ECS, você está vivendo errado( brincadeira 😅 ). 

Mas, vamos lá. Primeiro, vou falar rapidamente, de forma bem básica, o que é o ECS.

---

### O que é o Amazon ECS?

O Amazon Elastic Container Service (ECS), é um serviço totalmente gerenciado de orquestração de containers que ajuda a gerenciar e dimensionar facilmente aplicações conteinerizadas. Por ser um serviço gerenciado pela AWS, ele tem uma enorme facilidade de integração nativa com serviços como ECR, CloudWatch, ELB, GuardDuty e diversos outros.

“Tá, mas de que é composto o ECS? Estou acostumado com o meu Kubernetes.” Beleza, tudo bem, vamos lá.

No ECS, temos:

* **Cluster:** É basicamente um agrupamento lógico de máquinas/instâncias/nós operando para a execução de containers. De forma bem básica, seria isso.
* **Task Definition:** A Task Definition é como se fosse um blueprint de tudo que será criado. Fazendo um paralelo ao Kubernetes, é como se você estivesse escrevendo o manifesto (YAML) do seu Pod, no qual você define quais containers irão subir, define volume, recursos operacionais (CPU/Memória) que podem ser utilizados, imagem a ser utilizada, e etc.
* **Task:** A Task é como se fosse o output do que a Task Definition mandou para o Cluster. Ela será o resultado do que você definiu na Task Definition. É como se você desse um `kubectl apply -f meupod.yaml` e o Pod subisse no seu Cluster. A Task é, como se fosse, o seu Pod em execução.
* **Service:** O Service é o conjunto de Tasks criadas com base no seu Task Definition. E, como vocês aqui são tudo kuberneteiros, é como se fosse o seu Deployment. Nele, você consegue definir estratégia de rollout, o número de Tasks a serem criadas, definições de rede e mais algumas coisinhas.

---

### Por que escolher o ECS em vez do Kubernetes?

Beleza, mas o fato de ser algo que lembra muito o Kubernetes, por que eu iria escolher ele ao invés do Kubernetes? Vamos a alguns pontos:

**Complexidade e Curva de Aprendizado**
O Kubernetes exige uma curva de aprendizado infinitamente maior que o ECS. Sua equipe precisará aprender conceitos de rede, segurança, escalabilidade, observabilidade, entre outros, que são facilitados e/ou abstraídos no ECS, fazendo dele uma solução simples de orquestração de containers.

**Gerenciamento da Infraestrutura**
Você terá muito pouco esforço no que diz respeito à sobrecarga operacional, pois a AWS gerencia a maior parte dos requisitos, tudo por debaixo dos panos, e suas integrações. Você pode, por exemplo, utilizar o Fargate, que é o modo serverless, e basicamente só se preocupar com suas Tasks/containers, sem gerenciar uma única instância EC2.

**Velocidade de Entrega**
Com o ECS o famoso Time-to-Market (período de tempo que leva desde “start” de uma ideia para um produto ou serviço até o seu lançamento) pode ser bem menor. Isso para startups, equipes de produtos ou projetos que precisam ser lançados rapidamente é essencial. Dessa forma, a simplicidade do ECS pode ser um grande aliado. Isso ocorre porque você gasta menos tempo tanto na curva de aprendizado quanto definindo as melhores práticas de configuração a serem adotadas, para aquele cenario.

**Custo**
Para muitas cargas de trabalho, o ECS pode ser muito mais econômico que o EKS, visto que não é cobrado o gerenciamento do control plane . É cobrado apenas as instâncias usadas (no caso de EC2) ou os recursos consumidos (no Fargate), e o custo operacional de uma equipe para manter um cluster ECS é bem menor do que um custo operacional de uma equipe para manter um cluster Kubernetes.

**Integração Nativa**
Por ser um serviço da AWS, a sua integração com outras ferramentas do ecossistema da AWS é excelente, fazendo com que não precise utilizar ferramentas de terceiros, ou configurações mirabolantes para resolver determinados problemas. Hoje com a evolução de serviços como a stack Developer Tools, você pode utilizar:

* CodeCommit (Git)
* CodeArtifact (Gerenciador de artefatos)
* CodeBuild (Build)
* CodeDeploy (Deploy)
* CodePipeline (Orquestrador de CI/CD)

Isso faz com que você possa criar toda uma esteira para entrega para deploy ao ECS.

* **ECR:** Tem o ECR, o registry, que já tem a sua integração ao criar uma taskdefinition, que no console você pode visualmente escolher qual versão da imagem inserir no seu taskdefinition.
* **CloudWatch:** Você tem o CloudWatch, no qual tem uma ótima stack de observabilidade, com:
    * CloudWatch Metrics
    * CloudWatch Logs
    * CloudWatch Logs Insights
    * CloudWatch Alarms
    * CloudWatch Container Insights
* **Parameter Store:** Temos também o parameter-store, que para os kuberneteiros de plantão seria como se fosse um *ConfigMap ou um Secret*, no qual podemos guardar os env da aplicação e compartilhar o mesmo entre os services (caso o ambiente exija).

E diversos outros serviços complementares que podemos bater um papo depois.

---

### E quanto a estrategia de deployment?

Bom, agora nós já vimos que temos vários pontos positivos para se usar ECS em alguns workload ao invés do Kubernetes. Mas um dos pontos fortes do Kubernetes é sua maneira de realizar o rollout, deployment e também o seu “serviço de rede” que seria a forma que com os serviços de comunicação dentro do cluster.

**Estratégias de Deployment**
No ECS temos vários tipos de rollout que podemos utilizar de forma nativa, são eles:

* Rolling Update
* Blue/Green 
* Linear Deployment 
* Canary Deployment

Sendo assim, você pode escolher o melhor cenário para a aplicação, pois algumas aplicações o melhor cenário seria o Blue/Green, outras seria o Canary. Então o ECS consegue atender bem a necessidade da sua aplicação. E podemos ter todo um fluxo de GitOps, fazendo com que o github por exemplo seja a fonte da verdade, e sempre que subir um push para branch X ou Y todo o fluxo de CI/CD iniciar e realizar a entrega no cluster.

### E como minhas aplicações se comunicação internamente?

**Rede e Service Discovery**
No ECS nos também temos algumas configurações de rede que podem deixar a comunicação entre serviços internamente no cluster algo muito bom. Para isso podemos usar o ECS Service Connect e também o AWS Cloud Map, no qual você ira poder, por exemplo, fazer chamadas entre suas aplicações utilizando DNS interno (como meuapp.dns.local), de forma simples, aumentando inclusive a segurança na comunicação entre apps dentro do cluster.

---

### Escalabilidade
**Auto Scaling service**

O serviço de autoscaling do ECS também funciona muito bem, nele você pode fazer trigger com base em métricas que são enviadas para o cloudwatch, sendo assim com base nessas métricas você pode expandir o seu serviço adicionando mais tasks a sua aplicação e então conseguindo então suportar aquela demanda requerida no momento, é possível fazer o scaling por schedule, caso você saiba o periodo em que sua aplicação é mais exigida como por exemplo ao rodar determinada cron as 00hrs você pode colocar um autoscaling schedule para aumentar o número de task para X e após determinado horário voltar para Y.

**Auto scaling group**
Que ira gerenciar a quantidade de instancias do cluster, nele você ira poder determinar o minimo de instancia do cluster, o valor desejado e também o numero maximo de instancias que aquele cluster pode ter. 

Dessa forma em um cenario que você definiu que o cluster deve ter no minimo 1 instancia, o valor que deseja 2 instancia, e o maximo 5. Ele ira monitorar para atender essas necessidades, lembrando que o upscaling ira ocorrer com base em metricas defina por você.

**Capacity Provider**
É onde você pode por exemplo balancear para que 90% do sua aplicação suba em instancias spot e os outros 10% em instancia ondemand. Você pode também determinar tipos de instancias diferentes mais adequada para cada aplicação dentro do seu cluster. Por exemplo, eu posso definir que app batatinha suba no capacity com instancia voltara para consumo de cpu, e o app abrobinha suba no capacity com instancias voltadas para consumo de memoria. 
Dessa forma eu consigo ter um cluster de apps compartilhado com melhor aproveitamento dos recursos computacionais.

---

Bom, após uma introdução simples falando que nem sempre precisamos de Kubernetes para solucionar nossos desafios de entrega, espero ter mostrado que o ECS é uma alternativa madura, poderosa, resilientem, confiavel e pratica.

E com isso creio que chegamos a uma conclusão, que  a decisão entre os dois não é sobre qual é "melhor", mas sim qual é melhor para aquele determinado cenário, dadas as circunstâncias apresentadas, a experiência da equipe e o contexto estratégico do projeto.

Muitas vezes, a ferramenta certa é aquela que resolve o seu problema com a menor complexidade e custo operacional possível, e o ECS é muito bom quando se trata disso.

Vou deixar para vocês um pequeno lab, totalmente focado para o lado didático no qual temos algumas configurações interessantes. Nesse lab vamos utilizar o github para ter o nosso fluxo GitOps, iremos ter uma pipeline até a entrega de uma aplicação no ECS, colocando alguns testes simples para verificar a nossa imagem enviada para o registry que ira no cluster, vamos ter também alguns testes em nosso código terraform que será responsável por criar esse laboratório.


> ## ***Lembrando, isso é apenas para fins de estudo, não pegue o código e coloque em seu ambiente de produção sem antes analisar o cenário do seu workload.***


### ***Laboratorio:. [Você pode fazer um fork, clonar e brincar com o lab que criei de demonstração.](https://github.com/rafaelfernandessilva/ecs-workload-laboratorio).***

#### *** [https://github.com/rafaelfernandessilva/ecs-workload-laboratorio](https://github.com/rafaelfernandessilva/ecs-workload-laboratorio).***