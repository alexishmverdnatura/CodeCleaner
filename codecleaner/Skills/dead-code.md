# SKILL: DEAD CODE — Eliminación de Código Muerto

## Objetivo
Detectar y eliminar todo el código que no se ejecuta, no se usa, o no aporta valor: imports, variables, funciones, clases, bloques condicionales inalcanzables y scripts obsoletos.

---

## CHECKLIST DE DETECCIÓN

### 1. Imports y Requires sin uso
```
DETECTAR:
□ import X from 'module' → X nunca referenciado en el fichero
□ import { a, b, c } from 'module' → alguno de a/b/c sin usar
□ const X = require('module') → X nunca referenciado
□ const { a, b } = require('module') → alguno sin usar
□ import 'side-effect' → verificar si el side-effect es necesario

CUIDADO — no eliminar si:
- El import tiene side effects conocidos (p.ej. import 'reflect-metadata' en NestJS)
- Es un tipo TypeScript usado solo en anotaciones
- Está referenciado dinámicamente (p.ej. como string en un objeto de configuración)
```

**Ejemplos:**
```javascript
// ❌ ELIMINAR — fs nunca se usa en el fichero:
const fs = require('fs');
const path = require('path');

function greet(name) {
  return `Hello, ${name}`; // fs y path no se usan en ningún sitio
}

// ❌ ELIMINAR — { isEmpty } importado pero no usado:
import { merge, isEmpty, cloneDeep } from 'lodash';
// isEmpty no aparece en el resto del fichero

// ✅ RESULTADO:
import { merge, cloneDeep } from 'lodash';
```

### 2. Variables y Constantes sin uso
```
DETECTAR:
□ Variables declaradas y nunca leídas
□ Variables asignadas múltiples veces sin leer el valor intermedio
□ Constantes definidas con valor que nunca se referencia
□ Parámetros de función nunca usados (con cuidado en callbacks de API)
□ Destructuring con variables no usadas

CUIDADO:
- Parámetros de middleware Express (req, res, next) aunque no se usen
- Variables prefijadas con _ (convención de "intencionalmente no usada")
- Variables usadas solo en bloques eliminados (eliminar en conjunto)
```

**Ejemplos:**
```javascript
// ❌ ELIMINAR — result se asigna pero nunca se lee:
function processUser(user) {
  const result = validateUser(user); // result no se usa
  const processed = transform(user);
  return processed;
}

// ❌ ELIMINAR — valores intermedios sobrescritos sin leer:
let config = {};
config = loadFromFile();  // primera asignación nunca se lee
config = loadFromEnv();   // esta es la que vale
return config;

// ❌ ELIMINAR — destructuring parcialmente usado:
const { id, name, email, createdAt, updatedAt } = user;
// Solo se usan id y name en el resto de la función
const { id, name } = user; // ✅
```

### 3. Funciones sin uso
```
DETECTAR:
□ Funciones declaradas y nunca llamadas en el mismo fichero
□ Funciones no exportadas y sin llamadas detectables
□ Métodos de clase nunca invocados
□ Helpers/utils creados y luego reemplazados por otro enfoque
□ Funciones de debug o testing dejadas en producción

PROCESO DE VERIFICACIÓN:
1. ¿Está exportada? → puede usarse externamente, no eliminar sin verificar
2. ¿Está referenciada como string? (require(funcName)) → no eliminar
3. ¿Es un handler de eventos? → verificar si el evento se emite
4. ¿Aparece en configuración (router, middleware array)? → no eliminar
```

**Ejemplos:**
```javascript
// ❌ ELIMINAR — formatDate nunca se llama y no se exporta:
function formatDate(date) {
  return new Date(date).toLocaleDateString();
}

function formatCurrency(amount) { // Esta SÍ se usa
  return `€${amount.toFixed(2)}`;
}

module.exports = { formatCurrency }; // formatDate no se exporta
```

