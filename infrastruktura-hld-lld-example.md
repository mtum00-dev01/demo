HLD i LLD Aplikacji X - przykład z perspektywy infrastruktury

# Diagram HLD
```mermaid

graph LR

    %% --- Kolumna 1: Klienci ---
    subgraph C1["Klienci / Konsumenci"]
        Web[Przeglądarka / Aplikacja X]
        Mobile[Aplikacja Mobilna]
    end

    %% --- Kolumna 2: Edge / Security ---
    subgraph C2["Warstwa Ingress & Security (GCP)"]
        LB[Global Cloud Load Balancer / WAF]
        Entra[Microsoft Entra ID]
    end

    %% --- Kolumna 3: Aplikacja ---
    subgraph C3["Warstwa Aplikacyjna (GCP)"]
        Core[System Centralny Aplikacji X]
        MS[Dedykowane Mikroserwisy]
        AI[Narzędzia AI / Vertex AI]
    end

    %% --- Kolumna 4: Dane ---
    subgraph C4["Warstwa Danych & Backup (GCP)"]
        DB_Centr[(Centralna Baza Danych)]
        DB_Ded[(Dedykowane Bazy Danych)]
        BKP[(Cloud Storage - Backup)]
    end

    %% --- Kolumna 5: Systemy zewnętrzne ---
    subgraph C5["Systemy Zewnętrzne / On-Premise"]
        SAP[System SAP]
        AMMS[AMMS / InfoMedica]
        ExtSys[Inne Systemy Zewnętrzne]
    end

    %% Główne przepływy
    Web -->|HTTPS / WSS| LB
    Mobile -->|HTTPS| LB
    LB -->|OIDC / OAuth2| Entra
    LB -->|HTTPS / gRPC| Core
    LB -->|HTTPS / gRPC| MS

    Core <-->|REST / gRPC| MS
    Core -->|API / Python SDK| AI
    MS -->|API / Python SDK| AI

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

    %% Niewidoczne krawędzie stabilizujące układ (bez renderowanej linii)
    C1 ~~~ C2
    C2 ~~~ C3
    C3 ~~~ C4
    C4 ~~~ C5
```

# Diagram LLD

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

```

# Macierz komunikacji
| Źródło (Source) | Cel (Destination) | Protokół | Port | Opis / Przeznaczenie |
| :--- | :--- | :--- | :--- | :--- |
| **Internet (User)** | Cloud Load Balancer | HTTPS | `443` | Szyfrowany ruch kliencki (Web/Mobile) |
| **Cloud LB** | Entra ID | HTTPS | `443` | Autentykacja użytkowników (OIDC/OAuth2 Redirects) |
| **Cloud LB** | GKE (App X / MS) | HTTP / HTTP2 | `80`, `443` | Przekazywanie ruchu z LB do Ingress Controller |
| **Pod: App X** | Pod: Mikroserwisy | HTTP / gRPC | `8080`, `50051` | Komunikacja wewnętrzna między usługami |
| **GKE Pods** | Vertex AI | HTTPS | `443` | Wywołania modeli LLM przez Private Google Access |
| **GKE Pods** | Cloud SQL (Central) | TCP | `5432` / `3306` | Dostęp do bazy danych (PostgreSQL lub MySQL) |
| **GKE Pods** | Cloud SQL (Ded.) | TCP | `5432` / `6379` | Dostęp do baz dedykowanych / Cache (Redis) |
| **GKE / Cloud SQL** | Cloud Storage | HTTPS | `443` | Zrzuty backupowe i eksport danych |
| **GKE Pods** | SAP (On-Premise) | TCP / HTTPS | `443`, `33xx` | Integracja z SAP przez tunel VPN |
| **GKE Pods** | AMMS / InfoMedica | TCP / HTTP | `HL7` / `FHIR` | Wymiana danych medycznych przez Cloud VPN |
| **GKE Pods** | Vertex AI | HTTPS | `443` | Wywołania modeli LLM przez Private Google Access |
| **GKE Pods** | Cloud SQL (Central) | TCP | `5432` / `3306` | Dostęp do bazy danych (PostgreSQL lub MySQL) |
| **GKE Pods** | Cloud SQL (Ded.) | TCP | `5432` / `6379` | Dostęp do baz dedykowanych / Cache (Redis) |
| **GKE / Cloud SQL** | Cloud Storage | HTTPS | `443` | Zrzuty backupowe i eksport danych |
| **GKE Pods** | SAP (On-Premise) | TCP / HTTPS | `443`, `33xx` | Integracja z SAP przez tunel VPN |
| **GKE Pods** | AMMS / InfoMedica | TCP / HTTP | `HL7` / `FHIR` | Wymiana danych medycznych przez Cloud VPN |
