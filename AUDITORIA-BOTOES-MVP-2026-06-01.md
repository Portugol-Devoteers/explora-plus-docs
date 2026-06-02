# Auditoria de Botões e Ações do MVP

Data de referência: 1 de junho de 2026

## Legenda

- `Conectado`: já aciona algo real e útil hoje
- `Parcial`: tem ação local ou navegação, mas ainda não entrega a funcionalidade final
- `Estático`: aparece como botão, mas não aciona nada útil

## Resumo executivo

Hoje o app tem três blocos em estados bem diferentes:

| Bloco | Estado atual | Observação |
|---|---|---|
| Auth | Bom | Login, cadastro, `me` e logout já têm backend real |
| Places / Explorar | Bom | Lista e detalhe de lugares já consomem `places` do backend |
| Rotas / Ingressos / Perfil expandido | Incompleto | Existem botões e telas, mas grande parte ainda está estática ou ligada a endpoints placeholder |

## Backend real disponível hoje

### Já existe e responde

| Estrutura | Endpoints | Uso natural no frontend |
|---|---|---|
| `accounts` | `POST /api/auth/register/`, `POST /api/auth/login/`, `POST /api/auth/refresh/`, `GET /api/me/` | login, cadastro, perfil, logout |
| `places` | `GET /api/places/`, `GET /api/places/<slug>/` | explorar lugares, detalhe do lugar |
| `tour_routes` | `POST /api/tour-routes/` | gerar rota turística real com mapa e POIs |

### Estrutura pronta no banco, mas API ainda placeholder

| Estrutura | Endpoint atual | Situação |
|---|---|---|
| `routes.Route` | `GET/POST /api/routes/` | modelo existe, view ainda devolve `[]` e `{}` |
| `tickets.Ticket` | `GET/POST /api/tickets/` | modelo existe, view ainda devolve `[]` e `{}` |

## Auditoria por tela

### Navegação global

| Tela / botão | Estado | Hoje faz o quê | Deveria conectar em | Decisão MVP |
|---|---|---|---|---|
| Tab `Explorar` | `Conectado` | abre a tela principal | `places` | manter |
| Tab `Minhas Rotas` | `Parcial` | abre a tela, mas a lista vem de `/api/routes/` placeholder | `routes.Route` e possivelmente integração com `tour_routes` | ocultar se não implementarmos persistência hoje |
| Tab `Ingressos` | `Parcial` | abre a tela, mas a lista vem de `/api/tickets/` placeholder | `tickets.Ticket` | ocultar se não implementarmos compra mockada hoje |
| Tab `Perfil` | `Parcial` | abre a tela e carrega `/api/me/`, mas vários botões internos estão estáticos | `accounts` | manter simplificado |

### Tela de Login

Arquivo: `explora-plus-frontend/src/screens/LoginScreen.tsx`

| Botão | Estado | Hoje faz o quê | Backend / estrutura alvo | Decisão MVP |
|---|---|---|---|---|
| `Entrar` | `Conectado` | chama `signIn`, que faz `POST /api/auth/login/` | `accounts` | manter |
| `Criar conta` | `Conectado` | navega para cadastro | fluxo auth | manter |
| Ícone `olho` da senha | `Conectado` | alterna visibilidade da senha localmente | frontend local | manter |

### Tela de Cadastro

Arquivo: `explora-plus-frontend/src/screens/RegisterScreen.tsx`

| Botão | Estado | Hoje faz o quê | Backend / estrutura alvo | Decisão MVP |
|---|---|---|---|---|
| `Voltar` | `Conectado` | volta para login | navegação local | manter |
| `Criar conta` | `Conectado` | chama `signUp`, que faz `POST /api/auth/register/` | `accounts` | manter |
| `Entrar` | `Conectado` | volta para login | fluxo auth | manter |
| Ícone `olho` da senha | `Conectado` | alterna visibilidade da senha localmente | frontend local | manter |

### Tela Explorar

Arquivo: `explora-plus-frontend/src/screens/ExploreScreen.tsx`

