# CLAUDE.md

Este archivo proporciona una guía completa a Claude Code cuando se trabaja con código Java en este repositorio.

## Filosofía de Desarrollo Principal

### KISS (Keep It Simple, Stupid - Mantenlo Simple, Estúpido)

La simplicidad debe ser un objetivo clave en el diseño. Elige soluciones sencillas sobre las complejas siempre que sea posible. Las soluciones simples son más fáciles de entender, mantener y depurar.

### YAGNI (You Aren't Gonna Need It - No lo vas a necesitar)

Evita construir funcionalidades por especulación. Implementa características solo cuando sean necesarias, no cuando anticipes que podrían ser útiles en el futuro.

### Principios de Diseño

- **Inversión de Dependencias**: Los módulos de alto nivel no deben depender de los de bajo nivel. Ambos deben depender de abstracciones.
- **Principio Abierto/Cerrado**: Las entidades de software deben estar abiertas a la extensión pero cerradas a la modificación.
- **Responsabilidad Única**: Cada clase, método y módulo debe tener un propósito claro.
- **Fallar Rápido**: Valida las entradas temprano y lanza excepciones inmediatamente cuando ocurran problemas.

## 🤖 Directrices para el Asistente de IA

### Conciencia del Contexto

- Al implementar características, siempre revisa primero los patrones existentes.
- Prefiere la composición sobre la herencia en todos los diseños.
- Usa utilidades existentes antes de crear nuevas.
- Revisa si hay funcionalidades similares en otros dominios/características.

### Errores Comunes a Evitar

- Crear funcionalidades duplicadas.
- Sobrescribir pruebas existentes.
- Modificar frameworks principales sin instrucción explícita.
- Añadir dependencias sin revisar alternativas existentes.

### Patrones de Flujo de Trabajo

- Preferiblemente, crea pruebas ANTES de la implementación (TDD).
- Usa "pensar detenidamente" para las decisiones de arquitectura.
- Descompón tareas complejas en unidades más pequeñas y comprobables.
- Valida la comprensión antes de la implementación.

### Requisitos del Comando de Búsqueda

**CRÍTICO**: Siempre usa `rg` (ripgrep) en lugar de los comandos tradicionales `grep` y `find`:

```bash
# ❌ No uses grep
grep -r "patron" .

# ✅ Usa rg en su lugar
rg "patron"

# ❌ No uses find con name
find . -name "*.java"

# ✅ Usa rg con filtrado de archivos
rg --files | rg "\.java$"
# o
rg --files -g "*.java"
```

**Reglas de Aplicación:**

```
(
    r"^grep\b(?!.*\|)",
    "Usa 'rg' (ripgrep) en lugar de 'grep' para un mejor rendimiento y características",
),
(
    r"^find\s+\S+\s+-name\b",
    "Usa 'rg --files | rg patron' o 'rg --files -g patron' en lugar de 'find -name' para un mejor rendimiento",
),
```

## 🧱 Estructura del Código y Modularidad

### Límites de Archivos y Métodos

- **Nunca crees un archivo de clase de más de 500 líneas**. Si te acercas a este límite, refactoriza extrayendo clases.
- **Los métodos deben tener menos de 50 líneas** para una mejor comprensión por parte de la IA y mantenibilidad.
- **Las clases deben centrarse en un concepto** - sigue el Principio de Responsabilidad Única.
- **La complejidad ciclomática no debe exceder de 10** por método (regla de SonarQube).

### Estructura del Proyecto (Diseño Estándar de Gradle)

```
raiz-del-proyecto/
├── build.gradle.kts (o build.gradle)
├── settings.gradle.kts (o settings.gradle)
├── gradle.properties
├── CLAUDE.md
├── .claude/
│   └── commands/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/empresa/proyecto/
│   │   │       ├── controller/
│   │   │       ├── service/
│   │   │       ├── repository/
│   │   │       ├── entity/
│   │   │       ├── dto/
│   │   │       ├── exception/
│   │   │       ├── config/
│   │   │       └── util/
│   │   └── resources/
│   │       ├── application.yml
│   │       ├── application-prod.yml
│   │       └── logback-spring.xml
│   └── test/
│       ├── java/
│       │   └── com/empresa/proyecto/
│       └── resources/
├── build/
└── gradle/
    └── wrapper/
        ├── gradle-wrapper.jar
        └── gradle-wrapper.properties
```

