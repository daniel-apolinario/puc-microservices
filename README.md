# puc-microservices
Repositório para materiais da disciplina "Arquitetura de Microsserviços e Microcontainer: o Negócio como Serviço"

# Estudo de Caso - Banco Pinhão


'''mermaid

flowchart TD
    %% Atores
    Cliente([Clientes do Banco\nPF e PMEs])

    %% Infraestrutura Legada
    subgraph Datacenter [Datacenter Físico On-Premises]
        direction TB
        
        subgraph ServidorApp [Servidor Tomcat - Aplicação Monolítica]
            direction TB
            JSF[Interface Gráfica\nJSF]
            Spring[Backend\nJava + Spring MVC]
            
            JSF -- Chamadas internas\nÚnico Binário --> Spring
        end
        
        BD[(Banco de Dados\nMySQL Compartilhado)]
    end

    %% Conexões
    Cliente -- Requisições HTTP/S --> JSF
    Spring -- Consultas SQL\nGargalo e SPOF --> BD

    %% Estilos
    classDef monolith fill:#fdedec,stroke:#cb4335,stroke-width:2px;
    class ServidorApp monolith;
