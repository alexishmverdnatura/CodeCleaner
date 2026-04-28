# SKILL: BUGS — Detección y Corrección de Errores Lógicos

## Objetivo
Identificar y corregir bugs, comportamientos inesperados, condiciones de carrera, manejo incorrecto de errores y cualquier código que haga algo distinto a lo que parece hacer o a lo que el contexto indica que debería hacer.

---

## REGLA DE ORO

Antes de corregir cualquier bug: **anunciar el problema y explicar por qué es un bug**. Si hay ambigüedad sobre la intención original, preguntar antes de modificar.

---

## CHECKLIST DE DETECCIÓN

### 1. Manejo de Errores Incorrecto
```
DETECTAR:
□ Promesas sin .catch() ni try/catch (unhandled rejections)
□ async/await sin try/catch en código que puede fallar
□ Errores silenciados (catch vacío o con solo console.log)
□ Re-throw de errores que pierden el stack trace original
□ Callbacks que no comprueban el primer parámetro de error (Node.js style)
□ Errores de tipo sincrónico en funciones async que no se capturan
```

**Ejemplos:**
```javascript
// ❌ BUG — Promesa sin manejo de error (unhandled rejection):
app.get('/users', (req, res) => {
  User.find().then(users => res.json(users));
  // Si User.find() falla, crash silencioso
});

// ✅ FIX:
app.get('/users', async (req, res) => {
  try {
    const users = await User.find();
    res.json(users);
  } catch (err) {
    res.status(500).json({ error: 'Failed to fetch users' });
  }
});

// ❌ BUG — Re-throw que pierde stack trace:
try {
  riskyOp();
} catch (err) {
  throw new Error(err.message); // nuevo Error, stack trace perdido
}

// ✅ FIX:
try {
  riskyOp();
} catch (err) {
  throw err; // preserva stack trace original
}

// ❌ BUG — Error ignorado en callback estilo Node:
fs.readFile('./config.json', (err, data) => {
  const config = JSON.parse(data); // si err existe, data es null → crash
});

// ✅ FIX:
fs.readFile('./config.json', (err, data) => {
  if (err) {
    console.error('Failed to read config:', err);
    return;
  }
  const config = JSON.parse(data);
});
```

### 2. Operaciones Asíncronas Incorrectas
```
DETECTAR:
□ await dentro de forEach (no funciona como se espera)
□ Promise.all() cuando se necesita orden garantizado (usar for...of)
□ Race conditions por múltiples operaciones async sin coordinación
□ Variables compartidas modificadas en operaciones async concurrentes
□ setTimeout/setInterval sin clearTimeout/clearInterval en cleanup
□ Variables de bucle capturadas incorrectamente (closure bug con var)
```

**Ejemplos:**
```javascript
// ❌ BUG CLÁSICO — await en forEach no espera:
async function saveAll(items) {
  items.forEach(async (item) => {
    await db.save(item); // forEach no espera estas promesas
  });
  console.log('All saved!'); // Se ejecuta ANTES de que terminen los saves
}

// ✅ FIX — for...of:
async function saveAll(items) {
  for (const item of items) {
    await db.save(item);
  }
  console.log('All saved!');
}

// ✅ FIX — o en paralelo con Promise.all:
async function saveAll(items) {
  await Promise.all(items.map(item => db.save(item)));
  console.log('All saved!');
}

// ❌ BUG — closure con var en loop:
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // imprime 3, 3, 3
}

// ✅ FIX:
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 100); // imprime 0, 1, 2
}
```

### 3. Comparaciones y Tipos Incorrectos
```
DETECTAR:
□ == en lugar de === (coerción de tipos inesperada)
□ Comparación con NaN usando === (siempre false, usar Number.isNaN())
□ typeof null === 'object' (bug clásico de JS)
□ Operaciones matemáticas en strings que parecen números
□ Comparación de objetos/arrays por referencia cuando se espera por valor
□ parseInt sin radix (puede interpretar "08" como octal en algunos engines)
```

**Ejemplos:**
```javascript
// ❌ BUG — NaN nunca es igual a nada, ni a sí mismo:
if (result === NaN) { /* nunca se ejecuta */ }

// ✅ FIX:
if (Number.isNaN(result)) { ... }

// ❌ BUG — parseInt sin radix:
parseInt('09') // puede dar 0 en engines antiguos (octal)

// ✅ FIX:
parseInt('09', 10) // siempre base 10

// ❌ BUG — suma de string + número:
const total = req.query.price + req.query.tax; // "10" + "2" = "102"

// ✅ FIX:
const total = Number(req.query.price) + Number(req.query.tax);

// ❌ BUG — comparación de arrays por referencia:
if (selectedIds === defaultIds) { /* siempre false */ }

// ✅ FIX (según intención):
if (JSON.stringify(selectedIds) === JSON.stringify(defaultIds)) { ... }
// O mejor:
if (selectedIds.length === defaultIds.length && 
    selectedIds.every((id, i) => id === defaultIds[i])) { ... }
```

