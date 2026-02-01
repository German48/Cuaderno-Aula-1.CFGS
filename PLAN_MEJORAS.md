# Plan de Mejoras - CUADERNO-WIN

## 📊 Resumen Ejecutivo

Este documento detalla el plan completo de implementación de mejoras para la aplicación CUADERNO-WIN, priorizadas por impacto y esfuerzo.

---

## 🎯 Matriz de Priorización

| Mejora | Impacto | Esfuerzo | Prioridad | Fase |
|--------|---------|----------|-----------|------|
| Script de Título Mejorado | Alto | Bajo | 🔴 CRÍTICA | 1 |
| Validaciones de Errores | Alto | Medio | 🔴 CRÍTICA | 1 |
| Documentación Código | Alto | Alto | 🟡 ALTA | 2 |
| Optimización Imágenes | Medio | Medio | 🟡 ALTA | 2 |
| Backup Automático | Alto | Bajo | 🟡 ALTA | 2 |
| Exportación PDF | Medio | Alto | 🟢 MEDIA | 3 |
| Tests Unitarios | Alto | Alto | 🟢 MEDIA | 3 |
| Tooltips y UX | Medio | Bajo | 🟢 MEDIA | 3 |
| Atajos de Teclado | Medio | Medio | 🔵 BAJA | 4 |
| Lazy Loading Fotos | Bajo | Medio | 🔵 BAJA | 4 |
| Sincronización Nube | Bajo | Muy Alto | 🔵 OPCIONAL | 5 |
| Guía Desarrollo | Medio | Alto | 🔵 OPCIONAL | 5 |

---

## 🚀 FASE 1: Quick Wins Críticos (1-2 semanas)

### 1.1 Script de Título Mejorado

**Objetivo**: Hacer el script `Cambiar-Titulo.bat` más robusto y configurable

**Problemas actuales**:
- Ruta hardcodeada: `c:\\Users\\gmedc\\Descargas\\CUADERNO-WIN`
- Sin validación de errores robusta
- Sin detección automática de ubicación

**Implementación**:

```batch
@echo off
REM ============================================
REM Cambiar Titulo - Version Mejorada 2.0
REM ============================================

setlocal enabledelayedexpansion

title Cambiando Titulo...

echo.
echo ========================================
echo   Actualizando Titulo de la Aplicacion
echo ========================================
echo.

REM Detectar directorio actual del script
set "SCRIPT_DIR=%~dp0"
cd /d "%SCRIPT_DIR%"

echo [INFO] Directorio de trabajo: %SCRIPT_DIR%

REM Verificar que existe la aplicación
if not exist "resources\app.asar" (
    echo [ERROR] No se encuentra app.asar en: %SCRIPT_DIR%resources\
    echo [ERROR] Asegurese de ejecutar este script desde la carpeta CUADERNO-WIN
    pause
    exit /b 1
)

REM Verificar que Node.js está instalado
where node >nul 2>&1
if %ERRORLEVEL% neq 0 (
    echo [ERROR] Node.js no esta instalado
    echo [INFO] Descargue Node.js desde: https://nodejs.org/
    pause
    exit /b 1
)

REM Verificar que npm está disponible
where npm >nul 2>&1
if %ERRORLEVEL% neq 0 (
    echo [ERROR] npm no esta disponible
    pause
    exit /b 1
)

echo [1/5] Verificando herramienta asar...
call npm list -g asar >nul 2>&1
if %ERRORLEVEL% neq 0 (
    echo [INFO] Instalando herramienta asar...
    call npm install -g asar
    if %ERRORLEVEL% neq 0 (
        echo [ERROR] Fallo la instalacion de asar
        pause
        exit /b 1
    )
) else (
    echo [INFO] asar ya esta instalado
)

echo.
echo [2/5] Extrayendo app.asar...
if exist "resources\app" (
    echo [INFO] Limpiando extraccion anterior...
    rmdir /s /q "resources\app"
)

call asar extract "resources\app.asar" "resources\app"
if %ERRORLEVEL% neq 0 (
    echo [ERROR] Fallo la extraccion de app.asar
    pause
    exit /b 1
)

echo.
echo [3/5] Cambiando titulo...
if not exist "resources\app\index.html" (
    echo [ERROR] No se encuentra index.html en la extraccion
    pause
    exit /b 1
)

powershell -Command "(Get-Content 'resources\app\index.html') -replace 'Cuaderno de Aula - Prototipos', 'Cuaderno de Aula - Formación Profesional' | Set-Content 'resources\app\index.html'"
if %ERRORLEVEL% neq 0 (
    echo [ERROR] Fallo el cambio de titulo
    pause
    exit /b 1
)

echo.
echo [4/5] Creando backup...
if exist "resources\app.asar.backup" (
    echo [INFO] Eliminando backup anterior...
    del "resources\app.asar.backup"
)
move "resources\app.asar" "resources\app.asar.backup"
if %ERRORLEVEL% neq 0 (
    echo [ERROR] Fallo la creacion del backup
    pause
    exit /b 1
)

echo.
echo [5/5] Reempaquetando...
call asar pack "resources\app" "resources\app.asar"
if %ERRORLEVEL% neq 0 (
    echo [ERROR] Fallo el reempaquetado
    echo [INFO] Restaurando backup...
    move "resources\app.asar.backup" "resources\app.asar"
    pause
    exit /b 1
)

REM Limpiar carpeta temporal
echo [INFO] Limpiando archivos temporales...
rmdir /s /q "resources\app"

echo.
echo ========================================
echo   TITULO ACTUALIZADO EXITOSAMENTE
echo ========================================
echo.
echo Nuevo titulo: Cuaderno de Aula - Formación Profesional
echo.
echo Backup guardado en: resources\app.asar.backup
echo.
echo Ahora puedes abrir "Cuaderno de Aula.exe"
echo.

endlocal
pause
```

