# Diagrama de Flujo - ApkBridge

## Resumen Visual del Funcionamiento

### 🏠 Escenario: Red Local

```
┌─────────────────────────────────────────────────────────────────┐
│                        RED LOCAL (Wi-Fi)                         │
│                                                                   │
│  ┌──────────────────┐              ┌───────────────────────┐   │
│  │                  │              │                       │   │
│  │   PC / Laptop    │              │   Dispositivo Android │   │
│  │   (Cliente)      │◄────────────►│      (ApkBridge)      │   │
│  │                  │   HTTP       │                       │   │
│  │  Envía requests  │              │  Servidor en          │   │
│  │  con APK en      │              │  puerto 8080          │   │
│  │  Base64          │              │                       │   │
│  └──────────────────┘              └───────────────────────┘   │
│         │                                     │                  │
│         │                                     │                  │
│         └─────── http://192.168.1.X:8080 ────┘                  │
│                                                                   │
└─────────────────────────────────────────────────────────────────┘
```

## Flujo Detallado de una Solicitud

### 1️⃣ Preparación
```
Cliente
  │
  ├─ Lee archivo APK de la extensión
  ├─ Codifica el APK en Base64
  ├─ Prepara parámetros (método, página, búsqueda, etc.)
  └─ Construye request JSON
```

### 2️⃣ Envío
```
Cliente ──POST──► http://192.168.1.50:8080/dalvik
              │
              └─ Body JSON:
                 {
                   "data": "UEsDBAoAAA...",  // APK en Base64
                   "method": "getSearchManga",
                   "page": 1,
                   "search": "naruto"
                 }
```

### 3️⃣ Procesamiento en ApkBridge
```
WebServer (NanoHTTPD)
    ↓
DalvikHandler
    ↓
┌─────────────────────────────────────┐
│ 1. Decodifica Base64 → archivo APK │
│ 2. Crea archivo temporal           │
│ 3. DexClassLoader carga el APK     │
│ 4. Lee metadatos del APK           │
│ 5. Identifica tipo (Manga/Anime)   │
│ 6. Instancia la clase de extensión │
│ 7. Aplica preferencias del usuario │
│ 8. Ejecuta método solicitado       │
│ 9. Serializa resultado a JSON      │
│ 10. Elimina archivo temporal       │
└─────────────────────────────────────┘
    ↓
Resultado JSON
```

### 4️⃣ Respuesta
```
ApkBridge ──Response──► Cliente
              │
              └─ JSON con resultados:
                 [
                   {
                     "title": "Naruto",
                     "url": "...",
                     "thumbnail": "..."
                   },
                   ...
                 ]
```

## Ciclo de Vida del Servidor

```
Usuario abre la app
    ↓
┌─────────────────┐
│  MainActivity   │ ← Muestra UI con Jetpack Compose
└────────┬────────┘
         │ Usuario presiona "Start Server"
         ↓
┌─────────────────┐
│   WebService    │ ← Servicio de primer plano
│   (foreground)  │   con notificación persistente
└────────┬────────┘
         │ Inicia servidor
         ↓
┌─────────────────┐
│   WebServer     │ ← NanoHTTPD escuchando
│   Puerto 8080   │   en puerto 8080
└────────┬────────┘
         │ Espera requests
         │
         ├──► Request 1 → DalvikHandler → Response 1
         ├──► Request 2 → DalvikHandler → Response 2
         ├──► Request 3 → DalvikHandler → Response 3
         │    ...
         │
         │ Usuario presiona "Stop Server"
         ↓
    Servidor se detiene
    Servicio se destruye
```

## Ejemplo Práctico: Buscar Manga

### Paso a Paso

#### 1. Configuración Inicial
```bash
# En Android:
1. Abre ApkBridge
2. Presiona "Start Server"
3. Anota la IP: 192.168.1.50
```

#### 2. Desde tu PC/Laptop
```javascript
// Ejemplo en JavaScript/Node.js
const fs = require('fs');
const axios = require('axios');

// Leer el APK de la extensión
const apkBuffer = fs.readFileSync('mangakakalot.apk');
const apkBase64 = apkBuffer.toString('base64');

// Preparar request
const request = {
  data: apkBase64,
  method: 'getSearchManga',
  page: 1,
  search: 'one piece'
};

// Enviar request
const response = await axios.post(
  'http://192.168.1.50:8080/dalvik',
  { postData: JSON.stringify(request) }
);

console.log('Resultados:', response.data);
// [{ title: "One Piece", url: "...", ... }, ...]
```

