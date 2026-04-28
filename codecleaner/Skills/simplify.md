# SKILL: SIMPLIFY — Reducción y Simplificación de Código

## Objetivo
Detectar y simplificar código innecesariamente complejo, duplicado o verboso. Aplicar principios DRY, KISS y buenas prácticas de JavaScript/TypeScript moderno para reducir tamaño y mejorar legibilidad sin cambiar comportamiento.

---

## PRINCIPIOS GUÍA

- **DRY** (Don't Repeat Yourself): si el mismo bloque aparece 2+ veces, extraer función
- **KISS** (Keep It Simple): la solución más simple que funcione
- **Early return**: reducir anidamiento con retornos tempranos
- **Modern JS**: usar las herramientas del lenguaje (optional chaining, nullish coalescing, etc.)
- **Legibilidad > Brevedad**: no comprimir a costa de claridad

---

## CHECKLIST DE SIMPLIFICACIÓN

### 1. Código Duplicado (DRY)
```
DETECTAR:
□ Bloques idénticos o casi idénticos en 2+ lugares
□ Condiciones repetidas en múltiples funciones
□ Misma lógica de transformación aplicada en varios sitios
□ Patrones copy-paste con pequeñas variaciones (parametrizables)
```

**Ejemplos:**
```javascript
// ❌ DUPLICADO — misma validación en 3 endpoints:
app.post('/users', (req, res) => {
  if (!req.body.email || !req.body.name) {
    return res.status(400).json({ error: 'Missing fields' });
  }
  // ...
});

app.post('/admin/users', (req, res) => {
  if (!req.body.email || !req.body.name) {
    return res.status(400).json({ error: 'Missing fields' });
  }
  // ...
});

// ✅ EXTRAER:
const requireFields = (...fields) => (req, res, next) => {
  const missing = fields.filter(f => !req.body[f]);
  if (missing.length) {
    return res.status(400).json({ error: `Missing fields: ${missing.join(', ')}` });
  }
  next();
};

app.post('/users', requireFields('email', 'name'), handler);
app.post('/admin/users', requireFields('email', 'name'), adminHandler);
```

### 2. Anidamiento Excesivo (Early Return)
```
DETECTAR:
□ Funciones con más de 3 niveles de indentación
□ if/else encadenados que podrían ser early returns
□ Callbacks anidados (callback hell) — convertir a async/await
□ Condicionales que envuelven el cuerpo principal de la función
```

**Ejemplos:**
```javascript
// ❌ ANIDAMIENTO EXCESIVO:
function processUser(user) {
  if (user) {
    if (user.isActive) {
      if (user.hasPermission) {
        return doSomething(user);
      } else {
        return { error: 'No permission' };
      }
    } else {
      return { error: 'Inactive user' };
    }
  } else {
    return { error: 'No user' };
  }
}

// ✅ EARLY RETURN:
function processUser(user) {
  if (!user) return { error: 'No user' };
  if (!user.isActive) return { error: 'Inactive user' };
  if (!user.hasPermission) return { error: 'No permission' };
  return doSomething(user);
}
```

### 3. JavaScript Moderno (ES6+)
```
DETECTAR y MODERNIZAR:
□ var → const/let
□ function() {} en callbacks → arrow functions (cuando no se usa 'this')
□ String concatenación con + → template literals
□ .then().catch() encadenado → async/await (cuando mejora legibilidad)
□ Object.assign({}, obj) → spread: { ...obj }
□ Array.prototype.concat() → spread: [...arr1, ...arr2]
□ if (x !== null && x !== undefined) → x != null (o optional chaining)
□ x && x.y && x.y.z → x?.y?.z (optional chaining)
□ x !== null && x !== undefined ? x : defaultVal → x ?? defaultVal
□ arguments object → rest parameters: ...args
□ Array(n).fill(0).map() → Array.from({length: n}, ...)
```

**Ejemplos:**
```javascript
// ❌ ANTIGUO:
var users = [];
var message = 'Hello, ' + user.name + '! You have ' + count + ' messages.';
var config = Object.assign({}, defaultConfig, userConfig);
var first = arr[0] !== undefined ? arr[0] : 'default';
var name = user && user.profile && user.profile.name;

// ✅ MODERNO:
const users = [];
const message = `Hello, ${user.name}! You have ${count} messages.`;
const config = { ...defaultConfig, ...userConfig };
const first = arr[0] ?? 'default';
const name = user?.profile?.name;

// ❌ THEN/CATCH verboso cuando async/await es más claro:
function getUser(id) {
  return db.findById(id)
    .then(user => {
      return formatUser(user);
    })
    .then(formatted => {
      return saveToCache(formatted);
    })
    .catch(err => {
      console.error(err);
      throw err;
    });
}

// ✅ ASYNC/AWAIT:
async function getUser(id) {
  try {
    const user = await db.findById(id);
    const formatted = formatUser(user);
    return saveToCache(formatted);
  } catch (err) {
    console.error(err);
    throw err;
  }
}
```

### 4. Condiciones Simplificables
```
DETECTAR:
□ Boolean traps: if (x === true) → if (x)
□ Ternarios innecesarios: x ? true : false → !!x o Boolean(x)
□ Ternarios que devuelven el propio valor: x ? x : y → x || y (con cuidado: || vs ??)
□ Comparaciones redundantes con booleanos
□ Negaciones dobles: !(!x) → x
□ Switch que puede ser objeto de mapeo
```

**Ejemplos:**
```javascript
// ❌ REDUNDANTE:
if (isValid === true) { ... }
if (error === false) { ... }
const result = condition ? true : false;
return arr.length > 0 ? true : false;

// ✅:
if (isValid) { ... }
if (!error) { ... }
const result = Boolean(condition); // o !!condition
return arr.length > 0;

// ❌ SWITCH como objeto de mapeo (cuando no hay lógica compleja):
function getStatusLabel(status) {
  switch (status) {
    case 'pending': return 'Pendiente';
    case 'active': return 'Activo';
    case 'inactive': return 'Inactivo';
    default: return 'Desconocido';
  }
}

// ✅:
const STATUS_LABELS = {
  pending: 'Pendiente',
  active: 'Activo',
  inactive: 'Inactivo',
};
const getStatusLabel = status => STATUS_LABELS[status] ?? 'Desconocido';
```

### 5. Funciones Largas (Single Responsibility)
```
DETECTAR:
□ Funciones de más de 40-50 líneas (señal de alerta)
□ Funciones que hacen más de una cosa claramente diferente
□ Funciones con muchos niveles de abstracción mezclados
□ Funciones con comentarios tipo "// Step 1:", "// Step 2:" (candidatas a extraer)

PROCESO:
1. Identificar las "secciones" de la función (separadas por línea en blanco o comentario)
2. Extraer cada sección como función auxiliar con nombre descriptivo
3. La función original queda como orquestadora de alto nivel
```

**Ejemplos:**
```javascript
// ❌ FUNCIÓN QUE HACE DEMASIADO:
async function handleRegistration(req, res) {
  // Validate input
  if (!req.body.email || !req.body.password) {
    return res.status(400).json({ error: 'Missing fields' });
  }
  if (req.body.password.length < 8) {
    return res.status(400).json({ error: 'Password too short' });
  }
  
  // Check if user exists
  const existing = await User.findOne({ email: req.body.email });
  if (existing) {
    return res.status(409).json({ error: 'Email already in use' });
  }
  
  // Hash password
  const salt = await bcrypt.genSalt(12);
  const hash = await bcrypt.hash(req.body.password, salt);
  
  // Create user
  const user = await User.create({ email: req.body.email, password: hash });
  
  // Send welcome email
  await mailer.send({ to: user.email, template: 'welcome', data: { name: user.name } });
  
  res.status(201).json({ id: user.id, email: user.email });
}

// ✅ EXTRAÍDO:
const validateRegistrationInput = (body) => {
  if (!body.email || !body.password) return 'Missing fields';
  if (body.password.length < 8) return 'Password too short';
  return null;
};

const hashPassword = (password) => bcrypt.hash(password, 12);

async function handleRegistration(req, res) {
  const validationError = validateRegistrationInput(req.body);
  if (validationError) return res.status(400).json({ error: validationError });

  const existing = await User.findOne({ email: req.body.email });
  if (existing) return res.status(409).json({ error: 'Email already in use' });

  const password = await hashPassword(req.body.password);
  const user = await User.create({ email: req.body.email, password });

  await mailer.send({ to: user.email, template: 'welcome' });
  res.status(201).json({ id: user.id, email: user.email });
}
```

### 6. Complejidad de Arrays y Objetos
```
DETECTAR:
□ Loops for/while que podrían ser .map/.filter/.reduce
□ Múltiples .filter().map() que podrían combinarse con .reduce() o un solo .map() con condición
□ Creación manual de objetos que podría ser Object.fromEntries()
□ Spread innecesario o duplicado
□ flatMap vs map+flat
```

**Ejemplos:**
```javascript
// ❌ FOR LOOP innecesario:
const names = [];
for (let i = 0; i < users.length; i++) {
  if (users[i].isActive) {
    names.push(users[i].name);
  }
}

// ✅:
const names = users.filter(u => u.isActive).map(u => u.name);

// ❌ REDUCE manual innecesario:
const byId = {};
users.forEach(user => {
  byId[user.id] = user;
});

// ✅:
const byId = Object.fromEntries(users.map(u => [u.id, u]));
```

### 7. Configuraciones y Constantes Inline
```
DETECTAR:
□ Números mágicos sin nombre (0.21, 86400, 1000 * 60 * 60)
□ Strings repetidos en múltiples lugares
□ URLs hardcodeadas inline
□ Configuración inline que debería ser constante nombrada
```

**Ejemplos:**
```javascript
// ❌ NÚMEROS MÁGICOS:
const tax = price * 0.21;
const expires = Date.now() + 86400000;
setTimeout(retry, 5000);

// ✅:
const TAX_RATE = 0.21;
const ONE_DAY_MS = 24 * 60 * 60 * 1000;
const RETRY_DELAY_MS = 5_000;

const tax = price * TAX_RATE;
const expires = Date.now() + ONE_DAY_MS;
setTimeout(retry, RETRY_DELAY_MS);
```

---

## FORMATO DE REPORTE POR HALLAZGO

```
✂️ [SIMPLIFY] Tipo: DRY / Early Return / Modern JS / Función larga / etc.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Fichero/Línea: src/controllers/user.js:45-89
Razón:         Función de 44 líneas que mezcla validación, DB y email

ANTES: [X líneas]
  [código original]

DESPUÉS: [Y líneas] (ahorro: Z líneas)
  [código simplificado]

Impacto: Legibilidad ↑ / Mantenibilidad ↑ / Líneas -XX
```
