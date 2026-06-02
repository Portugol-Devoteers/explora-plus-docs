# Modelagem e Diagramas — Explora+

> Especificacoes de modelagem e diagramas UML do MVP entregue.
> Fontes Mermaid em `diagramas/mermaid/`. Projeto Modelio em `diagramas/explora_mais-diagramas-uml/`.

---

## Diagrama ER

```mermaid
erDiagram
    USER ||--o{ TOUR_ROUTE : cria
    USER ||--o{ USER_PLACE_STATE : possui

    PLACE_CATEGORY ||--o{ PLACE : categoriza

    PLACE ||--o{ PLACE_IMAGE : tem
    PLACE ||--o{ USER_PLACE_STATE : rastreada_em
    PLACE ||--o{ TOUR_ROUTE_STOP : aparece_em

    ROUTE_SEARCH_CACHE ||--o{ TOUR_ROUTE : origina

    TOUR_ROUTE ||--o{ TOUR_ROUTE_STOP : contem
    TOUR_ROUTE }o--o{ USER_PLACE_STATE : ultima_rota_vista

    USER {
        bigint id PK
        string username UK
        string email UK
        string password_hash
        datetime date_joined
    }

    PLACE_CATEGORY {
        bigint id PK
        string slug UK "culture | park | food"
        string name
        string icon_name
        bool is_active
    }

    PLACE {
        bigint id PK
        string slug UK
        bigint category_id FK
        string name
        string source_ref UK "stop_id publico"
        string osm_type "node | way | relation"
        bigint osm_id
        string wikidata_id
        string wikipedia_title
        point location "PostGIS Point (lng,lat)"
        string address
        text summary
        string opening_hours
        string website
        string source_url
        string detail_status "pending|complete|unavailable|error"
        datetime details_fetched_at
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

    USER_PLACE_STATE {
        bigint id PK
        bigint user_id FK
        bigint place_id FK
        bool is_visited
        datetime visited_at
        datetime first_seen_at
        datetime last_seen_at
        int seen_count
        bigint last_seen_route_id FK "nullable"
    }

    ROUTE_SEARCH_CACHE {
        bigint id PK
        string cache_key UK
        json canonical_payload
        string origin_query
        string destination_query
        json route_payload
        json map_payload
        int hit_count
        datetime created_at
        datetime updated_at
    }

    TOUR_ROUTE {
        bigint id PK
        bigint user_id FK
        bigint search_cache_id FK
        string origin_label
        string destination_label
        point origin_location
        point destination_location
        string mode "tour | direct_fallback"
        int distance_m
        int duration_s
        int direct_distance_m
        int direct_duration_s
        linestring route_geometry
        linestring direct_route_geometry
        datetime created_at
        datetime updated_at
    }

    TOUR_ROUTE_STOP {
        bigint id PK
        bigint route_id FK
        bigint place_id FK
        int display_order
        int waypoint_order "null se nao esta na rota"
        string state "active | visited | excluded"
        string source
        float distance_from_route_m
    }
```

### Resumo das entidades

| Entidade | Papel |
|---|---|
| **User** (Django nativo) | autenticacao JWT; dono de rotas e estados de lugares |
| **PlaceCategory** | taxonomia de POIs: `culture`, `park`, `food` |
| **Place** | fonte unica de verdade de um ponto de interesse; campo PostGIS `location`; enriquecido progressivamente via APIs externas |
| **PlaceImage** | galeria de imagens de um lugar (URL, order) |
| **UserPlaceState** | estado global usuario x lugar: visitado, datas, contagem, ultima rota |
| **RouteSearchCache** | cache de rota calculada por chave canonica (origem + destino); evita recalculo desnecessario via Nominatim/OSRM/Overpass |
| **TourRoute** | snapshot relacional da rota personalizada do usuario; geometria LineString PostGIS |
| **TourRouteStop** | cada POI na rota do usuario com estado `active / visited / excluded` |

### Decisoes de modelagem

- **`source_ref`**: identificador publico do POI (ex: ID OSM). E o `stop_id` no contrato HTTP.
- **`detail_status`**: controla o ciclo de enriquecimento. `pending` -> busca ao abrir; `complete` -> tem dados; `unavailable` -> APIs nao retornaram nada; `error` -> falha de rede.
- **Sem tabela de Ticket**: tickets existem como endpoint mockado sem fluxo de compra real no MVP.
- **Sem modal de transporte**: so caminhada suportada. OSRM opera em modo `foot`.
- **Sem Administrador no app**: lugares populados via `seed_demo` e descoberta automatica pelo Overpass.

