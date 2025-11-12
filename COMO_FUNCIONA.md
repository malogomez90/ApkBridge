# Cómo Funciona ApkBridge

## Descripción General

ApkBridge es un servidor proxy HTTP para Android que permite invocar funciones de extensiones APK (específicamente extensiones de Tachiyomi para manga y anime). La aplicación actúa como un puente entre clientes externos y las extensiones instaladas en formato APK, permitiendo la ejecución dinámica de código de las extensiones sin necesidad de instalarlas permanentemente en el dispositivo.

> **⚠️ IMPORTANTE - USO EN RED LOCAL**: ApkBridge está diseñado para uso en red local (LAN). El servidor **NO tiene autenticación ni cifrado**, por lo que **NO debe exponerse a Internet** directamente. Úsalo solo dentro de tu red Wi-Fi local o conectando dispositivos mediante hotspot/USB.

## Arquitectura de la Aplicación

### 1. Componentes Principales

#### 1.1 MainActivity
- **Ubicación**: `MainActivity.kt`
- **Función**: Actividad principal de la aplicación Android
- **Responsabilidades**:
  - Inicializar la interfaz de usuario con Jetpack Compose
  - Gestionar permisos de notificaciones
  - Monitorear la conectividad de red y obtener la dirección IP del dispositivo
  - Verificar actualizaciones de la aplicación desde GitHub
  - Gestionar descargas e instalaciones de actualizaciones

**Características clave**:
```kotlin
- networkCallback: Detecta cambios en la conexión de red y obtiene la IP actual
- checkUpdate(): Verifica si hay nuevas versiones disponibles
- requestDownload(): Descarga e instala actualizaciones automáticamente
```

#### 1.2 WebService
- **Ubicación**: `web/service/WebService.kt`
- **Función**: Servicio de primer plano que mantiene el servidor HTTP activo
- **Responsabilidades**:
  - Iniciar y detener el servidor web
  - Mantener una notificación persistente mientras el servidor está activo
  - Asegurar que el servidor continúe ejecutándose en segundo plano

**Acciones del servicio**:
- `START`: Inicia el servidor web en el puerto 8080
- `STOP`: Detiene el servidor y elimina el servicio

#### 1.3 WebServer
- **Ubicación**: `web/WebServer.java`
- **Función**: Servidor HTTP basado en NanoHTTPD
- **Responsabilidades**:
  - Escuchar en el puerto 8080
  - Definir rutas y mapear handlers
  - Gestionar el ciclo de vida del servidor

**Rutas configuradas**:
- `/`: Ruta principal (IndexHandler - no implementado aún)
- `/dalvik`: Endpoint principal para invocar funciones de extensiones

### 2. Sistema de Manejo de Extensiones (DalvikHandler)

El `DalvikHandler` es el corazón de la aplicación. Permite cargar dinámicamente extensiones APK y ejecutar sus funciones.

#### 2.1 Proceso de Carga de Extensiones

1. **Recepción de Datos**: 
   - El cliente envía una solicitud POST a `/dalvik`
   - Los datos incluyen el APK codificado en Base64 y los parámetros de la operación

2. **Decodificación y Preparación**:
   ```java
   - Decodifica el APK de Base64
   - Crea un archivo temporal (.apk)
   - Establece permisos de solo lectura
   ```

3. **Carga Dinámica con DexClassLoader**:
   ```java
   DexClassLoader loader = new DexClassLoader(
       file.getAbsolutePath(), 
       null, 
       null, 
       this.getClass().getClassLoader()
   );
   ```

4. **Identificación de la Fuente**:
   - Lee los metadatos del APK usando PackageManager
   - Identifica si es una extensión de Manga o Anime
   - Busca la clase principal definida en los metadatos

5. **Instanciación**:
   - Carga la clase usando reflexión
   - Instancia la fuente (MangaSource/AnimeSource o Factory)
   - Aplica preferencias del usuario si están disponibles

6. **Ejecución**:
   - Ejecuta el método solicitado usando Kotlin Coroutines
   - Devuelve el resultado en formato JSON

7. **Limpieza**:
   - Elimina el archivo temporal del APK

#### 2.2 Métodos Soportados