**Mejoras implementadas**:
- ✅ Detección automática de directorio
- ✅ Validación de Node.js y npm
- ✅ Verificación de asar instalado
- ✅ Manejo robusto de errores
- ✅ Limpieza de archivos temporales
- ✅ Restauración automática en caso de fallo
- ✅ Mensajes informativos detallados

**Tiempo estimado**: 2-3 horas

---

### 1.2 Validaciones de Errores en la Aplicación

**Objetivo**: Agregar validaciones robustas en el código principal

**Archivos a modificar**:
- `electron.js`
- Componentes React principales

**Implementación en `electron.js`**:

```javascript
const { app, BrowserWindow, dialog } = require('electron');
const path = require('path');
const { spawn } = require('child_process');
const fs = require('fs');

let mainWindow;
let viteProcess;

// Detectar si estamos en modo desarrollo o producción
const isDev = !app.isPackaged;

// Función de logging mejorada
function log(level, message) {
    const timestamp = new Date().toISOString();
    const logMessage = `[${timestamp}] [${level}] ${message}`;
    console.log(logMessage);
    
    // En producción, guardar logs en archivo
    if (!isDev) {
        const logPath = path.join(app.getPath('userData'), 'app.log');
        fs.appendFileSync(logPath, logMessage + '\n');
    }
}

// Manejo global de errores no capturados
process.on('uncaughtException', (error) => {
    log('ERROR', `Uncaught Exception: ${error.message}`);
    log('ERROR', error.stack);
    
    dialog.showErrorBox(
        'Error Crítico',
        `Ha ocurrido un error inesperado:\n\n${error.message}\n\nLa aplicación se cerrará.`
    );
    
    app.quit();
});

process.on('unhandledRejection', (reason, promise) => {
    log('ERROR', `Unhandled Rejection at: ${promise}, reason: ${reason}`);
});

function createWindow() {
    try {
        mainWindow = new BrowserWindow({
            width: 1400,
            height: 900,
            minWidth: 1024,
            minHeight: 768,
            title: 'Cuaderno de Aula',
            icon: path.join(__dirname, 'public', 'icon.png'),
            webPreferences: {
                nodeIntegration: false,
                contextIsolation: true,
                enableRemoteModule: false
            },
            autoHideMenuBar: true,
            backgroundColor: '#f9fafb',
            show: false // No mostrar hasta que esté listo
        });

        // Mostrar ventana cuando esté lista
        mainWindow.once('ready-to-show', () => {
            mainWindow.show();
            log('INFO', 'Ventana principal mostrada');
        });

        // Cargar la aplicación según el modo
        if (isDev) {
            log('INFO', 'Modo desarrollo: cargando desde Vite');
            mainWindow.loadURL('http://localhost:3001')
                .catch(err => {
                    log('ERROR', `Error cargando URL de desarrollo: ${err.message}`);
                    dialog.showErrorBox(
                        'Error de Desarrollo',
                        'No se pudo conectar al servidor de desarrollo.\nAsegúrese de que Vite esté ejecutándose.'
                    );
                });
        } else {
            const indexPath = path.join(__dirname, 'dist', 'index.html');
            
            if (!fs.existsSync(indexPath)) {
                log('ERROR', `No se encuentra index.html en: ${indexPath}`);
                dialog.showErrorBox(
                    'Error de Aplicación',
                    'No se encontraron los archivos de la aplicación.\nReinstale la aplicación.'
                );
                app.quit();
                return;
            }
            
            log('INFO', 'Modo producción: cargando archivos locales');
            mainWindow.loadFile(indexPath)
                .catch(err => {
                    log('ERROR', `Error cargando archivo: ${err.message}`);
                });
        }

        mainWindow.on('closed', () => {
            log('INFO', 'Ventana principal cerrada');
            mainWindow = null;
        });

        // Manejo de errores de renderizado
        mainWindow.webContents.on('crashed', () => {
            log('ERROR', 'El proceso de renderizado se ha bloqueado');
            dialog.showErrorBox(
                'Error de Renderizado',
                'La aplicación ha dejado de responder.\nSe reiniciará automáticamente.'
            );
            mainWindow.reload();
        });

    } catch (error) {
        log('ERROR', `Error creando ventana: ${error.message}`);
        throw error;
    }
}

function startViteServer() {
    return new Promise((resolve, reject) => {
        try {
            log('INFO', 'Iniciando servidor Vite...');
            
            viteProcess = spawn('npm', ['run', 'dev'], {
                cwd: __dirname,
                shell: true,
                stdio: 'pipe'
            });

            viteProcess.stdout.on('data', (data) => {
                const output = data.toString();
                log('VITE', output.trim());

                if (output.includes('Local:') || output.includes('localhost:3001')) {
                    log('INFO', 'Servidor Vite iniciado correctamente');
                    setTimeout(resolve, 1000);
                }
            });

            viteProcess.stderr.on('data', (data) => {
                log('VITE-ERROR', data.toString().trim());
            });

            viteProcess.on('error', (error) => {
                log('ERROR', `Error al iniciar Vite: ${error.message}`);
                reject(error);
            });

            viteProcess.on('exit', (code) => {
                if (code !== 0) {
                    log('ERROR', `Vite finalizó con código: ${code}`);
                }
            });

            // Timeout de seguridad
            setTimeout(() => {
                log('WARN', 'Timeout esperando a Vite, continuando...');
                resolve();
            }, 15000);

        } catch (error) {
            log('ERROR', `Error en startViteServer: ${error.message}`);
            reject(error);
        }
    });
}

app.whenReady().then(async () => {
    try {
        log('INFO', `Aplicación iniciada - Modo: ${isDev ? 'Desarrollo' : 'Producción'}`);
        log('INFO', `Versión de Electron: ${process.versions.electron}`);
        log('INFO', `Versión de Node: ${process.versions.node}`);
        
        if (isDev) {
            await startViteServer();
        }
        
        createWindow();
        
    } catch (error) {
        log('ERROR', `Error al iniciar la aplicación: ${error.message}`);
        log('ERROR', error.stack);
        
        dialog.showErrorBox(
            'Error de Inicio',
            `No se pudo iniciar la aplicación:\n\n${error.message}`
        );
        
        app.quit();
    }
});

app.on('window-all-closed', () => {
    log('INFO', 'Todas las ventanas cerradas');
    
    if (viteProcess) {
        log('INFO', 'Deteniendo proceso Vite...');
        viteProcess.kill();
    }
    
    if (process.platform !== 'darwin') {
        app.quit();
    }
});

app.on('activate', () => {
    if (mainWindow === null) {
        log('INFO', 'Reactivando aplicación');
        createWindow();
    }
});

app.on('before-quit', () => {
    log('INFO', 'Aplicación cerrándose...');
    
    if (viteProcess) {
        viteProcess.kill();
    }
});

// Manejo de segundo instancia
const gotTheLock = app.requestSingleInstanceLock();

if (!gotTheLock) {
    log('WARN', 'Ya existe una instancia de la aplicación');
    app.quit();
} else {
    app.on('second-instance', () => {
        if (mainWindow) {
            if (mainWindow.isMinimized()) mainWindow.restore();
            mainWindow.focus();
        }
    });
}
```

