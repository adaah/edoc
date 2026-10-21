# EDOC

Biblioteca de templates de documentos e recursos para a escola.

O professor (ou o aluno) escolhe um modelo, preenche só o que muda, confere a versão final na tela, corrige o que precisar direto na página e baixa o PDF. A formatação fica pronta dentro do sistema, como num projeto LaTeX, mas sem precisar escrever código.

Este guia explica como o projeto está organizado, o que cada parte do código faz e **onde mexer** para mudar cada coisa.

---

## Sumário

1. [Visão geral](#1-visão-geral)
2. [Arquivos do projeto](#2-arquivos-do-projeto)
3. [Como rodar e publicar](#3-como-rodar-e-publicar)
4. [Mapa do `index.html`](#4-mapa-do-indexhtml)
5. [Conceitos que aparecem no código todo](#5-conceitos-que-aparecem-no-código-todo)
6. [Parte por parte](#6-parte-por-parte)
7. [Receitas: “quero mudar X”](#7-receitas-quero-mudar-x)
8. [Criar um modelo novo](#8-criar-um-modelo-novo)
9. [Criar a sua própria versão (outra escola)](#9-criar-a-sua-própria-versão-outra-escola)
10. [Limites conhecidos](#10-limites-conhecidos)

---

## 1. Visão geral

São dois sistemas que saem do **mesmo código**:

| Edição | Arquivo | Quem usa | Como abre |
|---|---|---|---|
| **EDOC** (equipe) | `index.html` | Professores e coordenação | Pede o código de acesso; mostra os documentos internos (Prova objetiva) |
| **EDOC - Aluno** | `aluno/index.html` | Estudantes | Abre direto; mostra os documentos para alunos (TCC) |

A diferença entre os dois é uma única linha (`const EDICAO = 'interno'` ou `'aluno'`). Você só edita o `index.html` e gera o do aluno com um comando (veja a seção 3).

O fluxo de qualquer modelo é sempre o mesmo, em 3 etapas:

1. **Preencher**: formulário com os campos variáveis.
2. **Revisar e corrigir**: as páginas A4 aparecem como vão sair; o que é conteúdo do usuário fica com marca-texto e pode ser editado clicando.
3. **Baixar PDF**: só libera depois de marcar “Conferi … e está tudo certo”.

Não há servidor. Tudo roda no navegador, e os rascunhos ficam salvos no próprio navegador (`localStorage`).

---

## 2. Arquivos do projeto

```
edoc/
├── index.html               ← EDOC (equipe). É o código-fonte: edite aqui.
├── aluno/
│   └── index.html           ← EDOC - Aluno. Gerado a partir do index.html. Não edite à mão.
├── ferramentas/
│   ├── gerar_aluno.py       ← gera aluno/index.html a partir do index.html
│   └── novo_codigo.py       ← gera a "impressão digital" de um novo código de acesso
└── README.md
```

Cada `index.html` é **um arquivo só**, com HTML, CSS e JavaScript juntos. As imagens (cabeçalho do colégio e logo do CEAAT) estão embutidas no próprio arquivo, em base64.

Bibliotecas externas carregadas por CDN (no fim do `<body>`):

| Biblioteca | Para quê |
|---|---|
| `html2canvas 1.4.1` | Fotografa cada página A4 para montar o PDF |
| `jsPDF 2.5.1` | Monta o arquivo PDF, os links do sumário e os marcadores |
| `mammoth 1.8.0` | Lê arquivos `.docx` enviados pelo usuário |

Fontes (Google Fonts): **Montserrat** (títulos e botões), **Inter** (textos da interface) e **Arimo** (o documento em si; tem as mesmas medidas do Arial).

---

## 3. Como rodar e publicar

**Rodar no computador:** basta abrir o `index.html` no navegador. Para testar o código de acesso, prefira um endereço `http://localhost` ou `https://`, porque a conferência do código usa a criptografia do navegador (`crypto.subtle`), que alguns navegadores bloqueiam em arquivos abertos direto do disco.

Um jeito simples de servir a pasta:

```bash
cd edoc
python3 -m http.server 8000
# abra http://localhost:8000 (equipe) e http://localhost:8000/aluno/ (aluno)
```

**Depois de editar o `index.html`, gere o do aluno:**

```bash
python3 ferramentas/gerar_aluno.py
```

O script troca a edição para `'aluno'`, muda o título da aba, tira o código de acesso e tira o exemplo de prova de Humanas (para o arquivo ficar menor). Se ele disser que “não encontrou o trecho esperado”, alguma dessas linhas mudou de formato no `index.html`.

**Publicar no GitHub Pages:** suba a pasta inteira para um repositório e ative o Pages (Settings → Pages → branch `main`). Os endereços ficam `https://SEU-USUARIO.github.io/edoc/` (equipe) e `.../edoc/aluno/` (aluno).

---

## 4. Mapa do `index.html`

O arquivo tem cerca de 3.300 linhas. Os números abaixo são **aproximados** e mudam conforme o código cresce; para achar uma parte, use a busca do editor (Ctrl+F) pelo **nome da função** ou pelo **comentário de seção** (as linhas que começam com `/* =====` ou `/* -----`).

| Linhas (aprox.) | O que é | Procure por |
|---|---|---|
| 1–13 | `<head>`: meta, fontes, script que aplica o tema claro/escuro antes de a página aparecer | `edoc.tema` |
| 14–549 | **CSS** (todo o visual) | `<style>` |
| 47–412 | CSS da interface (barra do topo, botões, telas, formulário, revisão, janelas) | `/* ---------- top bar` |
| 413–549 | CSS **do documento** (página A4 da prova e do TCC) | `/* ============ PAPER` |
| 551–585 | HTML fixo: barra do topo, `<main id="app">`, áreas invisíveis de medição e de exportação, barra de formatação N/I/S | `<header class="top">` |
| 586–588 | Bibliotecas por CDN | `html2canvas` |
| 590–596 | Dados embutidos e **edição** do arquivo | `const EDICAO` |
| 597–713 | Utilidades, números por extenso, texto formatado | `/* ===================== utilidades` |
| 714–816 | **Registro de modelos** e dados fixos da prova | `registro de templates` |
| 817–839 | Estado e salvamento | `/* ===================== estado` |
| 840–889 | Regras da prova (nome, caminhos editáveis) | `regras do template` |
| 890–1093 | **Motor de diagramação** da prova (cabeçalho, gabarito, colunas, páginas) | `motor de diagramação` |
| 1094–1156 | Leitura de questões coladas/importadas | `function parseQuestoes` |
| 1157–1230 | Barra de formatação N/I/S e botão Citar | `barra de formatação` |
| 1231–1260 | Navegação entre telas e etapas | `/* ===================== VIEWS` |
| 1261–1422 | **Tela inicial**: escolas, áreas, código de acesso, lista de modelos e rascunhos | `/* ---------- HOME` |
| 1423–1774 | **Formulário da prova** | `/* ---------- FORM` |
| 1775–1953 | Questões: alternativas, **repetidas**, **excluídas**, colar uma questão | `questões: utilidades` |
| 1954–2125 | Janela “Colar todas as questões / Importar .docx” | `function openImport` |
| 2126–2257 | **Revisão** (páginas na tela, edição direta, lista “Vale conferir”) | `/* ---------- REVIEW` |
| 2258–2333 | **Geração do PDF** | `/* ---------- PDF` |
| 2334–3255 | **Modelo TCC** inteiro (dados, importação, referências, paginação ABNT, formulário) | `MODELO: TCC` |
| 3256–3261 | Ajustes por edição (o que cada arquivo mostra) | `if (EDICAO !== 'aluno')` |
| 3262–3292 | Modo claro/escuro, avisos (toast) e início do sistema | `modo claro / escuro` |

---

## 5. Conceitos que aparecem no código todo

### 5.1 O documento (`doc`)

Cada prova ou TCC é um objeto JavaScript salvo no navegador. A variável global `doc` é o documento aberto no momento.

**Prova objetiva:**

```js
{
  id, templateId: 'prova-objetiva-ceaat', v: 3, updatedAt,
  f: {                       // campos do cabeçalho
    area, unidade, profs: [..], curso, turno, serie, turma, data, valor,
    nAlt: 4 | 5 | 'custom',  // alternativas por questão
    colunas: 1 | 2,          // diagramação das questões
    gabarito: 'grade' | 'bolhas',
    custom: { curso: true, … } // quais escolhas estão em "Personalizável"
  },
  inst: ['texto da instrução 1', …],
  sections: [                // disciplinas
    { id, name: 'HISTÓRIA', noMerge: [..], questions: [
        { id, text: 'enunciado', alts: ['A', 'B', …], n: 5, img: { src, w, h } | null }
    ] }
  ]
}
```

**TCC:**

```js
{
  id, templateId: 'tcc-ceaat', v: 'tcc2', updatedAt, refMode: 'campos' | 'texto',
  f: { escola, curso, autores: [..], titulo, subtitulo, natureza, orientador, coorientador, cidade, ano },
  resumo, palavras, abstract, keywords,
  caps: [                    // capítulos, apêndices e anexos
    { id, kind: 'cap' | 'apendice' | 'anexo', titulo, hint, blocos: [
        { t: 'p', text } | { t: 'h2', text } | { t: 'h3', text } | { t: 'list', items: [..] }
      | { t: 'fig', cap, src, w, h, fonte } | { t: 'tab', cap, rows: [[..]], fonte }
    ] }
  ],
  refs: [ { id, tipo: 'livro'|'artigo'|'site'|'outro', autores, titulo, …, livre } ]
}
```

### 5.2 Texto formatado sem código à vista

O usuário vê negrito, itálico e sublinhado normalmente, mas por dentro o texto é guardado com marcas simples:

| Marca | Resultado |
|---|---|
| `**texto**` | **negrito** |
| `*texto*` | *itálico* |
| `++texto++` | sublinhado |
| linha em branco | novo parágrafo |
| `{valor}`, `{questoes}` | número que se atualiza sozinho (aparece como etiqueta azul no formulário) |
| `{cite:ID}`, `{citeonline:ID}` | citação de uma referência do TCC: `(TARDIF, 2014)` ou `Tardif (2014)` |

Funções responsáveis:

- `md(texto, modo)`: transforma as marcas em HTML. Com `modo = 'edit'`, os códigos `{...}` viram etiquetas não editáveis.
- `htmlToMd(elemento)`: o caminho de volta. Lê o que o usuário digitou ou colou (inclusive do Word e do Google Docs) e devolve o texto com marcas.
- `normalizeMd(texto)`: limpa espaços e remove marcas que ficaram sem par.

### 5.3 Caminhos editáveis (`data-path`)

Na tela de revisão, cada trecho editável é um `<span class="v" data-path="...">`. O caminho diz onde aquele texto mora dentro do `doc`:

| Caminho | Significa |
|---|---|
| `f.area` | campo “Avaliação de” |
| `inst.3` | 4ª instrução |
| `sec.0.name` | nome da 1ª disciplina |
| `q.0.2.p.1` | 1ª disciplina, 3ª questão, 2º parágrafo do enunciado |
| `q.0.2.a.4` | mesma questão, alternativa E |
| `c.1.b.3.p.0` | TCC: capítulo 2, bloco 4, 1º parágrafo |
| `ref.2` | TCC: 3ª referência |

`getRaw(caminho)` lê o texto, `setRaw(caminho, valor)` grava e `displayFor(caminho, valor)` mostra. Quando um parágrafo é dividido entre duas colunas ou páginas, cada pedaço ganha também `data-frag="início:fim"` (posição das palavras), e a edição de um pedaço é encaixada de volta no parágrafo inteiro.

No formulário, a mesma ideia usa `data-bind` em vez de `data-path`.

### 5.4 Medir antes de montar

O documento é montado como páginas A4 de verdade (794 × 1123 px, a 96 dpi). Para saber onde cada coisa cabe, o sistema coloca os blocos numa área invisível com a mesma largura da coluna (`#measure`), mede a altura de cada um e só então distribui pelas colunas e páginas. Por isso fonte, margem e espaçamento são definidos em **pixels** no CSS do documento: o que é medido é exatamente o que vai para o PDF.

---

## 6. Parte por parte

### 6.1 CSS da interface

- **Cores e fontes da marca** ficam no começo do `<style>`, em `:root`, como variáveis:
  `--accent` (verde-petróleo `#297971`), `--amber` (âmbar `#F59702`), `--ink` (preto suave `#171717`), fundos e linhas.
- Há **três blocos** de cores: claro (padrão), escuro pela preferência do sistema (`@media (prefers-color-scheme:dark)`) e escuro escolhido no botão (`:root[data-theme="dark"]`). Para mudar uma cor, mude nos três.
- `--display` (Montserrat) e `--ui` (Inter) definem as fontes da interface.

### 6.2 CSS do documento (`/* ============ PAPER`)

Aqui fica a aparência da prova e do TCC impressos:

| Classe | O que controla |
|---|---|
| `.page` | tamanho da folha e margens da prova (`padding: 26px 45px 46px`) |
| `.page .content` | altura útil da prova (`1051px`); deve bater com `CONTENT_H` no JavaScript |
| `.b-h1` … `.b-aluno` | linhas do cabeçalho da prova (título, professor, curso/turno/série/turma, aluno) |
| `.b-inst` | instruções |
| `.gab` | gabarito em grade |
| `.bub-*`, `.bubt` | gabarito de bolinhas |
| `.cols`, `.col.c1` | colunas das questões e a linha vertical entre elas |
| `.b-q0`, `.b-a0`, `.b-alast` | espaços antes/depois do enunciado e das alternativas |
| `.blk` | tamanho da letra das questões (`13px`) e entrelinha |
| `.tcc-page` | margens ABNT do TCC (3 cm em cima e à esquerda, 2 cm embaixo e à direita) |
| `.b-tp p.tp` | parágrafo do TCC: recuo de 1,5 cm (`57px`) e justificado |
| `.b-th1/2/3` | títulos de capítulo, seção e subseção do TCC |
| `.capa`, `.rosto` | capa e folha de rosto |
| `.b-toc` | linhas do sumário |
| `.tcc-page .pn` | posição do número da página do TCC (canto inferior direito) |

As classes `.reviewing .v` e `.vv` criam o **marca-texto** da revisão. Elas só valem dentro da tela de revisão; o PDF sai sem marcação.

### 6.3 Dados embutidos e edição

```js
const EDICAO = 'interno';   // 'interno' = EDOC (equipe) · 'aluno' = EDOC - Aluno
const HEADER_IMG = "data:image/jpeg;base64,…";   // cabeçalho do colégio (prova)
const CEAAT_LOGO = "data:image/png;base64,…";    // logo do CEAAT (capa do TCC e telas)
const EXEMPLO_HUMANAS = {…};                     // 40 questões do exemplo de Humanas
```

Para trocar uma imagem, converta o arquivo para base64 (por exemplo, `base64 -w0 imagem.png` no terminal) e cole no lugar, mantendo o prefixo `data:image/png;base64,`. Se mudar a proporção do cabeçalho, ajuste `HEADER_RATIO` (altura ÷ largura).

### 6.4 Utilidades

- `extenso(n, feminino)`: número por extenso (“quarenta”, “duas”).
- `valorFmt('4,0')`: devolve `4,0 (quatro)`.
- `tokVal(nome, doc)`: valor de cada código automático (`{valor}`, `{questoes}`…). **Para criar um código novo**, acrescente o nome em `TOKENS`, uma descrição em `TOK_TITLE` e o cálculo em `tokVal`.

### 6.5 Registro de modelos (`TEMPLATES`)

Cada modelo é uma entrada do objeto `TEMPLATES`:

```js
TEMPLATES['prova-objetiva-ceaat'] = {
  id, nome, org, resumo,     // textos do cartão na tela inicial
  school: 'ceaat',           // a que escola pertence
  area: 'internos',          // 'internos' (com código) ou 'alunos'
  fixo: [...],               // lista "parte fixa" (texto informativo)
  novo()    { ... },         // documento em branco
  exemplo() { ... }          // documento de exemplo
};
```

Logo acima ficam as **opções fixas do cabeçalho da prova**:

```js
const CURSOS = ['Parcial','Integral','TEC Informática','TEC Redes','TEC Desenvolvimento'];
const TURNOS = [['MAT','MAT','Matutino'], ['VESP','VESP','Vespertino']];   // [valor, como sai na prova, como aparece no formulário]
const SERIES = [['1','1ª','1ª série'], …, ['F','Fluxo 2/3','Fluxo 2/3']];
const TURMAS = ['A','B','C','D'];
```

E as instruções padrão: `INSTRUCOES_MODELO` (as do template oficial) e `INSTRUCOES_HUMANAS` (as do exemplo).

`EM_BREVE` e `EM_BREVE_ALUNOS` são os cartões “Em breve” de cada área.

`migrate(d)` atualiza rascunhos salvos em versões antigas do sistema. Se você mudar a estrutura do `doc`, acrescente aqui a conversão do formato antigo para o novo e aumente o número de versão (`v`).

### 6.6 Estado e salvamento

- Tudo fica em `localStorage`, na chave `dossie.v1`: `{ docs: {id: doc}, trash: [...] }`.
- `touch()` é chamado a cada alteração: marca a data, desfaz a confirmação de “Conferi” e agenda o salvamento (`persist()`, 400 ms depois).
- `trash` é o banco de **questões excluídas** (vale para todas as provas daquele navegador).
- Outras chaves: `edoc.tema` (claro/escuro) e, na sessão (`sessionStorage`), `edoc.unlock.ceaat` (código já digitado).

### 6.7 Motor de diagramação da prova

A prova é montada em três passos:

1. **`buildBlocks(d)`** transforma o `doc` numa lista de blocos:
   - `full`: largura total (cabeçalho via `headerBlocks`, instruções, gabarito);
   - `body`: as questões (título da disciplina, parágrafos do enunciado, imagem, alternativas).
   Cada bloco pode ter `keep: true` (“não deixe sozinho no fim da coluna”) e `para` (parágrafo que pode ser dividido).
2. **`paginateProva(d)`** mede os blocos e distribui:
   - a 1ª página recebe o cabeçalho, as instruções e o gabarito;
   - as questões **sempre começam na página seguinte**;
   - em 1 ou 2 colunas (`d.f.colunas`), enchendo a coluna da esquerda e depois a da direita;
   - parágrafos longos são divididos por palavras quando não cabem (busca binária pela maior parte que cabe).
3. **`pagesHTML(pages)`** gera o HTML final das páginas.

Funções e constantes úteis:

| Item | Para mudar… |
|---|---|
| `headerBlocks(d)` | o desenho do cabeçalho da prova (ordem das linhas, textos “AVALIAÇÃO DE”, “PROFESSOR(A)”, “ALUNO(A)”) |
| `ck(marcado, texto)` | como sai cada opção `( X ) MAT` / `(   ) VESP` |
| `BL(largura)` | as linhas em branco para o aluno escrever |
| `gabaritoGrade(d)` | o gabarito em grade (até 20 questões por tabela, título da disciplina sobre as colunas dela) |
| `gabaritoBolhas(d)` | o gabarito de bolinhas (colunas de 10, título centralizado sobre as colunas de cada disciplina) |
| `CONTENT_H = 1051` | a altura útil da página da prova (mude junto com `.page .content` no CSS) |
| `REGION_GAP` | espaço entre o gabarito e as colunas |
| texto `QUESTÕES 01 A 10` | dentro de `buildBlocks`, no título de cada disciplina |

### 6.8 Leitura de questões (`parseQuestoes`)

Recebe texto (já convertido com `htmlToMd`) e reconhece:

- linhas que começam com `QUESTÃO 01` (com ou sem negrito, ponto, hífen);
- alternativas `A)` a `H)`, inclusive várias na mesma linha (`B) 2 C) 3`);
- nomes de disciplina em MAIÚSCULAS sozinhos na linha (`HISTÓRIA`);
- `PROFESSORES: ...` e `AVALIAÇÃO DE ...`.

Para aceitar outro formato (por exemplo `a.` em vez de `A)`), ajuste as expressões regulares dentro dessa função. A função `isHeader` decide o que é nome de disciplina.

### 6.9 Barra de formatação N / I / S e Citar

- O HTML da barra está no `<body>` (`<div id="rtbar">`).
- Ela aparece sobre qualquer campo com o atributo `data-rich` (função `placeBar`).
- Os botões usam `document.execCommand('bold' | 'italic' | 'underline' | 'removeFormat')`.
- O botão **Citar** só aparece em campos de capítulo do TCC; a lista de referências é montada no clique.
- `richPaste(evento, tipo)`: ao colar, mantém só negrito, itálico e sublinhado e descarta o resto da formatação de Word e sites.

### 6.10 Navegação e etapas

- `setView('home' | 'form' | 'review')` troca a tela.
- `syncSteps()` acende as etapas 1-2-3 do topo.
- `nav = { school, area }` guarda onde o usuário está na tela inicial.

### 6.11 Tela inicial e código de acesso (`/* ---------- HOME`)

- `SCHOOLS`: as escolas (nome, sigla, cidade e logo).
- `AREAS`: nomes e descrições de “Documentos internos” e “Documentos para alunos”.
- `LOCKS`: a **impressão digital** do código de acesso de cada escola (PBKDF2-SHA256, 150 mil rodadas, com sal). O código em si não está no arquivo.
- `checkCode(escola, código)`: confere o código digitado.
- `renderHome()`: decide o que mostrar. Na edição `aluno`, vai direto para a área dos alunos; na `interno`, mostra a tela do código (`renderGate`) ou, se já liberado na sessão, a área interna (`renderArea`).
- `renderArea()`: cartões dos modelos da área e a lista de rascunhos.
- Proteção contra tentativas: 5 erros travam por 30 segundos (`failCount`, `lockUntil`).

**Trocar o código:**

```bash
python3 ferramentas/novo_codigo.py NOVOCODIGO ceaat
```

Copie a linha impressa para dentro de `LOCKS` no `index.html`, substituindo a da escola, e gere de novo o arquivo do aluno.

### 6.12 Formulário da prova (`/* ---------- FORM`)

| Função | O que faz |
|---|---|
| `renderForm()` | monta o formulário inteiro (cabeçalho, instruções, questões, formato) |
| `choiceField(chave, rótulo)` | lista com opção vazia + opções fixas + “Personalizável…” (curso, turno, série, turma, unidade) |
| `CHOICES` | as opções de cada lista do formulário |
| `instListHTML()` | lista de instruções |
| `renderSections()` | abas das disciplinas, link “+ Adicionar questão”, aviso de nomes repetidos, “Mesclar com…” |
| `renderQ(si, qi, nº)` | o cartão de cada questão (Colar uma questão, Mover para…, imagem, subir/descer, excluir, alternativas) |
| `mergeSections(a, b)` | junta duas disciplinas (fica a que vem primeiro) |
| `onFormInput / onFormChange / onFormClick` | reações a digitação, seleção e cliques; cada botão tem um `data-act` tratado num `switch` |
| `schedulePageCount()` | calcula quantas páginas a prova teria em 1 e em 2 colunas |
| `updateHeaderPreview()` | prévia ao vivo do cabeçalho |
| `refreshPills()` | atualiza os números automáticos nas instruções (ex.: ao mudar o valor da prova) |

Para **acrescentar um botão** numa questão: crie o botão em `renderQ` com `data-act="minha-acao" data-si="${si}" data-qi="${qi}"` e trate o caso `'minha-acao'` dentro de `onFormClick`.

### 6.13 Questões: alternativas, repetidas, excluídas e colar uma questão

- `fitAltMode(novas)`: quando entram questões com outra quantidade de alternativas, decide se a prova fica com 4, 5 ou “Personalizável”.
- **Repetidas:** `normQ`, `qSig`, `simInfo`, `bestMatch`, `docDupMap`. A comparação ignora acentos, maiúsculas, pontuação e formatação. `DUP_MIN = 0.9` é o limite para “muito parecida” (90% das palavras em comum). Aumente para ficar mais rigoroso; diminua para pegar variações maiores.
- **Excluídas:** `toTrash`, `openTrash`. O banco guarda até 300 questões (`if (t.length > 300)`).
- **Colar uma questão:** `openQPaste(si, qi)` e `parseOne(texto)`.

### 6.14 Colar todas / importar `.docx` (`openImport`)

- Lê o `.docx` com mammoth, ou o texto colado, e usa `parseQuestoes`.
- Se a prova já tem questões, pergunta **em qual disciplina** colocar e se é para adicionar ou substituir.
- Marca as repetidas (`checkDups`) e, por padrão, não as adiciona.

### 6.15 Revisão (`/* ---------- REVIEW`)

- `issues(d)`: a lista “Vale conferir” da prova (enunciado ou alternativa vazios, questões repetidas, disciplinas com o mesmo nome). **Para criar um aviso novo**, faça um `out.push({ msg:'texto', go:'q:0.3' })`. O `go` diz para onde o clique leva: `q:disciplina.questão`, `form:#id-do-campo`, `sec:índice` ou `cap:índice` (TCC).
- `layoutPages()` e `scalePages()`: desenham as páginas e as encolhem para caber na tela.
- `bindEditing()`: edição direta na página. Ao clicar, o trecho mostra o texto original; ao sair, grava com `setRaw` e remonta as páginas (`maybeRelayout`).

### 6.16 PDF (`exportPDF`)

1. Monta as páginas numa área invisível (`#exportRoot`).
2. Para cada página, `html2canvas` gera uma imagem (`scale: 2.4`, isto é, cerca de 230 dpi) e o jsPDF a coloca numa folha A4.
3. No TCC, lê a posição de cada linha do sumário (`[data-goto]`) e cria os **links clicáveis** e os **marcadores** do PDF.
4. Entrega o arquivo: dentro do claude.ai, pelo recurso de downloads; fora dele, por download normal do navegador; se tudo falhar, abre a impressão (`printFallback`) para “Salvar como PDF”.

Qualidade × tamanho do arquivo: mude `scale: 2.4` (resolução) e `.93` (qualidade do JPEG) em `exportPDF`.

O nome do arquivo vem de `docName(doc)`, passado por `slug()`.

### 6.17 Modelo TCC (`MODELO: TCC`)

Segue a estrutura do projeto LaTeX/abnTeX2 do colégio.

| Parte | Onde mexer |
|---|---|
| Medidas da página (área útil 605 × 934 px) | `T_W`, `T_H` e o CSS `.tcc-page` |
| Textos de “Orientação” | objeto `OR` |
| Capítulos e seções de um TCC novo | `tccEstrutura()` |
| Natureza do trabalho padrão | `NATUREZA_PADRAO` |
| Exemplo SisLab | `exemplo()` dentro de `TEMPLATES[TCC_ID]` |
| Numeração de capítulos, apêndices e anexos | `tccOutline(d)` |
| Corpo do texto (títulos, parágrafos, listas, figuras, tabelas, referências) | `tccBodyBlocks(d)` |
| Paginação de uma sequência de blocos | `tccFlow(blocos, …)` |
| Montagem completa: capa, folha de rosto, resumo, abstract, sumário, corpo | `tccPaginate(d)` |
| Número da página (a partir de qual página aparece) | final de `tccPaginate` (`p.num ? i : 0`) |
| Formulário | `renderTccForm`, `renderCaps`, `renderBlock`, `renderRefs` |
| Avisos da revisão | `tccIssues(d)` |

**Referências e citações:**

- `refMd(r)` monta a referência no formato ABNT a partir dos campos (livro, artigo, site). Para criar um tipo novo (ex.: “capítulo de livro”), acrescente em `REF_TIPOS`, crie os campos em `renderRefs` e o formato em `refMd`.
- `autRef(r)`: autores no formato `SOBRENOME, N. N.` (mais de 3 autores vira `et al.`). Autor escrito **sem vírgula** é tratado como instituição.
- `citeText(r, 'fim' | 'online')`: `(SILVA, 2024)` ou `Silva (2024)`.
- `refMode`: “Por campos” ou “Texto pronto”.

**Importar um TCC pronto:**

- `domToItems` (HTML do Word ou colado) e `plainToItems` (texto simples, como o copiado de PDF) transformam a entrada numa lista de itens.
- `tccParseItems(itens)` reconhece RESUMO, ABSTRACT, palavras-chave, SUMÁRIO (ignorado), capítulos numerados ou com estilo de título, seções, listas, tabelas, figuras com “Figura N –” e “Fonte:”, REFERÊNCIAS, APÊNDICE e ANEXO.
- `CAP_NOMES`: nomes de capítulo reconhecidos mesmo sem número (Introdução, Metodologia, Conclusão…). Acrescente outros aqui.

### 6.18 Ajustes por edição

Perto do fim do script:

```js
if (EDICAO !== 'aluno') delete TEMPLATES[TCC_ID];     // equipe não mostra o TCC
if (EDICAO === 'aluno') { … título da aba, etiqueta "Aluno" … }
```

E, na seção HOME:

```js
if (EDICAO === 'aluno'){ delete TEMPLATES['prova-objetiva-ceaat']; … }  // aluno não mostra a prova
```

Para **mudar qual modelo aparece em qual edição**, mexa nessas linhas e no campo `area` do modelo.

### 6.19 Tema e início

- `applyTheme('light' | 'dark')` e o botão `#themeBtn`. O padrão é claro; a escolha fica em `edoc.tema`.
- O sistema começa em `loadStore()` e `setView('home')`, depois de carregar as fontes (para as medidas das páginas saírem certas).

---

## 7. Receitas: “quero mudar X”

| Quero… | Onde |
|---|---|
| Mudar as cores da marca | `:root` no começo do CSS (os 3 blocos: claro, escuro do sistema, escuro do botão) |
| Trocar a logo do EDOC | SVG dentro de `<button class="brand" id="goHome">` |
| Trocar o cabeçalho do colégio na prova | `HEADER_IMG` (e `HEADER_RATIO`) |
| Trocar a logo da capa do TCC | `CEAAT_LOGO` |
| Mudar o código de acesso | `ferramentas/novo_codigo.py` → `LOCKS` |
| Mudar as instruções padrão da prova | `INSTRUCOES_MODELO` |
| Acrescentar um curso, turno, série ou turma fixos | `CURSOS`, `TURNOS`, `SERIES`, `TURMAS` |
| Mudar a ordem/texto das linhas do cabeçalho da prova | `headerBlocks(d)` |
| Mudar a letra das questões | `.blk { font-size:13px; line-height:1.33 }` |
| Mudar as margens da prova | `.page { padding }` e, se mudar a altura útil, `.page .content` e `CONTENT_H` |
| Fazer as questões começarem na 1ª página | em `paginateProva`, o trecho comentado “as questões sempre começam numa folha nova” |
| Mudar o gabarito de bolinhas (10 por coluna) | `gabaritoBolhas` (`last.items.length < 10`) |
| Mudar o gabarito em grade (20 por tabela) | `gabaritoGrade` (`items.length <= 10 ? 10 : 20`) |
| Mais ou menos rigor nas questões repetidas | `DUP_MIN` |
| Aceitar outro formato de questão ao colar | `parseQuestoes` |
| Mudar o recuo de parágrafo do TCC | `.b-tp p.tp { text-indent:57px }` (57 px ≈ 1,5 cm) |
| Mudar a posição do número da página do TCC | `.tcc-page .pn` |
| Mudar como sai a referência ABNT | `refMd`, `autRef` |
| Mudar o formato das citações | `citeText` |
| Mudar textos de orientação do TCC | `OR` |
| Mudar a qualidade do PDF | `exportPDF` (`scale`, qualidade do JPEG) |
| Mudar os avisos “Vale conferir” | `issues` (prova) e `tccIssues` (TCC) |
| Mudar a etiqueta e o título da edição Aluno | bloco `if (EDICAO === 'aluno')` no fim do script e `ferramentas/gerar_aluno.py` |

---

## 8. Criar um modelo novo

O jeito mais seguro é começar copiando o modelo que mais se parece com o novo.

**Modelo parecido com a prova** (ex.: prova discursiva, simulado):

1. Em `TEMPLATES`, crie uma entrada nova com outro `id`, `nome`, `area` e `novo()` / `exemplo()`.
2. Se o documento tiver os mesmos campos da prova, ele já funciona com o formulário, a paginação e o PDF da prova, porque tudo o que não é TCC usa o caminho da prova.
3. Para diferenças de visual, crie uma condição pelo `templateId` dentro de `headerBlocks` e `buildBlocks` (por exemplo, linhas para resposta em vez de alternativas).
4. Tire o cartão correspondente de `EM_BREVE`.

**Modelo bem diferente** (ex.: relatório de estágio, ofício, ata): siga o exemplo do TCC, que é um modelo completo à parte. Os pontos que precisam saber do modelo novo são os mesmos em que o TCC aparece com `isTcc(...)`:

| Ponto | Função |
|---|---|
| Formulário | `renderForm` → `renderTccForm` |
| Páginas | `paginate` → `tccPaginate` |
| Caminhos editáveis | `getRaw`, `setRaw`, `displayFor` |
| Avisos e números da revisão | `renderSide` (`tccIssues`, `tccStats`) |
| Nome do rascunho e do PDF | `docName` |
| Resumo na lista de rascunhos | `draftSummary` |
| Rascunhos antigos | `migrate` |

Crie as funções do seu modelo (por exemplo `relRenderForm`, `relPaginate`…) e acrescente uma condição em cada um desses pontos.

Dicas:

- Use `RT(...)` para campos de texto com formatação e `V(...)` para trechos editáveis na revisão.
- Para paginar um documento de uma coluna, reaproveite `tccFlow`: ele já divide parágrafos longos entre páginas e respeita “não deixar título sozinho”.
- Toda página precisa ter a classe `page` (é ela que o PDF fotografa).

---

## 9. Criar a sua própria versão (outra escola)

1. **Escola:** em `SCHOOLS`, troque ou acrescente nome, sigla, cidade e logo.
2. **Código:** gere com `ferramentas/novo_codigo.py CODIGO sigla` e coloque em `LOCKS`. Cada escola pode ter o seu código; o código digitado decide qual escola abre.
3. **Cabeçalho da prova:** troque `HEADER_IMG` e ajuste `CURSOS`, `TURNOS`, `SERIES`, `TURMAS` e o texto de `headerBlocks`.
4. **Instruções:** `INSTRUCOES_MODELO`.
5. **Modelos:** em cada entrada de `TEMPLATES`, ajuste `school`, `org` e os textos. No TCC, ajuste `f.escola`, `f.curso`, `NATUREZA_PADRAO` e `CEAAT_LOGO`.
6. **Marca:** cores em `:root`, logo do EDOC no topo e o texto da aba (`<title>`).
7. **Exemplos:** troque ou esvazie `EXEMPLO_HUMANAS` e o `exemplo()` do TCC.
8. Gere a edição do aluno (`python3 ferramentas/gerar_aluno.py`) e publique.

Se mudar a chave de salvamento (`STORE_KEY`), cada versão terá rascunhos separados mesmo quando estiver no mesmo endereço.

---

## 10. Limites conhecidos

- **O código de acesso não é uma proteção forte.** Como tudo roda no navegador, quem entende de programação consegue pular a tela do código. Ele evita o acesso casual. Para proteção de verdade (cada professor com login próprio, provas guardadas num servidor), é preciso um serviço como o Supabase.
- **Rascunhos ficam só no navegador.** Trocar de computador ou limpar os dados do navegador apaga os rascunhos e o banco de questões excluídas. Imagens grandes podem encher o espaço disponível (cerca de 5 MB); o sistema avisa quando isso acontece.
- **O PDF é feito de imagens.** Ele fica igual ao que aparece na tela, mas o texto não pode ser selecionado nem copiado dentro do PDF. Os links do sumário do TCC funcionam normalmente.
- **A detecção de questões repetidas é por texto.** Questões com textos diferentes que dependem da mesma imagem não são marcadas como repetidas.
- **A importação depende de padrões.** Questões precisam começar com “QUESTÃO” e o número; capítulos do TCC precisam estar numerados ou com estilo de Título no Word. Fora disso, o conteúdo entra, mas pode precisar de ajuste no formulário.
