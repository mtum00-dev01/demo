#LLD Aplikacji
```mermaid
graph LR
    %% Subnet: Ingress
    subgraph "Subnet: Ingress / DMZ"
        LB[Cloud Armor + External LB]
    end

    %% Subnet: Application
    subgraph "Subnet: Application Private"
        GKE_Core[GKE: Core App X]
        GKE_MS[GKE: Microservices]
        Vertex[Vertex AI / GenAI]
    end

    %% Subnet: Data
    subgraph "Subnet: Data Private"
        DB_Main[(Cloud SQL: Central DB)]
        DB_Ded[(Cloud SQL / Spanner)]
    end

    %% Inne usługi GCP
    subgraph "GCP Platform Services"
        GCS[(Cloud Storage: Backup)]
        VPN[Cloud VPN Gateway]
    end

    %% Chmury zewnętrzne i On-Premise
    subgraph "Identity Provider"
        Entra[Entra ID / IDP]
    end

    subgraph "On-Premise Data Center"
        SAP[SAP ERP]
        AMMS[AMMS / InfoMedica]
    end

    %% Przepływy i Porty
    Internet((Klienci)) -->|HTTPS / 443| LB
    LB -->|HTTPS / 443 Auth| Entra
    
    LB -->|HTTPS / 443| GKE_Core
    LB -->|HTTPS / 443| GKE_MS

    GKE_Core <-->|gRPC:50051 / HTTP:8080| GKE_MS
    GKE_Core -->|HTTPS / 443 API| Vertex
    GKE_MS -->|HTTPS / 443 API| Vertex

    GKE_Core -->|PostgreSQL:5432 / MySQL:3306| DB_Main
    GKE_MS -->|PostgreSQL:5432 / Redis:6379| DB_Ded

    DB_Main -->|HTTPS / 443 Export| GCS
    DB_Ded -->|HTTPS / 443 Export| GCS

    GKE_Core -->|Ruch Wewnętrzny| VPN
    GKE_MS -->|Ruch Wewnętrzny| VPN

    VPN <-->|IPsec Tunnel / RFC / HTTPS| SAP
    VPN <-->|IPsec Tunnel / HL7 / FHIR| AMMS