**Mejoras implementadas**:
- ✅ Sistema de logging completo
- ✅ Manejo de errores no capturados
- ✅ Validación de archivos existentes
- ✅ Diálogos de error informativos
- ✅ Prevención de múltiples instancias
- ✅ Recuperación automática de crashes
- ✅ Logs persistentes en producción

**Tiempo estimado**: 4-6 horas

---

## 📦 FASE 2: Optimización y Funcionalidad Core (2-3 semanas)

### 2.1 Optimización de Imágenes

**Objetivo**: Reducir el tamaño de las fotos en base64

**Estrategia**:
1. Comprimir imágenes antes de convertir a base64
2. Implementar diferentes calidades según uso
3. Crear utilidad de optimización

**Implementación**:

```javascript
// utils/imageOptimizer.js
/**
 * Optimizador de imágenes para CUADERNO-WIN
 * Comprime y redimensiona imágenes antes de guardarlas
 */

/**
 * Comprime una imagen y la convierte a base64
 * @param {File} file - Archivo de imagen
 * @param {Object} options - Opciones de compresión
 * @returns {Promise<string>} - Imagen en base64
 */
export async function compressImageToBase64(file, options = {}) {
    const {
        maxWidth = 400,
        maxHeight = 400,
        quality = 0.8,
        format = 'image/jpeg'
    } = options;

    return new Promise((resolve, reject) => {
        const reader = new FileReader();
        
        reader.onload = (e) => {
            const img = new Image();
            
            img.onload = () => {
                // Calcular nuevas dimensiones manteniendo aspect ratio
                let width = img.width;
                let height = img.height;
                
                if (width > maxWidth || height > maxHeight) {
                    const ratio = Math.min(maxWidth / width, maxHeight / height);
                    width = width * ratio;
                    height = height * ratio;
                }
                
                // Crear canvas y redimensionar
                const canvas = document.createElement('canvas');
                canvas.width = width;
                canvas.height = height;
                
                const ctx = canvas.getContext('2d');
                ctx.drawImage(img, 0, 0, width, height);
                
                // Convertir a base64 con compresión
                const base64 = canvas.toDataURL(format, quality);
                
                // Calcular reducción de tamaño
                const originalSize = e.target.result.length;
                const compressedSize = base64.length;
                const reduction = ((1 - compressedSize / originalSize) * 100).toFixed(2);
                
                console.log(`Imagen comprimida: ${reduction}% de reducción`);
                console.log(`Tamaño original: ${(originalSize / 1024).toFixed(2)} KB`);
                console.log(`Tamaño comprimido: ${(compressedSize / 1024).toFixed(2)} KB`);
                
                resolve(base64);
            };
            
            img.onerror = reject;
            img.src = e.target.result;
        };
        
        reader.onerror = reject;
        reader.readAsDataURL(file);
    });
}

/**
 * Valida que el archivo sea una imagen válida
 * @param {File} file - Archivo a validar
 * @returns {boolean}
 */
export function isValidImage(file) {
    const validTypes = ['image/jpeg', 'image/jpg', 'image/png', 'image/webp'];
    const maxSize = 10 * 1024 * 1024; // 10MB
    
    if (!validTypes.includes(file.type)) {
        throw new Error('Formato de imagen no válido. Use JPG, PNG o WebP.');
    }
    
    if (file.size > maxSize) {
        throw new Error('La imagen es demasiado grande. Máximo 10MB.');
    }
    
    return true;
}

/**
 * Crea una miniatura de la imagen
 * @param {string} base64 - Imagen en base64
 * @returns {Promise<string>} - Miniatura en base64
 */
export async function createThumbnail(base64) {
    return new Promise((resolve, reject) => {
        const img = new Image();
        
        img.onload = () => {
            const canvas = document.createElement('canvas');
            const size = 100; // Tamaño de miniatura
            
            canvas.width = size;
            canvas.height = size;
            
            const ctx = canvas.getContext('2d');
            
            // Calcular crop para mantener aspecto cuadrado
            const minDim = Math.min(img.width, img.height);
            const sx = (img.width - minDim) / 2;
            const sy = (img.height - minDim) / 2;
            
            ctx.drawImage(img, sx, sy, minDim, minDim, 0, 0, size, size);
            
            resolve(canvas.toDataURL('image/jpeg', 0.7));
        };
        
        img.onerror = reject;
        img.src = base64;
    });
}
```

