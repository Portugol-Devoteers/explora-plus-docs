# Modelagem e Diagramas — Explora+

> Especificações de modelagem e diagramas UML do MVP. Base para a construção dos diagramas finais.

---

## Modelagem MVP (ER)

```mermaid
erDiagram
    USER ||--o{ ROUTE : creates
    USER ||--o{ TICKET : owns

    CATEGORY ||--o{ PLACE : categorizes

    PLACE ||--o{ PLACE_IMAGE : "has gallery"
    PLACE ||--o{ TICKET : "sells access to"
    PLACE ||--o{ ROUTE : "is destination of"

    USER {
        bigint id PK
        string email UK
        string username UK
        string password_hash
        datetime date_joined
    }
    CATEGORY {
        bigint id PK
        string slug UK "monumento|evento|transporte"
        string name
        string icon_name
    }
    PLACE {
        bigint id PK
        string slug UK
        bigint category_id FK
        string name
        text description
        point location "PostGIS Point (lng,lat)"
        string address
        string hours_open "ex: 09:00-18:00"
        int price_cents "null = grátis"
        string currency "BRL"
        datetime event_start_at "null se não é evento"
        datetime event_end_at "null se não é evento"
        bool is_active
        datetime created_at
        datetime updated_at
    }
    PLACE_IMAGE {
        bigint id PK
        bigint place_id FK
        string url
        int order
        string caption
    }
    ROUTE {
        bigint id PK
        bigint user_id FK
        bigint destination_place_id FK
        point origin "PostGIS Point"
        string transport_mode "transit|rideshare|walking|driving"
        int distance_m
        int duration_s
        text polyline "encoded ou JSON LineString"
        datetime created_at
    }
    TICKET {
        bigint id PK
        bigint user_id FK
        bigint place_id FK
        int quantity
        int total_cents
        string currency
        string status "reserved|paid|cancelled|used"
        datetime purchased_at
        datetime used_at "null se não usado"
    }
```

### Resumo das entidades

| Entidade | Por que existe no MVP |
|---|---|
| **User** (Django nativo) | autenticação, dono de rotas e ingressos |
| **Category** | popula os chips de filtro (Monumentos / Eventos / Transporte) |
| **Place** | unidade central do app. Contém localização PostGIS, conteúdo (PT), preço, e *opcionalmente* janela temporal pra eventos (`event_start_at` / `event_end_at` nullable — assim "evento" é só um Place com data) |
| **PlaceImage** | a tela Detalhe tem carousel de N imagens |
| **Route** | "Rota Inteligente" salva: origem, destino, modal, polyline, duração, distância |
| **Ticket** | aba Ingressos: histórico de compras com status |

### Decisões já tomadas (MVP)

- **Sem i18n**: tudo em PT, campo único `name`/`description` no Place
- **Evento ≠ tabela separada**: campos nullable no Place
- **Rota single-destination**: sem tabela `RouteWaypoint` — destino é um único `Place`
- **Sem Favorite, sem UserProfile**: cortados do MVP
- **Sem PlaceTranslation**: cortado junto com i18n
- **Polyline na Route**: campo `text` (encoded polyline) — mais barato de armazenar e o frontend decoda

---

## Diagramas UML

### Casos de Uso

```mermaid
flowchart LR
    Visitante((Visitante))
    Turista((Turista))
    Admin((Administrador))

    Visitante --> UC1[Buscar lugares]
    Visitante --> UC2[Filtrar por categoria]
    Visitante --> UC3[Ver detalhes do lugar]
    Visitante --> UC4[Criar conta / Login]

    Turista --> UC1
    Turista --> UC2
    Turista --> UC3
    Turista --> UC5[Gerar rota inteligente]
    Turista --> UC6[Escolher modo de transporte]
    Turista --> UC7[Iniciar navegação]
    Turista --> UC8[Comprar ingresso]
    Turista --> UC9[Ver meus ingressos]
    Turista --> UC10[Ver histórico de rotas]
    Turista --> UC11[Editar perfil]

    Admin --> UC12[Cadastrar lugar]
    Admin --> UC13[Editar lugar]
    Admin --> UC14[Gerenciar categorias]
    Admin --> UC15[Visualizar rotas e ingressos]
```

### Classes

```mermaid
classDiagram
    class User {
        +Long id
        +String email
        +String username
        +String passwordHash
        +DateTime dateJoined
        +authenticate(password) bool
    }

    class Category {
        +Long id
        +String slug
        +String name
        +String iconName
    }

    class Place {
        +Long id
        +String slug
        +String name
        +String description
        +Point location
        +String address
        +String hoursOpen
        +int priceCents
        +String currency
        +DateTime eventStartAt
        +DateTime eventEndAt
        +bool isActive
        +distanceTo(point) float
        +isEvent() bool
        +isFree() bool
    }

    class PlaceImage {
        +Long id
        +String url
        +int order
        +String caption
    }

    class Route {
        +Long id
        +Point origin
        +String transportMode
        +int distanceM
        +int durationS
        +String polyline
        +DateTime createdAt
        +decodePolyline() List~Point~
    }

    class Ticket {
        +Long id
        +int quantity
        +int totalCents
        +String currency
        +String status
        +DateTime purchasedAt
        +DateTime usedAt
        +markAsUsed() void
        +cancel() void
        +isValid() bool
    }

    User "1" --> "0..*" Route : creates
    User "1" --> "0..*" Ticket : owns
    Category "1" --> "0..*" Place : categorizes
    Place "1" *-- "0..*" PlaceImage : has
    Place "1" --> "0..*" Route : destination
    Place "1" --> "0..*" Ticket : access
```