---

## Casos de Uso

```mermaid
flowchart LR
    Visitante((Visitante))
    Turista((Turista))

    Turista -.->|estende| Visitante

    Visitante --> UC1[Visualizar mapa com rota e POIs]
    Visitante --> UC2[Filtrar POIs por categoria\nCultura / Parques / Comida]
    Visitante --> UC3[Criar conta]
    Visitante --> UC4[Fazer login]

    Turista --> UC5[Gerar rota turistica\norigem e destino com POIs]
    Turista --> UC6[Abrir detalhe de POI\nimagem, resumo, horarios, website]
    Turista --> UC7[Marcar POI como visitado]
    Turista --> UC8[Desmarcar POI como visitado]
    Turista --> UC9[Excluir POI da rota atual]
    Turista --> UC10[Ver biblioteca pessoal de lugares]
    Turista --> UC11[Ver perfil basico]
    Turista --> UC12[Sair da conta]
```

> Turista **estende** Visitante: herda UC1-UC4 e adiciona acoes que exigem autenticacao.

---

## Diagrama de Classes

```mermaid
classDiagram
    class User {
        +Long id
        +String username
        +String email
        +String passwordHash
        +DateTime dateJoined
        +authenticate(password) bool
    }

    class PlaceCategory {
        +Long id
        +String slug
        +String name
        +String iconName
        +bool isActive
    }

    class Place {
        +Long id
        +String slug
        +String name
        +String sourceRef
        +String osmType
        +Long osmId
        +String wikidataId
        +String wikipediaTitle
        +Point location
        +String address
        +String summary
        +String openingHours
        +String website
        +String sourceUrl
        +String detailStatus
        +DateTime detailsFetchedAt
        +bool isActive
        +primaryImageUrl() String
        +distanceTo(point) float
    }

    class PlaceImage {
        +Long id
        +String url
        +int order
        +String caption
    }

    class UserPlaceState {
        +Long id
        +bool isVisited
        +DateTime visitedAt
        +DateTime firstSeenAt
        +DateTime lastSeenAt
        +int seenCount
        +markVisited() void
        +markUnvisited() void
    }

    class RouteSearchCache {
        +Long id
        +String cacheKey
        +String originQuery
        +String destinationQuery
        +dict routePayload
        +dict mapPayload
        +int hitCount
        +DateTime createdAt
        +bumpHit() void
    }

    class TourRoute {
        +Long id
        +String originLabel
        +String destinationLabel
        +Point originLocation
        +Point destinationLocation
        +String mode
        +int distanceM
        +int durationS
        +LineString routeGeometry
        +DateTime createdAt
    }

    class TourRouteStop {
        +Long id
        +int displayOrder
        +int waypointOrder
        +String state
        +float distanceFromRouteM
    }

    User "1" --> "0..*" TourRoute : cria
    User "1" --> "0..*" UserPlaceState : possui
    PlaceCategory "1" --> "0..*" Place : categoriza
    Place "1" *-- "0..*" PlaceImage : tem
    Place "1" --> "0..*" UserPlaceState : rastreada_em
    Place "1" --> "0..*" TourRouteStop : aparece_em
    RouteSearchCache "1" --> "0..*" TourRoute : origina
    TourRoute "1" *-- "0..*" TourRouteStop : contem
    TourRoute "0..1" --> "0..*" UserPlaceState : ultima_rota_vista
```

---

## Diagrama de Componentes