**Uso en componentes**:

```javascript
import { compressImageToBase64, isValidImage, createThumbnail } from '../utils/imageOptimizer';

// En el componente de subida de foto
async function handlePhotoUpload(event) {
    const file = event.target.files[0];
    
    try {
        // Validar imagen
        isValidImage(file);
        
        // Comprimir para vista completa
        const fullPhoto = await compressImageToBase64(file, {
            maxWidth: 400,
            maxHeight: 400,
            quality: 0.85
        });
        
        // Crear miniatura para listados
        const thumbnail = await createThumbnail(fullPhoto);
        
        // Guardar ambas versiones
        updateStudent({
            photo: fullPhoto,
            photoThumbnail: thumbnail
        });
        
    } catch (error) {
        console.error('Error procesando imagen:', error);
        alert(error.message);
    }
}
```

**Beneficios**:
- ✅ Reducción de 70-90% en tamaño de imágenes
- ✅ Carga más rápida de la aplicación
- ✅ Menor uso de memoria
- ✅ Miniaturas para listados

**Tiempo estimado**: 6-8 horas

---

### 2.2 Backup Automático de Datos

**Objetivo**: Implementar sistema de backup automático

**Implementación**:

```javascript
// utils/backupManager.js
import { format } from 'date-fns';
import { db } from '../db/database';

/**
 * Gestor de backups automáticos
 */
class BackupManager {
    constructor() {
        this.backupInterval = null;
        this.backupFrequency = 24 * 60 * 60 * 1000; // 24 horas
        this.maxBackups = 7; // Mantener últimos 7 backups
    }

    /**
     * Inicia el sistema de backup automático
     */
    start() {
        // Hacer backup inmediato al iniciar
        this.createBackup();
        
        // Programar backups periódicos
        this.backupInterval = setInterval(() => {
            this.createBackup();
        }, this.backupFrequency);
        
        console.log('Sistema de backup automático iniciado');
    }

    /**
     * Detiene el sistema de backup automático
     */
    stop() {
        if (this.backupInterval) {
            clearInterval(this.backupInterval);
            this.backupInterval = null;
            console.log('Sistema de backup automático detenido');
        }
    }

    /**
     * Crea un backup de todos los datos
     */
    async createBackup() {
        try {
            const timestamp = format(new Date(), 'yyyy-MM-dd_HH-mm-ss');
            const backupName = `backup_${timestamp}`;
            
            // Exportar todos los datos
            const data = await this.exportAllData();
            
            // Guardar en localStorage (para backups rápidos)
            this.saveToLocalStorage(backupName, data);
            
            // Opcionalmente, descargar archivo
            // this.downloadBackup(backupName, data);
            
            // Limpiar backups antiguos
            this.cleanOldBackups();
            
            console.log(`Backup creado: ${backupName}`);
            
            return backupName;
            
        } catch (error) {
            console.error('Error creando backup:', error);
            throw error;
        }
    }

    /**
     * Exporta todos los datos de la aplicación
     */
    async exportAllData() {
        const settings = await db.settings.toArray();
        const students = await db.students.toArray();
        
        return {
            version: '2.0.0',
            timestamp: new Date().toISOString(),
            settings: settings[0] || {},
            students: students
        };
    }

    /**
     * Guarda backup en localStorage
     */
    saveToLocalStorage(name, data) {
        try {
            const compressed = JSON.stringify(data);
            localStorage.setItem(`backup_${name}`, compressed);
        } catch (error) {
            console.error('Error guardando backup en localStorage:', error);
            // Si localStorage está lleno, eliminar backups antiguos
            this.cleanOldBackups();
            // Reintentar
            localStorage.setItem(`backup_${name}`, JSON.stringify(data));
        }
    }

    /**
     * Descarga backup como archivo JSON
     */
    downloadBackup(name, data) {
        const blob = new Blob([JSON.stringify(data, null, 2)], {
            type: 'application/json'
        });
        
        const url = URL.createObjectURL(blob);
        const a = document.createElement('a');
        a.href = url;
        a.download = `${name}.json`;
        a.click();
        
        URL.revokeObjectURL(url);
    }

    /**
     * Limpia backups antiguos manteniendo solo los más recientes
     */
    cleanOldBackups() {
        const backupKeys = Object.keys(localStorage)
            .filter(key => key.startsWith('backup_'))
            .sort()
            .reverse();
        
        // Eliminar backups excedentes
        backupKeys.slice(this.maxBackups).forEach(key => {
            localStorage.removeItem(key);
            console.log(`Backup antiguo eliminado: ${key}`);
        });
    }

    /**
     * Lista todos los backups disponibles
     */
    listBackups() {
        return Object.keys(localStorage)
            .filter(key => key.startsWith('backup_'))
            .map(key => {
                const data = JSON.parse(localStorage.getItem(key));
                return {
                    name: key.replace('backup_', ''),
                    timestamp: data.timestamp,
                    studentsCount: data.students?.length || 0
                };
            })
            .sort((a, b) => new Date(b.timestamp) - new Date(a.timestamp));
    }

    /**
     * Restaura datos desde un backup
     */
    async restoreBackup(backupName) {
        try {
            const data = localStorage.getItem(`backup_${backupName}`);
            
            if (!data) {
                throw new Error('Backup no encontrado');
            }
            
            const backup = JSON.parse(data);
            
            // Confirmar con el usuario
            const confirmed = confirm(
                `¿Está seguro de restaurar el backup del ${new Date(backup.timestamp).toLocaleString()}?\n\n` +
                `Esto sobrescribirá los datos actuales.`
            );
            
            if (!confirmed) {
                return false;
            }
            
            // Crear backup de seguridad antes de restaurar
            await this.createBackup();
            
            // Restaurar datos
            await db.settings.clear();
            await db.students.clear();
            
            if (backup.settings) {
                await db.settings.add(backup.settings);
            }
            
            if (backup.students) {
                await db.students.bulkAdd(backup.students);
            }
            
            console.log(`Backup restaurado: ${backupName}`);
            
            // Recargar la aplicación
            window.location.reload();
            
            return true;
            
        } catch (error) {
            console.error('Error restaurando backup:', error);
            alert(`Error restaurando backup: ${error.message}`);
            return false;
        }
    }
}

export const backupManager = new BackupManager();
```