## 🛠️ Configuración de Gradle

### Configuración Esencial de build.gradle.kts

```kotlin
plugins {
    java
    id("org.springframework.boot") version "3.5.0"
    id("io.spring.dependency-management") version "1.1.5"
    id("com.diffplug.spotless") version "6.25.0"
    id("com.github.spotbugs") version "6.0.18"
    id("jacoco")
    id("org.sonarqube") version "5.0.0.4638"
}

group = "com.empresa"
version = "0.0.1-SNAPSHOT"

java {
    toolchain {
        languageVersion = JavaLanguageVersion.of(21)
    }
}

repositories {
    mavenCentral()
}

dependencies {
    // Starters de Spring Boot
    implementation("org.springframework.boot:spring-boot-starter-web")
    implementation("org.springframework.boot:spring-boot-starter-validation")
    implementation("org.springframework.boot:spring-boot-starter-actuator")

    // Documentación OpenAPI
    implementation("org.springdoc:springdoc-openapi-starter-webmvc-ui:2.5.0")

    // Lombok
    compileOnly("org.projectlombok:lombok")
    annotationProcessor("org.projectlombok:lombok")

    // Pruebas
    testImplementation("org.springframework.boot:spring-boot-starter-test")
    testImplementation("org.assertj:assertj-core")

    // Herramientas de desarrollo
    developmentOnly("org.springframework.boot:spring-boot-devtools")
}

tasks.withType<Test> {
    useJUnitPlatform()
}

// Configuración de JaCoCo
jacoco {
    toolVersion = "0.8.12"
}

tasks.jacocoTestReport {
    reports {
        xml.required = true
        html.required = true
    }
}

tasks.jacocoTestCoverageVerification {
    violationRules {
        rule {
            limit {
                minimum = "0.80".toBigDecimal()
            }
        }
    }
}

// Configuración de SpotBugs
spotbugs {
    ignoreFailures = false
    showStackTraces = true
    showProgress = true
}

// Configuración de Spotless
spotless {
    java {
        googleJavaFormat()
        removeUnusedImports()
    }
}
```

### Comandos de Gradle

```bash
# Limpiar y compilar
./gradlew clean compileJava

# Ejecutar pruebas
./gradlew test

# Ejecutar pruebas con cobertura
./gradlew test jacocoTestReport

# Empaquetar la aplicación
./gradlew bootJar

# Ejecutar la aplicación
./gradlew bootRun

# Ejecutar análisis de SonarQube
./gradlew sonarqube

# Comprobar actualizaciones de dependencias
./gradlew dependencyUpdates

# Formatear código
./gradlew spotlessApply

# Comprobar el formato del código
./gradlew spotlessCheck

# Ejecutar análisis de SpotBugs
./gradlew spotbugsMain

# Ejecutar todas las comprobaciones
./gradlew check

# Compilar sin pruebas
./gradlew build -x test
```

## 📋 Estilo de Código y Convenciones

### Guía de Estilo de Java

- **Seguir la Guía de Estilo de Java de Google** con estas especificaciones:
  - Longitud de línea: 100 caracteres
  - Indentación: 4 espacios (sin tabuladores)
  - Llaves: Estilo egipcio (en la misma línea)
- **Usar la palabra clave `final` con criterio** - para variables y parámetros, pero evitarla en clases que usan características de AOP de Spring.
- **Preferir objetos inmutables** - seguros para hilos por diseño.
- **Sin importaciones con comodín** - solo importaciones explícitas.
- **Una clase por archivo** - excepto para clases internas.

### Convenciones de Nomenclatura

