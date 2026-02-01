# 🎓 CUADERNO-WIN - Cuaderno de Aula Digital

<div align="center">

![Version](https://img.shields.io/badge/version-2.0.0-blue.svg)
![Electron](https://img.shields.io/badge/Electron-Latest-47848F.svg)
![React](https://img.shields.io/badge/React-19.2-61DAFB.svg)
![License](https://img.shields.io/badge/license-MIT-green.svg)

**Aplicación de escritorio para gestión de cuadernos de aula**  
*CFGS Diseño y Amueblamiento - IES Ana Luisa Benítez*

[Características](#-características) •
[Instalación](#-instalación) •
[Uso](#-uso) •
[Desarrollo](#-desarrollo) •
[Documentación](#-documentación)

</div>

---

## 📋 Descripción

**CUADERNO-WIN** es una aplicación de escritorio moderna y completa para la gestión de cuadernos de aula en entornos de Formación Profesional. Construida con Electron y React, ofrece una experiencia fluida y offline-first.

### ✨ Características Principales

- 📝 **Gestión de Estudiantes** - Fichas completas con fotos y datos personales
- 📊 **Visualización de Datos** - Gráficos y estadísticas interactivas
- 💾 **Backups Automáticos** - Sistema de respaldo cada 24 horas
- 📄 **Exportación PDF** - Genera informes y fichas en PDF
- 🖼️ **Optimización de Imágenes** - Compresión automática de fotos
- 🌓 **Tema Claro/Oscuro** - Interfaz adaptable
- ⌨️ **Atajos de Teclado** - Navegación rápida
- 🔒 **100% Offline** - Sin necesidad de internet

---

## 🚀 Instalación

### Requisitos Previos

- Windows 10 o superior
- ~300 MB de espacio en disco

### Instalación Rápida

1. **Descargar** la última versión desde [Releases](releases)
2. **Ejecutar** `Cuaderno de Aula.exe`
3. ¡Listo! La aplicación está lista para usar

### Instalación para Desarrollo

```bash
# Clonar repositorio
git clone [URL]
cd CUADERNO-WIN

# Instalar dependencias
npm install

# Modo desarrollo
npm run dev
```

Ver [DESARROLLO.md](DESARROLLO.md) para más detalles.

---

## 💡 Uso

### Inicio Rápido

1. **Configurar Curso**
   - Ir a Configuración
   - Completar datos del centro y módulo
   - Subir logo institucional

2. **Añadir Estudiantes**
   - Click en "Nuevo Estudiante"
   - Completar datos
   - Subir foto (opcional)

3. **Gestionar Datos**
   - Ver lista de estudiantes
   - Editar información
   - Añadir observaciones

4. **Exportar**
   - Generar PDF de lista completa
   - Exportar fichas individuales
   - Descargar backups

### Atajos de Teclado

| Atajo | Acción |
|-------|--------|
| `Ctrl + S` | Guardar cambios |
| `Ctrl + N` | Nuevo estudiante |
| `Ctrl + F` | Buscar |
| `Ctrl + P` | Imprimir/Exportar PDF |
| `Ctrl + B` | Gestión de backups |

---

## 🛠️ Tecnologías

### Frontend
- **React 19.2** - Framework UI
- **Redux Toolkit** - Gestión de estado
- **Tailwind CSS** - Estilos
- **Lucide React** - Iconos
- **Recharts** - Gráficos
- **@dnd-kit** - Drag & Drop

### Backend
- **Electron** - Runtime de escritorio
- **Dexie** - Base de datos IndexedDB
- **jsPDF** - Generación de PDFs

### Build Tools
- **Vite** - Build tool
- **Electron Builder** - Empaquetado

---

## 📦 Estructura del Proyecto

```
CUADERNO-WIN/
├── src/
│   ├── components/      # Componentes React
│   ├── hooks/           # Custom hooks
│   ├── utils/           # Utilidades
│   ├── db/              # Base de datos
│   └── App.jsx          # App principal
├── public/              # Archivos estáticos
├── electron.js          # Proceso Electron
└── package.json         # Configuración
```

---

## 🔧 Mejoras Implementadas

### ✅ Fase 1 - Quick Wins (Completado)
- [x] Script de título mejorado con auto-detección
- [x] Validaciones robustas de errores
- [x] Sistema de logging completo

### ✅ Fase 2 - Core (Completado)
- [x] Optimización de imágenes (70-90% reducción)
- [x] Sistema de backup automático
- [x] Documentación completa del código

### ✅ Fase 3 - Avanzado (Completado)
- [x] Exportación a PDF
- [x] Tooltips y mejoras UX
- [x] Componentes con lazy loading

### ✅ Fase 4 - Opcional (Completado)
- [x] Atajos de teclado
- [x] Lazy loading de imágenes
- [x] Componentes de UI avanzados

---

## 📚 Documentación

- [📖 Guía de Desarrollo](DESARROLLO.md) - Setup y arquitectura
- [📋 Plan de Mejoras](PLAN_MEJORAS.md) - Roadmap completo
- [🔍 Análisis Técnico](analisis_cuaderno_win.md) - Análisis detallado

---

## 🔄 Backups

### Sistema Automático

- ✅ Backup cada 24 horas
- ✅ Mantiene últimos 7 backups
- ✅ Restauración con un click
- ✅ Descarga como JSON

### Crear Backup Manual

1. Ir a **Configuración** → **Backups**
2. Click en **Crear Backup Manual**
3. Esperar confirmación

### Restaurar Backup

1. Ir a **Configuración** → **Backups**
2. Seleccionar backup deseado
3. Click en **Restaurar**
4. Confirmar acción

---

## 📄 Exportación PDF

### Lista de Estudiantes

```javascript
import { PDFExporter } from './utils/pdfExporter';

const exporter = new PDFExporter(settings);
exporter.exportStudentList(students);
```

### Ficha Individual

```javascript
exporter.exportStudentCard(student);
```

### Estadísticas

```javascript
exporter.exportStatistics(students);
```

---

## 🎨 Personalización

### Cambiar Título de la Aplicación

Ejecutar el script mejorado:

```bash
Cambiar-Titulo-Mejorado.bat
```

El script:
- ✅ Detecta automáticamente la ubicación
- ✅ Valida requisitos (Node.js, npm)
- ✅ Crea backup automático
- ✅ Restaura en caso de error

### Temas

La aplicación soporta tema claro y oscuro:

```javascript
// Cambiar tema
document.documentElement.classList.toggle('dark');
```

---

## 🐛 Troubleshooting

### Problema: Aplicación no inicia

**Solución**:
1. Verificar logs en `%APPDATA%\cuaderno-win\app.log`
2. Reinstalar la aplicación
3. Contactar soporte

### Problema: Datos no se guardan

**Solución**:
1. Verificar permisos de escritura
2. Restaurar desde backup
3. Limpiar caché del navegador

### Problema: Fotos muy pesadas

**Solución**:
- Las fotos se comprimen automáticamente
- Tamaño máximo: 10 MB
- Formatos soportados: JPG, PNG, WebP

Ver [DESARROLLO.md](DESARROLLO.md#troubleshooting) para más soluciones.

---

## 🤝 Contribuir

Las contribuciones son bienvenidas! Ver [DESARROLLO.md](DESARROLLO.md#contribuir) para:

- Workflow de contribución
- Estándares de código
- Process de code review

---

## 📝 Changelog

### v2.0.0 (2026-02-01)

#### ✨ Nuevas Características
- Sistema de backups automáticos
- Exportación a PDF
- Optimización de imágenes
- Atajos de teclado
- Lazy loading de imágenes
- Tooltips informativos

#### 🔧 Mejoras
- Script de título mejorado
- Validaciones robustas
- Sistema de logging
- Documentación completa

#### 🐛 Correcciones
- Manejo de errores mejorado
- Prevención de múltiples instancias
- Recuperación de crashes

---

## 📞 Soporte

- 📧 **Email**: [email de soporte]
- 🐛 **Issues**: [GitHub Issues](issues)
- 📖 **Wiki**: [GitHub Wiki](wiki)

---

## 📄 Licencia

Este proyecto está bajo la Licencia MIT. Ver [LICENSE](LICENSE) para más detalles.

---

## 👥 Créditos

**Desarrollado para**:  
IES Ana Luisa Benítez  
CFGS Diseño y Amueblamiento

**Tecnologías**:  
React • Electron • Dexie • Tailwind CSS • Vite

---

<div align="center">

**⭐ Si te gusta este proyecto, dale una estrella en GitHub! ⭐**

Hecho con ❤️ para la comunidad educativa

</div>