**Integración en la aplicación**:

```javascript
// En el componente principal o App.jsx
import { useEffect } from 'react';
import { backupManager } from './utils/backupManager';

function App() {
    useEffect(() => {
        // Iniciar sistema de backup al montar la aplicación
        backupManager.start();
        
        // Detener al desmontar
        return () => {
            backupManager.stop();
        };
    }, []);
    
    // ... resto del componente
}
```

**Componente de gestión de backups**:

```jsx
// components/BackupManager.jsx
import { useState, useEffect } from 'react';
import { backupManager } from '../utils/backupManager';
import { Download, Upload, Trash2 } from 'lucide-react';

export function BackupManager() {
    const [backups, setBackups] = useState([]);
    
    useEffect(() => {
        loadBackups();
    }, []);
    
    function loadBackups() {
        setBackups(backupManager.listBackups());
    }
    
    async function handleCreateBackup() {
        await backupManager.createBackup();
        loadBackups();
    }
    
    async function handleRestore(backupName) {
        await backupManager.restoreBackup(backupName);
    }
    
    function handleDownload(backupName) {
        const data = localStorage.getItem(`backup_${backupName}`);
        const backup = JSON.parse(data);
        backupManager.downloadBackup(backupName, backup);
    }
    
    return (
        <div className="p-6 bg-white dark:bg-slate-800 rounded-lg shadow">
            <h2 className="text-2xl font-bold mb-4">Gestión de Backups</h2>
            
            <button
                onClick={handleCreateBackup}
                className="mb-4 px-4 py-2 bg-primary text-white rounded hover:bg-primary/90"
            >
                Crear Backup Manual
            </button>
            
            <div className="space-y-2">
                {backups.map(backup => (
                    <div
                        key={backup.name}
                        className="flex items-center justify-between p-3 border rounded"
                    >
                        <div>
                            <div className="font-medium">{backup.name}</div>
                            <div className="text-sm text-gray-600">
                                {new Date(backup.timestamp).toLocaleString()} - 
                                {backup.studentsCount} estudiantes
                            </div>
                        </div>
                        
                        <div className="flex gap-2">
                            <button
                                onClick={() => handleDownload(backup.name)}
                                className="p-2 hover:bg-gray-100 rounded"
                                title="Descargar"
                            >
                                <Download size={20} />
                            </button>
                            
                            <button
                                onClick={() => handleRestore(backup.name)}
                                className="p-2 hover:bg-gray-100 rounded"
                                title="Restaurar"
                            >
                                <Upload size={20} />
                            </button>
                        </div>
                    </div>
                ))}
            </div>
        </div>
    );
}
```