```mermaid
flowchart TB
    subgraph Frontend["Aplicativo (Expo + React Native)"]
        Screens[Telas\nExploreScreen / PlacesScreen / ProfileScreen]
        Auth[AuthContext\nJWT tokens]
        Services[Services\napi.ts / tourRoutes.ts / auth.ts]
        Map[MapView\nLeaflet via WebView/iframe]
        Screens --> Auth
        Screens --> Services
        Screens --> Map
    end

    subgraph Backend["API Backend (Django 5.1 + DRF)"]
        AuthAPI[accounts\nPOST /api/auth/*\nGET /api/me/]
        PlacesAPI[places\nGET /api/places/*]
        TourRoutesAPI[tour_routes\nPOST /api/tour-routes/\nGET /api/tour-routes/current/\nGET /api/tour-routes/places/\nGET /api/tour-routes/pois/id/\nPATCH .../stops/id/state/\nDELETE .../stops/id/]
        CoreAPI[core\nGET /api/health/]
        TicketsAPI[tickets\nEndpoint mockado]
    end

    DB[(PostgreSQL 16\n+ PostGIS 3.4)]

    subgraph Externos["APIs Externas (gratuitas/open)"]
        Nominatim["Nominatim\nGeocodificacao e extratags OSM"]
        OSRM["OSRM\nRoteamento pedestre"]
        Overpass["Overpass API\nBusca de POIs via OpenStreetMap"]
        Wikidata["Wikidata API\nImagem principal do lugar P18"]
        Wikipedia["Wikipedia REST API\nResumo e imagem fallback\npt e en"]
    end

    Services -->|HTTPS / JSON| AuthAPI
    Services -->|HTTPS / JSON| TourRoutesAPI
    Services -->|HTTPS / JSON| PlacesAPI

    AuthAPI --> DB
    PlacesAPI --> DB
    TourRoutesAPI --> DB
    TourRoutesAPI -->|geocoding| Nominatim
    TourRoutesAPI -->|rota pedestre| OSRM
    TourRoutesAPI -->|POIs OSM| Overpass
    TourRoutesAPI -->|imagem| Wikidata
    TourRoutesAPI -->|resumo e fallback| Wikipedia
```

---

## Diagrama de Implantacao

```mermaid
flowchart TB
    subgraph Turista["Dispositivo do Turista"]
        App["App Expo Web / React Native\nhttp://localhost:8082"]
    end

    subgraph DevMachine["Maquina de Desenvolvimento (Docker Compose)"]
        subgraph DockerNet["Rede Docker Interna"]
            BackendC["Container: backend\nDjango 5.1 + Gunicorn\nporta host 8080"]
            FrontendC["Container: frontend\nExpo Web\nporta host 8082"]
            DBC["Container: db\npostgis/postgis 16-3.4\nporta host 5433"]
            BackendC <--> DBC
        end
    end

    subgraph Externos["Provedores Externos"]
        Nominatim["Nominatim"]
        OSRM["OSRM"]
        Overpass["Overpass API"]
        Wikidata["Wikidata"]
        Wikipedia["Wikipedia"]
    end

    App -->|HTTPS / JSON porta 8080| BackendC
    BackendC --> Nominatim
    BackendC --> OSRM
    BackendC --> Overpass
    BackendC --> Wikidata
    BackendC --> Wikipedia
```

---

## Sequencia — Gerar Rota Turistica

```mermaid
sequenceDiagram
    actor T as Turista
    participant App as ExploreScreen
    participant API as Backend API
    participant Cache as RouteSearchCache
    participant Nom as Nominatim
    participant OSRM as OSRM
    participant OVP as Overpass API
    participant DB as PostgreSQL

    T->>App: Preenche origem e destino, toca "Gerar Rota"
    App->>API: POST /api/tour-routes/ {origin, destination}

    API->>Cache: Busca por cache_key canonico
    alt Cache valido encontrado
        Cache-->>API: route_payload + map_payload
        API->>DB: Upsert Place para cada POI do cache
    else Cache miss
        API->>Nom: Geocodifica origem e destino
        Nom-->>API: coordenadas lat/lng
        API->>OSRM: GET /route/v1/foot/{coords}
        OSRM-->>API: polyline + distance_m + duration_s
        API->>OVP: Busca POIs na bbox da rota
        OVP-->>API: Lista de nos OSM com tags
        API->>DB: Upsert Place para cada POI
        API->>Cache: INSERT RouteSearchCache
    end

    alt Usuario autenticado
        API->>DB: INSERT TourRoute + TourRouteStop + UserPlaceState
        API-->>App: 200 {route, map, saved_route_id}
    else Usuario anonimo
        API-->>App: 200 {route, map}
    end

    App->>App: Renderiza polyline e marcadores no mapa Leaflet
    App-->>T: Exibe rota com paradas e metricas
```

---

## Sequencia — Enriquecimento de Detalhe de POI

