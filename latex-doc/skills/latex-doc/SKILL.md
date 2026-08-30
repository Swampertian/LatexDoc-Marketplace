---
name: latex-doc
description: >
  Use this skill whenever the user wants to create, write, scaffold, or generate a LaTeX academic document
  for UFT (Universidade Federal do Tocantins) / Ciência da Computação. Trigger on: "gerar documento",
  "criar relatório", "novo trabalho", "escrever TCC", "montar artigo UFT", "gerar .tex", "novo relatório UFT",
  "documento LaTeX UFT", "trabalho de disciplina", "projeto de pesquisa", "relatório técnico UFT",
  "/latex-doc", "latex-doc". Always use this skill when the user mentions UFT documents, uftreport,
  or wants a LaTeX document scaffolded — even if they don't say "latex-doc" explicitly.
version: 1.0.0
---

# latex-doc — UFT/CC LaTeX Document Generator

Scaffolds complete, compilable `.tex` documents using `uftreport.cls` for UFT/CC academic work.

## Assets bundled in this plugin

All files live at `${CLAUDE_PLUGIN_ROOT}/skills/latex-doc/assets/`:

- `uftreport.cls` — the document class
- `logos/logouft.pdf` — official UFT logo
- `referencias_abnt/` — ABNT citation style files (abntex2cite.sty, .bst, .bib)

**Always copy these to the working directory before writing the .tex.**

## Workflow

### Step 1 — Determine output directory

Ask the user where to create the document (or use current working directory as default).
If user gives a relative path, resolve against their project root.
Create the directory if it doesn't exist.

### Step 2 — Collect metadata (use AskUserQuestion for missing fields)

| Field | Options / Notes |
|-------|----------------|
| **Tipo** | `report` (padrão), `techreport`, `projectresearch`, `projectextension`, `manual` |
| **Título** | Livre |
| **Título em inglês** | Opcional (`\foreigntitle`) |
| **Autor(es)** | Nome + Sobrenome separados; múltiplos autores = repetir |
| **Orientador(es)** | Título, Nome, Sobrenome, Grau (ex: Prof., João, Silva, Dr.) — opcional |
| **Departamento** | `CC` (padrão), `LC`, `EC`, `EE` |
| **Disciplina/Turma** | Ex: `Algoritmos II -- 2025.1` — opcional |
| **Data** | Dia, Mês, Ano numérico |
| **Palavras-chave PT** | Lista |
| **Keywords EN** | Lista — opcional |
| **Número TR** | Só para `techreport`; default `000` |

Collect all required fields. Optional fields can be omitted if user doesn't provide them.

### Step 3 — Copy assets

```bash
cp "${CLAUDE_PLUGIN_ROOT}/skills/latex-doc/assets/uftreport.cls" "<output_dir>/"
mkdir -p "<output_dir>/logos"
cp "${CLAUDE_PLUGIN_ROOT}/skills/latex-doc/assets/logos/logouft.pdf" "<output_dir>/logos/"
```

### Step 4 — Generate .tex file

Use the template below. Fill ALL metadata collected. Filename = slugified title (lowercase, hyphens, `.tex`).

#### Complete template

```latex
\documentclass[<tipo>, 12pt]{uftreport}

% ── Metadados ─────────────────────────────────────────────────
\title{<TÍTULO>}
% \foreigntitle{<TÍTULO EM INGLÊS>}   % descomente se fornecido

\author{<NOME>}{<SOBRENOME>}
% Repita \author para coautores

% \advisor{<TÍTULO>}{<NOME>}{<SOBRENOME>}{<GRAU>}  % descomente se fornecido

\department{<CC|LC|EC|EE>}
% \class{<DISCIPLINA -- ANO.SEMESTRE>}   % descomente se fornecido

\date{<DIA>}{<MES>}{<ANO>}

% \TRNumber{<NNN>}   % só para techreport

\keyword{<PALAVRA-CHAVE 1>}
% \keyword{<PALAVRA-CHAVE 2>}

% \foreignkeyword{<KEYWORD 1>}   % descomente se fornecido

% ── Início do documento ───────────────────────────────────────
\begin{document}

\frontmatter
\maketitle
\makefrontpage

% \dedication{Texto da dedicatória.}

% \begin{acknowledgement}
%   Texto dos agradecimentos.
% \end{acknowledgement}

\begin{abstract}
  % Escreva o resumo aqui.
\end{abstract}

% \begin{foreignabstract}
%   Abstract text here.
% \end{foreignabstract}

\tableofcontents
% \listoffigures
% \listoftables

\mainmatter
\ChapterStart{first}{Introdução}

\chapter{Introdução}

Escreva a introdução aqui.

\chapter{Desenvolvimento}

Escreva o desenvolvimento aqui.

\chapter{Conclusão}

Escreva a conclusão aqui.

\backmatter

\begin{thebibliography}{9}
  % \bibitem{ref1} AUTOR, A. \textit{Título}. Editora, Ano.
\end{thebibliography}

\end{document}
```

**Rules for filling the template:**
- Replace ALL `<PLACEHOLDER>` with actual values
- Uncomment optional lines that have data
- Remove comment-only lines for optional fields that were NOT provided (keep it clean)
- For multiple authors: repeat `\author{Nome}{Sobrenome}` lines
- For multiple keywords: repeat `\keyword{}` lines

### Step 5 — Compile

Check if `pdflatex` is available:

```bash
which pdflatex
```

**If available:** compile automatically:

```bash
cd "<output_dir>"
pdflatex -interaction=nonstopmode "<filename>.tex"
pdflatex -interaction=nonstopmode "<filename>.tex"
pdflatex -interaction=nonstopmode "<filename>.tex"
```

Run 3× for correct cross-references and pagination (the class uses `zref`).
If user mentioned bibliography, also run `bibtex <filename>` between passes 1 and 2.
Report the PDF location on success.

**If NOT available:** Output this block exactly:

```
pdflatex não encontrado. Para compilar:

# Ubuntu / Debian
sudo apt install texlive-full

# Arch
sudo pacman -S texlive-most

# macOS (Homebrew)
brew install --cask mactex

# Depois de instalar:
cd <output_dir>
pdflatex -interaction=nonstopmode <filename>.tex
pdflatex -interaction=nonstopmode <filename>.tex
pdflatex -interaction=nonstopmode <filename>.tex
```

### Step 6 — Report result

Tell the user:
- Path to the `.tex` file
- Path to the PDF (if compiled)
- Any compilation warnings/errors that need attention

## Document type reference

| Opção | Uso |
|-------|-----|
| `report` | Trabalho de disciplina, relatório acadêmico (padrão) |
| `techreport` | Relatório técnico numerado (exibe TR na capa) |
| `projectresearch` | PIBIC, TCC, proposta de pesquisa |
| `projectextension` | Projeto de extensão universitária |
| `manual` | Manual técnico (cabeçalho de capítulo maior) |

## Notes

- Class already loads: babel, inputenc, graphicx, xcolor, geometry, setspace, listings, hyperref, tikz, amsmath, booktabs, etc. — don't add them again.
- `\ChapterStart{first}{<título do 1º capítulo>}` must match the first `\chapter{}` title.
- For ABNT citations: copy `${CLAUDE_PLUGIN_ROOT}/skills/latex-doc/assets/referencias_abnt/` to the output directory and use `\usepackage{abntex2cite}` before `\begin{document}`.
- Never redefine `\titleformat{\section}`, `\titleformat{\chapter}`, or `hyperref` — the class owns them.
