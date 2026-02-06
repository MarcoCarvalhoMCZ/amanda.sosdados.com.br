# Relatório de Bug - Sistema de Recompensas

## Problema Identificado

O sistema está calculando a porcentagem de conclusão mensal baseado apenas nos dias que já passaram. Quando o usuário completa todos os hábitos de um único dia (especialmente no início do mês), ele atinge 100% imediatamente, desbloqueando todas as recompensas.

### Cenário do Bug:
1. Usuário cadastra seus hábitos
2. No dia 1 do mês, ele completa todos os hábitos do dia
3. Sistema calcula: 5/5 hábitos = **100%** ✓
4. Todas as recompensas são desbloqueadas imediatamente
5. Tela de sorteio aparece

## Código Atual (com problema)

```javascript
function calculateMonthlyCompletion() {
    const daysInMonth = new Date(new Date().getFullYear(), new Date().getMonth() + 1, 0).getDate();
    let totalPossible = 0;
    let totalCompleted = 0;
    
    for (let day = 1; day <= daysInMonth; day++) {
        const d = new Date(new Date().getFullYear(), new Date().getMonth(), day);
        const dayOfWeek = d.getDay();
        const dateStr = d.toDateString();
        
        if (d > new Date()) break;  // ← Para no dia atual
        
        const dayHabits = data.habits.filter(h => h.days.includes(dayOfWeek));
        totalPossible += dayHabits.length;
        
        dayHabits.forEach(h => {
            if (h.completed[dateStr]) totalCompleted++;
        });
    }
    
    return totalPossible > 0 ? (totalCompleted / totalPossible) * 100 : 0;
}
```

## Solução Proposta

Existem duas abordagens possíveis:

### Opção A: Exigir mínimo de dias (Recomendada)

Só permitir que recompensas sejam desbloqueadas após um mínimo de dias no mês (ex: 7 dias ou 50% do mês):

```javascript
function calculateMonthlyCompletion() {
    const now = new Date();
    const daysInMonth = new Date(now.getFullYear(), now.getMonth() + 1, 0).getDate();
    const currentDay = now.getDate();
    
    // Só conta se pelo menos 25% do mês passou (mínimo ~7 dias)
    const minimumDaysRequired = Math.ceil(daysInMonth * 0.25);
    
    let totalPossible = 0;
    let totalCompleted = 0;
    let daysWithHabits = 0;
    
    for (let day = 1; day <= daysInMonth; day++) {
        const d = new Date(now.getFullYear(), now.getMonth(), day);
        const dayOfWeek = d.getDay();
        const dateStr = d.toDateString();
        
        if (d > now) break;
        
        const dayHabits = data.habits.filter(h => h.days.includes(dayOfWeek));
        if (dayHabits.length > 0) daysWithHabits++;
        totalPossible += dayHabits.length;
        
        dayHabits.forEach(h => {
            if (h.completed[dateStr]) totalCompleted++;
        });
    }
    
    // Se não passou dias suficientes, limita a porcentagem máxima
    if (daysWithHabits < minimumDaysRequired) {
        const maxAllowed = (daysWithHabits / minimumDaysRequired) * 100;
        const rawPercent = totalPossible > 0 ? (totalCompleted / totalPossible) * 100 : 0;
        return Math.min(rawPercent, maxAllowed);
    }
    
    return totalPossible > 0 ? (totalCompleted / totalPossible) * 100 : 0;
}
```

### Opção B: Calcular sobre o mês inteiro

Calcular a porcentagem considerando TODOS os dias do mês (passados e futuros):

```javascript
function calculateMonthlyCompletion() {
    const now = new Date();
    const daysInMonth = new Date(now.getFullYear(), now.getMonth() + 1, 0).getDate();
    
    let totalPossible = 0;
    let totalCompleted = 0;
    
    // Conta hábitos possíveis para o MÊS INTEIRO
    for (let day = 1; day <= daysInMonth; day++) {
        const d = new Date(now.getFullYear(), now.getMonth(), day);
        const dayOfWeek = d.getDay();
        const dateStr = d.toDateString();
        
        const dayHabits = data.habits.filter(h => h.days.includes(dayOfWeek));
        totalPossible += dayHabits.length;
        
        // Só conta completados para dias passados
        if (d <= now) {
            dayHabits.forEach(h => {
                if (h.completed[dateStr]) totalCompleted++;
            });
        }
    }
    
    return totalPossible > 0 ? (totalCompleted / totalPossible) * 100 : 0;
}
```

### Opção C: Também validar no sorteio

Adicionar validação antes de permitir o sorteio:

```javascript
function sortearRecompensa() {
    const completionRate = calculateMonthlyCompletion();
    const now = new Date();
    const currentDay = now.getDate();
    const daysInMonth = new Date(now.getFullYear(), now.getMonth() + 1, 0).getDate();
    
    // Só permite sorteio se passou pelo menos 50% do mês
    if (currentDay < Math.ceil(daysInMonth * 0.5)) {
        showToast('Aguarde! 📅', `O sorteio estará disponível após o dia ${Math.ceil(daysInMonth * 0.5)}`);
        return;
    }
    
    // Só permite se tiver pelo menos 70% de conclusão
    if (completionRate < 70) {
        showToast('Continue assim! 💪', `Você precisa de pelo menos 70% para sortear. Atual: ${Math.round(completionRate)}%`);
        return;
    }
    
    // ... resto do código de sorteio
}
```

## Recomendação Final

Implementar **Opção A + Opção C** juntas para:
1. Mostrar progresso realista durante o mês
2. Evitar que usuários "trapaceiem" completando só o primeiro dia
3. Manter a motivação mostrando progresso parcial
4. Garantir que o sorteio só aconteça quando fizer sentido

---

**Data do relatório:** 01/02/2026  
**Arquivo afetado:** index.html (linhas 2935-2956)
