# 📱 Plan de Implementación: **Studio Mochi 22px**
Aplicación multiplataforma para gestión y visualización de un estudio fotográfico.

---

## 🛠 Herramientas Requeridas
| Categoría | Herramienta | Propósito |
|-----------|-------------|-----------|
| **SDK & Lenguaje** | Flutter SDK + Dart SDK | Framework y lenguaje base |
| **IDE Principal** | VS Code (recomendado) | Desarrollo, depuración, integración con Firebase |
| **IDE Alternativo** | Antigravity | Editor de texto plano (compatible, pero sin soporte nativo Flutter; se recomienda usar solo como editor secundario) |
| **Backend** | Firebase Console + Firebase CLI | Autenticación, Firestore, Storage, Hosting, reglas de seguridad |
| **Diseño UI/UX** | Figma / Penpot | Prototipado, sistema de diseño, exportación de assets |
| **Control de Versiones** | Git + GitHub/GitLab | Historial, ramas, CI/CD básico |
| **Emulación/Pruebas** | Android Studio (emuladores), Xcode (simuladores iOS), Chrome/Edge (web) | Validación multiplataforma |
| **Monitoreo & Depuración** | Flutter DevTools, Sentry (opcional) | Performance, logs, tracking de errores |

---

## 🎨 Consideraciones UI/UX (Contexto: Estudio Fotográfico)
- **Enfoque visual**: Las fotografías deben ser el centro. Interfaz minimalista, fondos neutros (blanco, gris oscuro, negro mate) para evitar distracciones.
- **Tipografía**: Sans-serif moderna (ej. `Inter`, `Montserrat`, `SF Pro`). Escala tipográfica clara para jerarquía (títulos, metadatos, botones).
- **Componentes clave**: 
  - Grid responsivo para galerías (masonry o fixed columns según plataforma)
  - Skeletons/shimmers durante carga de imágenes
  - Viewers con zoom, deslizamiento táctil y metadatos (EXIF, fecha, sesión)
  - Formularios de acceso limpios con validación visual inmediata
- **Navegación**: Bottom Navigation (móvil) / NavigationRail (tablet & web). Rutas protegidas post-autenticación.
- **Accesibilidad**: Contraste mínimo 4.5:1, soporte para modo oscuro, etiquetas semánticas, navegación por teclado/lector de pantalla.
- **Rendimiento visual**: Lazy loading, compresión adaptativa, caché inteligente para evitar recargas innecesarias.

---

## 📦 Dependencias Clave (`pubspec.yaml`)
*(Listado referencial. Se instalarán con `flutter pub add`)*
- `firebase_core`
- `firebase_auth`
- `cloud_firestore`
- `firebase_storage` *(imprescindible para fotos, aunque no se mencionó explícitamente)*
- `provider`
- `go_router`
- `cached_network_image`
- `flutter_spinkit` o `shimmer`
- `intl`
- `flutter_dotenv` o `envied`
- `image_picker` / `file_picker` *(si se habilita subida desde cliente)*
- `uuid` *(para IDs locales o metadatos)*

---

## 🗺 Procedimiento Paso a Paso

### 🔹 Fase 0: Definición de Arquitectura y Alcance
1. Establecer arquitectura **Feature-First + MVVM ligero**.
2. Definir módulos: `auth`, `gallery`, `sessions`, `profile`, `core`.
3. Crear estructura de carpetas en `lib/` separando `models`, `services`, `providers`, `screens`, `widgets`, `utils`.
4. Documentar roles de usuario (cliente, fotógrafo, administrador) y permisos por colección en Firestore.

### 🔹 Fase 1: Configuración del Entorno y Proyecto
1. Instalar/actualizar Flutter y Dart. Verificar con `flutter doctor`.
2. Configurar VS Code con extensiones oficiales: Flutter, Dart, Firebase, Error Lens, Pubspec Assist.
3. Crear proyecto: `flutter create studio_mochi_22px --platforms=android,ios,web`.
4. Inicializar Git y crear rama `main` + `develop`.
5. Registrar app en Firebase Console (Android, iOS, Web). Descargar archivos de configuración y colocarlos en rutas correspondientes.
6. Validar compilación limpia en las 3 plataformas objetivo.

### 🔹 Fase 2: Base de Dependencias y Routing
1. Agregar dependencias listadas en `pubspec.yaml`.
2. Configurar `go_router` con rutas públicas (login, registro) y rutas protegidas (dashboard, galería).
3. Implementar interceptor de redirección: si no hay sesión activa → redirigir a login.
4. Configurar variables de entorno (Firebase config, claves de proyecto) usando `.env` seguro.
5. Verificar que el árbol de dependencias compile sin conflictos.

### 🔹 Fase 3: Sistema de Diseño y UI Base
1. Crear wireframes en Figma: Login, Registro, Recuperar contraseña, Dashboard, Galería, Detalle de foto, Perfil.
2. Definir `ThemeData` global: paleta, tipografía, espaciados, radios, sombras.
3. Construir widgets reutilizables: `AppButton`, `AppTextField`, `PhotoCard`, `LoadingOverlay`, `ErrorBanner`.
4. Implementar navegación base y estructura de pantallas vacías (placeholders).
5. Validar responsividad en móvil, tablet y web con `LayoutBuilder` y `MediaQuery`.

