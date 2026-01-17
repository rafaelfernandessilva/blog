---
title: "O que é ConfigMap no Kubernetes?"
date: 2026-01-17 09:00:00 -0300
categories: kubernetes
tags: [kubernetes, configmap, devops, configuração]
description: "Nesse artigo vamos ter um resumo sobre o configMap e tentar trazer com palavras trazer um entendimento dele com palavras mais simples."
image: /assets/img/wha_is_configmap.png
---



Eu já vi algumas pessoas (até eu mesmo), confundindo ConfigMap com Secrets, ou não sabendo quando utilizar um ou outro. Então vamos tentar esclarecer o que é configmap e quando utilizar.

Os ConfigMaps são objetos da API do Kubernetes utilizados para armazenar dados não confidenciais em pares chave-valor. Eles funcionam como uma estrutura para guardar informações de configuração separadamente do código da aplicação, permitindo que artefatos de configuração sejam desacoplados do conteúdo da imagem do conteiner.

Você pode esta se perguntando, mas qual o benefício de utilizar configMap?

Bom uma das ou talvez a maior motivação para se usar configmap é fazer com que tenha maior desacoplamento e portabilidade da sua aplicação, fazendo por exemplo que você não precise fazer um novo build da imagem para atualizar as variáveis ou algum tipo de arquivo de configuração.

Vamos tentar fazer uma analogia para simplificar.

Imagine que você tem uma mochila que comprou para colocar um tênis e ir para academia, nesse cenário a mochila seria a imagem e o tênis algo "externo" seria o configmap. Se você precisar trocar o cadarço do tênis, ou a palmilha você não precisa comprar outra mochila, ou seja não vai precisar fazer um novo build da imagem, você simplesmente troca o tênis (configmap) e continua usando a mesma mochila (imagem).

Agora que já entendemos um pouquinho sobre confiMap, vamos abordar algumas formas de utilizar.

Para utilizar configMap temos as seguintes formas:

***Variavel de ambiente para um container*** :

 Nesse cenario os dados do configmap são injetados como variavel de ambiente, sendo assim é possível definir uma variavel individualmente a partir de uma chave especifica ou carregar todos os pares chave-valor do configmap com o campo envFrom.
 Exemplo:
 
 ```yaml
    apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: registry.k8s.io/busybox
      command: [ "/bin/sh", "-c", "env" ]
      envFrom:
      - configMapRef:
          name: special-config
  restartPolicy: Never
```

***Command e Args*** :

  Utilizando command e args é uma forma de parametrizar a inicialização de uma aplicação por exemplo, e para isso você deve definir uma variável de ambiente do container que recebe o valor de uma chave do configmap, junto disso você referencia essa variavel no command ou args.
  Exemplo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: registry.k8s.io/busybox
      command: [ "/bin/echo", "$(SPECIAL_LEVEL_KEY) $(SPECIAL_TYPE_KEY)" ]
      env:
        - name: SPECIAL_LEVEL_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config
              key: SPECIAL_LEVEL
        - name: SPECIAL_TYPE_KEY
          valueFrom:
            configMapKeyRef:
              name: special-config
              key: SPECIAL_TYPE
  restartPolicy: Never

```
* Nota:. valores consumidos via command ou args são definidos no momento que o kubelet lança o container, ou seja caso atualize algum valor o pod precisa ser reiniciado.


***Volume ReadOnly***:


  O configMap pode ser montado como um volume no sistema de arquivos de um container, cada chave no configMap torna-se um arquivo individual dentro do diretório de montagem especificado, aqui nesse cenário os configMaps montado como volumes são atualizados automaticamente pelo kubelet, embora aplicações subPath não recebam essa atualização 

Exemplo:

```yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: special-config
  namespace: default
data:
  SPECIAL_LEVEL: very
  SPECIAL_TYPE: charm
---

apiVersion: v1
kind: Pod
metadata:
  name: dapi-test-pod
spec:
  containers:
    - name: test-container
      image: registry.k8s.io/busybox
      command: [ "/bin/sh", "-c", "ls /etc/config/" ]
      volumeMounts:
      - name: config-volume
        mountPath: /etc/config
  volumes:
    - name: config-volume
      configMap:
        name: special-config
  restartPolicy: Never


```

***API do kubernetes para ler um configMap***:


 Essa creio que que seja a maneira mais avançada pois transfere parte da responsabilidade para a propria aplicação. Diferente das outras formas, aqui nesse caso a aplicação se comunica com o kube-apiserver, o desenvolvedor deve incluir na aplicação uma lib de cliente para realizar as requisições. Sendo assim a aplicação pode usar a funcionalidade de watch da API para receber notificações em tempo real quando um configMap for alterado. 
 Essa forma mesmo sendo a mais "complexa", trás vantagens como o acesso entre namespace, diferente dos outros metodos que só podem acessar configMaps do mesmo namespace.

* Nota:. para que um pod consiga consultar a API do kubernetes ele precisa ter as permissões adequadas via RBAC.

Agora que já sabemos o básico de configMap, mas em quais situações devemos utilizar ele no workload?

Em um workload do kubernetes, o uso de configMap é possível em varios cenários e ele é essencial para garantir a portabilidade, escalabilidade, e a manutenção do sistema. O objetivo é desacoplar os artefatos de configuração da imagem do container, com isso a mesma imagem pode ser usada em diversos ambientes 

Exemplos de uso:
* Variavel ``DB_HOST`` que define o host do banco de dados pode ser diferente entre ambientes (desenvolvimento, homologação, produção), com configMap você pode definir essa variavel de ambiente para cada ambiente sem precisar criar imagens diferentes.

* Configurações de log: niveis de log, formatos, destinos podem ser configurados via configMap para cada ambiente.

* Arquivos de configuração: Pode definir por exemplo arquivos como um ``redis.conf``, ``httpd.conf``, ou qualquer outro arquivo de configuração que sua aplicação utilize.

* Feature flags: Habilitar ou desabilitar funcionalidades específicas da aplicação com base em configMaps.

Em resumo, o configMap é uma ferramenta poderosa no Kubernetes para gerenciar configurações de forma flexível e eficiente, promovendo boas práticas de desenvolvimento e operações. 

Espero que esse post tenha de forma resumida ajudar a compreender um pouco sobre configMap.


Referência: 
- [ConfigMap — Configurar Pod/Container (pt-BR)](https://kubernetes.io/pt-br/docs/tasks/configure-pod-container/configure-pod-configmap/)
- [ConfigMap — Conceito (en)](https://kubernetes.io/docs/concepts/configuration/configmap/)
- [EKS: auth ConfigMap (AWS)](https://docs.aws.amazon.com/pt_br/eks/latest/userguide/auth-configmap.html)
- [Tutorial: Configure Redis usando ConfigMap (pt-BR)](https://kubernetes.io/pt-br/docs/tutorials/configuration/configure-redis-using-configmap/)
