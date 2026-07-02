# HLD Aplikacji
```mermaid
graph TD
    %% Definicja stylów
    classDef userStyle fill:#f9f,stroke:#333,stroke-width:2px;
    classDef ingressStyle fill:#bbf,stroke:#333,stroke-width:2px;
    classDef appStyle fill:#ddf,stroke:#333,stroke-width:2px;
    classDef dataStyle fill:#fdf,stroke:#333,stroke-width:2px;
    classDef extStyle fill:#ffb,stroke:#333,stroke-width:2px;

    %% Klienci
    subgraph "Klienci / Konsumenci"
        Web[Przeglądarka / Aplikacja X]
        Mobile[Aplikacja Mobilna]
    end
    class Web,Mobile userStyle;

    %% Warstwa Brzegowa
    subgraph "Warstwa Ingress & Security (GCP)"
        LB[Global Cloud Load Balancer / WAF]
        Entra[Microsoft Entra ID]
    end
    class LB ingressStyle;
    class Entra extStyle;

    %% Warstwa Aplikacyjna
    subgraph "Warstwa Aplikacyjna (GCP)"
        Core[System Centralny Aplikacji X]
        MS[Dedykowane Mikroserwisy]
        AI[Narzędzia AI / Vertex AI]
    end
    class Core,MS,AI appStyle;

    %% Warstwa Danych
    subgraph "Warstwa Danych & Backup (GCP)"
        DB_Centr[(Centralna Baza Danych)]
        DB_Ded[(Dedykowane Bazy Danych)]
        BKP[(Cloud Storage - Backup)]
    end
    class DB_Centr,DB_Ded,BKP dataStyle;

    %% Systemy Zewnętrzne
    subgraph "Systemy Zewnętrzne / On-Premise"
        SAP[System SAP]
        AMMS[AMMS / InfoMedica]
        ExtSys[Inne Systemy Zewnętrzne]
    end
    class SAP,AMMS,ExtSys extStyle;

    %% Relacje i komunikacja
    Web -->|HTTPS / WSS| LB
    Mobile -->|HTTPS| LB
    LB -->|OIDC / OAuth2| Entra
    LB -->|HTTPS / gRPC| Core
    LB -->|HTTPS / gRPC| MS
    
    Core <-->|REST / gRPC| MS
    Core <-->|API / Python SDK| AI
    MS <-->|API / Python SDK| AI
    
    Core -->|SQL| DB_Centr
    MS -->|SQL / NoSQL| DB_Ded
    
    DB_Centr -->|Backup Policy| BKP
    DB_Ded -->|Backup Policy| BKP
    
    Core <-->|VPN / Interconnect| SAP
    Core <-->|VPN / Interconnect| AMMS
    MS <-->|VPN / Interconnect| SAP
    MS <-->|VPN / Interconnect| AMMS
    Core -->|HTTPS| ExtSys
    MS -->|HTTPS| ExtSys