- **Clases**: `PascalCase` (p. ej., `UserService`)
- **Interfaces**: `PascalCase` sin prefijo "I"
- **Métodos**: `camelCase` (p. ej., `getUserById`)
- **Constantes**: `UPPER_SNAKE_CASE`
- **Paquetes**: `lowercase` (p. ej., `com.empresa.proyecto`)
- **Parámetros de Tipo**: Letras mayúsculas únicas (p. ej., `T`, `E`, `K`, `V`)

## 🎯 Seguridad de Tipos y Anotaciones

### Validación de Beans

- **Usar Validación de Beans** (JSR-380) para la validación.
- **Usar `@Valid`** para parámetros de método.
- **Usar `@Validated`** para valores de retorno de método.

### Requisitos de Tipado Estricto

- **Sin tipos crudos (raw types)** - Siempre usar genéricos.
- **Sin tipo `Object`** a menos que sea absolutamente necesario.
- **Usar `Optional<T>`** en lugar de devolver nulo.
- **Anotar todo** - `@NonNull`, `@Nullable`.
- **Sin advertencias suprimidas** sin justificación.

### Anotaciones Esenciales

```java
// Anotaciones de nulidad (JSR-305)
import javax.annotation.Nonnull;
import javax.annotation.Nullable;
import javax.annotation.ParametersAreNonnullByDefault;

// Por defecto a nivel de paquete
@ParametersAreNonnullByDefault
package com.empresa.proyecto;

// Lombok para reducir el código repetitivo
import lombok.Data;
import lombok.Builder;
import lombok.NonNull;
import lombok.extern.slf4j.Slf4j;

// Anotaciones de validación
import jakarta.validation.constraints.*;
```
### Mejores Prácticas de Tipos Genéricos
```java
// ❌ Mal: Tipos crudos
List list = new ArrayList();
Map map = new HashMap();

// ✅ Bien: Tipos parametrizados
List<String> list = new ArrayList<>();
Map<String, User> map = new HashMap<>();

// ✅ Bien: Parámetros de tipo acotados
public interface Repository<T extends Entity> {
    Optional<T> findById(Long id);
    List<T> findAll();
}

// ✅ Bien: Múltiples cotas
public <T extends Comparable<T> & Serializable> void process(T item) {
    // Implementación
}
```

## 🤖 Claude Code

### Mejores Prácticas

- Documentar las convenciones específicas del repositorio en CLAUDE.md.
- Especificar las preferencias de estilo de codificación en CLAUDE.md.
- Listar comportamientos inesperados o advertencias en CLAUDE.md.
- Incluir instrucciones de configuración del entorno en CLAUDE.md.

### Directrices para el Desarrollo Asistido por IA

- Proporcionar un contexto claro en los nombres de los métodos y en Javadoc.
- Incluir ejemplos de entradas/salidas en la documentación.
- Usar nombres de variables descriptivos que transmitan la intención.
- Estructurar el código para que sea fácilmente comprensible por los asistentes de IA.
- Mantener los métodos enfocados y de menos de 50 líneas para una mejor comprensión por parte de la IA.
- Usar patrones de nomenclatura consistentes en toda la base de código.
- Documentar claramente los casos borde y la lógica de negocio.
- Incluir pruebas unitarias que demuestren los patrones de uso.

## 📖 Estándares de Documentación

### Requisitos de Documentación OpenAPI/Swagger (OBLIGATORIO)

**CRÍTICO**: Cada controlador REST y DTO DEBE incluir anotaciones OpenAPI completas para los desarrolladores de frontend.

#### Anotaciones de Controlador Requeridas

Cada clase `@RestController` DEBE incluir:

```java
@RestController
@RequestMapping("/api/recurso")
@Tag(name = "Gestión de Recursos", description = "Operaciones para gestionar recursos")
@Validated
public class ResourceController {

    @Operation(
        summary = "Breve descripción de la acción",
        description = "Explicación detallada de lo que hace este endpoint, incluyendo la lógica de negocio"
    )
    @ApiResponses(value = {
        @ApiResponse(responseCode = "200", description = "Descripción del caso de éxito"),
        @ApiResponse(responseCode = "400", description = "Solicitud incorrecta - validación fallida"),
        @ApiResponse(responseCode = "404", description = "Recurso no encontrado"),
        @ApiResponse(responseCode = "500", description = "Error interno del servidor")
    })
    @GetMapping("/{id}")
    public ResponseEntity<ResourceResponse> getById(
        @Parameter(description = "Identificador único del recurso", example = "123", required = true)
        @PathVariable Long id,

        @Parameter(description = "Incluir datos relacionados", example = "true")
        @RequestParam(defaultValue = "false") Boolean includeDetails
    ) {
        // Implementación
    }
}
```

#### Anotaciones de DTO Requeridas

Cada clase DTO DEBE incluir:

```java
@Schema(description = "Respuesta del recurso que contiene toda la información del recurso")
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class ResourceResponse {

    @Schema(description = "Identificador único", example = "123", accessMode = Schema.AccessMode.READ_ONLY)
    private Long id;

    @Schema(description = "Nombre del recurso", example = "Recurso de Muestra", requiredMode = Schema.RequiredMode.REQUIRED)
    @NotBlank(message = "El nombre no puede estar en blanco")
    @Size(min = 2, max = 100, message = "El nombre debe tener entre 2 y 100 caracteres")
    private String name;

    @Schema(description = "Marca de tiempo de creación del recurso", example = "2024-01-15T10:30:00Z", accessMode = Schema.AccessMode.READ_ONLY)
    private LocalDateTime createdAt;

    @Schema(description = "Lista de elementos relacionados", implementation = RelatedItemResponse.class)
    private List<RelatedItemResponse> relatedItems;
}
```

#### Lista de Verificación de Documentación OpenAPI

Cada endpoint DEBE documentar:

- ✅ **Métodos HTTP** - Anotación explícita @Operation
- ✅ **Parámetros de ruta** - @Parameter con descripción y ejemplo
- ✅ **Parámetros de consulta** - @Parameter con descripción, ejemplo y valores por defecto
- ✅ **Esquemas de Solicitud/Respuesta** - @Schema en todos los campos de DTO
- ✅ **Códigos de estado HTTP** - @ApiResponses completas con todos los resultados posibles
- ✅ **Reglas de validación** - @Schema.requiredMode, descripciones de restricciones
- ✅ **Ejemplos** - Valores de ejemplo del mundo real para todos los campos
- ✅ **Contexto de negocio** - Qué hace el endpoint y por qué

#### Integración de la Validación

Combina OpenAPI con la Validación de Beans para la generación automática de esquemas:

```java
public class CreateResourceRequest {

    @Schema(description = "Nombre del recurso", example = "Nuevo Recurso")
    @NotBlank(message = "El nombre es obligatorio")
    @Size(min = 2, max = 100, message = "El nombre debe tener entre 2 y 100 caracteres")
    private String name;

    @Schema(description = "Categoría del recurso", example = "CATEGORIA_A", allowableValues = {"CATEGORIA_A", "CATEGORIA_B", "CATEGORIA_C"})
    @NotNull(message = "La categoría es obligatoria")
    private ResourceCategory category;

    @Schema(description = "Prioridad del recurso", example = "5", minimum = "1", maximum = "10")
    @Min(value = 1, message = "La prioridad debe ser al menos 1")
    @Max(value = 10, message = "La prioridad debe ser como máximo 10")
    private Integer priority;
}
```

#### Documentación de Respuesta de Error

DEBE documentar las respuestas de error de manera consistente:

```java
@Schema(description = "Respuesta de error para fallos de validación")
public class ErrorResponse {

    @Schema(description = "Mensaje de error", example = "Validación fallida")
    private String message;

    @Schema(description = "Código de estado HTTP", example = "400")
    private Integer status;

    @Schema(description = "Marca de tiempo de la solicitud", example = "2024-01-15T10:30:00Z")
    private LocalDateTime timestamp;

    @Schema(description = "Errores de validación de campos", implementation = FieldError.class)
    private List<FieldError> fieldErrors;
}
```

