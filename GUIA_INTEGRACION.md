# 🔌 Guía de Integración - CUADERNO-WIN

Esta guía muestra cómo integrar todas las mejoras implementadas en la aplicación existente.

---

## 📦 1. Instalación de Dependencias

```bash
# Dependencias principales
npm install jspdf jspdf-autotable @radix-ui/react-tooltip

# Dependencias de desarrollo (para tests)
npm install -D vitest @testing-library/react @testing-library/jest-dom
```

---

## 🗂️ 2. Estructura de Archivos

Asegúrate de que la estructura sea:

```
src/
├── components/
│   ├── BackupManager.jsx       ✅ Nuevo
│   ├── LazyImage.jsx           ✅ Nuevo
│   ├── Tooltip.jsx             ✅ Nuevo
│   └── ... (componentes existentes)
├── hooks/
│   ├── useKeyboardShortcuts.js ✅ Nuevo
│   └── ... (hooks existentes)
├── utils/
│   ├── backupManager.js        ✅ Nuevo
│   ├── imageOptimizer.js       ✅ Nuevo
│   ├── pdfExporter.js          ✅ Nuevo
│   └── ... (utils existentes)
├── db/
│   └── database.js             (debe existir)
└── App.jsx
```

---

## 🚀 3. Integración en App.jsx

### Paso 1: Importar Dependencias

```jsx
// App.jsx
import { useEffect } from 'react';
import { backupManager } from './utils/backupManager';
import { useKeyboardShortcuts } from './hooks/useKeyboardShortcuts';
```

### Paso 2: Iniciar Sistema de Backups

```jsx
function App() {
    // Iniciar backups automáticos
    useEffect(() => {
        console.log('🔄 Iniciando sistema de backups...');
        backupManager.start();
        
        return () => {
            console.log('⏹️ Deteniendo sistema de backups...');
            backupManager.stop();
        };
    }, []);
    
    // ... resto del componente
}
```

### Paso 3: Configurar Atajos de Teclado Globales

```jsx
function App() {
    // ... código anterior
    
    // Atajos de teclado globales
    const globalShortcuts = [
        {
            key: 's',
            ctrl: true,
            action: () => {
                console.log('💾 Guardando...');
                // Implementar lógica de guardado
            },
            description: 'Guardar cambios'
        },
        {
            key: 'b',
            ctrl: true,
            shift: true,
            action: () => {
                console.log('📦 Creando backup...');
                backupManager.createBackup();
            },
            description: 'Crear backup manual'
        },
        {
            key: 'f',
            ctrl: true,
            action: () => {
                console.log('🔍 Buscando...');
                // Implementar lógica de búsqueda
            },
            description: 'Buscar'
        }
    ];
    
    useKeyboardShortcuts(globalShortcuts);
    
    return (
        <div className="app">
            {/* Tu contenido */}
        </div>
    );
}
```

---

## 📸 4. Integración de Optimización de Imágenes

### En Componente de Subida de Fotos

```jsx
// components/StudentPhotoUpload.jsx
import { useState } from 'react';
import { processImage, isValidImage } from '../utils/imageOptimizer';
import { LazyImage } from './LazyImage';

export function StudentPhotoUpload({ student, onPhotoChange }) {
    const [isUploading, setIsUploading] = useState(false);
    const [error, setError] = useState(null);
    
    async function handleFileSelect(event) {
        const file = event.target.files[0];
        if (!file) return;
        
        setIsUploading(true);
        setError(null);
        
        try {
            // Validar imagen
            isValidImage(file);
            
            // Procesar y comprimir
            const { photo, thumbnail } = await processImage(file, {
                maxWidth: 400,
                maxHeight: 400,
                quality: 0.85
            });
            
            // Actualizar estudiante
            onPhotoChange({
                photo,
                photoThumbnail: thumbnail
            });
            
            console.log('✅ Foto procesada exitosamente');
            
        } catch (err) {
            setError(err.message);
            console.error('❌ Error procesando foto:', err);
        } finally {
            setIsUploading(false);
        }
    }
    
    return (
        <div className="photo-upload">
            {/* Vista previa con lazy loading */}
            {student.photo && (
                <LazyImage
                    src={student.photo}
                    alt={student.name}
                    className="w-32 h-32 rounded-full object-cover"
                    placeholder={student.photoThumbnail}
                />
            )}
            
            {/* Input de archivo */}
            <input
                type="file"
                accept="image/jpeg,image/png,image/webp"
                onChange={handleFileSelect}
                disabled={isUploading}
                className="mt-2"
            />
            
            {/* Estado de carga */}
            {isUploading && (
                <p className="text-sm text-blue-600">
                    📸 Procesando imagen...
                </p>
            )}
            
            {/* Error */}
            {error && (
                <p className="text-sm text-red-600">
                    ❌ {error}
                </p>
            )}
        </div>
    );
}
```

