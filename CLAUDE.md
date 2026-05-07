# Pulse Protótipo — Plural Orto

Protótipo de sistema SaaS para clínica odontológica. **Manter simples**: sem build system, sem framework, sem CSS compartilhado. Cada página é um HTML standalone com CSS inline.

## Arquitetura

3 arquivos HTML, navegação real entre eles via `href`:

```
prototipo1.html   → Dashboard (visão macro da clínica)
consultorio.html  → Meu Dia (fila de pacientes do dia)
atendimento.html  → Atendimento em curso (detalhe de um paciente)
```

Para criar uma nova página, duplicar um HTML existente e ajustar só o conteúdo específico da tela. O bloco `:root`, header e nav são copiados intencionalmente — não extrair para arquivo externo.

## Design Tokens

Bloco `:root` idêntico em todas as páginas. Nunca inventar valores fora desses tokens.

### Cores
```css
/* Fundo — escala quente */
--paper:       #F4EEE2   /* fundo geral */
--paper-2:     #EDE5D2   /* fundo de elementos nested */
--paper-3:     #E3D9C0   /* fundo mais escuro */
--cream:       #FBF6EE   /* superfície de cards */

/* Texto — 3 níveis (ink-2 não é usado como nível de hierarquia) */
--ink:         #17140F   /* primário: nomes, valores, headings */
--ink-3:       #6B6456   /* secundário: descrições, meta, labels */
--ink-4:       #9B937F   /* metadados: timestamps, labels fracos */

/* Divisores */
--line:        #D9CFB8
--line-2:      #E5DCC6

/* Semânticas */
--accent:      #C15F3C   /* brand / ativo / em atendimento */
--accent-ink:  #8A3E25   /* texto sobre accent-soft */
--accent-soft: #EDD6C3   /* fundo suave accent */
--accent-50:   #F7E7D8
--gold:        #B08A4A   /* pendente / aguardando */
--gold-soft:   #E8D9B2
--red:         #B03030   /* alerta / erro / inadimplência */
--red-soft:    #F5DDD8
--green:       #6E7F4C   /* confirmado / concluído / pago */
```

### Estados por cor
| Estado | Cor |
|---|---|
| Em atendimento / Ativo | `--accent` |
| Confirmado / Pago / Concluído | `--green` |
| Aguardando / Pendente | `--gold` |
| Alerta / Erro / Inadimplência | `--red` |
| Neutro / Sem estado | `--paper-2` / `--ink-4` |

### Tipografia
Três famílias com papéis **estritos** — não misturar:

| Família | Variável | Usar SOMENTE para |
|---|---|---|
| DM Serif Display | `--font-serif` | Valores display/hero: KPI numbers, nomes de pacientes em destaque, títulos de página |
| Inter | `--font-sans` | Tudo que for UI: labels, eyebrows, badges, nav, breadcrumbs, botões, textarea, avatares |
| JetBrains Mono | `--font-mono` | Dados literais tabulares: horários (08:30), valores monetários (R$ 320), timer da sessão, numeração de dentes |

**Escala de tamanhos (6 passos):**
- `11px` — metadata: badges, labels, eyebrows (substitui toda a faixa 9px–10.5px)
- `13px` — secondary: body de apoio, descriptions (substitui 11.5px–13.5px)
- `15px` — primary: corpo base
- `17px` — subtitle: títulos de seção
- `20px` — display-md: nomes de pacientes em destaque
- `28px+` — display-lg: valores KPI, page titles, timer de sessão

**Exceções permitidas:** `tooth-num` em 9px (odontograma muito compacto); `timer-display` em 40px (stopwatch da sessão).

**Letter-spacing:**
- `-0.02em` — DM Serif Display (display type)
- `0` — Inter corpo
- `0.04–0.06em` — Inter uppercase labels
- `0.04em` — JetBrains Mono dados

### Sombras e raios
```css
--shadow-sm: 0 1px 2px rgba(26,20,14,0.04), 0 2px 6px rgba(26,20,14,0.04)
--shadow-md: 0 2px 4px rgba(26,20,14,0.04), 0 12px 30px -12px rgba(26,20,14,0.12)

--radius-sm: 10px
--radius:    16px
```

## Shell de Layout

Toda página usa header fixo (56px) + nav fixo (48px) com frosted glass. O `main` sempre compensa com `margin-top: calc(var(--header-h) + var(--nav-h))`.

## Componentes Padrão

### Card (superfície base)
```css
background: var(--cream);
border: 1px solid var(--line-2);
border-radius: var(--radius);
padding: 16px;
box-shadow: var(--shadow-sm);
```
Variação ativa: `border-left: 3px solid var(--accent)` + fundo levemente tintado.
Variação KPI: `::before` de 2px no topo com a cor do estado.

### Eyebrow (label de seção)
```css
font-family: var(--font-sans);
font-size: 11px; letter-spacing: 0.06em; text-transform: uppercase;
color: var(--ink-4); font-weight: 500;
```

### Section title (título dentro de card)
```css
font-family: var(--font-serif);
font-size: 17px; font-weight: 500; color: var(--ink);
letter-spacing: -0.02em;
```

### Status Badge (pill)
```css
border-radius: 999px; font-family: var(--font-sans);
font-size: 11px; text-transform: uppercase; font-weight: 500;
```
Variantes: `.on` (accent), `.conf` (green), `.wait` (gold), `.done` (paper-2 / ink-4).

### Alert tag (pill de alerta clínico/administrativo)
```css
font-family: var(--font-sans); font-size: 11px; letter-spacing: 0;
text-transform: uppercase; padding: 2px 8px; border-radius: 999px; font-weight: 500;
```
Variantes: `.medical` (red-soft + borda red, font-weight 600), `.payment` (gold-soft), `.exam` (paper-2), `.new-pt` (green-soft).

### Avatar
Círculo com iniciais em Inter. A cor do fundo comunica contexto:
- Doutor: gradiente accent (`#E37A54 → #B84C2B`)
- Paciente ativo: `accent-soft`
- Paciente com alerta médico: `red-soft`
- Paciente novo / aguardando: `gold-soft`

### Dot indicator
`7px` circle, `border-radius: 50%`. Mesmo sistema de cores dos badges.

### Breadcrumb
```css
font-family: var(--font-sans); font-size: 11px; letter-spacing: 0.04em;
color: var(--ink-4);
```
Link ativo em `accent-ink`; página atual em `ink` com `font-weight: 500`.

### Footer link
```css
font-family: var(--font-sans); font-size: 11px;
letter-spacing: 0.06em; text-transform: uppercase;
color: var(--accent-ink); font-weight: 500;
```
Sempre com texto no padrão "Ver todos →".

## Padrão de Grid por Página

Todas as páginas de conteúdo usam grid 2 colunas. Coluna direita: contexto/suporte (sticky). Coluna esquerda: conteúdo principal.

```
prototipo1:   grid-template-columns: 1fr 390px
consultorio:  grid-template-columns: 1fr 300px
atendimento:  grid-template-columns: 1fr 300px
```

## Regras

- Não criar arquivo CSS externo compartilhado — duplicação é intencional neste protótipo
- Não introduzir JavaScript além do estritamente necessário (só o timer do atendimento existe hoje)
- Não usar frameworks (React, Vue, etc.)
- Sempre usar tokens do `:root` — nunca valores de cor ou tamanho hardcoded fora do bloco de variáveis
- Manter o tom visual: warm, editorial, denso em informação mas legível