#### 3. Resultado
```json
[
  {
    "url": "/manga/one_piece",
    "title": "One Piece",
    "thumbnail_url": "https://example.com/cover.jpg"
  },
  {
    "url": "/manga/one_piece_colored",
    "title": "One Piece (Colored Edition)",
    "thumbnail_url": "https://example.com/cover2.jpg"
  }
]
```

## Métodos Disponibles por Tipo

### 📚 Para Extensiones de Manga

```
getPopularManga      → Lista de manga populares
    ↓
getLatestManga       → Últimas actualizaciones
    ↓
getSearchManga       → Búsqueda con filtros
    ↓
getDetailsManga      → Información del manga
    ↓
getChapterList       → Lista de capítulos
    ↓
getPageList          → URLs de las páginas
```

### 🎬 Para Extensiones de Anime

```
getPopularAnime      → Lista de anime populares
    ↓
getLatestAnime       → Últimas actualizaciones
    ↓
getSearchAnime       → Búsqueda con filtros
    ↓
getDetailsAnime      → Información del anime
    ↓
getEpisodeList       → Lista de episodios
    ↓
getVideoList         → URLs de videos y subtítulos
```

## Comparación: Con vs Sin ApkBridge

### ❌ Sin ApkBridge (Método Tradicional)
```
1. Descargar APK de extensión
2. Instalar APK en Android
3. Abrir app compatible (Tachiyomi)
4. Usar dentro de la app
5. Para cambiar extensión: Instalar otra APK
```

### ✅ Con ApkBridge
```
1. Instalar ApkBridge una vez
2. Iniciar servidor
3. Enviar APK en cada request (no instalar)
4. Usar desde cualquier cliente (PC, otro Android, etc.)
5. Cambiar extensión: Solo cambiar el APK en el request
```

## Ventajas del Diseño

```
┌─────────────────────────────────────────┐
│  ✅ VENTAJAS                            │
├─────────────────────────────────────────┤
│ • No requiere instalar cada extensión  │
│ • Acceso desde múltiples dispositivos  │
│ • Fácil testing de extensiones         │
│ • API HTTP simple y universal          │
│ • Aislamiento de extensiones           │
│ • Preferencias persistentes            │
└─────────────────────────────────────────┘
```

## ⚠️ Consideraciones de Seguridad

```
┌─────────────────────────────────────────┐
│  ⚠️  IMPORTANTE - SOLO RED LOCAL       │
├─────────────────────────────────────────┤
│                                         │
│  ✅ SÍ USAR:                            │
│    • Red Wi-Fi doméstica               │
│    • Hotspot personal                  │
│    • Localhost (127.0.0.1)             │
│    • USB tethering                     │
│                                         │
│  ❌ NO USAR:                            │
│    • Wi-Fi público                     │
│    • Expuesto a Internet               │
│    • Sin VPN en redes no confiables    │
│                                         │
│  Razón: No hay autenticación ni        │
│         cifrado (HTTP plano)           │
└─────────────────────────────────────────┘
```

## Tecnologías Clave

```
┌──────────────────┐
│  Android (Java)  │
└────────┬─────────┘
         │
    ┌────┴────┬─────────┬──────────┐
    │         │         │          │
    ↓         ↓         ↓          ↓
┌────────┐ ┌─────┐ ┌────────┐ ┌─────────┐
│Jetpack │ │Nano │ │ Dex    │ │ Kotlin  │
│Compose │ │HTTPD│ │Class   │ │Coroutines│
│  (UI)  │ │(Web)│ │Loader  │ │ (Async) │
└────────┘ └─────┘ └────────┘ └─────────┘
                        │
                        └─► Carga APKs
                            dinámicamente
```

## Resumen en 3 Puntos

1. **🖥️ ApkBridge = Servidor HTTP en tu Android**
   - Escucha en puerto 8080
   - Recibe APKs y ejecuta sus funciones
   - Devuelve resultados en JSON

2. **🌐 Uso en Red Local**
   - Accede desde PC/Laptop en la misma Wi-Fi
   - NO expongas a Internet (sin seguridad)
   - Perfecto para desarrollo y testing

3. **📦 Extensiones Sin Instalar**
   - Envía APK en cada request
   - No necesitas instalar extensiones
   - Cambia de extensión fácilmente
