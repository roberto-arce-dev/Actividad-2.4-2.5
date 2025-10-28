# 🌐 Guía: Cómo Agregar API REST a Tu Proyecto Android

> **Para:** Cualquier proyecto Android con Jetpack Compose
> **Tiempo estimado:** 30-45 minutos
> **Nivel:** Básico-Intermedio

---

## 📋 ¿Qué vas a lograr?

Al terminar esta guía, tu app podrá:
- ✅ Conectarse a internet
- ✅ Enviar peticiones HTTP a un servidor
- ✅ Recibir y mostrar datos en tiempo real
- ✅ Manejar estados de carga y errores

---

## 🛠️ Requisitos Previos

Antes de empezar, asegúrate de tener:

- [x] Proyecto Android con **Jetpack Compose**
- [x] **Kotlin** como lenguaje
- [x] Arquitectura **MVVM** (ViewModel + UI)
- [x] Min SDK 24 o superior
- [x] Android Studio instalado

---

## 📦 PASO 1: Agregar Dependencias

### 1.1 Ubicar el archivo de dependencias

Abre el archivo: **`app/build.gradle.kts`** (o `build.gradle` si usas Groovy)

### 1.2 Agregar las librerías

Busca la sección `dependencies {` y agrega **al final** (antes del último `}`):

```kotlin
dependencies {
    // ... tus dependencias existentes ...

    // ========================================
    // NETWORKING - API REST
    // ========================================

    // OkHttp - Cliente HTTP
    implementation("com.squareup.okhttp3:okhttp:4.12.0")
    implementation("com.squareup.okhttp3:logging-interceptor:4.12.0")

    // Retrofit - Cliente REST
    implementation("com.squareup.retrofit2:retrofit:2.11.0")
    implementation("com.squareup.retrofit2:converter-gson:2.11.0")

    // Coroutines (si no las tienes)
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.9.0")

    // DataStore - Para guardar tokens
    implementation("androidx.datastore:datastore-preferences:1.0.0")

    // Coil - Para cargar imágenes desde URLs (opcional)
    implementation("io.coil-kt:coil-compose:2.6.0")
}
```

### 1.3 Sincronizar el proyecto

1. Haz clic en **"Sync Now"** (barra amarilla arriba)
2. Espera a que diga **"BUILD SUCCESSFUL"**

**¿Qué acabas de agregar?**

| Librería | ¿Para qué sirve? |
|----------|------------------|
| **OkHttp** | Maneja las conexiones HTTP |
| **Logging Interceptor** | Muestra las peticiones en Logcat (debugging) |
| **Retrofit** | Simplifica las llamadas a APIs REST |
| **Gson Converter** | Convierte JSON ↔ Objetos Kotlin automáticamente |
| **Coroutines** | Ejecuta peticiones sin bloquear la UI |
| **DataStore** | Guarda tokens/preferencias de forma segura |
| **Coil** | Carga imágenes desde URLs (opcional) |

---

## 🔐 PASO 2: Agregar Permiso de Internet

### 2.1 Ubicar el archivo Manifest

Abre: **`app/src/main/AndroidManifest.xml`**

### 2.2 Agregar el permiso

**ANTES** de la etiqueta `<application>`, agrega:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">

    <!-- ✅ AGREGAR AQUÍ -->
    <uses-permission android:name="android.permission.INTERNET" />

    <application
        android:name=".MainActivity"
        ...>
        <!-- resto del archivo -->
    </application>
