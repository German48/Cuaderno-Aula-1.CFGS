# Guía de Desarrollo - CUADERNO-WIN

## 📋 Tabla de Contenidos

1. [Requisitos del Sistema](#requisitos-del-sistema)
2. [Instalación](#instalación)
3. [Estructura del Proyecto](#estructura-del-proyecto)
4. [Arquitectura](#arquitectura)
5. [Convenciones de Código](#convenciones-de-código)
6. [Testing](#testing)
7. [Debugging](#debugging)
8. [Deployment](#deployment)
9. [Troubleshooting](#troubleshooting)
10. [Contribuir](#contribuir)

---

## 🔧 Requisitos del Sistema

### Para Desarrollo

- **Node.js** >= 18.0.0
- **npm** >= 9.0.0
- **Git** >= 2.0.0
- **Editor recomendado**: VS Code con extensiones:
  - ESLint
  - Prettier
  - Tailwind CSS IntelliSense
  - ES7+ React/Redux/React-Native snippets

### Para Compilación

- **Electron Builder**
- **Windows SDK** (para compilar en Windows)
- **Espacio en disco**: ~500 MB libres

---

## 📦 Instalación

### Clonar el Repositorio

```bash
git clone [URL_DEL_REPOSITORIO]
cd CUADERNO-WIN
```

### Instalar Dependencias

```bash
npm install
```

### Modo Desarrollo

```bash
# Iniciar servidor de desarrollo
npm run dev

# En otra terminal, iniciar Electron
npm run electron:dev
```

### Compilar para Producción

```bash
# Build del frontend
npm run build

# Empaquetar aplicación
npm run package

# Crear instalador
npm run dist
```

---

## 📁 Estructura del Proyecto

```
CUADERNO-WIN/
├── src/
│   ├── components/          # Componentes React
│   │   ├── BackupManager.jsx
│   │   ├── LazyImage.jsx
│   │   ├── Tooltip.jsx
│   │   └── ...
│   ├── hooks/               # Custom hooks
│   │   ├── useKeyboardShortcuts.js
│   │   └── ...
│   ├── utils/               # Utilidades
│   │   ├── backupManager.js
│   │   ├── imageOptimizer.js
│   │   ├── pdfExporter.js
│   │   └── ...
│   ├── db/                  # Configuración Dexie
│   │   └── database.js
│   ├── styles/              # Estilos globales
│   │   └── index.css
│   ├── App.jsx              # Componente principal
│   └── main.jsx             # Punto de entrada React
├── public/                  # Archivos estáticos
│   ├── icon.png
│   └── ...
├── electron.js              # Proceso principal Electron
├── package.json             # Configuración del proyecto
├── vite.config.js           # Configuración Vite
├── tailwind.config.js       # Configuración Tailwind
└── README.md                # Documentación
```

---

## 🏗️ Arquitectura

### Frontend

**React 19** con arquitectura basada en componentes:

- **Componentes**: UI reutilizables
- **Hooks**: Lógica compartida
- **Utils**: Funciones auxiliares
- **State Management**: Redux Toolkit

### Persistencia

**Dexie (IndexedDB)** para almacenamiento local:

```javascript
// db/database.js
import Dexie from 'dexie';

export const db = new Dexie('CuadernoAula');

db.version(1).stores({
    settings: '++id',
    students: '++id, number, name'
});
```

### Backend (Electron)

**Proceso Principal** (`electron.js`):
- Gestión de ventanas
- Comunicación IPC
- Acceso al sistema de archivos
- Manejo de eventos del sistema

---

## 📝 Convenciones de Código

### Nombres

```javascript
// Componentes: PascalCase
StudentCard.jsx
BackupManager.jsx

// Funciones y variables: camelCase
const handleSubmit = () => {};
const studentData = {};

// Constantes: UPPER_SNAKE_CASE
const MAX_STUDENTS = 100;
const API_URL = 'https://...';

// Archivos de utilidades: camelCase
imageOptimizer.js
backupManager.js
```

### Estructura de Componentes

```jsx
/**
 * Descripción del componente
 * 
 * @param {Object} props - Props del componente
 */
export function ComponentName({ prop1, prop2 }) {
    // 1. Hooks
    const [state, setState] = useState();
    useEffect(() => {}, []);
    
    // 2. Funciones auxiliares
    function handleAction() {}
    
    // 3. Render
    return (
        <div>
            {/* JSX */}
        </div>
    );
}
```

### Commits

Seguir [Conventional Commits](https://www.conventionalcommits.org/):

```bash
feat: añadir exportación a PDF
fix: corregir error en backup automático
docs: actualizar README
style: formatear código con Prettier
refactor: reorganizar componentes
test: añadir tests para imageOptimizer
chore: actualizar dependencias
```

---

## 🧪 Testing

### Setup

```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom
```

### Ejecutar Tests

```bash
# Todos los tests
npm test

# Watch mode
npm run test:watch

# Coverage
npm run test:coverage
```

### Ejemplo de Test

```javascript
// utils/imageOptimizer.test.js
import { describe, it, expect } from 'vitest';
import { isValidImage } from './imageOptimizer';

describe('imageOptimizer', () => {
    describe('isValidImage', () => {
        it('debe aceptar imágenes JPEG válidas', () => {
            const file = new File([''], 'test.jpg', { 
                type: 'image/jpeg' 
            });
            expect(() => isValidImage(file)).not.toThrow();
        });
        
        it('debe rechazar archivos muy grandes', () => {
            const largeFile = new File(
                [new ArrayBuffer(11 * 1024 * 1024)],
                'large.jpg',
                { type: 'image/jpeg' }
            );
            expect(() => isValidImage(largeFile))
                .toThrow('demasiado grande');
        });
    });
});
```

---

## 🐛 Debugging

### Modo Desarrollo

```javascript
// electron.js
if (isDev) {
    mainWindow.webContents.openDevTools();
}
```

**Atajos útiles**:
- `F12`: Abrir DevTools
- `Ctrl+R`: Recargar
- `Ctrl+Shift+I`: Inspeccionar elemento

### Logs

```javascript
// Desarrollo
console.log('Debug info:', data);

// Producción (electron.js)
const logPath = path.join(app.getPath('userData'), 'app.log');
fs.appendFileSync(logPath, message + '\n');
```

**Ubicación de logs**:
- Windows: `%APPDATA%/cuaderno-win/app.log`
- macOS: `~/Library/Application Support/cuaderno-win/app.log`
- Linux: `~/.config/cuaderno-win/app.log`

### Debugging de Base de Datos

```javascript
// Inspeccionar Dexie
import { db } from './db/database';

// Ver todos los estudiantes
const students = await db.students.toArray();
console.table(students);

// Limpiar base de datos
await db.students.clear();
```

---

## 🚀 Deployment

### Build de Producción

```bash
# 1. Limpiar builds anteriores
npm run clean

# 2. Build del frontend
npm run build

# 3. Verificar build
npm run preview

# 4. Empaquetar aplicación
npm run package
```

### Crear Instalador

```bash
# Windows
npm run dist:win

# macOS
npm run dist:mac

# Linux
npm run dist:linux
```

### Configuración de Electron Builder

```json
// package.json
{
  "build": {
    "appId": "com.cuaderno.aula",
    "productName": "Cuaderno de Aula",
    "directories": {
      "output": "dist-electron"
    },
    "files": [
      "dist/**/*",
      "electron.js",
      "package.json"
    ],
    "win": {
      "target": "nsis",
      "icon": "public/icon.ico"
    }
  }
}
```

---

## 🔍 Troubleshooting

### Problema: Vite no inicia

**Síntomas**: Error al ejecutar `npm run dev`

**Solución**:
```bash
# Verificar puerto 3001 libre
netstat -ano | findstr :3001

# Limpiar caché
npm run clean
rm -rf node_modules
npm install
```

### Problema: Base de datos corrupta

**Síntomas**: Errores al cargar datos

**Solución**:
```javascript
// Restaurar desde backup
import { backupManager } from './utils/backupManager';

const backups = backupManager.listBackups();
await backupManager.restoreBackup(backups[0].name);
```

### Problema: Imágenes no cargan

**Síntomas**: Fotos aparecen rotas

**Solución**:
```javascript
// Verificar formato base64
const isValidBase64 = (str) => {
    return str.startsWith('data:image/');
};

// Recomprimir imagen
import { processImage } from './utils/imageOptimizer';
const { photo } = await processImage(file);
```

### Problema: Electron no abre

**Síntomas**: Aplicación no inicia

**Solución**:
```bash
# Verificar logs
type %APPDATA%\cuaderno-win\app.log

# Reinstalar Electron
npm uninstall electron
npm install electron --save-dev
```

---

## 🤝 Contribuir

### Workflow

1. **Fork** del repositorio
2. **Crear rama** feature:
   ```bash
   git checkout -b feature/nueva-funcionalidad
   ```
3. **Commit** cambios:
   ```bash
   git commit -m "feat: añadir nueva funcionalidad"
   ```
4. **Push** a la rama:
   ```bash
   git push origin feature/nueva-funcionalidad
   ```
5. **Crear Pull Request**

### Code Review

Antes de crear PR:
- ✅ Tests pasan
- ✅ Código formateado
- ✅ Sin warnings de ESLint
- ✅ Documentación actualizada

### Estándares de Calidad

```bash
# Formatear código
npm run format

# Lint
npm run lint

# Type check (si usa TypeScript)
npm run type-check

# Build exitoso
npm run build
```

---

## 📚 Recursos Adicionales

### Documentación

- [React Docs](https://react.dev/)
- [Electron Docs](https://www.electronjs.org/docs)
- [Dexie.js](https://dexie.org/)
- [Tailwind CSS](https://tailwindcss.com/)

### Herramientas

- [React DevTools](https://react.dev/learn/react-developer-tools)
- [Redux DevTools](https://github.com/reduxjs/redux-devtools)
- [Electron DevTools](https://www.electronjs.org/docs/latest/tutorial/devtools-extension)

---

## 📞 Soporte

Para problemas o preguntas:
- 📧 Email: [email de soporte]
- 💬 Issues: [URL del repositorio]/issues
- 📖 Wiki: [URL del repositorio]/wiki

---

**Última actualización**: 2026-02-01  
**Versión**: 2.0.0
