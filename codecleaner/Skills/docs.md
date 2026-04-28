# SKILL: DOCS — Documentación de Código

## Objetivo
Añadir y mejorar la documentación del código: JSDoc/TSDoc para funciones y clases, comentarios explicativos donde hagan falta, y README cuando proceda. El objetivo es que cualquier desarrollador entienda el código sin necesidad de ejecutarlo ni preguntar al autor.

---

## FILOSOFÍA DE DOCUMENTACIÓN

**Documentar el "por qué", no el "qué".**
El código dice *qué* hace. Los comentarios deben decir *por qué* lo hace así.

```javascript
// ❌ MAL COMENTARIO — explica el "qué" (obvio):
// Increment counter by 1
counter++;

// ✅ BUEN COMENTARIO — explica el "por qué":
// Offset by 1 because the API uses 1-based pagination
const page = req.query.page + 1;
```

---

## CHECKLIST DE DOCUMENTACIÓN

### 1. JSDoc para Funciones

**Cuándo añadir JSDoc:**
- Funciones exportadas o de la API pública del módulo
- Funciones con parámetros no obvios (objetos complejos, flags booleanos)
- Funciones con comportamiento no evidente (side effects, throws)
- Funciones públicas de clases

**Cuándo NO es necesario JSDoc:**
- Funciones privadas/internas con nombres autodescriptivos y parámetros simples
- Arrow functions de una línea como callbacks
- Getters/setters simples

**Plantilla JSDoc:**
```javascript
/**
 * [Descripción de una línea de qué hace la función]
 *
 * [Descripción adicional si hace falta: comportamiento especial,
 *  efectos secundarios, casos edge, contexto de uso]
 *
 * @param {tipo} nombre - Descripción del parámetro
 * @param {Object} options - Opciones de configuración
 * @param {number} options.timeout - Tiempo máximo de espera en ms (default: 5000)
 * @param {boolean} [options.retry=false] - Si debe reintentar en caso de error
 * @returns {Promise<tipo>} Descripción de lo que devuelve
 * @throws {Error} Cuándo lanza error y por qué
 *
 * @example
 * const user = await getUser('123', { timeout: 3000 });
 */
async function getUser(id, options = {}) { ... }
```

**Tipos JSDoc más usados:**
```javascript
@param {string}           - string
@param {number}           - number
@param {boolean}          - boolean
@param {Object}           - objeto genérico
@param {Array<string>}    - array de strings
@param {string[]}         - array de strings (alternativa)
@param {string|number}    - union type
@param {?string}          - nullable string
@param {string} [name]    - parámetro opcional
@param {string} [name='default'] - opcional con default
@returns {Promise<void>}  - promesa que no devuelve valor
@returns {Object}         - objeto (especificar propiedades si no es obvio)
```

### 2. TSDoc para TypeScript

En TypeScript, el sistema de tipos ya documenta mucho. El TSDoc debe centrarse en semántica, no en tipos:

```typescript
/**
 * Procesa un pedido y genera la factura correspondiente.
 * 
 * Esta función es idempotente: llamarla múltiples veces con el mismo
 * orderId no crea facturas duplicadas (verifica existencia previa).
 *
 * @param orderId - ID único del pedido en el sistema
 * @param options - Opciones de generación
 * @throws {OrderNotFoundError} Si el pedido no existe
 * @throws {InvoiceExistsError} Si ya existe una factura para el pedido
 */
async function generateInvoice(orderId: string, options: InvoiceOptions): Promise<Invoice> { ... }
```

### 3. Comentarios Inline

**Añadir comentario cuando:**
- La lógica no es obvia a primera vista
- Hay un workaround o hack con una razón específica
- Hay una decisión de diseño que podría parecer incorrecta
- Hay un comportamiento del lenguaje/librería que no es evidente
- El código implementa un algoritmo no trivial

**Formato:**
```javascript
// Frase corta explicando el por qué, no el qué.

// Si necesita más contexto, usar múltiples líneas.
// Mantener el comentario junto al código que explica.
```