</manifest>
```

**¿Por qué?**
Android bloquea el acceso a internet por defecto. Este permiso le dice al sistema operativo que tu app necesita conectarse.

---

## 📁 PASO 3: Crear la Estructura de Carpetas (Arquitectura MVVM por Capas)

Vas a crear esta estructura organizada por **CAPAS**:

```
app/src/main/java/com/tuempresa/tuapp/
├── data/                          ← CAPA DE DATOS
│   ├── local/                     ← Datos locales (Room, DataStore)
│   │   ├── SessionManager.kt      ← Guarda el token JWT
│   │   ├── dao/
│   │   │   └── UserDao.kt         ← (Opcional) DAO de Room
│   │   ├── database/
│   │   │   └── AppDatabase.kt     ← (Opcional) Base de datos Room
│   │   └── entity/
│   │       └── User.kt            ← (Opcional) Entity de Room
│   └── remote/                    ← Datos remotos (API)
│       ├── ApiService.kt          ← Define los endpoints
│       ├── RetrofitClient.kt      ← Configura Retrofit
│       ├── AuthInterceptor.kt     ← Inyecta el token automáticamente
│       └── dto/                   ← Data Transfer Objects
│           ├── UserDto.kt
│           ├── LoginRequest.kt
│           ├── LoginResponse.kt
│           └── UsersResponse.kt
├── repository/                    ← CAPA DE REPOSITORIO
│   └── UserRepository.kt          ← Coordina local + remote
├── ui/                            ← CAPA DE UI
│   ├── navigation/
│   │   └── AppNavigation.kt       ← Navegación entre pantallas
│   └── screens/                   ← Pantallas Compose
│       ├── LoginScreen.kt
│       ├── RegisterScreen.kt
│       ├── ProfileScreen.kt
│       └── HomeScreen.kt
├── viewmodel/                     ← CAPA DE VIEWMODEL
│   ├── LoginViewModel.kt
│   ├── RegisterViewModel.kt
│   ├── ProfileViewModel.kt
│   └── HomeViewModel.kt
├── utils/                         ← UTILIDADES
│   └── ValidationUtils.kt         ← Funciones helper
└── AppDependencies.kt             ← Service Locator (raíz)
```
### 📊 Responsabilidades de cada Capa

| Capa | Responsabilidad | Ejemplo |
|------|-----------------|---------|
| **`data/`** | Obtener/guardar datos | API, Room, DataStore |
| **`repository/`** | Coordinar fuentes de datos | Local + Remote, caché |
| **`viewmodel/`** | Lógica de negocio de UI | Estados, validaciones |
| **`ui/`** | Interfaz visual | Composables, navegación |
| **`utils/`** | Funciones reutilizables | Validaciones, formateo |

### ✅ Ventajas de esta Arquitectura

- 📚 **Didáctica**: Los alumnos ven claramente cada capa de MVVM
- 🎯 **Conceptualmente clara**: Cada carpeta = una responsabilidad
- 📖 **Sigue estándares**: La mayoría de tutoriales y libros usan esto
- 🔍 **Fácil de navegar**: "¿ViewModels? Van en `viewmodel/`"
- ⚡ **Perfecta para proyectos pequeños/medios** (5-15 pantallas)


### 🛠️ ¿Cómo crear las carpetas?

1. Click derecho en tu paquete principal (`com.tuempresa.tuapp`)
2. **New → Package**
3. Escribe: `data.local` (crea la carpeta)
4. Repite para:
   - `data.remote.dto`
   - `repository`
   - `ui.screens`
   - `ui.navigation`
   - `viewmodel`
   - `utils`

**💡 Tip:** Para proyectos grandes (> 15 pantallas), considera organizar por features (`ui/login/`, `ui/profile/`) en lugar de por capas.

---

## 🔧 PASO 4: Configurar Retrofit con Interceptores

### 4.1 Crear SessionManager (Gestor de Token)

**¿Qué es?** Un componente que guarda el token de autenticación JWT de forma persistente.

Crea el archivo: **`data/local/SessionManager.kt`**

```kotlin
package com.tuempresa.tuapp.data.local  // ⚠️ Cambia esto

import android.content.Context
import androidx.datastore.preferences.core.edit
import androidx.datastore.preferences.core.stringPreferencesKey
import androidx.datastore.preferences.preferencesDataStore
import kotlinx.coroutines.flow.first
import kotlinx.coroutines.flow.map

/**
 * SessionManager: Guarda y recupera el token JWT de forma segura
 */
class SessionManager(private val context: Context) {

    companion object {
        private val Context.dataStore by preferencesDataStore(name = "session_prefs")
        private val KEY_AUTH_TOKEN = stringPreferencesKey("auth_token")
    }

    /**
     * Guarda el token de autenticación
     */
    suspend fun saveAuthToken(token: String) {
        context.dataStore.edit { preferences ->
            preferences[KEY_AUTH_TOKEN] = token
        }
    }

    /**
     * Recupera el token guardado (o null si no existe)
     */
    suspend fun getAuthToken(): String? {
        return context.dataStore.data
            .map { preferences -> preferences[KEY_AUTH_TOKEN] }
            .first()
    }

    /**
     * Elimina el token (cerrar sesión)
     */
    suspend fun clearAuthToken() {
        context.dataStore.edit { preferences ->
            preferences.remove(KEY_AUTH_TOKEN)
        }
    }
}
```

**¿Cuándo usar DataStore?**
- ✅ Para guardar tokens, preferencias del usuario, configuraciones
- ✅ Alternativa moderna a SharedPreferences
- ✅ Seguro para operaciones asíncronas (no bloquea la UI)

**DataStore vs SharedPreferences:**

| Característica | DataStore | SharedPreferences |
|----------------|-----------|-------------------|
| **API** | Coroutines (suspend) | Síncrona (bloquea UI) |
| **Seguridad** | Type-safe con Preferences/Proto | Solo String/Int/Boolean/etc |
| **Crashes** | Maneja errores con try-catch | Puede corromper archivo |
| **Transacciones** | Atómicas (todo o nada) | No garantizadas |
| **Observabilidad** | Flow reactivo | Listeners manuales |

**Ejemplo de uso completo:**

```kotlin
// 1️⃣ Guardar el token tras login exitoso
viewModelScope.launch {
    sessionManager.saveAuthToken(loginResponse.accessToken)
}

