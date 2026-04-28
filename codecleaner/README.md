# 🧹 Code Cleaner

Agente de limpieza y mejora de código para Node.js/JavaScript/TypeScript potenciado por **Claude Code**.

Elimina código muerto, simplifica lógica compleja, corrige bugs y documenta — aplicando cambios directamente en los ficheros.

---

## 🚀 Inicio rápido

```bash
# Colocar esta carpeta en la raíz del proyecto a limpiar y ejecutar:
claude

# Limpiar un fichero completo:
clean src/routes/users.js

# Limpiar todo un directorio:
clean:full src/
```

---

## 📋 Comandos

| Comando | Descripción |
|---------|-------------|
| `clean <fichero>` | Limpieza completa (todas las áreas) |
| `clean:dead <fichero>` | Solo eliminar código muerto e imports sin uso |
| `clean:simplify <fichero>` | Solo simplificar y reducir complejidad |
| `clean:bugs <fichero>` | Solo revisar y corregir bugs lógicos |
| `clean:docs <fichero>` | Solo añadir documentación JSDoc/comentarios |
| `clean:full <ruta>` | Limpieza completa de directorio o proyecto |
| `clean:structure <ruta>` | Analizar y proponer reorganización de estructura |
| `report` | Informe final de todo lo hecho en la sesión |
| `explain <fragmento>` | Explicar qué hace un fragmento (sin modificar) |
| `undo` | Revertir el último cambio aplicado |

---

## 🎯 Qué hace

### 🗑️ Código Muerto
- Imports y requires sin usar
- Variables y constantes sin referenciar
- Funciones no exportadas y no llamadas
- Bloques inalcanzables (código tras `return`, condiciones siempre falsas)
- Scripts de `package.json` que apuntan a ficheros inexistentes
- Código comentado (bloques grandes de código antiguo)
- Comentarios obvios que solo repiten el código

### ✂️ Simplificación
- Código duplicado → funciones reutilizables (DRY)
- Anidamiento excesivo → early returns
- `var` → `const`/`let`, `.then()` → `async/await`
- String concatenation → template literals
- `x && x.y` → `x?.y` (optional chaining)
- `x !== null && x !== undefined` → `x ?? default` (nullish coalescing)
- Funciones largas → funciones más pequeñas con responsabilidad única
- Números mágicos → constantes nombradas

### 🐛 Corrección de Bugs
- `await` dentro de `forEach` (no espera)
- Promesas sin `.catch()` ni `try/catch`
- Errores silenciados en bloques `catch` vacíos
- `sort()` que muta el array original sin intención
- `JSON.parse()` sin protección ante JSON inválido
- `parseInt()` sin radix
- `NaN` comparado con `===`
- Closures con `var` en loops
- Memory leaks: event listeners sin cleanup, streams no cerrados

### 📝 Documentación
- JSDoc en funciones exportadas y de API pública
- Comentarios explicativos del "por qué" (no del "qué")
- Documentación de parámetros no obvios y casos de error
- README de módulo para directorios con lógica no trivial

---

## ⚙️ Skills incluidas

```
skills/
├── dead-code.md    ← Imports, variables, funciones, bloques sin uso
├── simplify.md     ← DRY, early return, JS moderno, complejidad
├── bugs.md         ← Errores lógicos, async, mutaciones, null handling
├── docs.md         ← JSDoc, comentarios, README
├── structure.md    ← Reorganización de ficheros y módulos
└── report.md       ← Generación del informe final de sesión
```

---

## 🛡️ Garantías

El agente **siempre**:
- Lee el fichero completo antes de modificar nada
- Anuncia cada cambio antes de aplicarlo
- Preserva el comportamiento funcional existente
- Pregunta antes de eliminar algo con uso incierto

El agente **nunca**:
- Elimina código sin entender qué hace
- Modifica lógica de negocio sin avisar explícitamente
- Rompe exports o interfaces públicas sin advertencia
- Toca ficheros de dependencias (`node_modules`, `package-lock.json`)

---

## 🔗 Combinar con Code Auditor

Para una revisión completa, usar junto con `code-auditor`:

```bash
# Primero limpiar:
clean:full src/

# Luego auditar seguridad:
# (desde el proyecto code-auditor)
audit:full src/
```