---

## 📄 5. Integración de Exportación PDF

### Crear Componente de Exportación

```jsx
// components/ExportMenu.jsx
import { useState } from 'react';
import { PDFExporter } from '../utils/pdfExporter';
import { Download, FileText, BarChart } from 'lucide-react';
import { Tooltip } from './Tooltip';

export function ExportMenu({ students, settings }) {
    const [isExporting, setIsExporting] = useState(false);
    
    async function handleExportList() {
        setIsExporting(true);
        try {
            const exporter = new PDFExporter(settings);
            exporter.exportStudentList(students);
            console.log('✅ Lista exportada');
        } catch (error) {
            console.error('❌ Error exportando:', error);
            alert(`Error: ${error.message}`);
        } finally {
            setIsExporting(false);
        }
    }
    
    async function handleExportStudent(student) {
        setIsExporting(true);
        try {
            const exporter = new PDFExporter(settings);
            exporter.exportStudentCard(student);
            console.log(`✅ Ficha de ${student.name} exportada`);
        } catch (error) {
            console.error('❌ Error exportando:', error);
            alert(`Error: ${error.message}`);
        } finally {
            setIsExporting(false);
        }
    }
    
    async function handleExportStats() {
        setIsExporting(true);
        try {
            const exporter = new PDFExporter(settings);
            exporter.exportStatistics(students);
            console.log('✅ Estadísticas exportadas');
        } catch (error) {
            console.error('❌ Error exportando:', error);
            alert(`Error: ${error.message}`);
        } finally {
            setIsExporting(false);
        }
    }
    
    return (
        <div className="export-menu flex gap-2">
            <Tooltip content="Exportar lista completa">
                <button
                    onClick={handleExportList}
                    disabled={isExporting}
                    className="btn-primary"
                >
                    <FileText size={18} />
                    Lista Completa
                </button>
            </Tooltip>
            
            <Tooltip content="Exportar estadísticas">
                <button
                    onClick={handleExportStats}
                    disabled={isExporting}
                    className="btn-primary"
                >
                    <BarChart size={18} />
                    Estadísticas
                </button>
            </Tooltip>
        </div>
    );
}
```

---

## 💾 6. Integración de Gestión de Backups

### Añadir a Página de Configuración

```jsx
// pages/SettingsPage.jsx
import { BackupManagerComponent } from '../components/BackupManager';

export function SettingsPage() {
    return (
        <div className="settings-page">
            <h1>Configuración</h1>
            
            {/* Otras secciones de configuración */}
            
            {/* Sección de Backups */}
            <section className="mt-8">
                <BackupManagerComponent />
            </section>
        </div>
    );
}
```

---

## 🎨 7. Uso de Tooltips

### Ejemplo en Botones

```jsx
import { Tooltip, HelpTooltip } from './components/Tooltip';
import { Save, Trash2 } from 'lucide-react';

function MyComponent() {
    return (
        <div>
            {/* Tooltip simple */}
            <Tooltip content="Guardar cambios" side="top">
                <button className="btn-primary">
                    <Save size={18} />
                </button>
            </Tooltip>
            
            {/* Tooltip con icono de ayuda */}
            <HelpTooltip content="Los cambios se guardan automáticamente cada 30 segundos">
                <label>Nombre del estudiante</label>
            </HelpTooltip>
            
            {/* Tooltip con contenido complejo */}
            <Tooltip 
                content={
                    <div>
                        <strong>Eliminar estudiante</strong>
                        <p className="text-xs mt-1">
                            Esta acción no se puede deshacer
                        </p>
                    </div>
                }
                side="right"
            >
                <button className="btn-danger">
                    <Trash2 size={18} />
                </button>
            </Tooltip>
        </div>
    );
}
```

---

## 🖼️ 8. Uso de Lazy Loading

### En Lista de Estudiantes

```jsx
// components/StudentList.jsx
import { LazyImage } from './LazyImage';

export function StudentList({ students }) {
    return (
        <div className="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
            {students.map(student => (
                <div key={student.id} className="student-card">
                    {/* Imagen con lazy loading */}
                    <LazyImage
                        src={student.photo}
                        alt={student.name}
                        placeholder={student.photoThumbnail}
                        className="w-24 h-24 rounded-full object-cover"
                        onLoad={() => console.log(`Foto de ${student.name} cargada`)}
                    />
                    
                    <h3>{student.name}</h3>
                    <p>Nº {student.number}</p>
                </div>
            ))}
        </div>
    );
}
```

---

## ⌨️ 9. Atajos de Teclado por Componente

### Ejemplo en Modal

