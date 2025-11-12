# ApkBridge

<p align="center">
 <img width=200px height=200px src="images/icon.webp"/>
</p>

<h1 align="center"> ApkBridge </h1>

<div align="center">

[![GitHub downloads](https://img.shields.io/github/downloads/Schnitzel5/ApkBridge/total?label=downloads&labelColor=27303D&color=0D1117&logo=github&logoColor=FFFFFF&style=flat)](https://github.com/kodjodevf/mangayomi/releases)
![star](https://img.shields.io/github/stars/Schnitzel5/ApkBridge)
[![Discord server](https://img.shields.io/discord/1157628512077893666.svg?label=&labelColor=6A7EC2&color=7389D8&logo=discord&logoColor=FFFFFF)](https://discord.com/invite/EjfBuYahsP)

Un servidor proxy HTTP para Android que permite invocar funciones de extensiones APK.
</div>

## ¿Qué es ApkBridge?

ApkBridge es una aplicación Android que funciona como un servidor proxy HTTP, permitiéndote ejecutar extensiones APK (específicamente extensiones de Tachiyomi para manga y anime) de forma dinámica sin necesidad de instalarlas permanentemente en tu dispositivo.

### Características Principales

- 🚀 **Servidor HTTP Local**: Ejecuta un servidor en tu dispositivo Android (puerto 8080)
- 📦 **Carga Dinámica de APK**: Carga y ejecuta extensiones APK bajo demanda
- 🎌 **Soporte para Manga**: Compatible con extensiones de Tachiyomi para manga
- 🎬 **Soporte para Anime**: Compatible con extensiones de Tachiyomi para anime
- ⚙️ **Gestión de Preferencias**: Configura y almacena preferencias por extensión
- 🔍 **Filtros Avanzados**: Soporta filtros de búsqueda complejos
- 🔄 **Actualizaciones Automáticas**: Verifica e instala actualizaciones desde GitHub
- 📱 **Interfaz Moderna**: UI construida con Jetpack Compose

## ¿Cómo Funciona?

1. **Inicia el servidor** en la aplicación
2. **Obtén la IP** de tu dispositivo desde la interfaz
3. **Envía solicitudes HTTP** desde cualquier cliente a `http://<IP>:8080/dalvik`
4. **Incluye el APK** de la extensión codificado en Base64
5. **Recibe los datos** en formato JSON

Para una explicación detallada de la arquitectura y funcionamiento interno, consulta [COMO_FUNCIONA.md](COMO_FUNCIONA.md).

## Instalación

Descarga [ApkBridge.apk](https://github.com/Schnitzel5/ApkBridge/releases/latest) y presiona "Start server" después de la instalación.

<img src="images/img.png"/>

## Compilar desde el Código Fuente

```bash
git clone https://github.com/Schnitzel5/ApkBridge.git
cd ApkBridge
chmod +x ./gradlew
./gradlew build
./gradlew assembleRelease
```

Asegúrate de proporcionar las claves de firma en un nuevo archivo "local.properties":

```properties
storePassword=<contraseña>
keyPassword=<contraseña>
keyAlias=<alias de la clave>
storeFile=<ruta al archivo keystore>
```

## Uso Básico

### Iniciar el Servidor

1. Abre la aplicación ApkBridge
2. Presiona el botón "Start server"
3. Muestra la IP del servidor (botón "Show Server IP")
4. Copia la IP al portapapeles si es necesario

### Realizar una Solicitud

Envía una solicitud POST a `http://<IP>:8080/dalvik` con el siguiente formato JSON:

```json
{
  "data": "<APK_CODIFICADO_EN_BASE64>",
  "method": "getPopularManga",
  "page": 1
}
```

### Métodos Disponibles

**Para Manga:**
- `getPopularManga` - Obtener manga populares
- `getLatestManga` - Obtener últimas actualizaciones
- `getSearchManga` - Buscar manga
- `getDetailsManga` - Detalles de un manga
- `getChapterList` - Lista de capítulos
- `getPageList` - Páginas de un capítulo
- `preferencesManga` - Obtener preferencias configurables

**Para Anime:**
- `getPopularAnime` - Obtener anime populares
- `getLatestAnime` - Obtener últimas actualizaciones
- `getSearchAnime` - Buscar anime
- `getDetailsAnime` - Detalles de un anime
- `getEpisodeList` - Lista de episodios
- `getVideoList` - Enlaces de video
- `preferencesAnime` - Obtener preferencias configurables

## Documentación

- 📖 [Documentación Completa en Español](COMO_FUNCIONA.md) - Explicación detallada de la arquitectura y funcionamiento
- 🌐 [English README](README.md) - Original README in English

## Permisos Requeridos

- **INTERNET**: Para el servidor HTTP
- **ACCESS_NETWORK_STATE**: Para detectar la IP del dispositivo
- **FOREGROUND_SERVICE**: Para mantener el servidor activo en segundo plano
- **POST_NOTIFICATIONS**: Para notificaciones (Android 13+)
- **REQUEST_INSTALL_PACKAGES**: Para instalar actualizaciones

## Casos de Uso

- **Desarrollo de Clientes**: Desarrolla aplicaciones que consuman extensiones de Tachiyomi
- **Testing de Extensiones**: Prueba extensiones sin instalarlas permanentemente
- **Acceso Remoto**: Accede a extensiones desde otros dispositivos en tu red local
- **Automatización**: Crea scripts para automatizar búsquedas y descargas

## Tecnologías

- Kotlin + Jetpack Compose
- NanoHTTPD (Servidor HTTP)
- DexClassLoader (Carga dinámica de APK)
- Kotlin Coroutines
- OkHttp + Jackson

## Licencia

```
Copyright 2025 Schnitzel5

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
```

## Disclaimer

El/Los desarrollador(es) de esta aplicación no tienen ninguna afiliación con los proveedores de contenido que están disponibles libremente en Internet.

## Contribuciones

Las contribuciones son bienvenidas! Por favor, abre un issue o pull request en GitHub.

## Soporte

- 💬 [Discord Server](https://discord.com/invite/EjfBuYahsP)
- 🐛 [Reportar un Bug](https://github.com/Schnitzel5/ApkBridge/issues)
- ⭐ Si te gusta el proyecto, dale una estrella en GitHub!