| Botão | Estado | Hoje faz o quê | Backend / estrutura alvo | Decisão MVP |
|---|---|---|---|---|
| Chip `Todos` | `Conectado` | filtra localmente a lista carregada | `places` já carregado | manter |
| Chip `Monumentos` | `Conectado` | filtra localmente por `kind` | `places` já carregado | manter |
| Chip `Eventos` | `Conectado` | filtra localmente por `kind` | `places` já carregado | manter |
| Chip `Transporte` | `Conectado` | filtra localmente por `kind` | `places` já carregado | manter |
| Marcadores do mapa | `Conectado` | abre detalhe do lugar | `places/<slug>` | manter |
| Botão `microfone` | `Estático` | não chama nada | não existe backend de busca por voz | ocultar no MVP |

Observação:

- O campo de busca existe visualmente, mas hoje não filtra `query` em nada. Não é botão, mas é uma interação incompleta que também merece ajuste ou simplificação.

### Tela Detalhe do Lugar

Arquivo: `explora-plus-frontend/src/screens/PlaceDetailScreen.tsx`

| Botão | Estado | Hoje faz o quê | Backend / estrutura alvo | Decisão MVP |
|---|---|---|---|---|
| `Voltar` | `Conectado` | retorna para a tela anterior | navegação local | manter |
| `Gerar Rota` | `Parcial` | navega para a tela de rota, mas sem rota real | `POST /api/tour-routes/` | essencial desenvolver hoje |
| `Comprar Ingressos` | `Estático` | não chama nada | `POST /api/tickets/` | ocultar no MVP se ticket não entrar hoje |
| Seta esquerda do carrossel | `Estático` | parece clicável, mas não troca imagem | controle do `FlatList` | ocultar ou implementar |
| Seta direita do carrossel | `Estático` | parece clicável, mas não troca imagem | controle do `FlatList` | ocultar ou implementar |

Comentário importante:

- `Gerar Rota` é o botão mais importante do produto neste momento.
- O backend certo para ele hoje é `tour_routes`, não `/api/routes/`.

### Tela de Rota

Arquivo: `explora-plus-frontend/src/screens/RouteScreen.tsx`

| Botão | Estado | Hoje faz o quê | Backend / estrutura alvo | Decisão MVP |
|---|---|---|---|---|
| `Voltar` | `Conectado` | retorna à tela anterior | navegação local | manter |
| Card `Ônibus / Metrô` | `Parcial` | só troca seleção visual local | não existe backend correspondente hoje | ocultar no MVP |
| Card `Carro de App` | `Parcial` | só troca seleção visual local | não existe backend correspondente hoje | ocultar no MVP |
| Card `A pé` | `Parcial` | só troca seleção visual local | `tour_routes` já trabalha com caminhada | manter e conectar |
| `Iniciar Navegação` | `Estático` | não chama nada | poderia abrir rota externa ou recalcular | ocultar no MVP por enquanto |

Comentário importante:

- Esta tela hoje não usa a API nova.
- A polyline atual é só `[origem, destino]`, sem rota real.
- O núcleo da entrega de hoje é ligar essa tela ao `POST /api/tour-routes/`.

### Tela Minhas Rotas

Arquivo: `explora-plus-frontend/src/screens/MyRoutesScreen.tsx`

| Botão | Estado | Hoje faz o quê | Backend / estrutura alvo | Decisão MVP |
|---|---|---|---|---|
| Chip `Todas` | `Parcial` | filtra o array local carregado | `GET /api/routes/` | só manter se a lista existir |
| Chip `Salvas` | `Parcial` | filtra o array local carregado | `GET /api/routes/` | só manter se a lista existir |
| Chip `Esta semana` | `Parcial` | filtra o array local carregado | `GET /api/routes/` | só manter se a lista existir |
| Card da rota | `Parcial` | abre a tela de rota do destino salvo | `routes.Route` + detalhe do destino | só manter se houver histórico real |

Comentário importante:

- A tela está bem desenhada, mas hoje depende de um endpoint placeholder.
- Para o MVP de hoje, existem duas opções honestas:
  - implementar persistência mínima em `routes`;
  - ou ocultar a aba inteira.

### Tela Ingressos

Arquivo: `explora-plus-frontend/src/screens/TicketsScreen.tsx`

