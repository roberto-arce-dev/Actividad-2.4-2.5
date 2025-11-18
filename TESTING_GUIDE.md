# Guía de Pruebas Unitarias - Mi App Modular

## 📋 Índice
1. [Configuración Inicial](#configuración-inicial)
2. [Estructura de Carpetas](#estructura-de-carpetas)
3. [Pruebas de Repository](#pruebas-de-repository)
4. [Pruebas de ViewModel](#pruebas-de-viewmodel)
5. [Buenas Prácticas](#buenas-prácticas)
6. [Ejecución de Tests](#ejecución-de-tests)

---

## 1. Configuración Inicial

### Dependencias Necesarias

Agrega estas dependencias a tu `app/build.gradle.kts`:

```kotlin
dependencies {
    // ... otras dependencias ...

    // Testing - JUnit
    testImplementation("junit:junit:4.13.2")

    // Testing - Coroutines
    testImplementation("org.jetbrains.kotlinx:kotlinx-coroutines-test:1.7.3")

    // Testing - MockK (mocking framework)
    testImplementation("io.mockk:mockk:1.13.8")
    testImplementation("io.mockk:mockk-android:1.13.8")

    // Testing - Turbine (para testar StateFlow/Flow)
    testImplementation("app.cash.turbine:turbine:1.0.0")

    // Testing - Core Testing (para InstantTaskExecutorRule)
    testImplementation("androidx.arch.core:core-testing:2.2.0")

    // Android Testing
    androidTestImplementation("androidx.test.ext:junit:1.1.5")
    androidTestImplementation("androidx.test.espresso:espresso-core:3.5.1")
}
```

---

## 2. Estructura de Carpetas

Tu proyecto debe tener esta estructura:

```
app/src/
├── main/
│   └── java/com/example/miappmodular/
│       ├── repository/
│       │   └── AuthSaborLocalRepository.kt
│       ├── viewmodel/
│       │   ├── LoginViewModel.kt
│       │   └── RegisterViewModel.kt
│       └── model/
│           └── SaborLocalModels.kt
└── test/  ← Aquí van los tests unitarios
    └── java/com/example/miappmodular/
        ├── repository/
        │   └── AuthSaborLocalRepositoryTest.kt
        └── viewmodel/
            ├── LoginViewModelTest.kt
            └── RegisterViewModelTest.kt
```

**Cómo crear la estructura:**
1. En Android Studio, click derecho en `app/src/test/java`
2. New → Package → `com.example.miappmodular.repository`
3. New → Package → `com.example.miappmodular.viewmodel`

---

## 3. Pruebas de Repository

### 3.1. AuthSaborLocalRepositoryTest

**Ubicación:** `app/src/test/java/com/example/miappmodular/repository/AuthSaborLocalRepositoryTest.kt`

```kotlin
package com.example.miappmodular.repository

import android.content.Context
import android.content.SharedPreferences
import com.example.miappmodular.data.remote.SaborLocalApiService
import com.example.miappmodular.data.remote.dto.*
import com.example.miappmodular.model.User
import io.mockk.*
import kotlinx.coroutines.test.runTest
import org.junit.After
import org.junit.Before
import org.junit.Test
import retrofit2.Response
import kotlin.test.assertEquals
import kotlin.test.assertTrue

/**
 * Pruebas unitarias para AuthSaborLocalRepository
 *
 * Cubre:
 * - Login exitoso
 * - Login con credenciales inválidas
 * - Registro exitoso
 * - Registro con email duplicado
 * - Creación de productor
 * - Obtener todos los usuarios
 */
class AuthSaborLocalRepositoryTest {

    // Mocks
    private lateinit var mockContext: Context
    private lateinit var mockApiService: SaborLocalApiService
    private lateinit var mockSharedPreferences: SharedPreferences
    private lateinit var mockEditor: SharedPreferences.Editor

    // SUT (System Under Test)
    private lateinit var repository: AuthSaborLocalRepository

    @Before
    fun setup() {
        // Crear mocks
        mockContext = mockk(relaxed = true)
        mockApiService = mockk()
        mockSharedPreferences = mockk(relaxed = true)
        mockEditor = mockk(relaxed = true)

        // Configurar comportamiento de SharedPreferences
        every { mockContext.getSharedPreferences(any(), any()) } returns mockSharedPreferences
        every { mockSharedPreferences.edit() } returns mockEditor
        every { mockEditor.putString(any(), any()) } returns mockEditor
        every { mockEditor.apply() } just Runs
        every { mockEditor.clear() } returns mockEditor

        // Mockear RetrofitClient para que use nuestro mock
        mockkObject(com.example.miappmodular.data.remote.RetrofitClient)
        every { com.example.miappmodular.data.remote.RetrofitClient.saborLocalApiService } returns mockApiService

        // Crear instancia del repository
        repository = AuthSaborLocalRepository(mockContext)
    }

    @After
    fun teardown() {
        // Limpiar todos los mocks
        unmockkAll()
    }

    // ==================== LOGIN TESTS ====================

    @Test
    fun `login exitoso debe retornar Success con User`() = runTest {
        // Given - Preparar datos de prueba
        val email = "test@example.com"
        val password = "password123"

        val userDto = UserDto(
            id = "user123",
            nombre = "Test User",
            email = email,
            role = "CLIENTE",
            telefono = "123456789",
            ubicacion = "Santiago",
            direccion = "Calle Falsa 123"
        )

        val authData = AuthSaborLocalData(
            user = userDto,
            accessToken = "mock_token_12345"
        )

        val apiResponse = ApiResponse(
            success = true,
            message = "Login exitoso",
            data = authData
        )

        val response = Response.success(apiResponse)

        // Configurar mock para retornar respuesta exitosa
        coEvery {
            mockApiService.login(
                LoginSaborLocalRequest(email, password)
            )
        } returns response

        // When - Ejecutar login
        val result = repository.login(email, password)

        // Then - Verificar resultado
        assertTrue(result.isSuccess, "El resultado debe ser Success")

        val user = result.getOrNull()
        assertEquals("user123", user?.id)
        assertEquals("Test User", user?.nombre)
        assertEquals(email, user?.email)
        assertEquals("CLIENTE", user?.role)

        // Verificar que se guardó el token
        verify { mockEditor.putString("auth_token", "mock_token_12345") }
        verify { mockEditor.putString("user_id", "user123") }
        verify { mockEditor.apply() }
    }

    @Test
    fun `login con credenciales inválidas debe retornar Failure`() = runTest {
        // Given
        val email = "wrong@example.com"
        val password = "wrongpassword"

        val response = Response.error<ApiResponse<AuthSaborLocalData>>(
            401,
            okhttp3.ResponseBody.create(null, "")
        )

        coEvery {
            mockApiService.login(any())
        } returns response

        // When
        val result = repository.login(email, password)

        // Then
        assertTrue(result.isFailure, "El resultado debe ser Failure")
        assertEquals(
            "Credenciales inválidas",
            result.exceptionOrNull()?.message
        )
    }

    @Test
    fun `login con error de red debe retornar Failure con mensaje de error`() = runTest {
        // Given
        coEvery {
            mockApiService.login(any())
        } throws Exception("Network error")

        // When
        val result = repository.login("test@example.com", "password")

        // Then
        assertTrue(result.isFailure)
        assertTrue(
            result.exceptionOrNull()?.message?.contains("Error de red") == true
        )
    }

    // ==================== REGISTER TESTS ====================

    @Test
    fun `registro exitoso debe retornar Success con User`() = runTest {
        // Given
        val nombre = "Nuevo Usuario"
        val email = "nuevo@example.com"
        val password = "password123"

        val userDto = UserDto(
            id = "newuser123",
            nombre = nombre,
            email = email,
            role = "CLIENTE",
            telefono = null,
            ubicacion = null,
            direccion = null
        )

        val authData = AuthSaborLocalData(
            user = userDto,
            accessToken = "new_token_12345"
        )

        val apiResponse = ApiResponse(
            success = true,
            data = authData
        )

        val response = Response.success(apiResponse)

        coEvery {
            mockApiService.register(any())
        } returns response

        // When
        val result = repository.register(
            nombre = nombre,
            email = email,
            password = password
        )

        // Then
        assertTrue(result.isSuccess)
        val user = result.getOrNull()
        assertEquals(nombre, user?.nombre)
        assertEquals(email, user?.email)
        assertEquals("CLIENTE", user?.role)
    }

    @Test
    fun `registro con email duplicado debe retornar Failure`() = runTest {
        // Given
        val response = Response.error<ApiResponse<AuthSaborLocalData>>(
            409,
            okhttp3.ResponseBody.create(null, "")
        )

        coEvery {
            mockApiService.register(any())
        } returns response

        // When
        val result = repository.register(
            nombre = "Test",
            email = "existing@example.com",
            password = "password123"
        )

        // Then
        assertTrue(result.isFailure)
        assertEquals(
            "El email ya está registrado",
            result.exceptionOrNull()?.message
        )
    }

    // ==================== CREATE PRODUCTOR TESTS ====================

    @Test
    fun `createProductorUser exitoso debe retornar Success`() = runTest {
        // Given
        val userDto = UserDto(
            id = "productor123",
            nombre = "Productor Test",
            email = "productor@example.com",
            role = "PRODUCTOR",
            telefono = "987654321",
            ubicacion = "Valparaíso",
            direccion = null
        )

        val authData = AuthSaborLocalData(
            user = userDto,
            accessToken = "productor_token"
        )

        val apiResponse = ApiResponse(
            success = true,
            data = authData
        )

        val response = Response.success(apiResponse)

        coEvery {
            mockApiService.createProductorUser(any())
        } returns response

        // When
        val result = repository.createProductorUser(
            nombre = "Productor Test",
            email = "productor@example.com",
            password = "password123",
            ubicacion = "Valparaíso",
            telefono = "987654321"
        )

        // Then
        assertTrue(result.isSuccess)
        val user = result.getOrNull()
        assertEquals("PRODUCTOR", user?.role)
        assertEquals("Valparaíso", user?.ubicacion)
    }

    @Test
    fun `createProductorUser sin permisos debe retornar Failure`() = runTest {
        // Given - Usuario no ADMIN
        val response = Response.error<ApiResponse<AuthSaborLocalData>>(
            403,
            okhttp3.ResponseBody.create(null, "")
        )

        coEvery {
            mockApiService.createProductorUser(any())
        } returns response

        // When
        val result = repository.createProductorUser(
            nombre = "Test",
            email = "test@example.com",
            password = "password",
            ubicacion = "Santiago",
            telefono = "123456789"
        )

        // Then
        assertTrue(result.isFailure)
        assertTrue(
            result.exceptionOrNull()?.message?.contains("No tienes permisos") == true
        )
    }

    // ==================== GET ALL USERS TESTS ====================

    @Test
    fun `getAllUsers debe retornar lista de usuarios`() = runTest {
        // Given
        val users = listOf(
            UserDto(
                id = "1",
                nombre = "User 1",
                email = "user1@example.com",
                role = "CLIENTE",
                telefono = null,
                ubicacion = null,
                direccion = null
            ),
            UserDto(
                id = "2",
                nombre = "User 2",
                email = "user2@example.com",
                role = "PRODUCTOR",
                telefono = "123456789",
                ubicacion = "Santiago",
                direccion = null
            )
        )

        val apiResponse = ApiResponse(
            success = true,
            data = users
        )

        val response = Response.success(apiResponse)

        coEvery {
            mockApiService.getAllUsers()
        } returns response

        // When
        val result = repository.getAllUsers()

        // Then
        assertTrue(result.isSuccess)
        val userList = result.getOrNull()
        assertEquals(2, userList?.size)
        assertEquals("User 1", userList?.get(0)?.nombre)
        assertEquals("PRODUCTOR", userList?.get(1)?.role)
    }

    // ==================== SESSION TESTS ====================

    @Test
    fun `isLoggedIn debe retornar true cuando hay token`() {
        // Given
        every { mockSharedPreferences.getString("auth_token" வேண்ட
        every { mockSharedPreferences.getString("auth_token", null) } returns "mock_token"

        // When
        val isLoggedIn = repository.isLoggedIn()

        // Then
        assertTrue(isLoggedIn)
    }

    @Test
    fun `isLoggedIn debe retornar false cuando no hay token`() {
        // Given
        every { mockSharedPreferences.getString("auth_token", null) } returns null

        // When
        val isLoggedIn = repository.isLoggedIn()

        // Then
        assertTrue(!isLoggedIn)
    }

    @Test
    fun `logout debe limpiar SharedPreferences`() {
        // When
        repository.logout()

        // Then
        verify { mockEditor.clear() }
        verify { mockEditor.apply() }
    }
}
```

---

## 4. Pruebas de ViewModel

### 4.1. LoginViewModelTest

**Ubicación:** `app/src/test/java/com/example/miappmodular/viewmodel/LoginViewModelTest.kt`

```kotlin
package com.example.miappmodular.viewmodel

import android.app.Application
import androidx.arch.core.executor.testing.InstantTaskExecutorRule
import app.cash.turbine.test
import com.example.miappmodular.model.User
import com.example.miappmodular.repository.AuthSaborLocalRepository
import io.mockk.*
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.ExperimentalCoroutinesApi
import kotlinx.coroutines.test.*
import org.junit.After
import org.junit.Before
import org.junit.Rule
import org.junit.Test
import kotlin.test.assertEquals
import kotlin.test.assertFalse
import kotlin.test.assertTrue

/**
 * Pruebas unitarias para LoginViewModel
 *
 * Cubre:
 * - Validación de email
 * - Validación de password
 * - Login exitoso
 * - Login fallido
 * - Estados de UI (Loading, Success, Error, Idle)
 */
@OptIn(ExperimentalCoroutinesApi::class)
class LoginViewModelTest {

    // Regla para ejecutar código en el mismo thread
    @get:Rule
    val instantExecutorRule = InstantTaskExecutorRule()

    // Dispatcher de prueba para coroutines
    private val testDispatcher = StandardTestDispatcher()

    // Mocks
    private lateinit var mockApplication: Application
    private lateinit var mockRepository: AuthSaborLocalRepository

    // SUT
    private lateinit var viewModel: LoginViewModel

    @Before
    fun setup() {
        // Configurar dispatcher de prueba
        Dispatchers.setMain(testDispatcher)

        // Crear mocks
        mockApplication = mockk(relaxed = true)
        mockRepository = mockk()

        // Mockear contexto de la aplicación
        every { mockApplication.applicationContext } returns mockApplication

        // TODO: Necesitarás modificar LoginViewModel para inyectar el repository
        // Por ahora, asumiremos que puedes pasar el repository al constructor
        // viewModel = LoginViewModel(mockApplication, mockRepository)
    }

    @After
    fun teardown() {
        Dispatchers.resetMain()
        unmockkAll()
    }

    // ==================== VALIDATION TESTS ====================

    @Test
    fun `email vacío debe mostrar error`() = runTest {
        // Given
        viewModel = LoginViewModel(mockApplication)

        // When
        viewModel.onEmailChange("")

        // Then
        viewModel.emailError.test {
            val error = awaitItem()
            assertEquals("El email es obligatorio", error)
        }
    }

    @Test
    fun `email inválido debe mostrar error`() = runTest {
        // Given
        viewModel = LoginViewModel(mockApplication)

        // When
        viewModel.onEmailChange("invalid-email")

        // Then
        viewModel.emailError.test {
            val error = awaitItem()
            assertTrue(error?.contains("Email inválido") == true)
        }
    }

    @Test
    fun `email válido no debe mostrar error`() = runTest {
        // Given
        viewModel = LoginViewModel(mockApplication)

        // When
        viewModel.onEmailChange("valid@example.com")

        // Then
        viewModel.emailError.test {
            val error = awaitItem()
            assertEquals(null, error)
        }
    }

    @Test
    fun `password vacío debe mostrar error`() = runTest {
        // Given
        viewModel = LoginViewModel(mockApplication)

        // When
        viewModel.onPasswordChange("")

        // Then
        viewModel.passwordError.test {
            val error = awaitItem()
            assertEquals("La contraseña es obligatoria", error)
        }
    }

    // ==================== LOGIN TESTS ====================

    @Test
    fun `login exitoso debe cambiar estado a Success`() = runTest {
        // Given
        val mockUser = User(
            id = "123",
            nombre = "Test User",
            email = "test@example.com",
            role = "CLIENTE"
        )

        coEvery {
            mockRepository.login(any(), any())
        } returns Result.success(mockUser)

        viewModel = LoginViewModel(mockApplication)

        // When
        viewModel.onEmailChange("test@example.com")
        viewModel.onPasswordChange("password123")
        viewModel.login()

        // Avanzar el dispatcher para ejecutar coroutines
        advanceUntilIdle()

        // Then
        viewModel.uiState.test {
            val state = awaitItem()
            assertTrue(state is LoginUiState.Success)
            assertEquals(mockUser, (state as LoginUiState.Success).user)
        }
    }

    @Test
    fun `login con credenciales inválidas debe cambiar estado a Error`() = runTest {
        // Given
        coEvery {
            mockRepository.login(any(), any())
        } returns Result.failure(Exception("Credenciales inválidas"))

        viewModel = LoginViewModel(mockApplication)

        // When
        viewModel.onEmailChange("test@example.com")
        viewModel.onPasswordChange("wrongpassword")
        viewModel.login()

        advanceUntilIdle()

        // Then
        viewModel.uiState.test {
            val state = awaitItem()
            assertTrue(state is LoginUiState.Error)
            assertEquals(
                "Credenciales inválidas",
                (state as LoginUiState.Error).message
            )
        }
    }

    @Test
    fun `login debe cambiar a Loading y luego a Success`() = runTest {
        // Given
        val mockUser = User(
            id = "123",
            nombre = "Test",
            email = "test@example.com",
            role = "CLIENTE"
        )

        coEvery {
            mockRepository.login(any(), any())
        } coAnswers {
            // Simular delay de red
            kotlinx.coroutines.delay(100)
            Result.success(mockUser)
        }

        viewModel = LoginViewModel(mockApplication)
        viewModel.onEmailChange("test@example.com")
        viewModel.onPasswordChange("password123")

        // When - Then
        viewModel.uiState.test {
            // Estado inicial debe ser Idle
            assertEquals(LoginUiState.Idle, awaitItem())

            viewModel.login()

            // Debe cambiar a Loading
            assertEquals(LoginUiState.Loading, awaitItem())

            advanceTimeBy(100)

            // Debe cambiar a Success
            val successState = awaitItem()
            assertTrue(successState is LoginUiState.Success)
        }
    }

    @Test
    fun `login con campos vacíos no debe llamar al repository`() = runTest {
        // Given
        viewModel = LoginViewModel(mockApplication)

        // When
        viewModel.login() // Email y password vacíos

        // Then
        coVerify(exactly = 0) {
            mockRepository.login(any(), any())
        }

        viewModel.uiState.test {
            val state = awaitItem()
            assertTrue(state is LoginUiState.Error)
        }
    }

    // ==================== PASSWORD VISIBILITY TESTS ====================

    @Test
    fun `togglePasswordVisibility debe cambiar isPasswordVisible`() = runTest {
        // Given
        viewModel = LoginViewModel(mockApplication)

        // When - Then
        viewModel.isPasswordVisible.test {
            assertFalse(awaitItem()) // Inicialmente false

            viewModel.togglePasswordVisibility()
            assertTrue(awaitItem()) // Ahora true

            viewModel.togglePasswordVisibility()
            assertFalse(awaitItem()) // Vuelve a false
        }
    }
}
```

---

## 5. Buenas Prácticas

### 5.1. Principios FIRST

✅ **F**ast - Los tests deben ejecutarse rápido
✅ **I**ndependent - Cada test debe ser independiente
✅ **R**epeatable - Deben dar el mismo resultado siempre
✅ **S**elf-validating - Deben pasar o fallar claramente
✅ **T**imely - Escribe tests junto con el código

### 5.2. Patrón AAA (Arrange-Act-Assert)

```kotlin
@Test
fun `ejemplo de patrón AAA`() = runTest {
    // ARRANGE (Given) - Preparar
    val email = "test@example.com"
    val password = "password123"

    // ACT (When) - Ejecutar
    val result = repository.login(email, password)

    // ASSERT (Then) - Verificar
    assertTrue(result.isSuccess)
}
```

### 5.3. Nombres Descriptivos

✅ **Bueno:**
```kotlin
@Test
fun `login con credenciales inválidas debe retornar Failure`()
```

❌ **Malo:**
```kotlin
@Test
fun testLogin()
```

### 5.4. Test de un Solo Concepto

Cada test debe verificar UNA SOLA cosa.

✅ **Bueno:**
```kotlin
@Test
fun `email vacío debe mostrar error`()

@Test
fun `email inválido debe mostrar error`()
```

❌ **Malo:**
```kotlin
@Test
fun `test validaciones de email`() {
    // Verifica múltiples casos
}
```

---

## 6. Ejecución de Tests

### 6.1. Desde Android Studio

1. **Ejecutar todos los tests:**
   - Click derecho en carpeta `test/java`
   - "Run 'Tests in 'com.example...'"

2. **Ejecutar tests de una clase:**
   - Click derecho en `AuthSaborLocalRepositoryTest.kt`
   - 
"Run 'AuthSaborLocalRepositoryTest'"

3. **Ejecutar un solo test:**
   - Click en el ícono verde al lado del `@Test`
   - "Run 'login exitoso debe...'"

### 6.2. Desde Terminal

```bash
# Ejecutar todos los tests unitarios
./gradlew test

# Ejecutar tests con reporte detallado
./gradlew test --info

# Ejecutar tests de un módulo específico
./gradlew :app:test

# Ver reporte HTML
# Se genera en: app/build/reports/tests/testDebugUnitTest/index.html
```

### 6.3. Ver Cobertura de Tests

```bash
# Generar reporte de cobertura
./gradlew testDebugUnitTestCoverage

# Ver reporte en:
# app/build/reports/coverage/test/debug/index.html
```

---

## 7. Checklist de Testing

Antes de considerar que tu código está completo, verifica:

- [ ] Cada función pública tiene al menos un test
- [ ] Casos felices están cubiertos (success paths)
- [ ] Casos de error están cubiertos (error paths)
- [ ] Casos edge (valores null, listas vacías, etc.)
- [ ] Tests pasan en tu máquina
- [ ] Tests son independientes (no dependen del orden)
- [ ] Nombres de tests son descriptivos
- [ ] No hay código de producción en los tests
- [ ] Mocks están configurados correctamente

---

## 8. Recursos Adicionales

- [Testing en Android - Documentación Oficial](https://developer.android.com/training/testing)
- [MockK Documentation](https://mockk.io/)
- [Turbine - Testing Flow](https://github.com/cashapp/turbine)
- [Coroutines Testing](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-test/)

---

## 9. Próximos Pasos

1. ✅ Agregar dependencias al `build.gradle.kts`
2. ✅ Crear estructura de carpetas en `test/`
3. ✅ Escribir tests para `AuthSaborLocalRepository`
4. ✅ Escribir tests para `LoginViewModel`
5. ⬜ Escribir tests para `RegisterViewModel` (similar a LoginViewModel)
6. ⬜ Ejecutar tests y verificar cobertura
7. ⬜ Configurar CI/CD para ejecutar tests automáticamente

---

**¡Happy Testing! 🧪🚀**
