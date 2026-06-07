# explora-plus-docs

Repositorio de documentacao, modelagem e artefatos academicos do projeto **Explora+**.

Este repositorio nao contem a aplicacao em si. Ele concentra:

- documentacao funcional e tecnica;
- modelagem do dominio e dos fluxos do sistema;
- diagramas fonte em Mermaid;
- exportacoes usadas nos papers;
- papers academicos em LaTeX;
- artefatos auxiliares de entrega e alinhamento do MVP.

Hoje, este repo e o lugar certo para entender:

- como o projeto esta organizado;
- como backend e frontend se encaixam;
- como o dominio de `places`, `tour_routes` e preferencias do usuario foi modelado;
- como preparar uma entrega academica ou tecnica do Explora+.

---

## Sumario

- [Como este repo se encaixa no projeto](#como-este-repo-se-encaixa-no-projeto)
- [Estrutura atual](#estrutura-atual)
- [O que existe em cada arquivo principal](#o-que-existe-em-cada-arquivo-principal)
- [Ordem recomendada de leitura](#ordem-recomendada-de-leitura)
- [Diagramas e fontes Mermaid](#diagramas-e-fontes-mermaid)
- [Papers academicos](#papers-academicos)
- [Como regenerar os diagramas PNG](#como-regenerar-os-diagramas-png)
- [Como compilar os papers](#como-compilar-os-papers)
- [Relacao com os outros repositorios](#relacao-com-os-outros-repositorios)
- [Convencoes deste repositorio](#convencoes-deste-repositorio)

---

## Como este repo se encaixa no projeto

O Explora+ esta dividido em tres repositorios irmaos:

- `explora-plus-backend`
- `explora-plus-frontend`
- `explora-plus-docs`

Os remotos publicos atualmente usados no workspace sao:

- `https://github.com/EXPLORA-PLUS/explora-plus-backend.git`
- `https://github.com/EXPLORA-PLUS/explora-plus-frontend.git`
- `https://github.com/EXPLORA-PLUS/explora-plus-docs.git`

Papel de cada um:

- `explora-plus-backend`: API Django, autenticacao, persistencia canonica de lugares, rota turistica, preferencias do usuario e biblioteca pessoal.
- `explora-plus-frontend`: app Expo/React Native Web com telas `Explore`, `Places`, `Profile` e configuracoes de busca.
- `explora-plus-docs`: explicacao do sistema, modelagem, setup, fluxos, diagramas e material academico.

Em outras palavras:

- se a pergunta for "como roda?", a resposta geralmente comeca em `SETUP.md`;
- se a pergunta for "como foi modelado?", a resposta geralmente comeca em `MODELAGEM.md`;
- se a pergunta for "qual e o material de entrega?", a resposta geralmente esta em `paper-pce/` ou `paper-pcg/`.

---

## Estrutura atual

Arvore principal do repositorio no estado atual:

```text
explora-plus-docs/
├── README.md
├── MODELAGEM.md
├── SETUP.md
├── AUDITORIA-BOTOES-MVP-2026-06-01.md
├── fluxograma-projeto.html
├── fluxograma-projeto.pdf
├── diagramas/
│   └── mermaid/
│       ├── atividade-explorar.mmd
│       ├── casos-de-uso.mmd
│       ├── classes.mmd
│       ├── componentes.mmd
│       ├── er.mmd
│       ├── implantacao.mmd
│       ├── sequencia-detalhe-poi.mmd
│       ├── sequencia-gerar-rota.mmd
│       └── sequencia-marcar-visitado.mmd
├── paper-pce/
│   ├── main.tex
│   ├── main.pdf
│   ├── referencias.bib
│   ├── README-compilar.md
│   ├── proposta-original.pdf
│   └── figuras/
└── paper-pcg/
    ├── main.tex
    ├── main.pdf
    ├── referencias.bib
    ├── README-compilar.md
    ├── entrega1-pcg.pdf
    └── figuras/
```

Observacoes importantes sobre essa estrutura:

- a pasta `paper/` antiga nao e mais a referencia principal neste repo;
- hoje os artefatos academicos estao separados em `paper-pce/` e `paper-pcg/`;
- os arquivos `.mmd` em `diagramas/mermaid/` sao a fonte editavel dos principais diagramas;
- as imagens PNG usadas nos papers sao exportacoes desses Mermaid;
- `fluxograma-projeto.html` e um documento operacional mais visual, complementar a `MODELAGEM.md`.

---

## O que existe em cada arquivo principal

### `MODELAGEM.md`

Documento central de modelagem do sistema.

Ele descreve:

- diagrama ER;
- casos de uso;
- diagrama de classes;
- diagrama de componentes;
- diagrama de implantacao;
- sequencias principais do planner;
- atividade da tela `Explore`;
- estados de `TourRouteStop`;
- configuracoes de busca por usuario.

Use este arquivo quando voce quiser entender:

- quais entidades existem;
- como elas se relacionam;
- como o planner funciona do ponto de vista conceitual;
- como o fluxo de visitado, excluido e preferencias foi pensado.

### `SETUP.md`

Guia operacional do projeto.

Ele cobre:

- repositorios necessarios;
- uso de Docker;
- `.env` de backend e frontend;
- portas locais;
- migrations;
- seeds;
- health check;
- testes;
- estrutura Django ativa;
- endpoints mais importantes;
- fluxo de preferencias de busca por usuario.

Use este arquivo quando voce quiser:

- subir o stack local;
- lembrar portas e servicos;
- conferir como rodar backend, frontend e banco;
- verificar quais endpoints estao ativos.

### `AUDITORIA-BOTOES-MVP-2026-06-01.md`

Registro de auditoria funcional do MVP de interface.

Ele foi criado para mapear:

- botoes existentes na UI;
- o que ja estava conectado;
- o que era essencial no MVP;
- o que devia ser ocultado ou adiado.

Nao e o documento central da arquitetura, mas ajuda a explicar decisoes de escopo e corte do MVP.

### `fluxograma-projeto.html`

Documento visual em HTML com uma visao operacional do sistema.

Ele resume:

- repositorios e responsabilidades;
- fluxo macro do sistema;
- fluxo da API tradicional;
- fluxo da API `tour_routes`;
- pontos de desalinhamento;
- proximos passos naturais.

O PDF gerado a partir dele (`fluxograma-projeto.pdf`) serve para compartilhar a mesma visao fora do navegador.

---

## Ordem recomendada de leitura

Se alguem esta chegando agora no projeto, a ordem mais util costuma ser:

1. `README.md` deste repo
2. `SETUP.md`
3. `MODELAGEM.md`
4. `fluxograma-projeto.html`
5. `paper-pce/main.pdf` ou `paper-pcg/main.pdf`, dependendo da entrega

Se a pessoa quer entender o produto antes da infraestrutura:

1. `fluxograma-projeto.html`
2. `MODELAGEM.md`
3. `SETUP.md`

Se a pessoa quer so compilar o material academico:

1. `paper-pce/README-compilar.md` ou `paper-pcg/README-compilar.md`
2. `diagramas/mermaid/`
3. `paper-*/main.tex`

---

## Diagramas e fontes Mermaid

Os diagramas editaveis ficam em:

`diagramas/mermaid/`

Arquivos fonte atuais:

- `atividade-explorar.mmd`
- `casos-de-uso.mmd`
- `classes.mmd`
- `componentes.mmd`
- `er.mmd`
- `implantacao.mmd`
- `sequencia-detalhe-poi.mmd`
- `sequencia-gerar-rota.mmd`
- `sequencia-marcar-visitado.mmd`

Esses arquivos sao a fonte de verdade para:

- documentacao Markdown;
- exportacoes PNG usadas nos papers;
- alinhamento entre codigo, modelagem e apresentacao academica.

Regra pratica de manutencao:

1. edite primeiro o `.mmd`;
2. atualize o trecho correspondente em `MODELAGEM.md`, se necessario;
3. regenere os PNGs usados nos papers;
4. so depois revise `main.tex` e PDFs finais.

Isso evita drift entre:

- o diagrama vivo;
- a documentacao tecnica;
- a imagem estavel incluida no paper.

---

## Papers academicos

Hoje existem duas frentes principais de material academico neste repo.

### `paper-pce/`

Material associado ao **Projeto de Conclusao / entrega de prototipo**.

Conteudo principal:

- `main.tex`: fonte LaTeX do documento;
- `main.pdf`: PDF compilado;
- `referencias.bib`: base bibliografica;
- `figuras/`: PNGs usados no paper;
- `proposta-original.pdf`: referencia historica;
- `README-compilar.md`: instrucoes de compilacao.

### `paper-pcg/`

Material associado a **Pesquisa Curricularizada da Graduacao**.

Conteudo principal:

- `main.tex`: fonte LaTeX do artigo;
- `main.pdf`: PDF compilado;
- `referencias.bib`: base bibliografica;
- `figuras/`: PNGs usados no paper;
- `entrega1-pcg.pdf`: artefato de entrega anterior;
- `README-compilar.md`: instrucoes de compilacao.

### Diferenca pratica entre os dois

- `paper-pce/` foca mais na entrega do prototipo e documentacao principal do sistema.
- `paper-pcg/` foca mais no detalhamento da inovacao, impacto e organizacao de continuidade academica.

Ambos compartilham a mesma base visual de diagramas, por isso os `.mmd` e a regeneracao das figuras importam para os dois.

---

## Como regenerar os diagramas PNG

Os papers usam imagens PNG dentro de `paper-pce/figuras/` e `paper-pcg/figuras/`.

Essas imagens devem ser derivadas dos Mermaid em `diagramas/mermaid/`.

### Pre-requisito

Ter Node.js e `npx` disponiveis.

Exemplo de verificacao:

```powershell
node --version
npx -y @mermaid-js/mermaid-cli --version
```

### Gerando um diagrama individual

Exemplo para o ER em `paper-pce`:

```powershell
npx -y @mermaid-js/mermaid-cli `
  -i .\diagramas\mermaid\er.mmd `
  -o .\paper-pce\figuras\er.png `
  -t neutral `
  -b transparent `
  -s 2
```

### Gerando todos para os dois papers

Exemplo em PowerShell:

```powershell
$base = "C:\Users\lucas\Documents\Projects\academic\PCE\explora-plus-docs"
$src  = Join-Path $base "diagramas\\mermaid"
$names = @(
  "er",
  "casos-de-uso",
  "classes",
  "componentes",
  "implantacao",
  "sequencia-gerar-rota",
  "sequencia-detalhe-poi",
  "sequencia-marcar-visitado",
  "atividade-explorar"
)

foreach ($paper in @("paper-pce", "paper-pcg")) {
  $fig = Join-Path $base "$paper\\figuras"
  foreach ($name in $names) {
    npx -y @mermaid-js/mermaid-cli `
      -i (Join-Path $src "$name.mmd") `
      -o (Join-Path $fig "$name.png") `
      -t neutral `
      -b transparent `
      -s 2
  }
}
```

Quando um diagrama for alterado, o ideal e regenerar os dois conjuntos de figuras, mesmo que voce esteja trabalhando em apenas um dos papers.

---

## Como compilar os papers

Cada paper tem seu proprio `README-compilar.md`, mas o fluxo geral e este.

### Pre-requisitos

- MiKTeX ou TeX Live com os pacotes necessarios;
- `biber`;
- figuras PNG ja atualizadas;
- Mermaid CLI se voce precisar regenerar as figuras.

### Compilando `paper-pce`

```powershell
Set-Location C:\Users\lucas\Documents\Projects\academic\PCE\explora-plus-docs\paper-pce
pdflatex main.tex
biber main
pdflatex main.tex
pdflatex main.tex
```

Saida esperada:

- `paper-pce/main.pdf`

### Compilando `paper-pcg`

```powershell
Set-Location C:\Users\lucas\Documents\Projects\academic\PCE\explora-plus-docs\paper-pcg
pdflatex main.tex
biber main
pdflatex main.tex
pdflatex main.tex
```

Saida esperada:

- `paper-pcg/main.pdf`

### Observacao importante

Os `README-compilar.md` internos ainda usam algumas referencias textuais genericas como `paper/` nos exemplos herdados de uma estrutura anterior. O caminho real atual deste repo e:

- `paper-pce/`
- `paper-pcg/`

Entao, ao executar comandos locais, adapte sempre para essas duas pastas reais.

---

## Relacao com os outros repositorios

Este repo deve andar sincronizado com:

- `explora-plus-backend`
- `explora-plus-frontend`

Sempre que houver mudanca estrutural relevante em:

- entidades Django;
- endpoints ativos;
- estados de rota;
- preferencias de busca;
- fluxo de autenticacao;
- telas principais do frontend;

o minimo esperado e revisar:

- `MODELAGEM.md`
- `SETUP.md`
- diagramas em `diagramas/mermaid/`
- paper(s) impactado(s)

Exemplos de mudancas que normalmente exigem atualizacao aqui:

- novo endpoint REST;
- troca de nome de entidade ou coluna;
- nova tela no frontend;
- nova relacao entre `Place`, `TourRoute`, `TourRouteStop` e `UserPlaceState`;
- alteracao de preferencias do planner;
- mudanca de porta, stack Docker ou instrucoes de setup.

---

## Convencoes deste repositorio

### 1. Fonte de verdade

Quando existir a mesma ideia em mais de um lugar:

- a fonte editavel deve ser preferida ao artefato derivado;
- Mermaid e preferivel ao PNG;
- Markdown e preferivel ao PDF;
- `main.tex` e preferivel ao `main.pdf`.

### 2. Documentacao viva

Este repo nao e so arquivo de entrega. Ele tambem funciona como documentacao viva de engenharia.

Por isso, ao atualizar o sistema:

- nao trate o docs como algo separado do software;
- revise diagramas, setup e explicacoes no mesmo ciclo da mudanca, quando possivel.

### 3. Nomes e escopo

Os documentos devem refletir o estado real do projeto:

- nomes canonicos atuais (`PlaceCategory`, `UserPlaceState`, `RouteSearchCache`, `UserRouteSearchPreference`, etc.);
- nomes reais das pastas (`paper-pce`, `paper-pcg`);
- portas e fluxos atuais do ambiente local.

### 4. Artefatos gerados

Arquivos como:

- `main.pdf`
- `fluxograma-projeto.pdf`
- PNGs em `figuras/`

sao artefatos derivados. Ao mexer neles, sempre considere se a fonte original tambem precisa ser atualizada.

---

## Leitura rapida por objetivo

Se o seu objetivo e:

- **subir o projeto localmente**: comece em `SETUP.md`
- **entender entidades e relacionamentos**: comece em `MODELAGEM.md`
- **revisar a visao operacional do MVP**: abra `fluxograma-projeto.html`
- **atualizar diagramas**: entre em `diagramas/mermaid/`
- **gerar material academico**: entre em `paper-pce/` ou `paper-pcg/`

---

## Estado esperado deste repo

No estado ideal, este repositorio deve responder com clareza:

- o que o Explora+ faz;
- como ele foi modelado;
- como ele sobe localmente;
- como os diagramas foram produzidos;
- como recompilar os papers;
- e como alinhar documentacao com backend e frontend sem ambiguidade.

Se alguma dessas respostas nao estiver clara, provavelmente este e o lugar certo para melhorar.