**Para Extensiones de Manga**:
- `headersManga`: Obtiene los headers HTTP de la fuente
- `filtersManga`: Obtiene los filtros de búsqueda disponibles
- `supportLatestManga`: Verifica si soporta "últimas actualizaciones"
- `getPopularManga`: Obtiene manga populares (con paginación)
- `getLatestManga`: Obtiene últimas actualizaciones de manga
- `getSearchManga`: Busca manga con filtros personalizados
- `getDetailsManga`: Obtiene detalles de un manga específico
- `getChapterList`: Obtiene la lista de capítulos
- `getPageList`: Obtiene las páginas de un capítulo
- `preferencesManga`: Obtiene las preferencias configurables de la extensión

**Para Extensiones de Anime**:
- `headersAnime`: Obtiene los headers HTTP de la fuente
- `filtersAnime`: Obtiene los filtros de búsqueda disponibles
- `supportLatestAnime`: Verifica si soporta "últimas actualizaciones"
- `getPopularAnime`: Obtiene anime populares (con paginación)
- `getLatestAnime`: Obtiene últimas actualizaciones de anime
- `getSearchAnime`: Busca anime con filtros personalizados
- `getDetailsAnime`: Obtiene detalles de un anime específico
- `getEpisodeList`: Obtiene la lista de episodios
- `getVideoList`: Obtiene los enlaces de video y subtítulos
- `preferencesAnime`: Obtiene las preferencias configurables de la extensión

### 3. Flujo de Datos

```
Cliente Externo
    ↓
    | HTTP POST /dalvik
    | (APK en Base64 + parámetros)
    ↓
WebServer (Puerto 8080)
    ↓
DalvikHandler
    ↓
1. Decodifica APK
2. Crea archivo temporal
3. Carga con DexClassLoader
4. Identifica tipo de extensión
5. Instancia la clase fuente
6. Aplica preferencias
7. Ejecuta método solicitado
8. Serializa resultado a JSON
    ↓
    | HTTP Response (JSON)
    ↓
Cliente Externo
```

### 4. Sistema de Preferencias

La aplicación soporta diferentes tipos de preferencias para extensiones:

- **CheckBoxPreference**: Casillas de verificación booleanas
- **EditTextPreference**: Campos de texto editables
- **ListPreference**: Listas de selección simple
- **MultiSelectListPreference**: Listas de selección múltiple
- **SwitchPreference**: Interruptores booleanos

Las preferencias se almacenan usando `SharedPreferences` con un ID único por fuente.

### 5. Sistema de Filtros

Soporta múltiples tipos de filtros para búsquedas avanzadas:

- **Text**: Filtros de texto libre
- **CheckBox**: Casillas de verificación
- **TriState**: Estados de tres valores (incluido, excluido, ninguno)
- **Select**: Listas desplegables
- **Sort**: Ordenamiento con dirección (ascendente/descendente)
- **Group**: Grupos de filtros anidados

### 6. Interfaz de Usuario

#### 6.1 Pantalla Principal (MainScreen)
- Botón para iniciar/detener el servidor
- Visualización de la IP del servidor
- Botón para copiar IP al portapapeles
- Acceso a logs de la aplicación
- Enlace al repositorio de GitHub
- Notificación de actualizaciones disponibles

#### 6.2 Pantalla de Logs (LogScreen)
- Visualiza los logs en tiempo real
- Filtra por niveles: INFO, WARNING, ERROR
- Permite copiar logs al portapapeles

### 7. Gestión de Red

- **Detección Automática de IP**: Monitorea cambios en la conexión de red
- **Preferencia por IPv4**: Prioriza direcciones IPv4 sobre IPv6
- **Notificación de Estado**: Informa cuando no hay conexión a internet

### 8. Seguridad y Permisos

Permisos requeridos:
- `INTERNET`: Para el servidor HTTP
- `REQUEST_INSTALL_PACKAGES`: Para instalar actualizaciones
- `ACCESS_WIFI_STATE` y `ACCESS_NETWORK_STATE`: Para detectar la IP
- `FOREGROUND_SERVICE`: Para mantener el servidor activo
- `POST_NOTIFICATIONS`: Para notificaciones (Android 13+)

### 9. Características Adicionales

#### 9.1 Actualizaciones Automáticas
- Verifica automáticamente nuevas versiones en GitHub
- Descarga e instala actualizaciones con un clic
- Muestra notas de la versión

#### 9.2 Gestión de Subtítulos (Anime)
- Convierte rutas de archivos locales a contenido inline
- Lee archivos de subtítulos desde el caché de la aplicación
- Limpia tracks de subtítulos vacíos

#### 9.3 Sistema de Logging
- Tres niveles: INFO, WARNING, ERROR
- Logs visibles en la interfaz
- Útil para depuración y monitoreo