// 2️⃣ Recuperar el token para peticiones autenticadas
val token = sessionManager.getAuthToken()  // Returns String? o null

// 3️⃣ Cerrar sesión (eliminar token)
viewModelScope.launch {
    sessionManager.clearAuthToken()
    // Navegar a LoginScreen
}
```

**⚠️ IMPORTANTE:** DataStore requiere que agregues esta dependencia si no la tienes:

```kotlin
implementation("androidx.datastore:datastore-preferences:1.0.0")
```

---

### 4.2 Crear AuthInterceptor (Inyección Automática de Token)

**¿Qué es?** Un interceptor que añade automáticamente el token JWT en TODAS las peticiones HTTP.

**Analogía:** Es como una pulsera de un club nocturno. Una vez que pasas la entrada (login), te ponen la pulsera (token). Cada vez que pides una bebida (petición HTTP), el bartender ve tu pulsera automáticamente y te atiende.

Crea el archivo: **`data/remote/AuthInterceptor.kt`**

```kotlin
package com.tuempresa.tuapp.data.remote  // ⚠️ Cambia esto

import com.tuempresa.tuapp.data.local.SessionManager
import kotlinx.coroutines.runBlocking
import okhttp3.Interceptor
import okhttp3.Response

/**
 * AuthInterceptor: Añade automáticamente el token JWT a las peticiones
 *
 * ¿Cuándo se ejecuta?
 * - ANTES de cada petición HTTP
 *
 * ¿Qué hace?
 * 1. Recupera el token del SessionManager
 * 2. Si existe, añade el header: Authorization: Bearer {token}
 * 3. Si no existe, deja la petición sin modificar
 */
class AuthInterceptor(
    private val sessionManager: SessionManager
) : Interceptor {

    override fun intercept(chain: Interceptor.Chain): Response {
        val originalRequest = chain.request()

        // Recuperar el token (usando runBlocking porque intercept no es suspend)
        val token = runBlocking {
            sessionManager.getAuthToken()
        }

        // Si no hay token, continuar con la petición original
        if (token.isNullOrEmpty()) {
            return chain.proceed(originalRequest)
        }

        // Crear nueva petición CON el token
        val authenticatedRequest = originalRequest.newBuilder()
            .header("Authorization", "Bearer $token")
            .build()

        // Continuar con la petición autenticada
        return chain.proceed(authenticatedRequest)
    }
}
```

**¿Qué es `runBlocking`?**
- Los interceptores NO son funciones `suspend`, pero necesitamos llamar a `getAuthToken()` que SÍ es `suspend`
- `runBlocking` convierte una función suspend en una función normal bloqueante
- Está bien usarlo aquí porque DataStore tiene caché en memoria (es muy rápido)

---

### 4.3 Configurar HttpLoggingInterceptor (Debugging)

**¿Qué es?** Un interceptor que muestra en Logcat TODAS las peticiones y respuestas HTTP.

**¿Para qué sirve?**
- 🐛 Ver exactamente qué datos envías y recibes
- 🔍 Debuggear errores de API (ver el JSON real)
- 📊 Ver códigos de estado (200, 404, 500, etc.)

**Niveles de logging:**

| Nivel | ¿Qué muestra? | ¿Cuándo usar? |
|-------|---------------|---------------|
| `NONE` | Nada | Producción (app publicada) |
| `BASIC` | Método, URL, código de estado | Producción con logging mínimo |
| `HEADERS` | + Headers de petición/respuesta | Debugging de autenticación |
| `BODY` | + Cuerpo completo del JSON | **Desarrollo** (¡mucha info!) |

---

### 4.4 Crear RetrofitClient COMPLETO

Ahora vamos a juntar todo: **AuthInterceptor + HttpLoggingInterceptor + Retrofit**

Crea el archivo: **`data/remote/RetrofitClient.kt`**

```kotlin
package com.tuempresa.tuapp.data.remote  // ⚠️ Cambia esto por tu paquete

import android.content.Context
import com.tuempresa.tuapp.data.local.SessionManager
import okhttp3.OkHttpClient
import okhttp3.logging.HttpLoggingInterceptor
import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory
import java.util.concurrent.TimeUnit

object RetrofitClient {

    // ⚠️ CAMBIA ESTA URL POR LA DE TU API
    private const val BASE_URL = "https://dummyjson.com/"

