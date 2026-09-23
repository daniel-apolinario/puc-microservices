# puc-microservices
Repositório para materiais da disciplina "Arquitetura de Microsserviços e Microcontainer: o Negócio como Serviço"

# Estudo de Caso - Banco Pinhão


@startuml
!theme plain
skinparam componentStyle uml2
skinparam padding 5
skinparam NodePadding 20

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

' Relações externas e de infraestrutura
Cliente -down-> JSF : Requisições HTTP(s)
Spring -down-> BD : Consultas SQL (JDBC/JPA)

' Notas para orientar a discussão na Live
note right of Monolito
  **Gargalos (Pain Points):**
  - Forte acoplamento (MVC)
  - Deploy demorado (Tudo ou nada)
  - Difícil escalar camadas separadamente
end note

note bottom of BD
  Único ponto de falha (SPOF)
  Gerenciado por equipe DBA em silo
end note

@enduml
