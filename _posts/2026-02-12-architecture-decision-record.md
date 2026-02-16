---
title: "Architecture Decision Record - ADR"
date: 2026-02-16 09:00:00 -0300
categories: Architecture
tags: [architecture, decision, record, adr, segurança, vulnerabilidade]
description: "Documentação de Architecture Decision Record, implementação de scan de vulnerabilidades em imagens de container."
image: /assets/img/adr.png
---

Nesse post vamos documentar a criação de um Architecture Decision Record (ADR) para a implementação de um scan de vulnerabilidades em imagens de container utilizando a ferramenta Trivy. O objetivo é garantir que as decisões arquiteturais sejam registradas de forma clara e acessível para toda a equipe, facilitando a comunicação e o entendimento das escolhas feitas.

___

# Criação de architecture decision record

***ADR-001:*** Implementação de Scan de Vulnerabilidades em Imagens de Container

***Status:*** Pendente

***Data:*** 16 de Fevereiro de 2026

***Decisor:*** Time de Engenharia e DevSecOps

### ***1. Contexto e Problema***

Foi detectado pelo time de segurança que existem falhas críticas e altas (CRITICAL e HIGH) nas aplicações atuais que comprometem workloads podendo gerar risco de exposição de dados, ou compromentimento total da infraestrutura. Atualmente, o pipeline de CI/CD não possui validação de segurança, permitindo o deploy de imagens vulneráveis. O desafio é implementar essa trava sem gerar atritos excessivos ou bloqueios desnecessários no fluxo de desenvolvimento.

### ***2. Decisão***
No intuito de analisar e eliminar vulnerabilidades em imagens de container, implementaremos o Trivy como ferramenta oficial de análise de vulnerabilidades no pipeline de CI/CD.

#### ***Detalhes da Implementação:***

* ***Integração no Pipeline***
    - O scan será executado imediatamente após o build, e analisar possiblidade de inserir cache para otimizar o tempo de execução.

* ***Política de Bloqueio:***
    - O pipeline falhará para vulnerabilidades CRITICAL ou HIGH que possuam correção oficial disponível (patch).

    - Vulnerabilidades sem correção conhecida serão reportadas, mas não bloquearão o build automaticamente, sendo tratadas no processo de exceção.

* ***Notificação***
    - Um alerta automático será enviado via Slack para o time de engenharia responsavel, contendo o log gerado pela ferramenta, detalhando as vulnerabilidades encontradas.
    - O time de segurança também será ira fazer  parte do suporte.

* ***Analise***
    - Além do sistema operacional da imagem, o scan incluirá bibliotecas de linguagem (npm, pip, etc.),e também Misconfigurations em Dockerfiles.

### 3. ***Consequências***
* #### ***Positivas***
* ***Redução de risco:***
    - Não permite que imagem com vulnerabilidade chegue em produção ambiente de produção.

* ***Agilidade no processo de remediação:***
    - O time de segurança é notificado em tempo real, agilizando a remediação.
* ***Analise detalhada:***
    - Criação de um log detalhado das vulnerabilidades encontradas, facilitando a anlise e correção.

* #### ***Negativas***

* ***Tempo de execução:***
    - O pipeline tera um tempo maior de execução devido ao processo de scan, o que pode impactar no deployment.
* ***Adaptação da cultura do time:***
    - Necessário treinamento e adaptação do time de engenharia para lidar com as novas exigências de segurança.

### 4. Alternativas Consideradas
* ***Testes Manuais***
    - Descartado por ser ineficiente, propenso a falhas humanas e impossível de escalar com a frequência necessária de deploys.

* ***Scan Informativo (Sem Falha)***
    - Descartado pois, embora gere alertas, não impede que a vulnerabilidade seja explorada em produção caso o alerta seja ignorado ou lido tarde demais.

### 5. Próximos Passos
* ***Configuração:***
    - Criar o de workflow do CI/CD para incluir o passo do Trivy.

* ***Performance e Otimização:***
    - Analisar a possibilidade de utilizar cache para otimizar o tempo de execução do scan.
* ***Alerta:***
    - Definir o canal de destino e o Webhook no Slack para as notificações.