#### Dependencias Requeridas (Gradle)

```kotlin
dependencies {
    implementation("org.springdoc:springdoc-openapi-starter-webmvc-ui:2.5.0")
}
```

**URLs de Acceso:**

- Swagger UI: `http://localhost:8080/swagger-ui.html`
- OpenAPI JSON: `http://localhost:8080/v3/api-docs`

### Requisitos de Javadoc

Cada clase, método y campo público DEBE tener Javadoc. Usa el estilo de Javadoc de Google:

```java
/**
 * Calcula el precio con descuento de un producto.
 *
 * <p>Este método aplica un descuento porcentual al precio original,
 * asegurando que el precio final no sea inferior al umbral mínimo.
 *
 * @param originalPrice el precio original del producto, debe ser positivo
 * @param discountPercent el porcentaje de descuento (0-100)
 * @param minPrice el precio mínimo permitido después del descuento
 * @return el precio con descuento calculado
 * @throws IllegalArgumentException si algún parámetro es inválido
 * @since 1.2.0
 */
@Nonnull
public BigDecimal calculateDiscount(
        @Nonnull BigDecimal originalPrice,
        double discountPercent,
        @Nonnull BigDecimal minPrice) {
    // Implementación
}
```

### Reglas de Documentación

- La primera oración es un resumen (termina con un punto).
- Usa `<p>` para saltos de párrafo.
- Documenta todos los parámetros con `@param`.
- Documenta el valor de retorno con `@return`.
- Documenta las excepciones con `@throws`.
- Usa `@since` para el versionado de la API.
- Enlaza elementos relacionados con `{@link}`.

## 🧪 Estrategia de Pruebas

### Organización de las Pruebas

- Pruebas unitarias: Misma estructura de paquetes que el código principal.
- Pruebas de integración: Carpeta separada `src/test/integration`.
- Nomenclatura de pruebas: `NombreClaseTest` para pruebas unitarias.
- Nomenclatura de métodos de prueba: `deberia_ComportamientoEsperado_Cuando_EstadoBajoPrueba`

### Mejores Prácticas de Pruebas

```java
// JUnit 5 + AssertJ + Mockito
import org.junit.jupiter.api.*;
import static org.assertj.core.api.Assertions.*;
import static org.mockito.Mockito.*;

@DisplayName("UserService")
class UserServiceTest {

    @Mock
    private UserRepository userRepository;

    @InjectMocks
    private UserService userService;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
    }

    @Test
    @DisplayName("debería devolver un usuario cuando se proporciona un ID válido")
    void deberia_DevolverUsuario_Cuando_IdValidoProporcionado() {
        // Dado
        Long userId = 1L;
        User expectedUser = User.builder()
            .id(userId)
            .name("John Doe")
            .build();
        when(userRepository.findById(userId))
            .thenReturn(Optional.of(expectedUser));

        // Cuando
        Optional<User> result = userService.findById(userId);

        // Entonces
        assertThat(result)
            .isPresent()
            .hasValue(expectedUser);
        verify(userRepository).findById(userId);
    }

    @Test
    @DisplayName("debería lanzar una excepción cuando el ID es nulo")
    void deberia_LanzarExcepcion_Cuando_IdEsNulo() {
        // Cuando/Entonces
        assertThatThrownBy(() -> userService.findById(null))
            .isInstanceOf(IllegalArgumentException.class)
            .hasMessage("El ID de usuario no puede ser nulo");
    }
}
```

### Requisitos de Cobertura de Pruebas

- Cobertura de línea mínima del 80%
- Cobertura de rama mínima del 80%
- Lógica de negocio crítica: cobertura del 90%+
- Todos los métodos públicos deben tener pruebas

## 🚀 Spring Boot

### Mejores Prácticas de Spring Boot

- Usar starters de Spring Boot para la gestión de dependencias.
- Habilitar DevTools para la productividad en el desarrollo.
- Configurar hilos virtuales para un mejor rendimiento (Java 21).
- Usar `@ConfigurationProperties` para una configuración segura en tipos.
- Implementar endpoints de actuador adecuados para el monitoreo.
- Habilitar el apagado gradual por defecto.