### Componentes

```mermaid
flowchart TB
    subgraph Mobile["Aplicativo Móvel (React Native + Expo)"]
        UI[Telas / UI]
        Nav[Navegação]
        State[Estado local]
        APIClient[Cliente HTTP]
        MapView[Mapa / Leaflet]
        UI --> Nav
        UI --> State
        State --> APIClient
        UI --> MapView
    end

    subgraph Backend["API Backend (Django + DRF)"]
        AuthAPI[Auth API<br/>JWT]
        PlacesAPI[Places API]
        RoutesAPI[Routes API]
        TicketsAPI[Tickets API]
        AdminUI[Django Admin]
        RoutingSvc[Routing Service<br/>wrapper]
    end

    DB[(PostgreSQL<br/>+ PostGIS)]
    ORS[OpenRouteService API]
    OSM[OpenStreetMap Tiles]

    APIClient -->|HTTPS / JSON| AuthAPI
    APIClient -->|HTTPS / JSON| PlacesAPI
    APIClient -->|HTTPS / JSON| RoutesAPI
    APIClient -->|HTTPS / JSON| TicketsAPI
    MapView -->|HTTPS| OSM

    AuthAPI --> DB
    PlacesAPI --> DB
    RoutesAPI --> DB
    RoutesAPI --> RoutingSvc
    TicketsAPI --> DB
    AdminUI --> DB
    RoutingSvc -->|HTTPS| ORS
```

### Implantação

```mermaid
flowchart TB
    subgraph Cliente["Dispositivo do Turista"]
        App["App Móvel<br/>iOS / Android / Web"]
    end

    subgraph Host["Servidor de Aplicação<br/>(VPS / Cloud)"]
        direction TB
        subgraph DockerNet["Rede Docker"]
            BackendC["Container: backend<br/>Django + Gunicorn<br/>porta 8000"]
            DBC["Container: db<br/>postgis/postgis:16-3.4<br/>porta 5432"]
            BackendC <--> DBC
        end
        Nginx["Nginx<br/>reverse proxy<br/>porta 443"]
        Nginx --> BackendC
    end

    subgraph Terceiros["Provedores Externos"]
        ORS["OpenRouteService"]
        OSM["OSM Tile Servers"]
    end

    App -->|HTTPS| Nginx
    App -->|HTTPS| OSM
    BackendC -->|HTTPS| ORS
```

### Sequência — Gerar Rota

```mermaid
sequenceDiagram
    actor T as Turista
    participant App as App Móvel
    participant API as Backend API
    participant DB as PostgreSQL
    participant ORS as OpenRouteService

    T->>App: Toca em "Gerar Rota"
    App->>API: POST /api/routes/ {origin, placeId, mode}
    API->>DB: SELECT * FROM place WHERE id = :placeId
    DB-->>API: Place(location, name)
    API->>ORS: POST /v2/directions/{mode}
    ORS-->>API: {polyline, distance, duration}
    API->>DB: INSERT INTO route (...)
    DB-->>API: Route(id)
    API-->>App: 201 Created {route}
    App->>App: Decodifica polyline
    App-->>T: Renderiza rota no mapa
```

### Sequência — Compra de Ingresso

```mermaid
sequenceDiagram
    actor T as Turista
    participant App as App Móvel
    participant API as Backend API
    participant DB as PostgreSQL

    T->>App: Toca em "Comprar Ingressos"
    App->>App: Valida sessão (JWT)
    alt Não autenticado
        App-->>T: Redireciona para Login
        T->>App: Informa credenciais
        App->>API: POST /api/auth/login
        API-->>App: {access, refresh}
    end
    App-->>T: Mostra modal de confirmação
    T->>App: Confirma quantidade
    App->>API: POST /api/tickets/ {placeId, quantity}
    API->>DB: INSERT INTO ticket (status='reserved')
    DB-->>API: Ticket(id, code)
    API-->>App: 201 Created {ticket}
    App-->>T: Exibe na aba Ingressos
```

### Atividade — Explorar e Gerar Rota

```mermaid
flowchart TD
    Start([Início]) --> Open[Abrir app]
    Open --> Permission{Permite<br/>localização?}
    Permission -->|Sim| FetchLoc[Obter GPS]
    Permission -->|Não| Manual[Usar localização padrão]
    FetchLoc --> LoadMap
    Manual --> LoadMap[Carregar mapa<br/>e pins próximos]
    LoadMap --> Filter{Aplicar<br/>filtro?}
    Filter -->|Sim| Apply[Filtrar por categoria]
    Filter -->|Não| Tap
    Apply --> Tap[Tocar em um pin]
    Tap --> Detail[Ver tela de detalhe]
    Detail --> Decide{Gerar rota?}
    Decide -->|Não| End([Fim])
    Decide -->|Sim| Route[Tela Rota Inteligente]
    Route --> ChooseMode[Escolher modal]
    ChooseMode --> StartNav[Iniciar navegação]
    StartNav --> End
```

### Estados — Ciclo do Ticket

```mermaid
stateDiagram-v2
    [*] --> Reservado : turista compra
    Reservado --> Pago : confirmação de pagamento
    Reservado --> Cancelado : turista desiste
    Pago --> Utilizado : check-in no local
    Pago --> Expirado : passou da data
    Pago --> Cancelado : cancelamento permitido
    Cancelado --> [*]
    Utilizado --> [*]
    Expirado --> [*]
```