    /**
     * Inicializa Retrofit con el contexto de la app
     * Llamar desde Application o ViewModel al inicio
     */
    fun create(context: Context): Retrofit {

        // 1️⃣ SessionManager para manejar el token
        val sessionManager = SessionManager(context)

        // 2️⃣ AuthInterceptor para inyectar el token automáticamente
        val authInterceptor = AuthInterceptor(sessionManager)

        // 3️⃣ HttpLoggingInterceptor para debugging
        val loggingInterceptor = HttpLoggingInterceptor().apply {
            level = HttpLoggingInterceptor.Level.BODY  // ⚠️ Cambiar a NONE en producción
        }

        // 4️⃣ OkHttpClient con AMBOS interceptores
        val okHttpClient = OkHttpClient.Builder()
            // ⚠️ ORDEN IMPORTANTE: AuthInterceptor primero, luego Logging
            .addInterceptor(authInterceptor)    // Añade el token
            .addInterceptor(loggingInterceptor)  // Muestra en Logcat (con token)
            .connectTimeout(15, TimeUnit.SECONDS)
            .readTimeout(20, TimeUnit.SECONDS)
            .build()

        // 5️⃣ Retrofit con el cliente configurado
        return Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .client(okHttpClient)
            .build()
    }
}
```

**¿Por qué el orden de interceptores importa?**

```
Petición HTTP (SIN token)
    ↓
[AuthInterceptor]   → Añade: Authorization: Bearer abc123
    ↓
[LoggingInterceptor] → Muestra en Logcat: Authorization: Bearer abc123
    ↓
Servidor recibe la petición CON token
```

Si inviertes el orden:
- ❌ El log mostrará la petición SIN el token (confuso para debugging)

---

### 4.5 Ejemplo de Logcat con Ambos Interceptores

Cuando ejecutes tu app, verás esto en Logcat:

```
D/OkHttp: --> POST https://dummyjson.com/user/login
D/OkHttp: Content-Type: application/json
D/OkHttp: {
D/OkHttp:   "username": "emilys",
D/OkHttp:   "password": "emilyspass",
D/OkHttp:   "expiresInMins": 30
D/OkHttp: }
D/OkHttp: --> END POST

D/OkHttp: <-- 200 OK (421ms)
D/OkHttp: Content-Type: application/json
D/OkHttp: {
D/OkHttp:   "id": 1,
D/OkHttp:   "username": "emilys",
D/OkHttp:   "email": "emily.johnson@x.dummyjson.com",
D/OkHttp:   "firstName": "Emily",
D/OkHttp:   "lastName": "Johnson",
D/OkHttp:   "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
D/OkHttp:   "refreshToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
D/OkHttp: }
D/OkHttp: <-- END HTTP

D/OkHttp: --> GET https://dummyjson.com/user/me
D/OkHttp: Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
D/OkHttp: --> END GET

D/OkHttp: <-- 200 OK (312ms)
D/OkHttp: Content-Type: application/json
D/OkHttp: {
D/OkHttp:   "id": 1,
D/OkHttp:   "username": "emilys",
D/OkHttp:   "email": "emily.johnson@x.dummyjson.com",
D/OkHttp:   "firstName": "Emily",
D/OkHttp:   "lastName": "Johnson",
D/OkHttp:   "image": "https://dummyjson.com/icon/emilys/128"
D/OkHttp: }
D/OkHttp: <-- END HTTP
```

**¿Qué ves aquí?**
- ✅ **Primera petición (LOGIN)**: Enviamos username/password, recibimos `accessToken`
- ✅ **Segunda petición (GET /user/me)**: AuthInterceptor añadió el token automáticamente
- ✅ El servidor reconoció el token y devolvió los datos del usuario autenticado
- ✅ Todo el flujo de autenticación JWT funcionando correctamente

---

### 4.6 ¿Cuándo Usar Cada Interceptor?

| Situación | AuthInterceptor | HttpLoggingInterceptor |
|-----------|-----------------|------------------------|
| **API pública (sin auth)** | ❌ No necesario | ✅ Útil para debugging |
| **API con login JWT** | ✅ **ESENCIAL** | ✅ Útil para debugging |
| **Producción (app publicada)** | ✅ Si usas auth | ⚠️ Cambiar a `NONE` |
| **Desarrollo/Testing** | ✅ Si usas auth | ✅ Nivel `BODY` |

---

## 📝 PASO 5: Crear el Modelo de Datos (DTO)

### 5.1 Crear el DTO

Crea el archivo: **`data/remote/dto/UserDto.kt`**

```kotlin
package com.tuempresa.tuapp.data.remote.dto  // ⚠️ Cambia esto por tu paquete

import com.google.gson.annotations.SerializedName

/**
 * DTO = Data Transfer Object
 * Este objeto representa los datos que VIAJAN entre tu app y el servidor
 */
data class UserDto(
    @SerializedName("id")
    val id: Int,

    @SerializedName("username")
    val username: String,

    @SerializedName("email")
    val email: String,

    @SerializedName("firstName")
    val firstName: String,

    @SerializedName("lastName")
    val lastName: String,

    @SerializedName("image")
    val image: String? = null  // URL de imagen de perfil (opcional)
)
```

### 5.2 Crear DTOs de Login

Para el login necesitamos **dos DTOs**: uno para enviar (request) y otro para recibir (response).

Crea el archivo: **`data/remote/dto/LoginRequest.kt`**

```kotlin
package com.tuempresa.tuapp.data.remote.dto

import com.google.gson.annotations.SerializedName

/**
 * DTO para la petición de login
 * Datos que ENVIAMOS al servidor
 */