**Beneficios**:
- ✅ Backups automáticos cada 24 horas
- ✅ Mantiene últimos 7 backups
- ✅ Restauración con un click
- ✅ Descarga de backups como JSON
- ✅ Backup de seguridad antes de restaurar

**Tiempo estimado**: 8-10 horas

---

### 2.3 Documentación del Código Fuente

**Objetivo**: Documentar completamente el código existente

**Estrategia**:
1. JSDoc para todas las funciones
2. README técnico
3. Comentarios explicativos
4. Diagramas de arquitectura

**Plantilla JSDoc**:

```javascript
/**
 * Descripción breve de la función
 * 
 * Descripción detallada de lo que hace la función,
 * casos de uso, consideraciones especiales, etc.
 * 
 * @param {Type} paramName - Descripción del parámetro
 * @param {Object} options - Opciones configurables
 * @param {number} options.maxWidth - Ancho máximo
 * @param {number} options.maxHeight - Alto máximo
 * @returns {Promise<string>} Descripción del valor retornado
 * @throws {Error} Cuándo y por qué lanza errores
 * 
 * @example
 * const result = await myFunction(param, { maxWidth: 100 });
 * console.log(result);
 */
```

**README Técnico** (crear `DESARROLLO.md`):

```markdown
# Guía de Desarrollo - CUADERNO-WIN

## Requisitos del Sistema

### Para Desarrollo
- Node.js >= 18.0.0
- npm >= 9.0.0
- Git

### Para Compilación
- Electron Builder
- Windows SDK (para compilar en Windows)

## Instalación

\`\`\`bash
# Clonar repositorio
git clone [URL]

# Instalar dependencias
npm install

# Modo desarrollo
npm run dev

# Compilar para producción
npm run build

# Empaquetar aplicación
npm run package
\`\`\`

## Estructura del Proyecto

\`\`\`
cuaderno-win/
├── src/
│   ├── components/      # Componentes React
│   ├── utils/           # Utilidades
│   ├── db/              # Configuración Dexie
│   ├── hooks/           # Custom hooks
│   └── App.jsx          # Componente principal
├── public/              # Archivos estáticos
├── electron.js          # Proceso principal Electron
└── package.json
\`\`\`

## Arquitectura

### Frontend
- **React 19** con hooks
- **Redux Toolkit** para estado global
- **Dexie** para persistencia local

### Backend
- **Electron** como runtime
- **IndexedDB** para almacenamiento

## Convenciones de Código

### Nombres
- Componentes: PascalCase (`StudentCard.jsx`)
- Funciones: camelCase (`handleSubmit`)
- Constantes: UPPER_SNAKE_CASE (`MAX_STUDENTS`)

### Commits
- feat: Nueva funcionalidad
- fix: Corrección de bug
- docs: Documentación
- style: Formato de código
- refactor: Refactorización
- test: Tests

## Testing

\`\`\`bash
# Ejecutar tests
npm test

# Coverage
npm run test:coverage
\`\`\`

## Debugging

### Modo Desarrollo
- DevTools disponibles: F12
- Hot reload automático
- Logs en consola

### Modo Producción
- Logs en: `%APPDATA%/cuaderno-win/app.log`
- Crash reports automáticos

## Deployment

### Compilar Release

\`\`\`bash
npm run build
npm run package:win
\`\`\`

### Distribución
1. Compilar aplicación
2. Crear instalador con Electron Builder
3. Firmar digitalmente (opcional)
4. Distribuir ejecutable

## Troubleshooting

### Problema: Vite no inicia
**Solución**: Verificar puerto 3001 libre

### Problema: Base de datos corrupta
**Solución**: Restaurar desde backup

## Contribuir

1. Fork del repositorio
2. Crear rama feature
3. Commit cambios
4. Push a la rama
5. Crear Pull Request
\`\`\`

**Tiempo estimado**: 12-16 horas

---

## 🎨 FASE 3: UX y Funcionalidades Avanzadas (3-4 semanas)

### 3.1 Exportación a PDF

**Objetivo**: Permitir exportar datos a PDF

**Implementación con jsPDF**:

```javascript
// utils/pdfExporter.js
import jsPDF from 'jspdf';
import 'jspdf-autotable';

export class PDFExporter {
    constructor(settings) {
        this.settings = settings;
        this.doc = new jsPDF();
    }

    /**
     * Exporta la lista completa de estudiantes
     */
    exportStudentList(students) {
        const { doc } = this;
        
        // Encabezado
        this.addHeader();
        
        // Título
        doc.setFontSize(16);
        doc.text('Lista de Estudiantes', 105, 40, { align: 'center' });
        
        // Tabla de estudiantes
        const tableData = students.map(student => [
            student.number,
            student.name,
            new Date(student.birthDate).toLocaleDateString(),
            student.repeater ? 'Sí' : 'No',
            student.pendingModules || '-'
        ]);
        
        doc.autoTable({
            startY: 50,
            head: [['Nº', 'Nombre', 'Fecha Nac.', 'Repetidor', 'Módulos Pendientes']],
            body: tableData,
            theme: 'grid',
            headStyles: { fillColor: [15, 118, 110] }
        });
        
        // Pie de página
        this.addFooter();
        
        // Guardar
        doc.save(`lista_estudiantes_${new Date().toISOString().split('T')[0]}.pdf`);
    }