### 4. Mutaciones Inesperadas
```
DETECTAR:
□ Modificar arrays/objetos recibidos como parámetros (side effects ocultos)
□ .sort() que muta el array original sin intención
□ Object.assign al primer argumento siendo un objeto compartido
□ splice() cuando se quiere slice()
□ push() cuando se quiere un nuevo array
```

**Ejemplos:**
```javascript
// ❌ BUG — sort() muta el array original:
function getTopUsers(users) {
  return users.sort((a, b) => b.score - a.score).slice(0, 10);
  // users externo queda ordenado como side effect no intencionado
}

// ✅ FIX:
function getTopUsers(users) {
  return [...users].sort((a, b) => b.score - a.score).slice(0, 10);
}

// ❌ BUG — mutar parámetro objeto:
function addDefaults(config) {
  config.timeout = config.timeout || 5000; // muta el objeto del llamador
  config.retries = config.retries || 3;
  return config;
}

// ✅ FIX:
function addDefaults(config) {
  return {
    timeout: 5000,
    retries: 3,
    ...config, // sobreescribe defaults con valores del usuario
  };
}
```

### 5. Null/Undefined no Gestionados
```
DETECTAR:
□ Acceso a propiedades sin verificar que el objeto existe
□ Llamadas a funciones que pueden devolver null/undefined sin verificar
□ Array.find() sin verificar que el resultado no sea undefined
□ JSON.parse() sin try/catch (puede lanzar SyntaxError)
□ req.body.x sin verificar que req.body existe (sin middleware de parsing)
```

**Ejemplos:**
```javascript
// ❌ BUG — find() puede devolver undefined:
const user = users.find(u => u.id === targetId);
console.log(user.name); // TypeError si no se encuentra

// ✅ FIX:
const user = users.find(u => u.id === targetId);
if (!user) throw new Error(`User ${targetId} not found`);
console.log(user.name);

// ❌ BUG — JSON.parse sin protección:
const config = JSON.parse(rawConfig); // SyntaxError si rawConfig es inválido

// ✅ FIX:
let config;
try {
  config = JSON.parse(rawConfig);
} catch {
  throw new Error('Invalid config format');
}
```

### 6. Lógica de Negocio Sospechosa
```
DETECTAR (señalar, no corregir sin confirmar):
□ Off-by-one errors en índices y rangos
□ Condiciones que parecen invertidas (< cuando parece que debería ser >)
□ Lógica AND/OR que podría ser incorrecta
□ Valores de retorno inconsistentes (a veces objeto, a veces null, a veces false)
□ Funciones que modifican estado Y devuelven valor (Command-Query separation)
```

**Ejemplo de señalización:**
```javascript
// ⚠️ POSIBLE BUG — condición potencialmente invertida:
function isExpired(token) {
  return token.expiresAt > Date.now(); // ¿no debería ser < ?
}
// → Señalar antes de corregir: "¿La intención es que expiresAt > now signifique expirado?"
```

### 7. Memory Leaks Comunes
```
DETECTAR:
□ Event listeners añadidos sin removeEventListener/off() en cleanup
□ setInterval sin clearInterval en cleanup
□ Closures que retienen referencias grandes innecesariamente
□ Cachés que crecen sin límite
□ Streams no cerrados tras su uso
```

**Ejemplos:**
```javascript
// ❌ BUG — event listener acumulado en cada llamada:
function setupHandler() {
  emitter.on('data', processData); // cada llamada añade un nuevo listener
}

// ✅ FIX:
function setupHandler() {
  emitter.removeAllListeners('data'); // limpiar antes de añadir
  emitter.on('data', processData);
}
// O mejor: llamar setupHandler solo una vez.

// ❌ BUG — stream no cerrado:
app.get('/download', (req, res) => {
  const stream = fs.createReadStream('./large-file.csv');
  stream.pipe(res);
  // Si la conexión se corta, el stream queda abierto
});

// ✅ FIX:
app.get('/download', (req, res) => {
  const stream = fs.createReadStream('./large-file.csv');
  stream.pipe(res);
  req.on('close', () => stream.destroy());
});
```

---

## FORMATO DE REPORTE POR HALLAZGO

```
🐛 [BUG] Tipo: Unhandled Promise / Mutación / Race Condition / etc.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Fichero/Línea: src/services/user.js:78
Severidad:     Alta / Media / Baja
Síntoma:       [Qué falla y cuándo]

CÓDIGO CON BUG:
  [fragmento]

EXPLICACIÓN:
  [Por qué es un bug y cuándo se manifiesta]

FIX APLICADO:
  [código corregido]
```