```jsx
// components/StudentModal.jsx
import { useEffect } from 'react';
import { useKeyboardShortcuts } from '../hooks/useKeyboardShortcuts';

export function StudentModal({ isOpen, onClose, onSave }) {
    const shortcuts = [
        {
            key: 'Escape',
            action: onClose,
            description: 'Cerrar modal'
        },
        {
            key: 'Enter',
            ctrl: true,
            action: onSave,
            description: 'Guardar y cerrar'
        }
    ];
    
    // Solo activar atajos cuando el modal está abierto
    useKeyboardShortcuts(shortcuts, isOpen);
    
    if (!isOpen) return null;
    
    return (
        <div className="modal">
            {/* Contenido del modal */}
            <div className="modal-footer">
                <button onClick={onClose}>
                    Cancelar <kbd>Esc</kbd>
                </button>
                <button onClick={onSave}>
                    Guardar <kbd>Ctrl+Enter</kbd>
                </button>
            </div>
        </div>
    );
}
```

---

## 🧪 10. Ejemplo de Test

### Test de Optimización de Imágenes

```javascript
// utils/imageOptimizer.test.js
import { describe, it, expect } from 'vitest';
import { isValidImage, getBase64Size } from './imageOptimizer';

describe('imageOptimizer', () => {
    describe('isValidImage', () => {
        it('acepta imágenes JPEG válidas', () => {
            const file = new File([''], 'test.jpg', { 
                type: 'image/jpeg',
                size: 1024 * 1024 // 1MB
            });
            
            expect(() => isValidImage(file)).not.toThrow();
        });
        
        it('rechaza archivos muy grandes', () => {
            const file = new File(
                [new ArrayBuffer(11 * 1024 * 1024)],
                'large.jpg',
                { type: 'image/jpeg' }
            );
            
            expect(() => isValidImage(file))
                .toThrow('demasiado grande');
        });
        
        it('rechaza formatos no válidos', () => {
            const file = new File([''], 'doc.pdf', { 
                type: 'application/pdf' 
            });
            
            expect(() => isValidImage(file))
                .toThrow('no válido');
        });
    });
    
    describe('getBase64Size', () => {
        it('calcula el tamaño correctamente', () => {
            const base64 = 'data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAYAAAAfFcSJAAAADUlEQVR42mNk+M9QDwADhgGAWjR9awAAAABJRU5ErkJggg==';
            const size = getBase64Size(base64);
            
            expect(parseFloat(size)).toBeGreaterThan(0);
        });
    });
});
```

---

## 📋 11. Checklist de Integración

### Antes de Empezar

- [ ] Hacer backup de la aplicación actual
- [ ] Crear rama de desarrollo: `git checkout -b feature/mejoras-v2`
- [ ] Instalar dependencias nuevas
- [ ] Verificar que la aplicación actual funciona

### Durante la Integración

- [ ] Copiar archivos nuevos a las carpetas correctas
- [ ] Importar componentes en App.jsx
- [ ] Iniciar sistema de backups
- [ ] Configurar atajos de teclado
- [ ] Probar cada funcionalidad individualmente

### Después de Integrar

- [ ] Ejecutar tests: `npm test`
- [ ] Verificar que no hay errores en consola
- [ ] Probar en modo desarrollo
- [ ] Compilar para producción: `npm run build`
- [ ] Probar aplicación compilada
- [ ] Crear backup antes de desplegar

---

## 🚨 Troubleshooting de Integración

### Error: "Cannot find module"

```bash
# Reinstalar dependencias
rm -rf node_modules package-lock.json
npm install
```

### Error: "db is not defined"

Asegúrate de que `database.js` existe y está correctamente configurado:

```javascript
// src/db/database.js
import Dexie from 'dexie';

export const db = new Dexie('CuadernoAula');

db.version(1).stores({
    settings: '++id',
    students: '++id, number, name'
});
```

### Error en Backups

```javascript
// Verificar localStorage disponible
if (typeof localStorage === 'undefined') {
    console.error('localStorage no disponible');
}

// Verificar espacio
try {
    localStorage.setItem('test', 'test');
    localStorage.removeItem('test');
} catch (e) {
    console.error('localStorage lleno o no disponible');
}
```

---

## 🎯 Próximos Pasos

1. **Integrar** los archivos siguiendo esta guía
2. **Probar** cada funcionalidad
3. **Ajustar** según necesidades específicas
4. **Documentar** cambios adicionales
5. **Desplegar** cuando esté listo

---

## 📞 Soporte

Si encuentras problemas durante la integración:

1. Revisa los logs en consola
2. Verifica que todas las dependencias están instaladas
3. Consulta `DESARROLLO.md` para troubleshooting
4. Revisa los ejemplos en esta guía

---

**Última actualización**: 2026-02-01  
**Versión**: 2.0.0