    /**
     * Exporta ficha individual de estudiante
     */
    exportStudentCard(student) {
        const { doc } = this;
        
        this.addHeader();
        
        // Foto del estudiante
        if (student.photo) {
            try {
                doc.addImage(student.photo, 'JPEG', 15, 40, 40, 40);
            } catch (error) {
                console.error('Error añadiendo foto:', error);
            }
        }
        
        // Datos del estudiante
        doc.setFontSize(14);
        doc.text('Ficha de Estudiante', 105, 45, { align: 'center' });
        
        const data = [
            ['Número', student.number],
            ['Nombre', student.name],
            ['Fecha de Nacimiento', new Date(student.birthDate).toLocaleDateString()],
            ['Repetidor', student.repeater ? 'Sí' : 'No'],
            ['Módulos Pendientes', student.pendingModules || 'Ninguno']
        ];
        
        doc.autoTable({
            startY: 90,
            body: data,
            theme: 'plain',
            columnStyles: {
                0: { fontStyle: 'bold', cellWidth: 60 },
                1: { cellWidth: 120 }
            }
        });
        
        // Observaciones
        if (student.observations) {
            const finalY = doc.lastAutoTable.finalY + 10;
            doc.setFontSize(12);
            doc.text('Observaciones:', 15, finalY);
            doc.setFontSize(10);
            const lines = doc.splitTextToSize(student.observations, 180);
            doc.text(lines, 15, finalY + 7);
        }
        
        this.addFooter();
        
        doc.save(`ficha_${student.name.replace(/\s+/g, '_')}.pdf`);
    }

    /**
     * Añade encabezado al PDF
     */
    addHeader() {
        const { doc, settings } = this;
        
        // Logo si existe
        if (settings.instituteLogo) {
            try {
                doc.addImage(settings.instituteLogo, 'PNG', 15, 10, 30, 20);
            } catch (error) {
                console.error('Error añadiendo logo:', error);
            }
        }
        
        // Información del centro
        doc.setFontSize(10);
        doc.text(settings.instituteName || '', 50, 15);
        doc.text(settings.department || '', 50, 20);
        doc.text(`Curso: ${settings.academicYear || ''}`, 50, 25);
    }

    /**
     * Añade pie de página
     */
    addFooter() {
        const { doc } = this;
        const pageCount = doc.internal.getNumberOfPages();
        
        for (let i = 1; i <= pageCount; i++) {
            doc.setPage(i);
            doc.setFontSize(8);
            doc.text(
                `Página ${i} de ${pageCount}`,
                105,
                290,
                { align: 'center' }
            );
            doc.text(
                `Generado el ${new Date().toLocaleString()}`,
                105,
                295,
                { align: 'center' }
            );
        }
    }
}
```

**Tiempo estimado**: 10-12 horas

---

### 3.2 Tests Unitarios

**Objetivo**: Implementar suite de tests

**Setup con Vitest**:

```bash
npm install -D vitest @testing-library/react @testing-library/jest-dom
```

**Configuración** (`vitest.config.js`):

```javascript
import { defineConfig } from 'vitest/config';
import react from '@vitejs/plugin-react';

export default defineConfig({
    plugins: [react()],
    test: {
        globals: true,
        environment: 'jsdom',
        setupFiles: './src/test/setup.js',
        coverage: {
            provider: 'v8',
            reporter: ['text', 'json', 'html'],
            exclude: [
                'node_modules/',
                'src/test/',
            ]
        }
    }
});
```

**Ejemplos de tests**:

```javascript
// utils/imageOptimizer.test.js
import { describe, it, expect } from 'vitest';
import { isValidImage } from './imageOptimizer';

describe('imageOptimizer', () => {
    describe('isValidImage', () => {
        it('debe aceptar imágenes JPEG válidas', () => {
            const file = new File([''], 'test.jpg', { type: 'image/jpeg' });
            expect(() => isValidImage(file)).not.toThrow();
        });
        
        it('debe rechazar archivos muy grandes', () => {
            const largeFile = new File(
                [new ArrayBuffer(11 * 1024 * 1024)],
                'large.jpg',
                { type: 'image/jpeg' }
            );
            expect(() => isValidImage(largeFile)).toThrow('demasiado grande');
        });
        
        it('debe rechazar formatos no válidos', () => {
            const file = new File([''], 'test.pdf', { type: 'application/pdf' });
            expect(() => isValidImage(file)).toThrow('no válido');
        });
    });
});
```

**Tiempo estimado**: 16-20 horas

---

### 3.3 Tooltips y Mejoras UX

**Objetivo**: Mejorar la experiencia de usuario

**Implementación con Radix UI**:

```bash
npm install @radix-ui/react-tooltip
```

```jsx
// components/Tooltip.jsx
import * as TooltipPrimitive from '@radix-ui/react-tooltip';