**Ejemplos:**
```javascript
// ✅ BUEN COMENTARIO — explica decisión no obvia:
// bcrypt.compare es resistente a timing attacks por diseño;
// no usar === para comparar contraseñas aunque parezca más simple.
const isValid = await bcrypt.compare(plaintext, hash);

// ✅ BUEN COMENTARIO — workaround documentado:
// Safari < 15 no soporta Intl.DisplayNames; caer al código ISO directamente.
const label = supportsDisplayNames
  ? new Intl.DisplayNames(['es'], { type: 'region' }).of(code)
  : code;

// ✅ BUEN COMENTARIO — algoritmo no trivial:
// Exponential backoff: esperar 2^attempt * 100ms, máximo 30s.
const delay = Math.min(Math.pow(2, attempt) * 100, 30_000);
```

### 4. Documentación de Módulos

Al inicio de cada fichero relevante, si no tiene encabezado:

```javascript
/**
 * @module auth
 * @description Middleware y utilidades de autenticación JWT.
 * 
 * Exporta:
 * - authenticate: middleware para rutas protegidas
 * - generateToken: crea JWT firmado con el payload del usuario
 * - refreshToken: renueva un token próximo a expirar
 */
```

### 5. Documentación de Clases

```javascript
/**
 * Gestiona la conexión y pool de conexiones a la base de datos.
 * 
 * Implementa el patrón Singleton: usar DatabaseManager.getInstance()
 * en lugar del constructor directamente.
 * 
 * @example
 * const db = DatabaseManager.getInstance();
 * await db.connect();
 * const users = await db.query('SELECT * FROM users');
 */
class DatabaseManager {
  /**
   * @private
   * @type {DatabaseManager}
   */
  static #instance = null;

  /**
   * Obtiene la instancia única del manager.
   * Crea la instancia en la primera llamada.
   * 
   * @returns {DatabaseManager}
   */
  static getInstance() { ... }
}
```

### 6. Documentación de Constantes y Enums

```javascript
/** Tiempo máximo de espera para peticiones externas (ms). */
const REQUEST_TIMEOUT_MS = 10_000;

/**
 * Estados posibles de un pedido en el flujo de trabajo.
 * @enum {string}
 */
const ORDER_STATUS = {
  /** Pedido creado, pendiente de confirmación de pago. */
  PENDING: 'pending',
  /** Pago confirmado, en preparación. */
  CONFIRMED: 'confirmed',
  /** Enviado al transportista. */
  SHIPPED: 'shipped',
  /** Entregado al destinatario. */
  DELIVERED: 'delivered',
  /** Cancelado por el usuario o por fallo de pago. */
  CANCELLED: 'cancelled',
};
```

### 7. README de Módulo/Directorio

Para directorios con lógica no trivial (p.ej. `/services`, `/middleware`, `/utils`), crear o actualizar un README.md:

```markdown
# [Nombre del módulo]

## Qué hace
[Descripción en 2-3 frases]

## Estructura
- `auth.js` — Middleware JWT y helpers de autenticación
- `validation.js` — Schemas de validación con Zod
- `errors.js` — Clases de error personalizadas

## Cómo usar
\`\`\`javascript
const { authenticate } = require('./middleware/auth');
app.get('/protected', authenticate, handler);
\`\`\`

## Decisiones de diseño
- [Por qué se eligió X sobre Y]
- [Cualquier constraint importante]
```

---

## CRITERIOS DE CALIDAD

Una buena documentación cumple:
- ✅ Cualquier dev del equipo entiende el propósito sin ejecutar el código
- ✅ Los parámetros no obvios están explicados
- ✅ Los casos de error están documentados
- ✅ Los side effects están declarados
- ✅ Hay un ejemplo si el uso no es evidente
- ❌ No repite lo que el nombre de la función ya dice
- ❌ No documenta el "qué" cuando el código es auto-explicativo

---

## FORMATO DE REPORTE

```
📝 [DOCS] Tipo: JSDoc / Comentario / README / Módulo
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Fichero/Línea: src/services/payment.js:34
Elemento:      función processPayment()
Razón:         Función exportada con parámetros complejos y throws sin documentar

DOCUMENTACIÓN AÑADIDA:
  [JSDoc o comentario añadido]
```