## 🔐 Validación de Entradas

### Validación de Beans (Jakarta Validation)

## 📊 ¡¡¡IMPORTANTE!!! Seguir la Configuración de SonarQube

### Reglas de Calidad del Código (reglas estándar de sonarqube)

- **Complejidad Cognitiva**: Máximo 15 por método.
- **Complejidad Ciclomática**: Máximo 10 por método.
- **Líneas Duplicadas**: Máximo 3%.
- **Cobertura de Código**: Mínimo 80%.
- **No introducir nuevos problemas (puerta de calidad por defecto de Sonar way)**.
- **Ratio de Deuda Técnica**: Máximo 5%.
- **Puntos Calientes de Seguridad**: Deben ser revisados.

## 🌱 Mejores Prácticas de Spring Boot

### Clases Finales y Limitaciones de AOP

- **CRÍTICO**: Evita el modificador `final` en las clases de servicio de Spring (`@Service`, `@Component`, `@Repository`).
- **Razón**: Spring AOP (incluyendo `@Transactional`, `@Cacheable`, `@Async`) usa proxies CGLIB.
- **Problema**: Las clases finales no pueden ser subclasificadas, lo que impide la creación de proxies.
- **Solución**: Usa clases no finales con inyección de constructor.

#### Cuándo Usar Final

✅ **SÍ usa final para:**

- Variables locales y parámetros de método.
- Campos que nunca deben cambiar.
- Clases de utilidad con solo métodos estáticos.
- DTOs y objetos de valor sin anotaciones AOP.

❌ **EVITA final para:**

- Clases `@Service`, `@Component`, `@Repository`.
- Clases que usan `@Transactional`, `@Cacheable`, `@Async`.
- Cualquier clase que requiera características de Spring AOP.

### Diseño Basado en Interfaces vs Clases

- **Spring Moderno**: Las interfaces son **opcionales** para la mayoría de los servicios.
- **Usa interfaces cuando**:
  - Existen múltiples implementaciones.
  - Quieres ocultar las anotaciones AOP de la API pública.
  - Sigues patrones de diseño dirigido por el dominio.
- **Omite las interfaces cuando**:
  - Hay una única implementación con inyección de constructor.
  - Son servicios internos sin API externa.
  - Son operaciones CRUD simples.

### Estrategia de Proxy

```java
// ✅ Bien: Clase de servicio no final
@Service
@Transactional
public class UserService {
    // Implementación
}

// ❌ Mal: La clase final impide AOP
@Service
@Transactional
public final class UserService { // ¡CGLIB no puede crear un proxy para esto!
    // Implementación
}
```

## 🚀 Directrices de Rendimiento

### Características de Rendimiento de Spring Boot 3.5

- **Hilos Virtuales (Java 21)**: Habilitar con `spring.threads.virtual.enabled=true`.
- **HTTP/2**: Habilitado por defecto para una mejor multiplexación.
- **Compresión**: Habilitar la compresión de respuestas.
- **Agrupación de Conexiones**: Configurar para el SDK de AWS (si es necesario).
- **Caché**: Usar la abstracción de Caché de Spring.

## 🔥 Atajos de Rendimiento en Java

### Victorias Rápidas

- Usa `@RestControllerAdvice` para el manejo global de excepciones.
- Aprovecha `@ConfigurationProperties` para una configuración segura en tipos.
- Usa `@Async` con `CompletableFuture` para el procesamiento en paralelo.
- Habilita los hilos virtuales: `spring.threads.virtual.enabled=true`.

### Impulsores de Productividad Instantáneos

```java
// Plantilla de manejador de excepciones global
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(ValidationException.class)
    public ResponseEntity<ErrorResponse> handleValidation(ValidationException e) {
        return ResponseEntity.badRequest().body(ErrorResponse.of(e));
    }
}
```

## 🛡️ Mejores Prácticas de Seguridad