export function Tooltip({ children, content, side = 'top' }) {
    return (
        <TooltipPrimitive.Provider delayDuration={300}>
            <TooltipPrimitive.Root>
                <TooltipPrimitive.Trigger asChild>
                    {children}
                </TooltipPrimitive.Trigger>
                
                <TooltipPrimitive.Portal>
                    <TooltipPrimitive.Content
                        side={side}
                        className="bg-slate-900 text-white px-3 py-2 rounded text-sm shadow-lg"
                        sideOffset={5}
                    >
                        {content}
                        <TooltipPrimitive.Arrow className="fill-slate-900" />
                    </TooltipPrimitive.Content>
                </TooltipPrimitive.Portal>
            </TooltipPrimitive.Root>
        </TooltipPrimitive.Provider>
    );
}
```

**Tiempo estimado**: 6-8 horas

---

## 🔵 FASE 4: Funcionalidades Opcionales (4-6 semanas)

### 4.1 Atajos de Teclado

**Implementación**:

```javascript
// hooks/useKeyboardShortcuts.js
import { useEffect } from 'react';

export function useKeyboardShortcuts(shortcuts) {
    useEffect(() => {
        function handleKeyDown(event) {
            const key = event.key.toLowerCase();
            const ctrl = event.ctrlKey;
            const shift = event.shiftKey;
            const alt = event.altKey;
            
            const shortcut = shortcuts.find(s => 
                s.key === key &&
                s.ctrl === ctrl &&
                s.shift === shift &&
                s.alt === alt
            );
            
            if (shortcut) {
                event.preventDefault();
                shortcut.action();
            }
        }
        
        window.addEventListener('keydown', handleKeyDown);
        return () => window.removeEventListener('keydown', handleKeyDown);
    }, [shortcuts]);
}

// Uso:
const shortcuts = [
    { key: 's', ctrl: true, action: () => saveData() },
    { key: 'n', ctrl: true, action: () => createNew() },
    { key: 'f', ctrl: true, action: () => focusSearch() }
];

useKeyboardShortcuts(shortcuts);
```

**Tiempo estimado**: 8-10 horas

---

### 4.2 Lazy Loading de Fotos

**Implementación**:

```jsx
// components/LazyImage.jsx
import { useState, useEffect, useRef } from 'react';

export function LazyImage({ src, alt, placeholder, className }) {
    const [imageSrc, setImageSrc] = useState(placeholder);
    const [isLoading, setIsLoading] = useState(true);
    const imgRef = useRef();
    
    useEffect(() => {
        const observer = new IntersectionObserver(
            (entries) => {
                entries.forEach(entry => {
                    if (entry.isIntersecting) {
                        setImageSrc(src);
                        observer.disconnect();
                    }
                });
            },
            { threshold: 0.1 }
        );
        
        if (imgRef.current) {
            observer.observe(imgRef.current);
        }
        
        return () => observer.disconnect();
    }, [src]);
    
    return (
        <img
            ref={imgRef}
            src={imageSrc}
            alt={alt}
            className={`${className} ${isLoading ? 'blur-sm' : ''}`}
            onLoad={() => setIsLoading(false)}
        />
    );
}
```

**Tiempo estimado**: 4-6 horas

---

## 🌐 FASE 5: Funcionalidades Avanzadas Opcionales (6-8 semanas)

### 5.1 Sincronización en Nube

**Advertencia**: Esta es la funcionalidad más compleja y requiere:
- Backend (Firebase, Supabase, o custom)
- Autenticación de usuarios
- Manejo de conflictos
- Seguridad de datos

**Tiempo estimado**: 40-60 horas

---

### 5.2 Guía de Desarrollo Completa

**Crear documentación exhaustiva**:
- Arquitectura detallada
- Diagramas de flujo
- API documentation
- Deployment guides

**Tiempo estimado**: 20-30 horas

---

## 📊 Resumen de Tiempos

| Fase | Duración Estimada | Esfuerzo (horas) |
|------|-------------------|------------------|
| Fase 1 | 1-2 semanas | 6-9 horas |
| Fase 2 | 2-3 semanas | 26-34 horas |
| Fase 3 | 3-4 semanas | 32-40 horas |
| Fase 4 | 4-6 semanas | 12-16 horas |
| Fase 5 | 6-8 semanas | 60-90 horas |
| **TOTAL** | **16-23 semanas** | **136-189 horas** |

---

## 🎯 Recomendación de Implementación

### Prioridad Inmediata (Hacer Ya)
1. ✅ Script de Título Mejorado
2. ✅ Validaciones de Errores
3. ✅ Backup Automático

### Prioridad Alta (Próximas 2-4 semanas)
4. ✅ Optimización de Imágenes
5. ✅ Documentación Básica

### Prioridad Media (1-2 meses)
6. ✅ Exportación PDF
7. ✅ Tooltips y UX
8. ✅ Tests Unitarios

### Prioridad Baja (Evaluar necesidad)
9. ⚠️ Atajos de Teclado
10. ⚠️ Lazy Loading
11. ⚠️ Sincronización Nube

---

## 📝 Notas Finales

- Cada mejora es independiente y puede implementarse por separado
- Se recomienda seguir el orden de prioridades
- Hacer commits frecuentes y crear backups antes de cambios mayores
- Probar exhaustivamente cada mejora antes de pasar a la siguiente
- Documentar mientras se desarrolla, no después

---

**Última actualización**: 2026-02-01
**Versión del documento**: 1.0