### 🔹 Fase 4: Autenticación (Email/Password)
1. Habilitar provider de Email/Password en Firebase Console.
2. Crear `AuthModel` (uid, email, displayName, role, createdAt).
3. Implementar `AuthProvider` con `ChangeNotifier`: métodos `signIn`, `register`, `signOut`, `resetPassword`, `checkAuthState`.
4. Conectar formularios de UI al provider. Manejar estados: `loading`, `success`, `error` (con mensajes traducidos).
5. Validar reglas de seguridad de Firebase Auth: solo usuarios autenticados acceden a recursos.
6. Probar flujo completo: registro → verificación email (opcional) → login → cierre de sesión → recuperación.

### 🔹 Fase 5: Gestión de Estado con Provider
1. Registrar `MultiProvider` en `main.dart` con: `AuthProvider`, `GalleryProvider`, `SessionProvider`, `ProfileProvider`.
2. Definir contratos claros entre UI y lógica: la UI solo escucha, el provider gestiona datos y llama a servicios.
3. Implementar manejo de errores centralizado (try/catch → estado `error` → UI muestra `SnackBar` o diálogo).
4. Añadir indicadores de carga coherentes (shimmer en grids, spinner en acciones).
5. Validar que los cambios de estado no provoquen rebuilds innecesarios (usar `Consumer`, `Selector` o `context.watch` selectivo).

### 🔹 Fase 6: Base de Datos (Firestore)
1. Diseñar esquema de colecciones:
   - `users`: perfil, preferencias, rol
   - `sessions`: nombre, fecha, cliente, fotógrafo, estado
   - `photos`: url (Storage), thumbnail, metadata, sesión_id, visibilidad
2. Crear servicios de repositorio (`FirestoreAuthService`, `FirestoreGalleryService`) que abstraigan `cloud_firestore`.
3. Implementar consultas paginadas (`startAfterDocument`, `limit`) para galerías grandes.
4. Habilitar caché offline de Firestore para resiliencia en redes inestables.
5. Configurar **Reglas de Seguridad** estrictas en Firebase Console:
   - Solo usuarios autenticados leen/escriben sus propios datos o sesiones asignadas.
   - Validación de tipos y rangos en campos críticos.
6. Probar operaciones CRUD con datos ficticios y validar consistencia.

### 🔹 Fase 7: Integración de Flujos Principales
1. Conectar `GalleryProvider` con Firestore: carga inicial, paginación, búsqueda/filtros (por fecha, sesión, etiqueta).
2. Implementar visor de imágenes con transiciones suaves y metadatos superpuestos.
3. (Opcional) Habilitar subida de fotos: `image_picker` → compresión → `FirebaseStorage` → guardar metadatos en Firestore → notificar provider.
4. Unir autenticación con perfil: al login, cargar datos de usuario y preferencias.
5. Añadir pull-to-refresh, estados vacíos personalizados y manejo de desconexión.
6. Validar navegación completa y persistencia de sesión entre reinicios.

### 🔹 Fase 8: Testing, Optimización y Despliegue
1. **Testing**:
   - Unit tests: lógica de providers, validaciones de formularios, servicios mock.
   - Widget tests: renders de pantallas críticas, estados de carga/error.
   - Integration tests: flujo login → galería → cierre.
2. **Optimización**:
   - Analizar con Flutter DevTools: frame drops, memoria, uso de red.
   - Optimizar imágenes: formatos WebP/AVIF, tamaños adaptativos, prefetching.
   - Minimizar rebuilds, usar `const` widgets, evitar `setState` innecesarios.
3. **Despliegue**:
   - Configurar firmas y iconos para Android/iOS.
   - Generar builds: `flutter build apk --release`, `flutter build ios --release`, `flutter build web --release`.
   - Subir a Play Console, App Store Connect, y Firebase Hosting (web).
   - Configurar monitoreo post-lanzamiento (Firebase Crashlytics, Analytics).
4. **Documentación**: README técnico, guía de instalación, diagrama de arquitectura, manual de despliegue.

---

## ✅ Entregables por Fase
| Fase | Entregable |
|------|------------|
| 0 | Documento de arquitectura + estructura de carpetas |
| 1 | Proyecto compilando en 3 plataformas + Firebase configurado |
| 2 | Routing funcional + variables de entorno seguras |
| 3 | Sistema de diseño implementado + componentes base |
| 4 | Flujo de autenticación completo + manejo de estados |
| 5 | Arquitectura Provider establecida + consumo UI validado |
| 6 | Esquema Firestore + servicios + reglas de seguridad |
| 7 | Flujos integrados (galería, perfil, navegación) |
| 8 | Tests pasados + builds optimizados + despliegue documentado |

---

## ⚠️ Recomendaciones Críticas
- **No almacenar imágenes en Firestore**. Usa Firebase Storage para binarios y guarda solo URLs/metadatos en Firestore.
- **Reglas de seguridad primero**. Prueba con el simulador de reglas antes de conectar la app.
- **Provider no es para lógica pesada**. Mantén la manipulación de datos en `services`, los providers solo orquestan y exponen estados.
- **Valida multiplataforma temprano**. Algunos paquetes o comportamientos de UI difieren entre web y móvil; prueba cada fase en las 3 plataformas.
- **Mantén `pubspec.yaml` limpio**. Solo agrega dependencias cuando sean estrictamente necesarias para evitar bloat y conflictos.

---

✅ **Siguiente paso**: Una vez valides este plan, puedo generar la estructura de carpetas exacta, el `pubspec.yaml` completo, o comenzar con la implementación fase por fase bajo tu aprobación. ¿Deseas ajustar algún alcance o proceder con la Fase 1?