### Directrices de Seguridad
- Nunca registrar información sensible.
- Usar consultas parametrizadas (sin concatenación de cadenas).
- Validar todas las entradas.
- Usar algoritmos de cifrado fuertes.
- Mantener las dependencias actualizadas.
- Seguir las directrices de OWASP.
- Usar anotaciones de seguridad.

## 🔧 Problemas Comunes y Soluciones

### Fijación de Hilos Virtuales (Virtual Thread Pinning)
- Evita los bloques `synchronized` en rutas críticas.
- Usa `ReentrantLock` en lugar de `synchronized`.
- Monitorea con `-Djdk.tracePinnedThreads=short`.
- Perfila con JDK Flight Recorder.

### Limitación de Tasa de Bedrock
- Implementa un retroceso exponencial con fluctuación (jitter).
- Usa interruptores de circuito (Resilience4j).
- Monitorea el uso de tokens con CloudWatch.
- Configura la cola de solicitudes.

### Gestión de Memoria de Spring AI
- Configura ventanas de contexto apropiadas.
- Implementa la poda de conversaciones.
- Usa streaming para respuestas grandes.
- Monitorea el uso del heap.

## 🔄 Flujo de Trabajo de Git

### Formato de Mensaje de Commit

 - NUNCA incluyas "claude code", "written by claude code" o similares en el mensaje de commit.

<tipo>(<ámbito>): <asunto>

<cuerpo>

<pie>
Tipos: feat, fix, docs, style, refactor, test, chore
Ejemplo:
feat(user): añadir verificación de correo electrónico con el modelo Nova

- Implementar servicio de verificación de correo electrónico.
- Añadir generación de token de verificación.
- Integrar con Amazon Nova para la generación de contenido.
- Actualizar el modelo de usuario con campos de verificación.

Cierra #234
```

## ⚠️ Directrices Críticas

1. **Sin tipos crudos** - Siempre usar genéricos.
2. **Sin retornos nulos** - Usar `Optional<T>`.
3. **Validar todas las entradas** - Usar Jakarta Validation.
4. **Documentar todas las APIs públicas** - Javadoc COMPLETO Y anotaciones OpenAPI.
5. **OpenAPI OBLIGATORIO** - Cada endpoint REST DEBE tener anotaciones @Operation, @ApiResponses, @Parameter y @Schema completas.
6. **Probar todo** - Cobertura mínima del 80%.
7. **Manejar todas las excepciones** - Sin bloques `catch` vacíos.
8. **Usar `final` con criterio** - Para variables/parámetros, evitar en clases de servicio de Spring.
9. **Sin números mágicos** - Extraer a constantes.
10. **Una clase por archivo** - Excepto clases internas.
11. **Seguir las reglas de SonarQube** - Cero bloqueadores/críticos.
12. **Diseño de API "Frontend-first"** - Todos los endpoints deben ser amigables para los desarrolladores de React con ejemplos y esquemas completos.

## 📋 Lista de Verificación Pre-commit

- [ ] Todas las advertencias del compilador resueltas.
- [ ] Javadoc para todos los métodos/clases públicos.
- [ ] **Anotaciones OpenAPI en TODOS los endpoints REST** (@Operation, @ApiResponses, @Parameter, @Schema).
- [ ] **Esquemas de DTO con ejemplos** (@Schema con descripción y ejemplo en todos los campos).
- [ ] **Documentación de la API accesible** en `/swagger-ui.html`.
- [ ] Pruebas unitarias escritas (cobertura del 80%+).
- [ ] Sin problemas críticos/bloqueadores de SonarQube.
- [ ] Sin advertencias de alta prioridad de SpotBugs.
- [ ] Código formateado (./gradlew spotlessApply).
- [ ] Todas las entradas validadas.
- [ ] Registro en los niveles apropiados (si el registro está configurado).
- [ ] **El desarrollador de frontend puede usar la API** sin hacer preguntas.

---

_Mantén esta guía actualizada a medida que los patrones evolucionen. Calidad sobre velocidad, siempre._
_Última actualización: Junio de 2025_