| Botão | Estado | Hoje faz o quê | Backend / estrutura alvo | Decisão MVP |
|---|---|---|---|---|
| Tab `Próximos` | `Parcial` | separa o array local por status | `GET /api/tickets/` | só manter se lista real existir |
| Tab `Histórico` | `Parcial` | separa o array local por status | `GET /api/tickets/` | só manter se lista real existir |
| Card do ingresso | `Parcial` | abre o detalhe do lugar | `tickets.Ticket` + `places.Place` | só manter se houver compra/lista real |

Comentário importante:

- A tela faz sentido estruturalmente.
- Mas, sem compra mockada e sem lista real, ela transmite sensação de produto incompleto.

### Tela Perfil

Arquivo: `explora-plus-frontend/src/screens/ProfileScreen.tsx`

| Botão | Estado | Hoje faz o quê | Backend / estrutura alvo | Decisão MVP |
|---|---|---|---|---|
| Botão lápis `Editar perfil` | `Estático` | não chama nada | futura edição de usuário | ocultar no MVP |
| Linha `Editar perfil` | `Estático` | sem `onPress` útil | futura edição de usuário | ocultar no MVP |
| Linha `Idioma` | `Estático` | sem `onPress` útil | não existe estrutura real hoje | ocultar no MVP |
| Linha `Notificações` | `Estático` | sem `onPress` útil | não existe estrutura real hoje | ocultar no MVP |
| Linha `Favoritos` | `Estático` | sem `onPress` útil | favorites não existem no backend | ocultar no MVP |
| Linha `Histórico de buscas` | `Estático` | sem `onPress` útil | não existe estrutura real hoje | ocultar no MVP |
| Linha `Sobre o Explora+` | `Estático` | sem `onPress` útil | poderia virar página estática | ocultar ou substituir por link estático |
| Linha `Termos de uso` | `Estático` | sem `onPress` útil | poderia virar página estática | ocultar ou substituir por link estático |
| Linha `Política de privacidade` | `Estático` | sem `onPress` útil | poderia virar página estática | ocultar ou substituir por link estático |
| `Sair da conta` | `Conectado` | limpa sessão local e volta para auth | `accounts` / contexto de auth | manter |

Comentário importante:

- O topo do perfil já tem valor porque usa `GET /api/me/`.
- Os stats ainda são mockados em zero.
- Para o MVP, faz muito sentido deixar só:
  - dados básicos do usuário
  - botão de sair

## Botões essenciais para desenvolver conexão hoje

Estes são os botões com maior impacto de entrega e melhor custo-benefício:

| Prioridade | Botão / fluxo | O que falta |
|---|---|---|
| Alta | `Gerar Rota` | ligar tela de rota ao `POST /api/tour-routes/` |
| Alta | Tela `Route` | desenhar polyline e POIs reais no mapa |
| Média | `Minhas Rotas` | decidir entre persistir rotas em `routes.Route` ou ocultar a aba |
| Média | `Comprar Ingressos` | decidir entre implementar `tickets` mockado ou ocultar botão/aba |
| Média | Perfil | remover/ocultar opções estáticas para não parecer quebrado |

## Recomendação de corte para o MVP de hoje

### Manter visível

- Login
- Cadastro
- Explorar
- Detalhe do lugar
- Perfil simplificado
- Geração de rota real

### Desenvolver agora se der tempo

- `Gerar Rota` conectado ao `tour_routes`
- tela `Route` consumindo o retorno real
- persistência mínima de rota, se quisermos preservar a aba `Minhas Rotas`

### Ocultar no MVP

- botão de microfone
- `Comprar Ingressos`, se `tickets` não entrar hoje
- aba `Ingressos`, se `tickets` não entrar hoje
- aba `Minhas Rotas`, se rota persistida não entrar hoje
- lápis de editar perfil
- linhas estáticas do menu de perfil
- setas do carrossel da tela de detalhe, enquanto não funcionarem
- `Iniciar Navegação`, enquanto não abrir nada real
- opções `Ônibus / Metrô` e `Carro de App`, enquanto o backend só suportar caminhada

## Decisão técnica sugerida para hoje

Para chegar numa entrega honesta e forte:

1. **Fechar bem auth + places + route turística**
2. **Conectar `Gerar Rota` ao `tour_routes`**
3. **Transformar a tela de rota em demonstração principal do produto**
4. **Ocultar tudo que pareça promessa não cumprida**

Essa combinação entrega um MVP pequeno, mas coerente.