data class LoginRequest(
    @SerializedName("username")
    val username: String,

    @SerializedName("password")
    val password: String,

    @SerializedName("expiresInMins")
    val expiresInMins: Int = 30  // Token expira en 30 minutos
)
```

Crea el archivo: **`data/remote/dto/LoginResponse.kt`**

```kotlin
package com.tuempresa.tuapp.data.remote.dto

import com.google.gson.annotations.SerializedName

/**
 * DTO para la respuesta de login
 * Datos que RECIBIMOS del servidor tras login exitoso
 */
data class LoginResponse(
    @SerializedName("id")
    val id: Int,

    @SerializedName("username")
    val username: String,

    @SerializedName("email")
    val email: String,

    @SerializedName("firstName")
    val firstName: String,

    @SerializedName("lastName")
    val lastName: String,

    @SerializedName("accessToken")
    val accessToken: String,  // 🔑 TOKEN JWT - Lo guardamos en SessionManager

    @SerializedName("refreshToken")
    val refreshToken: String?  // Opcional - Para renovar el token
)
```

### 5.3 Crear DTO para Lista de Usuarios

Crea el archivo: **`data/remote/dto/UsersResponse.kt`**

```kotlin
package com.tuempresa.tuapp.data.remote.dto

import com.google.gson.annotations.SerializedName

/**
 * DTO para la respuesta de lista de usuarios
 * La API devuelve un objeto con "users" y metadata de paginación
 */
data class UsersResponse(
    @SerializedName("users")
    val users: List<UserDto>,

    @SerializedName("total")
    val total: Int,  // Total de usuarios en la base de datos

    @SerializedName("skip")
    val skip: Int,   // Cuántos usuarios se saltaron (paginación)

    @SerializedName("limit")
    val limit: Int   // Límite de usuarios por página
)
```

**¿Qué es `@SerializedName`?**

Le dice a Gson qué campo del JSON corresponde a cada variable de Kotlin.

**Ejemplo:**
```json
{
  "id": 1,
  "name": "Juan Pérez",
  "email": "juan@ejemplo.com"
}
```

Gson automáticamente convierte este JSON → `UserDto` object.

**⚠️ IMPORTANTE:** Ajusta los campos según lo que devuelva TU API.

---

## 🌐 PASO 6: Definir los Endpoints (API Service)

### 6.1 Crear ApiService

Crea el archivo: **`data/remote/ApiService.kt`**

```kotlin
package com.tuempresa.tuapp.data.remote  // ⚠️ Cambia esto por tu paquete

import com.tuempresa.tuapp.data.remote.dto.*
import retrofit2.http.*

/**
 * Define los endpoints de tu API
 * Usando DummyJSON como ejemplo de API REST con autenticación JWT
 */
interface ApiService {

    /**
     * 🔐 LOGIN - Autenticar usuario
     * POST /user/login
     *
     * Ejemplo de uso:
     * val response = apiService.login(LoginRequest("emilys", "emilyspass"))
     * sessionManager.saveAuthToken(response.accessToken)
     */
    @POST("user/login")
    suspend fun login(@Body request: LoginRequest): LoginResponse

    /**
     * 👤 OBTENER USUARIO ACTUAL (requiere autenticación)
     * GET /user/me
     *
     * ⚠️ IMPORTANTE: Este endpoint REQUIERE el token JWT
     * El AuthInterceptor lo añade automáticamente
     *
     * Ejemplo de uso:
     * val currentUser = apiService.getCurrentUser()
     */
    @GET("user/me")
    suspend fun getCurrentUser(): UserDto

    /**
     * 📋 OBTENER LISTA DE USUARIOS
     * GET /users
     *
     * Ejemplo de uso:
     * val response = apiService.getUsers()
     * val usersList = response.users  // Lista de UserDto
     */
    @GET("users")
    suspend fun getUsers(): UsersResponse

    /**
     * 🔍 BUSCAR USUARIOS POR NOMBRE
     * GET /users/search?q={query}
     *
     * Ejemplo de uso:
     * val results = apiService.searchUsers("John")
     */
    @GET("users/search")
    suspend fun searchUsers(@Query("q") query: String): UsersResponse

    /**
     * 👤 OBTENER USUARIO POR ID
     * GET /users/{id}
     *
     * Ejemplo de uso:
     * val user = apiService.getUserById(1)
     */
    @GET("users/{id}")
    suspend fun getUserById(@Path("id") id: Int): UserDto
}
```

**Anotaciones de Retrofit:**

| Anotación | ¿Para qué sirve? | Ejemplo |
|-----------|------------------|---------|
| `@GET` | Petición GET | Obtener datos |
| `@POST` | Petición POST | Crear datos / Login |
| `@PUT` | Petición PUT | Actualizar datos |
| `@DELETE` | Petición DELETE | Eliminar datos |
| `@Path` | Variable en la URL | `/users/{id}` → id=1 |
| `@Body` | Cuerpo de la petición | JSON en POST/PUT |
| `@Query` | Parámetro de query | `/search?q=John` |

**Flujo de Login con DummyJSON:**

```
1. Usuario ingresa username/password en LoginScreen
   ↓
