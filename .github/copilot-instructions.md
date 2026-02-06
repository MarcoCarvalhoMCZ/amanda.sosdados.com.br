# Amanda - Copilot Instructions

## Visão Geral

**Done** é um PWA de rastreamento de hábitos e rotinas saudáveis. Toda a aplicação (~3000 linhas) vive em um único `index.html` — sem build, sem dependências, sem framework. Abrir o arquivo no navegador é suficiente para rodar.

## Arquitetura Monolítica

- **Um único arquivo**: `index.html` contém HTML + `<style>` + `<script>` embutidos. Não separar em múltiplos arquivos.
- **Vanilla JS puro**: sem React/Vue/Angular. DOM manipulado diretamente via `innerHTML` em funções `render*()`.
- **Persistência**: `localStorage` com chave `dailyTrackData`. Estrutura:

```javascript
{
  user: { nome, email, peso, altura, genero, objetivo },
  habits: [{ id, name, icon, category, timeOfDay, days[], dosage?, streak, color, completed: { [dateStr]: true } }],
  water: { current, goal },
  rewards: [{ id, name, icon, description, required, unlocked }],
  xp: number, level: number, theme: string, joinDate: string
}
```

- `habit.completed` usa `new Date().toDateString()` como chave (ex: `"Fri Feb 06 2026"`).
- `habit.id` é gerado por `Date.now()` na criação.

## Funções Críticas e Fluxo de Dados

| Função | Papel |
|---|---|
| `init()` | Carrega `localStorage` → decide auth/main → chama renders |
| `saveData()` | `JSON.stringify(data)` → `localStorage` (chamar após toda mutação) |
| `renderDaily()` | Agrupa hábitos por `timeOfDay` + separa `medication`/`supplement` → monta HTML |
| `renderWeekly()` | Calendário semanal com `selectedDate` |
| `renderRewards()` | Cards com progresso baseado em `calculateMonthlyCompletion()` |
| `toggleHabit(id)` | Toggle completado → XP ±25/10 → confetti → checkRewards → saveData → re-render |
| `setScreen(screen)` | Navegação entre telas: `daily`, `weekly`, `rewards`, `settings` |
| `calculateMonthlyCompletion()` | % mensal com trava de mínimo de dias (`max(7, 25% do mês)`) para evitar 100% prematuro |

## Categorias e Períodos

- **Categorias**: `medication`, `supplement`, `habit`, `exercise` — `medication`/`supplement` renderizam separadamente em `medSection`
- **Períodos**: `morning` 🌅, `afternoon` ☀️, `evening` 🌆, `night` 🌙
- **Cores por categoria**: `habit`→`var(--primary)`, `medication`→`#3b82f6`, `supplement`→`#8b5cf6`, `exercise`→`#10b981`

## Sistema de Temas (CSS Variables)

Temas aplicados via `data-theme` no `<html>`: `default`, `ocean`, `sunset`, `forest`, `midnight`, `berry`, `monochrome`. Sempre usar `var(--primary)`, `var(--surface)`, `var(--text)`, etc. Nunca hardcodar cores.

## Gamificação e Recompensas

- **XP**: +25 ao completar, -10 ao desmarcar. Level up em `xp >= level * 500`.
- **Recompensas**: desbloqueiam por `% mensal >= reward.required`. Proteção contra desbloqueio prematuro: `checkRewards()` exige mínimo de `max(7, 25% do mês)` dias passados.
- **Sorteio** (`sortearRecompensa()`): exige 50% do mês passado + 70% de conclusão. Sorteia aleatoriamente da lista de recompensas.

## Padrões de UI

- **Modais**: `.modal-overlay` + toggle classe `.active`. Clique fora fecha via event listener global.
- **Feedback**: `showToast('Título', 'Mensagem')` para notificações, `createConfetti(x, y)` para celebrações.
- **Design mobile-first**: viewport com `user-scalable=no`, componentes otimizados para toque.
- **Ícones**: Emojis como padrão (🎯, 💊, 🏋️), sem biblioteca de ícones.

## Regras ao Modificar

1. **Monolítico**: toda alteração dentro de `index.html` — não criar arquivos JS/CSS separados
2. **Sempre `saveData()`** após mutar `data` — esquecer causa perda de dados
3. **Re-render**: após mutação, chamar o `render*()` correspondente para atualizar a UI
4. **Compatibilidade**: não alterar estrutura de `dailyTrackData` sem migração — usuários reais dependem dos dados salvos
5. **Idioma**: interface 100% em **português brasileiro** — toda string, label e mensagem em pt-BR
6. **Sem console.log em produção** exceto em blocos `catch` para debug de erros
