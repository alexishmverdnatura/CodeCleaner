# SKILL: STRUCTURE — Análisis y Reorganización de Estructura

## Objetivo
Analizar la organización de ficheros y módulos de un proyecto Node.js/JS/TS y proponer (o aplicar) una estructura más clara, mantenible y coherente. Detectar módulos demasiado grandes, responsabilidades mezcladas y dependencias circulares.

---

## CUÁNDO ACTIVAR ESTA SKILL

- Ficheros individuales > 300 líneas
- Directorios con > 15 ficheros sin subdivisión
- El usuario ejecuta `clean:full` o `clean:structure`
- Se detectan múltiples responsabilidades en un módulo

---

## CHECKLIST DE ANÁLISIS

### 1. Tamaño de Ficheros
```
SEÑALES DE ALERTA:
□ Fichero > 300 líneas → candidato a dividir
□ Fichero > 500 líneas → dividir casi seguro
□ Un único fichero con > 10 exports → probablemente mezcla responsabilidades

PROCESO:
1. Listar todos los exports del fichero
2. Agrupar por responsabilidad/dominio
3. Proponer división en ficheros más pequeños
4. Crear index.js/barrel si hay múltiples ficheros del mismo módulo
```

### 2. Separación de Responsabilidades
```
PATRONES A DETECTAR:

Mezcla de capas:
□ Rutas con lógica de negocio inline (debería ir en services/controllers)
□ Controllers con queries directas a DB (debería ir en repositories/models)
□ Utils con lógica de negocio (debería estar en services)
□ Modelos con lógica de negocio compleja (thin models, fat services)

Señales en nombres de ficheros:
□ utils.js con > 5 funciones no relacionadas entre sí
□ helpers.js mezclando lógica de fechas, strings, y negocio
□ index.js con demasiada lógica (no solo re-exports)
```

**Estructura recomendada para APIs Express:**
```
src/
├── routes/          # Solo definición de rutas y middleware por ruta
│   ├── users.js
│   └── orders.js
├── controllers/     # Lógica HTTP: parsear req, llamar service, formatear res
│   ├── users.js
│   └── orders.js
├── services/        # Lógica de negocio pura (sin req/res)
│   ├── users.js
│   └── orders.js
├── repositories/    # Acceso a datos (queries, ORM calls)
│   ├── users.js
│   └── orders.js
├── models/          # Schemas y modelos de DB
│   ├── User.js
│   └── Order.js
├── middleware/      # Middleware de Express reutilizable
│   ├── auth.js
│   ├── validate.js
│   └── rateLimit.js
├── utils/           # Helpers puros sin dependencias de negocio
│   ├── date.js
│   ├── string.js
│   └── crypto.js
├── config/          # Configuración y variables de entorno
│   ├── database.js
│   └── index.js
└── app.js           # Setup de Express (sin lógica de negocio)
```

### 3. Dependencias Circulares
```
DETECTAR:
□ A importa de B, B importa de A
□ Cadenas de 3+ módulos que crean ciclo
□ index.js que importa de módulos que importan de index.js

CÓMO IDENTIFICAR:
- Buscar imports cruzados entre módulos del mismo nivel
- Si moduleA require('./moduleB') y moduleB require('./moduleA') → circular

SOLUCIONES:
1. Extraer la dependencia compartida a un tercer módulo
2. Inyección de dependencias (pasar como parámetro en lugar de importar)
3. Reorganizar en capas con dirección de dependencia clara (siempre hacia abajo)
```

### 4. Barrel Files (index.js)
```
CUÁNDO CREAR:
- Directorio con 3+ ficheros relacionados que se importan juntos frecuentemente
- Para simplificar imports externos al módulo

CUÁNDO NO USAR:
- Si solo hay 1-2 ficheros en el directorio
- Si cada fichero se importa de forma independiente

EJEMPLO:
// ❌ SIN BARREL — imports verbosos desde fuera:
import { createUser } from '../services/users/create';
import { updateUser } from '../services/users/update';
import { deleteUser } from '../services/users/delete';

// ✅ CON BARREL — services/users/index.js:
export { createUser } from './create';
export { updateUser } from './update';
export { deleteUser } from './delete';

// Import limpio desde fuera:
import { createUser, updateUser, deleteUser } from '../services/users';
```

### 5. Nombrado de Ficheros y Directorios
```
ESTÁNDARES A VERIFICAR y APLICAR:

Ficheros:
□ kebab-case para ficheros: user-service.js, not UserService.js (en JS puro)
□ PascalCase para clases/modelos: User.js, OrderController.js
□ camelCase evitarlo en ficheros (confuso en sistemas case-insensitive)
□ Nombres descriptivos: no utils.js → date-utils.js, string-helpers.js
□ Sin abreviaciones: usr.js → user.js, cfg.js → config.js

Directorios:
□ Singular para tipo: /service, /model, /controller (o plural consistente)
□ Consistencia en todo el proyecto
```

### 6. Configuración Centralizada
```
DETECTAR:
□ Variables de configuración definidas en múltiples sitios
□ process.env.X accedido directamente en múltiples módulos (en lugar de config central)
□ Valores por defecto definidos en múltiples lugares

PROPONER:
// config/index.js — único punto de acceso a configuración:
module.exports = {
  port: parseInt(process.env.PORT) || 3000,
  db: {
    url: process.env.DATABASE_URL,
    poolSize: parseInt(process.env.DB_POOL_SIZE) || 10,
  },
  jwt: {
    secret: process.env.JWT_SECRET,
    expiresIn: process.env.JWT_EXPIRES_IN || '1h',
  },
};

// En cualquier módulo:
const config = require('../config');
// No: process.env.JWT_SECRET directamente
```

---

## PROCESO DE REORGANIZACIÓN

Cuando se va a reorganizar estructura:

1. **Documentar estado actual** — listar ficheros y dependencias
2. **Proponer nueva estructura** — mostrar árbol antes de mover nada
3. **Confirmar con el usuario** antes de aplicar (cambios de estructura son disruptivos)
4. **Mover ficheros** uno a uno actualizando imports
5. **Verificar** que no quedan imports rotos
6. **Actualizar** package.json main/exports si afecta

---

## FORMATO DE REPORTE

```
📁 [STRUCTURE] Tipo: Fichero grande / Responsabilidades / Barrel / Nomenclatura
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Afecta:  src/utils.js (287 líneas, 12 exports de dominios distintos)
Razón:   Mezcla funciones de fecha, crypto, validación y formateo

PROPUESTA:
  src/utils/
  ├── date.js       (formatDate, parseDate, isExpired)
  ├── crypto.js     (generateToken, hashPassword)
  ├── validation.js (isEmail, isPhone, isURL)
  └── format.js     (formatCurrency, formatPhone, truncate)

⚠️ Este cambio requiere confirmar antes de aplicar.
```
