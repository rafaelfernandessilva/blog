---
title: "Permissions Boundary (limite de permissões) no IAM"
date: 2026-03-31 14:00:00 -0300
categories: security
tags: [iam, permission-boundary, aws, devsecops]
description: "Entenda o modelo de controle Permissions Boundary no IAM e como aplicar governança de permissões." 
image: /assets/img/permission-boundary.png
---


Aqui está o conteúdo do arquivo formatado em Markdown, com a organização das seções e blocos de código, removendo as marcações de citação conforme solicitado:

---

## 1. O que são essas permissions boundaries ?

As permissions boundaries não é uma permissão exatamente citada para determinado usuario mas talvez a melhor forma de definir seria o limite até onde se pode ir. E uma forma de fazer com que em alguns casos dê autonomia a determinadas equipes para criar suas proprias roles no IAM sem que crie algo com usuario Administrator por exemplo.

Vamos imaginar que equipe de como por exemplo DevSecOps ou InfraSec tenha por padrão uma policy chamada DevPermissionsBoundary. Esta policy vai permitir permite varias ações em serviços como EC2, S3, DynamoDB,SQS e mas nega explicitamente todas as ações referente ao IAM (iam:*). Em seguida, eles concedem a um desenvolvedor a permissão iam:CreateRole, mas com uma condição em sua política que exige que qualquer role que ele crie deve ter o DevPermissionsBoundary anexado.

O resultado é que o desenvolvedor ganha autonomia para criar roles para suas aplicações conforme necessário, mas é não é permitido criar uma role com  administrator ou de modificar o próprio boundary para escalar seus privilégios.

Isso ocorre por que a role criada ela já nasce com limitações, quem esta criando só consegue criar ela se inserir o boundary o que faz com que permita criar porém a criação ja nasce limitada.

E como se você fosse um porteiro (desenvolvedor(a)) e tivesse que permitir entrada de alguém da manutenção ao condominio (criar role), porém você não pode simplesmente liberar a entrada do prestador de serviço, se você apenas tentar liberar a entrada ele vai ser barrado, mas o sindico do condominio te deu algumas tags com limite de acesso a alguns lugares, então você da a ele uma tag (boundary), e após isso ele consegue entrar no condominio porém ele já tem sua entrada permitida com determinadas restrições, essa tag permite a ele ter acesso apenas de ir ao bloco A apartamento 203 e bloco B apartamento 501. 

Ou seja você tem a permissão de criar uma role para um determinado serviço ou usuario porém essa role já é criada com as limitações definida pela boundary.

Agora que temos uma ideia vamos saber como realmente funciona a boundary e sua interseção.

Para que a o usuario tenha sucesso na criação/utilização/manipulação de um recurso precisa existir uma combinação validando isso na interseção de permissão.

Vamos para alguns exemplos 

1 o usuario tem permissão para criar um RDS:
                "rds:CreateDBInstance",

mas o permission boundary fixado aquele usuario esta definido que ele tem permissão apenas de listar instancias ec2:

                "ec2:DescribeInstances"

Sendo assim o usuario ao tentar criar um RDS ira tomar um erro, pois a permission boundary define o seu teto de permissões.

## 2. Funcionamento e Interseção
De acordo com a documentação da AWS, o funcionamento da interseção de um Permissions Boundary (Limite de Permissões) baseia-se no princípio de que as permissões efetivas de um usuário ou função (role) são apenas as ações permitidas simultaneamente por dois tipos de políticas

Para que uma ação seja autorizada, ela deve estar presente tanto na policy/role adicionada ao usuario quanto no teto que seria o permission boundary.


1.  **Policy de Identidade** (Permissão de leitura): 
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "s3:Get*",
                    "s3:List*"
                ],
                "Resource": [
                    "arn:aws:s3:::NOME-DO-SEU-BUCKET",
                    "arn:aws:s3:::NOME-DO-SEU-BUCKET/*"
                ]
            }
        ]
    }
    ```
2.  **Boundary** também diz que pode fazer a leitura:: 
    ```json
    {
        "Version": "2012-10-17",
        "Statement": [
            {
                "Effect": "Allow",
                "Action": [
                    "s3:Get*",
                    "s3:List*"
                ],
                "Resource": "*"
            }
        ]
    }
    ```
* **Resultado**:Temos sucesso e então o usuario ira poder criar o acesso ou executar aquele procedimento.

![Resultado](/assets/img/boundary-ok.png)


Se a política de identidade permitir uma ação (ex: iam:CreateUser), mas essa ação não constar no Permissions Boundary, o acesso será negado. O boundary atua como o barreira privilégios

***Por exemplo***

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "iam:CreateUser",
            ],
            "Resource": "*"
        }
    ]
}
 ```

***O boundary permite apenas ações em cima do s3 e ec2:***
```json

{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "s3:*",
                "ec2:*"
            ],
            "Resource": "*"
        }
    ]
}
 ```

![Resultado](/assets/img/boundary-deny.png)

* **Resultado**: O usuário não poderá criar um usuário IAM, pois essa ação não está incluída no Permissions Boundary, mesmo que a política de identidade permita.

---

## 3. Pontos Importantes

É sempre bom lembrar que o boundary, por si só, não dá permissão ele apenas limita o que outras políticas podem conceder.

Outro ponto importante é que dentro da AWS um Deny explícito em qualquer uma das políticas envolvidas na interseção sempre vai ter um peso maior sobre qualquer "Allow" se o boundary contiver um Deny (que seria a barreira o tero) para uma ação específica, essa ação será bloqueada, mesmo que o usuario tenha uma role muito permissiva como a AdministratorAccess.

#### ***SCP (Service Control Policies)***

E esse tipo de interseção também se extende quando se trata de SCPs (Service Control Policies) em uma organização AWS, onde as permissões efetivas de um usuário ou role são a interseção entre a política de identidade, o Permissions Boundary e a SCP aplicada conta ou organização. Ou seja nesse caso a ação deve ser permitida por todas as três camadas para que o acesso seja concedido.

![Resultado](/assets/img/boundary-scp.png)


#### ***Resource-Based Policies***
Porém temos um cenario um pouco diferente quando falamos de políticas baseadas em recursos (Resource-Based Policies) como as políticas de bucket S3 ou políticas de fila SQS. Nesses casos, as permissões concedidas por essas políticas também precisam estar na interseção com o Permissions Boundary para que sejam efetivas, mas a forma como isso é aplicado pode variar dependendo do tipo de entidade (usuário ou role) e do serviço específico envolvido. Por exemplo:

Se um bucket esta dizendo que o usuario batatinha tem permissão para ler um objeto, mas o boundary não possui nenhuma negação explicita ou seja não fala que pode e nem que não pode, o usuario tem acesso a leitura daquele objeto dentro do bucket pois no boundary não existe uma negação explicita, ou seja um teto um limite de acesso.

Mas quando se trata de roles, funciona de uma forma mais rigorosa, se uma role tem permissão para acessar um recurso por Resource-Based Policy, mas o Permissions Boundary não diz nada sobre acesso ao bucket explicitamente e diz que aquela role tem permissão apenas para acessar o EC2, então a role não terá acesso ao bucket mesmo que a política de recurso permita, pois o boundary atua de uma forma um pouco mais rigida para roles.

---
## 4. Conclusão

As Permissions Boundaries são uma ferramenta poderosa para implementar o princípio de menor privilégio e garantir que os usuários e roles não escale permissões, mesmo que tenham políticas de identidade permissivas. Sendo assim permitindo uma governança mais granular e segura, principalmente em ambiente com muitiplas contas e equipes.