2. LoginViewModel llama a apiService.login(LoginRequest(...))
   ↓
3. Servidor valida credenciales y devuelve LoginResponse con accessToken
   ↓
4. Guardamos el token: sessionManager.saveAuthToken(response.accessToken)
   ↓
5. AuthInterceptor automáticamente añade el token a futuras peticiones
   ↓
6. Podemos llamar a getCurrentUser() sin pasar el token manualmente
```

**Credenciales de prueba para DummyJSON:**

| Username | Password | Descripción |
|----------|----------|-------------|
| `emilys` | `emilyspass` | Usuario de prueba |
| `michaelw` | `michaelwpass` | Otro usuario de prueba |

**⚠️ IMPORTANTE:** Ajusta los endpoints según tu API.

---

## 🗂️ PASO 7: Crear el Repository

### 7.1 Crear UserRepository

Crea el archivo: **`data/repository/UserRepository.kt`**

```kotlin
package com.tuempresa.tuapp.data.repository  // ⚠️ Cambia esto por tu paquete

import com.tuempresa.tuapp.data.remote.ApiService
import com.tuempresa.tuapp.data.remote.RetrofitClient
import com.tuempresa.tuapp.data.remote.dto.UserDto

/**
 * Repository: Abstrae la fuente de datos
 * El ViewModel NO sabe si los datos vienen de API, base de datos local, etc.
 */
class UserRepository(context: Context) {

    // Crear la instancia del API Service (pasando el contexto)
    private val apiService: ApiService = RetrofitClient
        .create(context)
        .create(ApiService::class.java)

    /**
     * Obtiene un usuario de la API
     *
     * Usa Result<T> para manejar éxito/error de forma elegante
     */
    suspend fun fetchUser(id: Int = 1): Result<UserDto> {
        return try {
            // Llamar a la API (esto puede tardar varios segundos)
            val user = apiService.getUser(id)

            // Retornar éxito
            Result.success(user)

        } catch (e: Exception) {
            // Si algo falla (sin internet, timeout, etc.)
            Result.failure(e)
        }
    }
}
```

**¿Por qué usar Repository?**

- ✅ **Abstracción**: El ViewModel no sabe de dónde vienen los datos
- ✅ **Testeable**: Puedes inyectar un repository falso en tests
- ✅ **Mantenible**: Si cambias de API, solo modificas el repository

---

## 🎨 PASO 8: Crear el ViewModel

### 8.1 Crear ProfileViewModel

Crea el archivo: **`ui/profile/ProfileViewModel.kt`**

```kotlin
package com.tuempresa.tuapp.ui.profile  // ⚠️ Cambia esto por tu paquete

import android.app.Application
import androidx.lifecycle.AndroidViewModel
import androidx.lifecycle.viewModelScope
import com.tuempresa.tuapp.data.repository.UserRepository
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import kotlinx.coroutines.launch

/**
 * Estado de la UI
 */
data class ProfileUiState(
    val isLoading: Boolean = false,
    val userName: String = "",
    val userEmail: String = "",
    val error: String? = null
)

/**
 * ViewModel: Maneja la lógica de UI y el estado
 * Usa AndroidViewModel para tener acceso al Application Context
 */
class ProfileViewModel(application: Application) : AndroidViewModel(application) {

    private val repository = UserRepository(application)

    // Estado PRIVADO (solo el ViewModel lo modifica)
    private val _uiState = MutableStateFlow(ProfileUiState())

    // Estado PÚBLICO (la UI lo observa)
    val uiState: StateFlow<ProfileUiState> = _uiState

    /**
     * Carga los datos del usuario desde la API
     */
    fun loadUser(id: Int = 1) {
        // Indicar que está cargando
        _uiState.value = _uiState.value.copy(
            isLoading = true,
            error = null
        )

        // Ejecutar en coroutine (no bloquea la UI)
        viewModelScope.launch {
            val result = repository.fetchUser(id)

            // Actualizar el estado según el resultado
            _uiState.value = result.fold(
                onSuccess = { user ->
                    // ✅ Éxito: mostrar datos
                    _uiState.value.copy(
                        isLoading = false,
                        userName = user.name,
                        userEmail = user.email ?: "Sin email",
                        error = null
                    )
                },
                onFailure = { exception ->
                    // ❌ Error: mostrar mensaje
                    _uiState.value.copy(
                        isLoading = false,
                        error = exception.localizedMessage ?: "Error desconocido"
                    )
                }
            )
        }
    }
}
```

**Flujo de estados:**

```
1. Usuario abre la pantalla
   → isLoading = true

2. Se hace la petición HTTP
   → Puede tardar 1-5 segundos...