```mermaid
sequenceDiagram
    actor T as Turista
    participant App as ExploreScreen
    participant API as Backend API
    participant DB as PostgreSQL
    participant Nom as Nominatim
    participant WD as Wikidata API
    participant WP as Wikipedia API

    T->>App: Toca em marcador do mapa ou item da lista
    App->>API: GET /api/tour-routes/pois/{stop_id}/
    API->>DB: SELECT Place WHERE source_ref = stop_id

    alt Detalhes incompletos
        API->>Nom: Lookup por osm_type/osm_id com extratags=1
        Nom-->>API: endereco, website, horarios, wikidata_id, wikipedia_title

        alt wikidata_id disponivel
            API->>WD: Busca entidade (claims P18, sitelinks)
            WD-->>API: imagem principal, titulo Wikipedia
        end

        alt wikipedia_title ainda ausente
            API->>WP: Busca por nome do lugar (pt, depois en)
            WP-->>API: Titulo do primeiro resultado
        end

        alt wikipedia_title disponivel
            API->>WP: GET /api/rest_v1/page/summary/{title}
            WP-->>API: resumo, imagem, source_url
        end

        API->>DB: UPDATE Place + INSERT PlaceImage
    end

    API-->>App: {name, address, summary, image_url, opening_hours, website}
    App-->>T: Exibe modal com detalhes enriquecidos
```

---

## Sequencia — Marcar como Visitado

```mermaid
sequenceDiagram
    actor T as Turista
    participant App as ExploreScreen
    participant API as Backend API
    participant DB as PostgreSQL
    participant Planner as Route Planner

    T->>App: Toca "Marcar como visitado" no modal
    App->>API: PATCH /api/tour-routes/saved/{route_id}/stops/{stop_id}/state/ {state: visited}

    API->>DB: Verifica TourRoute do usuario
    API->>DB: UPDATE UserPlaceState.is_visited = True
    API->>DB: UPDATE TourRouteStop.state = visited

    API->>Planner: Reconstroi rota sem stops visitados no trajeto ativo
    Planner-->>API: Nova polyline + waypoints
    API->>DB: UPDATE TourRoute (geometria, distancia, duracao)

    API-->>App: 200 {rota atualizada}
    App->>App: Move POI para secao "Ja visitados"
    App->>App: Atualiza marcador no mapa
    App-->>T: Rota atualizada sem o POI no trajeto
```

---

## Diagrama de Atividade — Explorar e Gerar Rota

```mermaid
flowchart TD
    Start([Inicio]) --> OpenApp[Abrir app]
    OpenApp --> AuthCheck{Sessao\nautenticada?}

    AuthCheck -->|Sim| TryCurrentRoute[GET /api/tour-routes/current/]
    TryCurrentRoute --> RouteFound{Rota salva\nencontrada?}
    RouteFound -->|200 OK| ShowRoute[Exibe rota salva]
    RouteFound -->|404| GenDefault[Gera rota padrao\nPOST /api/tour-routes/]

    AuthCheck -->|Nao| GenAnon[Gera rota anonima\nPOST /api/tour-routes/]

    ShowRoute --> RenderMap
    GenDefault --> RenderMap
    GenAnon --> RenderMap[Renderiza mapa Leaflet\npolyline + marcadores]

    RenderMap --> Idle{Aguarda\ninteracao}

    Idle -->|Filtro categoria| Filter[Atualiza visibilidade\ndos marcadores]
    Filter --> Idle

    Idle -->|Toca marcador ou item| OpenDetail[GET /api/tour-routes/pois/stop_id/]
    OpenDetail --> Modal[Exibe TourPoiDetailModal]

    Modal -->|Marcar visitado| PatchVisited[PATCH .../state/ visited]
    PatchVisited --> UpdateRoute[Atualiza rota e mapa]
    UpdateRoute --> CloseModal

    Modal -->|Excluir da rota| DeleteStop[DELETE .../stops/stop_id/]
    DeleteStop --> RebuildRoute[Reconstroi rota]
    RebuildRoute --> CloseModal

    Modal -->|Fechar| CloseModal[Fecha modal]
    CloseModal --> Idle

    Idle -->|Recalcular rota| NewRoute[POST /api/tour-routes/\nnova origem e destino]
    NewRoute --> RenderMap
```

---

## Estados — Stop na Rota

```mermaid
stateDiagram-v2
    [*] --> active : POI adicionado a rota
    active --> visited : usuario marca como visitado
    visited --> active : usuario desmarca
    active --> excluded : usuario remove da rota
    excluded --> [*]
```