### 4. Bloques de Código Inalcanzables
```
DETECTAR:
□ Código después de return/throw/break dentro del mismo bloque
□ Condiciones siempre verdaderas o falsas:
   - if (true) / if (false)
   - if (process.env.NODE_ENV === 'production') en fichero de dev hardcodeado
□ Ramas else tras return en if
□ Casos switch nunca alcanzables
□ try/catch vacío o con catch que ignora el error sin razón
```

**Ejemplos:**
```javascript
// ❌ ELIMINAR — código tras return:
function getUser(id) {
  return db.findById(id);
  console.log('User fetched'); // nunca se ejecuta
  return null;                 // nunca se ejecuta
}

// ❌ SIMPLIFICAR — else innecesario tras return:
function validate(value) {
  if (!value) {
    return false;
  } else {          // else innecesario
    return true;
  }
}
// ✅:
function validate(value) {
  if (!value) return false;
  return true;
}

// ❌ ELIMINAR o REVISAR — catch vacío:
try {
  riskyOperation();
} catch (err) {
  // silently ignore
}
// ✅ Al menos loguear:
try {
  riskyOperation();
} catch (err) {
  console.error('riskyOperation failed:', err);
}
```

### 5. Scripts de package.json sin uso
```
DETECTAR:
□ Scripts que referencian ficheros que no existen
□ Scripts duplicados con nombres distintos
□ Scripts de herramientas desinstaladas
□ Scripts comentados o con "TODO: remove"
□ devDependencies instaladas pero no referenciadas en ningún script ni config

VERIFICAR en:
- package.json scripts
- Ficheros de configuración (webpack.config.js, .eslintrc, jest.config.js)
- Comandos en Dockerfile o CI/CD si están accesibles
```

### 6. Ficheros sin uso (análisis de proyecto)
```
DETECTAR (en clean:full):
□ Ficheros .js/.ts que no son importados por nadie
□ Ficheros de configuración obsoletos (.babelrc cuando ya hay babel.config.js)
□ Ficheros duplicados (utils.js y helpers.js con funciones idénticas)
□ Ficheros de migración/scripts one-off sin documentar
□ Assets y estáticos sin referencias
□ Ficheros *.bak, *.old, *.copy

PROCESO:
1. Construir grafo de imports del proyecto
2. Identificar ficheros sin ningún importador (excepto entry points)
3. Verificar si son entry points en package.json main/exports
4. Verificar si están en scripts de package.json
5. Solo entonces proponer eliminación
```

### 7. Comentarios que son Código Muerto
```
ELIMINAR:
□ Código comentado (bloques grandes de código antiguo)
□ TODO/FIXME de hace meses sin contexto
□ Comentarios que solo repiten el código ("// increment counter" sobre counter++)

CONSERVAR:
□ Comentarios que explican el "por qué" (decisiones de diseño)
□ TODO con contexto claro y reciente
□ Comentarios de workarounds conocidos ("// Safari bug: ...")
□ JSDoc / TSDoc
```

**Ejemplos:**
```javascript
// ❌ ELIMINAR — código comentado:
function processOrder(order) {
  // const oldTotal = order.items.reduce((a, b) => a + b.price, 0);
  // const tax = oldTotal * 0.21;
  // return oldTotal + tax;
  
  return order.total; // nuevo cálculo desde backend
}

// ❌ ELIMINAR — comentario obvio:
// Increment the counter by 1
counter++;

// ✅ CONSERVAR — explica el por qué:
// Safari < 15 doesn't support the Intl.DisplayNames API, fall back to raw code
const displayName = supportsDisplayNames ? getDisplayName(locale) : locale;
```

---

## FORMATO DE REPORTE POR HALLAZGO

```
🗑️ [DEAD CODE] Tipo: Import / Variable / Función / Bloque / Fichero
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Fichero/Línea: src/utils.js:12-15
Elemento:      función formatDate()
Razón:         No exportada, no referenciada en ningún lugar del fichero

ANTES:
  [código a eliminar]

DESPUÉS:
  [código resultante o "(eliminado)"]

Líneas ahorradas: X
```
