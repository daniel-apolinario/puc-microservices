# PUC - Arquitetura de Microsserviços e Microcontainer: o Negócio como Serviço
Repositório para materiais da disciplina "Arquitetura de Microsserviços e Microcontainer: o Negócio como Serviço"

# Estudo de Caso - Banco Pinhão
## Arquitetura as-is
```plantuml
@startuml
!theme plain
skinparam componentStyle uml2

<style>
document {
  Padding 5
}
node {
  Padding 20
}
</style>

title Arquitetura As-Is: Banco Pinhão (Legado)

actor "Clientes do Banco\n(Pessoas Físicas e PMEs)" as Cliente

node "Datacenter Físico (On-Premises)" <<Servidores Linux>> {
    
    node "Servidor de Aplicação" <<Apache Tomcat>> {
        
        component "Aplicação Monolítica\n(Único Binário .war)" as Monolito {
            
            package "Camada de Apresentação" {
                [Interface Gráfica] <<JSF - JavaServer Faces>> as JSF
            }
            
            package "Camada de Negócio e Lógica" {
                [Backend] <<Java + Spring Framework>> as Spring
            }
            
            JSF -down-> Spring : Chamadas de método (MVC)
        }
    }
    
    database "Servidor de Banco de Dados" <<MySQL>> {
        [Tabelas: Contas, Cartões,\nBoletos, Clientes, etc.] as BD
    }
}

Cliente -down-> JSF : Requisições HTTP(s)
Spring -down-> BD : Consultas SQL (JDBC/JPA)

note right of Monolito
  **Gargalos (Pain Points):**
  - Forte acoplamento
  - Deploy demorado (Tudo ou nada)
  - Difícil escalar camadas separadamente
  - Maior complexidade
end note

note bottom of BD
  Único ponto de falha (SPOF)
  Gerenciado por equipe DBA em silo
end note
@enduml
```

## Fase 1: A Fachada (App Mobile consumindo o legado)
```plantuml
@startuml
!theme plain
skinparam componentStyle uml2

<style>
document {
  Padding 5
}
node {
  Padding 20
}
</style>

title Fase 1: A Fachada (App Mobile consumindo o Legado)

actor "Novo App Mobile" as App
component "API Gateway" as Gateway

package "Ecossistema Legado (Datacenter Físico)" {
    component "REST Adapter / ACL\n(Primeira versão da Fachada)" as Adapter
    component "Monólito Legado\n(Todas as funções do banco)" as Monolito
    database "MySQL Compartilhado" as LegadoDB
    
    Adapter -down-> Monolito
    Monolito -down-> LegadoDB
}

App -down-> Gateway
Gateway -down-> Adapter : Roteia todo o tráfego\ndo App para o legado

note right of Adapter
  Permite lançar o App rápido,
  usando a regra de negócio
  já homologada do banco.
end note

@enduml
```

## Fase 2: Inovação e Integração
```plantuml
@startuml
!theme plain
skinparam componentStyle uml2

<style>
document {
  Padding 5
}
node {
  Padding 20
}
</style>

title Fase 2: Inovação e Integração (PIX, Conta Digital e Broker)

actor "Novo App Mobile" as App
component "API Gateway" as Gateway
queue "Message Broker\n<<Eventos Assíncronos>>" as Broker

package "Novos Serviços (Cloud)" {
    component "Pix Service" as PixMS
    component "Digital Account Service" as DigAccountMS
    database "Novos Bancos Isolados" as CloudDB
    
    PixMS -down-> CloudDB
    DigAccountMS -down-> CloudDB
}

package "Ecossistema Legado" {
    component "Anti-Corruption Layer (ACL)" as ACL
    component "Monólito Legado\n(Boletos, Contas Antigas)" as Monolito
    database "MySQL Compartilhado" as LegadoDB
    
    ACL -down-> Monolito
    Monolito -down-> LegadoDB
}

App -down-> Gateway
Gateway -down-> PixMS : Roteia PIX
Gateway -down-> DigAccountMS : Roteia Conta Digital
Gateway -down-> ACL : Roteia funções antigas

PixMS .up.> Broker
DigAccountMS .up.> Broker
ACL .up.> Broker : Adaptador Assíncrono

@enduml
```



## Fase 3: O Estrangulamento
```plantuml
@startuml
!theme plain
skinparam componentStyle uml2

<style>
document {
  Padding 5
}
node {
  Padding 20
}
</style>

title Fase 3: O Estrangulamento

actor "Novo App Mobile" as App
component "API Gateway" as Gateway
queue "Message Broker" as Broker

package "Novos Serviços (Cloud)" {
    component "Pix Service" as PixMS
    database "Pix DB" as PixDB
    PixMS -down-> PixDB
    
    component "Digital Account Service" as DigAccountMS
    database "Digital Account DB" as DigAccountDB
    DigAccountMS -down-> DigAccountDB
    
    component "Boleto Service\n(Extraído do legado)" as BoletoMS
    database "Boleto DB" as BoletoDB
    BoletoMS -down-> BoletoDB
}

package "Legado (Encolhendo)" {
    component "Anti-Corruption Layer (ACL)" as ACL
    component "Monólito Legado\n(Funções residuais)" as Monolito
    database "MySQL Compartilhado" as LegadoDB
    
    ACL -down-> Monolito
    Monolito -down-> LegadoDB
}

App -down-> Gateway

Gateway -down-> PixMS
Gateway -down-> DigAccountMS
Gateway -down-> BoletoMS : Nova rota de boletos
Gateway -down-> ACL : Rotas residuais (Core antigo)

PixMS .up.> Broker
DigAccountMS .up.> Broker
BoletoMS .up.> Broker
ACL .up.> Broker : Sincronização final

note bottom of BoletoMS
  O legado é migrado 
  pedaço por pedaço para 
  a nova arquitetura.
end note

@enduml
```
@enduml
```