## Casos de Uso en Red Local

### Configuraciones de Red Típicas

#### Configuración 1: Desde PC/Laptop en la misma Wi-Fi
```
Android (ApkBridge) IP: 192.168.1.50:8080
    ↓ (Wi-Fi doméstica)
PC/Laptop: Envía request a http://192.168.1.50:8080/dalvik
```

#### Configuración 2: Android como Hotspot
```
Android (ApkBridge + Hotspot) IP: 192.168.43.1:8080
    ↓ (Hotspot personal)
Laptop conectado al hotspot → http://192.168.43.1:8080/dalvik
```

#### Configuración 3: Localhost (mismo dispositivo)
```
Android (ApkBridge + Cliente) 
Cliente accede a: http://127.0.0.1:8080/dalvik
o http://localhost:8080/dalvik
```

### Caso de Uso 1: Buscar Manga
1. Cliente envía POST a `/dalvik` con:
   - APK de extensión codificado en Base64
   - method: "getSearchManga"
   - page: número de página
   - search: término de búsqueda
   - filterList: filtros opcionales

2. ApkBridge:
   - Carga la extensión
   - Ejecuta la búsqueda
   - Devuelve lista de manga en JSON

### Caso de Uso 2: Obtener Videos de Anime
1. Cliente envía POST a `/dalvik` con:
   - APK de extensión codificado en Base64
   - method: "getVideoList"
   - episodeData: datos del episodio

2. ApkBridge:
   - Carga la extensión
   - Obtiene enlaces de video
   - Procesa subtítulos
   - Devuelve lista de videos en JSON

### Caso de Uso 3: Configurar Preferencias
1. Cliente envía POST a `/dalvik` con:
   - method: "preferencesManga" o "preferencesAnime"
   - APK de extensión

2. ApkBridge:
   - Obtiene preferencias disponibles
   - Devuelve estructura de preferencias en JSON

3. Cliente modifica preferencias y las incluye en solicitudes posteriores

## Tecnologías Utilizadas

- **Kotlin**: Lenguaje principal para la UI y servicios
- **Java**: Implementación del servidor web y handlers
- **Jetpack Compose**: Framework de UI moderna para Android
- **NanoHTTPD**: Servidor HTTP ligero
- **DexClassLoader**: Carga dinámica de clases desde APK
- **Kotlin Coroutines**: Manejo de operaciones asíncronas
- **OkHttp**: Cliente HTTP para red
- **Jackson**: Serialización/deserialización JSON
- **RxJava**: Programación reactiva (usado por extensiones)

## Ventajas del Diseño

1. **Sin Instalación Permanente**: Las extensiones se cargan dinámicamente sin instalarlas
2. **Aislamiento**: Cada extensión se ejecuta en su propio contexto
3. **Flexibilidad**: Soporta múltiples tipos de extensiones (manga/anime)
4. **API HTTP Simple**: Fácil integración desde cualquier cliente
5. **Preferencias Persistentes**: Configuración por extensión se mantiene
6. **Actualizaciones Automáticas**: Mantiene la aplicación actualizada

## Limitaciones y Consideraciones

### Seguridad
1. **⚠️ Sin Autenticación**: No hay autenticación, cualquiera en la red puede acceder
2. **⚠️ Sin Cifrado**: Las comunicaciones van en texto plano (HTTP, no HTTPS)
3. **⚠️ Solo Red Local**: **NUNCA expongas el servidor a Internet** - úsalo solo en tu red Wi-Fi local

### Técnicas
4. **Puerto Fijo**: El servidor siempre usa el puerto 8080
5. **Recursos**: Cada invocación crea y destruye un ClassLoader
6. **Compatibilidad**: Depende de la estructura de extensiones de Tachiyomi

### Escenarios de Uso Seguro
- ✅ Mismo dispositivo (localhost)
- ✅ Dispositivos en la misma red Wi-Fi doméstica
- ✅ Conexión mediante hotspot personal
- ✅ Conexión USB con tethering
- ❌ Exponer a Internet público
- ❌ Redes Wi-Fi públicas sin VPN

## Conclusión

ApkBridge es una solución elegante para ejecutar extensiones APK bajo demanda, funcionando como un puente entre clientes remotos y el ecosistema de extensiones de Tachiyomi. Su arquitectura modular y uso de carga dinámica de clases lo hace flexible y eficiente para su propósito específico.
