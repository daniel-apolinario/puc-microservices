# puc-microservices
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

## Fase 1: O lançamento do Mobile App
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

title Fase 1: O Lançamento Unificado (App Consome Velho e Novo)

actor "Novo App Mobile" as App
component "API Gateway / BFF" as Gateway

package "Novos Serviços (Cloud)" {
    component "Pix Service" as PixMS
    database "Pix DB" as PixDB
    PixMS -down-> PixDB
}

package "Ecossistema Legado (Datacenter Físico)" {
    component "REST Adapter / ACL\n(Nova fachada para o App)" as Adapter
    component "Monólito Legado\n(Boletos, Conta Corrente)" as Monolito
    database "MySQL Legado" as LegadoDB
    
    Adapter -down-> Monolito
    Monolito -down-> LegadoDB
}

App -down-> Gateway
Gateway -down-> PixMS : Roteia PIX
Gateway -down-> Adapter : Roteia Boletos\ne Extrato

note left of Adapter
  A equipe do legado constrói 
  endpoints REST básicos para 
  o App conseguir pagar boletos.
end note

@enduml
```



## Arquitetura to-be
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

title Arquitetura To-Be: TechBanco (Microsserviços & Cloud)

actor "Novo App Mobile" as App
actor "Sistemas Externos\n(Bacen/PIX, Open Banking)" as Externo

node "Ambiente Cloud / Kubernetes (Microcontainers)" {
    
    component "API Gateway / BFF\n(Autenticação, Roteamento, Rate Limit)" as Gateway

    queue "Message Broker (Ex: Kafka / RabbitMQ)\n<<Eventos Assíncronos / Saga Pattern>>" as Broker

    package "Novos Microsserviços (Domínios Isolados)" {
        
        component "Pix Service" as PixMS
        database "Pix DB" as PixDB
        PixMS -down-> PixDB
        
        component "Open Banking Service" as OBMS
        database "OpenBanking DB" as OBDB
        OBMS -down-> OBDB
        
        component "Account & Cards Service" as AccountMS
        database "Accounts DB" as AccountDB
        AccountMS -down-> AccountDB
    }
    
    package "Ecossistema Legado (Strangler Fig Pattern)" {
        component "Anti-Corruption Layer (ACL)" as ACL
        component "Monólito Legado\n(Boletos, Core antigo)" as Monolito
        database "MySQL Legado" as LegadoDB
        
        ACL -down-> Monolito
        Monolito -down-> LegadoDB
    }
}

App -down-> Gateway : REST / GraphQL
Externo -down-> Gateway : mTLS / APIs Seguras

Gateway -down-> PixMS
Gateway -down-> OBMS
Gateway -down-> AccountMS
Gateway -down-> ACL : Roteamento de rotas não migradas

PixMS .up.> Broker : Publica/Consome
OBMS .up.> Broker : Publica/Consome
AccountMS .up.> Broker : Publica/Consome
ACL .up.> Broker : Sincroniza dados legados

note right of Gateway
  **Ponto de Entrada Único:**
  Aplicativo mobile não fala 
  direto com os serviços.
end note

note bottom of LegadoDB
  O monólito vai encolhendo
  conforme novas lógicas vão 
  para os microsserviços.
end note

note right of PixDB
  **Database per Service:**
  O serviço PIX pode escalar
  seu banco independentemente.
end note

@enduml
```