3a. Si éxito:
    → isLoading = false
    → userName = "Juan"
    → userEmail = "juan@ejemplo.com"

3b. Si error:
    → isLoading = false
    → error = "Sin conexión a internet"
```

---

## 📱 PASO 9: Crear la Pantalla Compose

### 9.1 Crear ProfileScreen

Crea el archivo: **`ui/profile/ProfileScreen.kt`**

```kotlin
package com.tuempresa.tuapp.ui.profile  // ⚠️ Cambia esto por tu paquete

import androidx.compose.foundation.layout.*
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.unit.dp
import androidx.lifecycle.viewmodel.compose.viewModel

@Composable
fun ProfileScreen(
    viewModel: ProfileViewModel = viewModel()
) {
    // Observar el estado
    val state by viewModel.uiState.collectAsState()

    // Cargar datos cuando la pantalla se abre
    LaunchedEffect(Unit) {
        viewModel.loadUser(1)  // ⚠️ Cambia el ID según necesites
    }

    Box(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        when {
            // Estado: Cargando
            state.isLoading -> {
                CircularProgressIndicator(
                    modifier = Modifier.align(Alignment.Center)
                )
            }

            // Estado: Error
            state.error != null -> {
                Column(
                    modifier = Modifier.align(Alignment.Center),
                    horizontalAlignment = Alignment.CenterHorizontally
                ) {
                    Text(
                        text = "❌ Error",
                        style = MaterialTheme.typography.titleLarge,
                        color = MaterialTheme.colorScheme.error
                    )
                    Spacer(modifier = Modifier.height(8.dp))
                    Text(
                        text = state.error ?: "",
                        textAlign = TextAlign.Center,
                        color = MaterialTheme.colorScheme.error
                    )
                    Spacer(modifier = Modifier.height(16.dp))
                    Button(onClick = { viewModel.loadUser(1) }) {
                        Text("Reintentar")
                    }
                }
            }

            // Estado: Datos cargados
            else -> {
                Column(
                    modifier = Modifier.align(Alignment.TopCenter),
                    horizontalAlignment = Alignment.CenterHorizontally,
                    verticalArrangement = Arrangement.spacedBy(12.dp)
                ) {
                    Text(
                        text = "Perfil de Usuario",
                        style = MaterialTheme.typography.headlineMedium
                    )

                    Spacer(modifier = Modifier.height(16.dp))

                    // Nombre
                    Card(
                        modifier = Modifier.fillMaxWidth()
                    ) {
                        Column(modifier = Modifier.padding(16.dp)) {
                            Text(
                                text = "Nombre",
                                style = MaterialTheme.typography.labelMedium,
                                color = MaterialTheme.colorScheme.primary
                            )
                            Spacer(modifier = Modifier.height(4.dp))
                            Text(
                                text = state.userName,
                                style = MaterialTheme.typography.bodyLarge
                            )
                        }
                    }

                    // Email
                    Card(
                        modifier = Modifier.fillMaxWidth()
                    ) {
                        Column(modifier = Modifier.padding(16.dp)) {
                            Text(
                                text = "Email",
                                style = MaterialTheme.typography.labelMedium,
                                color = MaterialTheme.colorScheme.primary
                            )
                            Spacer(modifier = Modifier.height(4.dp))
                            Text(
                                text = state.userEmail,
                                style = MaterialTheme.typography.bodyLarge
                            )
                        }
                    }

                    Spacer(modifier = Modifier.height(16.dp))

                    Button(onClick = { viewModel.loadUser(1) }) {
                        Text("Refrescar")
                    }
                }
            }
        }
    }
}
```

---

## 🎯 PASO 10: Integrar en tu Navigation

### 10.1 Agregar la ruta

En tu archivo de navegación (ej: `Navigation.kt`), agrega:

```kotlin
composable("profile") {
    ProfileScreen()
}
```

### 10.2 Navegar a la pantalla

Desde cualquier otra pantalla:

```kotlin
Button(onClick = { navController.navigate("profile") }) {
    Text("Ver Perfil")
}
```

---

## ✅ PASO 11: Probar que Funciona

### 11.1 Ejecutar la app

1. Haz clic en el botón **Run** (triángulo verde)
2. Espera a que la app se instale
3. Navega a tu pantalla de perfil

### 11.2 Verificar en Logcat

Abre Logcat (parte inferior de Android Studio) y busca:

```
--> POST https://dummyjson.com/user/login
<-- 200 OK
{
  "accessToken": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  ...
}

--> GET https://dummyjson.com/user/me
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
<-- 200 OK
{
  "id": 1,
  "username": "emilys",
  "email": "emily.johnson@x.dummyjson.com"
}
```

Si ves esto, ¡funciona! ✅ El token se guardó y se está usando automáticamente.

### 11.3 Probar los estados

**Estado de carga:**
- Debería aparecer el `CircularProgressIndicator` brevemente

**Estado de éxito:**
- Debe mostrar el nombre y email del usuario

**Estado de error:**
- Desactiva el WiFi/datos
- Presiona "Refrescar"
- Debe mostrar el mensaje de error

---

## 🎨 PASO 12: Personalizar para TU API

### 12.1 Cambiar la URL base

En `RetrofitClient.kt`:

```kotlin
// Ejemplo con DummyJSON (usada en esta guía):
private const val BASE_URL = "https://dummyjson.com/"

