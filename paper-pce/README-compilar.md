# Como compilar o paper LaTeX

## Opcao 1: Overleaf (recomendado para colaboracao)

1. Acesse [overleaf.com](https://overleaf.com) e crie um projeto vazio
2. Faca upload de todos os arquivos da pasta `paper-pce/`:
   - `main.tex`
   - `referencias.bib`
   - Pasta `figuras/` com os PNGs dos diagramas
3. No Overleaf, va em **Menu > Compiler** e selecione **pdfLaTeX**
4. Va em **Menu > Bibliography** e selecione **Biber**
5. Compile com o botao verde

## Opcao 2: Compilacao local (pdflatex + biber)

### Pre-requisitos

- TeX Live (Linux/Mac) ou MiKTeX (Windows) com pacotes completos
- Biber (geralmente incluido no TeX Live)
- Mermaid CLI para gerar as figuras (ver secao abaixo)

### Gerar as figuras dos diagramas

As figuras ficam em `figuras/*.png` e sao geradas a partir dos `.mmd` em `../diagramas/mermaid/`.

**Com Mermaid CLI instalado** (`npm install -g @mermaid-js/mermaid-cli`):

```bash
cd ../diagramas/mermaid

mmdc -i er.mmd                    -o ../paper-pce/figuras/er.png                    -w 1400
mmdc -i casos-de-uso.mmd          -o ../paper-pce/figuras/casos-de-uso.png          -w 1200
mmdc -i classes.mmd               -o ../paper-pce/figuras/classes.png               -w 1400
mmdc -i componentes.mmd           -o ../paper-pce/figuras/componentes.png           -w 1200
mmdc -i implantacao.mmd           -o ../paper-pce/figuras/implantacao.png           -w 1200
mmdc -i sequencia-gerar-rota.mmd  -o ../paper-pce/figuras/sequencia-gerar-rota.png  -w 1400
mmdc -i sequencia-detalhe-poi.mmd -o ../paper-pce/figuras/sequencia-detalhe-poi.png -w 1400
mmdc -i sequencia-marcar-visitado.mmd -o ../paper-pce/figuras/sequencia-marcar-visitado.png -w 1200
mmdc -i atividade-explorar.mmd    -o ../paper-pce/figuras/atividade-explorar.png    -w 1200
```

**Alternativa online:** cole o conteudo de cada `.mmd` no [Mermaid Live Editor](https://mermaid.live) e exporte como PNG para a pasta `figuras/`.

### Compilar o PDF

```bash
cd paper-pce
pdflatex main.tex
biber main
pdflatex main.tex
pdflatex main.tex
```

O arquivo `main.pdf` sera gerado na pasta `paper-pce/`.

### Script de build completo (bash/WSL)

```bash
#!/usr/bin/env bash
set -e
BASE="$(cd "$(dirname "$0")/.." && pwd)"
MMD="$BASE/diagramas/mermaid"
FIG="$BASE/paper-pce/figuras"
mkdir -p "$FIG"

for f in er casos-de-uso classes componentes implantacao \
          sequencia-gerar-rota sequencia-detalhe-poi \
          sequencia-marcar-visitado atividade-explorar; do
  mmdc -i "$MMD/$f.mmd" -o "$FIG/$f.png" -w 1400
done

cd "$BASE/paper-pce"
pdflatex main.tex
biber main
pdflatex main.tex
pdflatex main.tex
echo "PDF gerado em paper-pce/main.pdf"
```

## Pacotes LaTeX necessarios

O `main.tex` usa os seguintes pacotes (todos incluidos no TeX Live Full):

- `inputenc`, `fontenc`, `babel` -- codificacao e idioma
- `geometry` -- margens ABNT
- `times` -- fonte Times New Roman
- `setspace` -- espacamento 1.5
- `graphicx`, `float` -- figuras
- `booktabs`, `longtable`, `array` -- tabelas
- `listings`, `xcolor` -- blocos de codigo
- `hyperref` -- links internos
- `biblatex` com estilo `abnt` -- referencias ABNT
- `indentfirst` -- identacao do primeiro paragrafo

## Notas

- O estilo `abnt` do biblatex requer o pacote `abntex2`. Se nao disponivel no seu TeX Live,
  substitua `style=abnt, citestyle=abnt` por `style=authoryear, citestyle=authoryear`
  no preambulo do `main.tex`.
- A proposta original do projeto esta em `proposta-original.pdf` para referencia historica.