// O cambia por la URL de TU API:
private const val BASE_URL = "https://tu-api.com/api/v1/"

// Ejemplo con backend local:
private const val BASE_URL = "http://10.0.2.2:3000/"  // Android Emulator → localhost
```

### 12.2 Ajustar el DTO

En `UserDto.kt`, cambia los campos según lo que devuelva tu API:

```kotlin
data class UserDto(
    @SerializedName("id")
    val id: Int,

    // ⚠️ Agrega/quita campos según tu API:
    @SerializedName("firstName")
    val firstName: String,

    @SerializedName("lastName")
    val lastName: String,

    @SerializedName("profilePicture")
    val profilePicture: String?
)
```

### 12.3 Ajustar los endpoints

En `ApiService.kt`:

```kotlin
// Cambia la ruta según tu API:
@GET("usuarios/{id}")  // ← En español
suspend fun getUser(@Path("id") id: Int): UserDto
```

---

## 🐛 Solución de Problemas Comunes

### Error: "Unable to resolve host"

**Causa:** No hay conexión a internet

**Solución:**
1. Verifica que el permiso `INTERNET` esté en el Manifest
2. Verifica que el dispositivo/emulador tenga internet
3. Prueba la URL en el navegador

### Error: "JSON parsing error"

**Causa:** Los campos del DTO no coinciden con el JSON

**Solución:**
1. Revisa el Logcat para ver el JSON real
2. Ajusta los campos del DTO
3. Asegúrate de que los `@SerializedName` sean correctos

### Error: "Timeout"

**Causa:** El servidor no responde a tiempo

**Solución:**
1. Aumenta los timeouts en `RetrofitClient`:
   ```kotlin
   .connectTimeout(30, TimeUnit.SECONDS)
   .readTimeout(30, TimeUnit.SECONDS)
   ```
2. Verifica que la URL sea correcta

### La pantalla se queda en "Loading" infinito

**Causa:** Error en la petición pero no se maneja

**Solución:**
1. Revisa Logcat para ver el error
2. Asegúrate de tener el `try-catch` en el Repository
3. Verifica que el ViewModel actualice el estado en `onFailure`

---

## 📚 Conceptos Clave para Recordar

| Concepto | Explicación |
|----------|-------------|
| **Retrofit** | Librería que simplifica las llamadas HTTP |
| **OkHttp** | Cliente HTTP que Retrofit usa internamente |
| **DTO** | Data Transfer Object - Objetos que viajan entre app y servidor |
| **suspend** | Función de coroutine que puede ejecutarse de forma asíncrona |
| **StateFlow** | Stream reactivo de estados (la UI se actualiza automáticamente) |
| **Repository** | Abstrae de dónde vienen los datos (API, BD local, etc.) |
| **ViewModel** | Maneja la lógica de UI y sobrevive a cambios de configuración |

---

## 🚀 Próximos Pasos

Una vez que tengas esto funcionando, puedes:

1. **Agregar caché local** con Room Database
2. **Agregar autenticación** con tokens JWT
3. **Mejorar el UI** con animaciones y estados de transición
4. **Agregar paginación** para listas grandes
5. **Implementar refresh** con SwipeRefresh

---

## 📝 Checklist Final

Marca cada punto cuando lo completes:

- [ ] Dependencias agregadas y sincronizadas (OkHttp, Retrofit, Coroutines)
- [ ] Permiso de INTERNET en el Manifest
- [ ] `SessionManager` creado para guardar tokens
- [ ] `AuthInterceptor` creado para inyectar token automáticamente
- [ ] `RetrofitClient` creado con ambos interceptores
- [ ] DTO creado con los campos correctos
- [ ] `ApiService` con los endpoints correctos
- [ ] `Repository` creado
- [ ] `ViewModel` creado con StateFlow
- [ ] `ProfileScreen` creada con manejo de estados
- [ ] Integrado en la navegación
- [ ] Probado que funciona (Logcat muestra peticiones y respuestas)
- [ ] Personalizado para MI API

---

## 🎓 ¿Necesitas Ayuda?

**Recursos útiles:**
- [Documentación de Retrofit](https://square.github.io/retrofit/)
- [Guía de Coroutines](https://developer.android.com/kotlin/coroutines)
- [StateFlow en Compose](https://developer.android.com/jetpack/compose/state)

**Tips para debugging:**
- Usa Logcat para ver las peticiones HTTP
- Prueba los endpoints en Postman primero
- Revisa que el JSON coincida con tu DTO

---

**Última actualización:** Octubre 2025
**Versión:** 1.0 - Guía Universal