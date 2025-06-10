# Guía Paso a Paso: CRUD de Mascotas con Spring Boot

## 1. Creación del proyecto con Spring Initializr


En Spring Initializr configura las dependencias de tu proyecto Spring Boot 

**Pasos:**

1. Ingresa a [https://start.spring.io/](https://start.spring.io/)
2. Completa los campos:

![Captura de pantalla 2025-06-05 204631](https://github.com/user-attachments/assets/615d084e-8a82-43e4-8ef4-e7f2ea562147)

3. Haz clic en **Add dependencies** y agrega:

   * Spring Web
   * Spring Data JPA
   * MySQL Driver
   * Lombok

4. Haz clic en **GENERATE** para descargar el proyecto.
5. Descomprime el ZIP y ábrelo en tu IDE favorito (IntelliJ, Eclipse, VSCode, etc).

---

## 2. Configuración de la Base de Datos

**¿Por qué?**
Tu aplicación necesita saber a qué base de datos conectarse para guardar los datos de las mascotas.

**Pasos:**

* Crea la base de datos `mascotas_db` en tu gestor de MySQL.
* Abre el archivo `src/main/resources/application.properties` y agrega:

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/mascotas_db
spring.datasource.username=root
spring.datasource.password=
spring.jpa.hibernate.ddl-auto=update
spring.jpa.show-sql=true
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL8Dialect
```

*(Ajusta el usuario y contraseña si es necesario)*

---

## 3. Creación del Modelo (Entidad Mascota)

**¿Por qué?**
El modelo representa los datos de tu sistema; cada objeto Mascota será una fila en la tabla `mascotas`.

**Pasos:**

* Crea el paquete: `com.pruebas.unitarias.model`
* Agrega el archivo `Mascota.java`:

```java
package com.pruebas.unitarias.model;

import jakarta.persistence.*;
import lombok.*;

@Data
@NoArgsConstructor
@AllArgsConstructor
@Entity
@Table(name = "mascotas")
public class Mascota {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Long id;

    @Column(nullable = false)
    private String nombre;

    @Column(nullable = false)
    private String tipo;

    private int edad;
}
```

---

## 4. Creación del Repositorio

**¿Por qué?**
El repositorio gestiona la interacción con la base de datos, usando métodos listos para guardar, buscar, eliminar, etc.

**Pasos:**

* Crea el paquete: `com.pruebas.unitarias.repository`
* Agrega el archivo `MascotaRepository.java`:

```java
package com.pruebas.unitarias.repository;

import com.pruebas.unitarias.model.Mascota;
import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.stereotype.Repository;

@Repository
public interface MascotaRepository extends JpaRepository<Mascota, Long> {}
```

---

## 5. Creación del Servicio

**¿Por qué?**
El servicio contiene la lógica de negocio y controla qué puede hacer tu aplicación.

**Pasos:**

* Crea el paquete: `com.pruebas.unitarias.service`
* Agrega el archivo `MascotaService.java`:

```java
package com.pruebas.unitarias.service;

import com.pruebas.unitarias.model.Mascota;
import com.pruebas.unitarias.repository.MascotaRepository;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import java.util.List;
import java.util.Optional;

@Service
public class MascotaService {
    @Autowired
    private MascotaRepository mascotaRepository;

    public Mascota guardarMascota(Mascota mascota) {
        return mascotaRepository.save(mascota);
    }

    public List<Mascota> listarMascotas() {
        return mascotaRepository.findAll();
    }

    public Optional<Mascota> obtenerMascotaPorId(Long id) {
        return mascotaRepository.findById(id);
    }

    public Mascota actualizarMascota(Long id, Mascota mascota) {
        Mascota existente = mascotaRepository.findById(id)
                .orElseThrow(() -> new RuntimeException("No existe la mascota"));
        existente.setNombre(mascota.getNombre());
        existente.setTipo(mascota.getTipo());
        existente.setEdad(mascota.getEdad());
        return mascotaRepository.save(existente);
    }

    public void eliminarMascota(Long id) {
        mascotaRepository.deleteById(id);
    }
}
```

---

## 6. Creación del Controlador REST

**¿Por qué?**
El controlador recibe las solicitudes HTTP y responde a clientes como Postman, frontends, etc.

**Pasos:**

* Crea el paquete: `com.pruebas.unitarias.controller`
* Agrega el archivo `MascotaController.java`:

```java
package com.pruebas.unitarias.controller;

import com.pruebas.unitarias.model.Mascota;
import com.pruebas.unitarias.service.MascotaService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import java.util.List;

@RestController
@RequestMapping("/api/mascotas")
public class MascotaController {

    @Autowired
    private MascotaService mascotaService;

    @PostMapping
    public Mascota crearMascota(@RequestBody Mascota mascota) {
        return mascotaService.guardarMascota(mascota);
    }

    @GetMapping
    public List<Mascota> obtenerTodas() {
        return mascotaService.listarMascotas();
    }

    @GetMapping("/{id}")
    public ResponseEntity<Mascota> obtenerPorId(@PathVariable Long id) {
        return mascotaService.obtenerMascotaPorId(id)
                .map(ResponseEntity::ok)
                .orElse(ResponseEntity.notFound().build());
    }

    @PutMapping("/{id}")
    public ResponseEntity<Mascota> actualizar(@PathVariable Long id, @RequestBody Mascota mascota) {
        try {
            Mascota actualizada = mascotaService.actualizarMascota(id, mascota);
            return ResponseEntity.ok(actualizada);
        } catch (RuntimeException e) {
            return ResponseEntity.notFound().build();
        }
    }

    @DeleteMapping("/{id}")
    public ResponseEntity<Void> eliminar(@PathVariable Long id) {
        mascotaService.eliminarMascota(id);
        return ResponseEntity.noContent().build();
    }
}
```

---
# ✅ CREACIÓN DE PRUEBAS UNITARIAS

## 7. Pruebas Unitarias del Servicio

**¿Por qué?**
Permiten asegurar que los métodos del servicio funcionan bien de forma aislada.

**Pasos:**

Ahora es momento de ocupar la carpeta de pruebas del proyecto, para ello revisa la estructura de carpetas
![carpetas01](https://github.com/user-attachments/assets/89f21cb7-373c-4088-973b-abd81c0bfb09)

Para realizar las pruebas unitarias utilizamos la carpeta test, dentro de ella encontraremos una ruta similar a lo que hay dentro de main

Probaremos la capa de servicio con pruebas unitarias


# Estructura típica de una prueba unitaria

La mayoría de las pruebas unitarias, sin importar el framework (JUnit, pytest, unittest, Mocha, etc.), siguen esta estructura general:

## 1. Preparación (Arrange / Setup)

Se preparan los datos y el entorno necesarios para la prueba.

Esto incluye crear objetos, definir variables de entrada y configurar mocks o dobles si es necesario.

## 2. Ejecución (Act / Exercise)

Se llama a la función o método que se desea probar con los datos preparados.

## 3. Verificación (Assert)

Se verifica el resultado obtenido comparándolo con el resultado esperado (con aserciones).

Aquí se comprueba si la unidad de código cumple el comportamiento que debería.

## 4. Limpieza (opcional) (Teardown)

En algunos casos se realiza limpieza del entorno si la prueba ha modificado algo global o externo (base de datos, archivos, etc.).

---

## Ejemplo de estructura genérica

```pseudo
TestNombreDeLaFuncion
    // 1. Preparación (Arrange)
    Preparar datos de entrada
    Configurar mocks o entorno si es necesario

    // 2. Ejecución (Act)
    resultado = FuncionAAProbar(datosDeEntrada)

    // 3. Verificación (Assert)
    Asegurarse de que resultado == resultadoEsperado

    // 4. Limpieza (Teardown, opcional)
    Restaurar entorno si es necesario
```
## Creamos nuestra prueba unitaria

**Pasos:**

* Crea el archivo `MascotaServiceTest.java` en el paquete de servicio. Copia y pega el siguiente código

```java
package com.pruebas.unitarias.service;

import com.pruebas.unitarias.model.Mascota;
import com.pruebas.unitarias.repository.MascotaRepository;
import org.junit.jupiter.api.BeforeEach;
import org.junit.jupiter.api.Test;
import org.mockito.InjectMocks;
import org.mockito.Mock;
import org.mockito.MockitoAnnotations;

import java.util.Optional;
import java.util.Arrays;
import java.util.List;

import static org.mockito.Mockito.*;
import static org.assertj.core.api.Assertions.assertThat;

class MascotaServiceTest {

    @Mock
    private MascotaRepository mascotaRepository;

    @InjectMocks
    private MascotaService mascotaService;

    @BeforeEach
    void setUp() {
        MockitoAnnotations.openMocks(this);
    }


    /* Test para guardar mascota en la capa servicio */
    @Test
    void testGuardarMascota() {
        Mascota mascota = new Mascota(null, "Rex", "Perro", 5);
        Mascota mascotaGuardada = new Mascota(1L, "Rex", "Perro", 5);
        when(mascotaRepository.save(mascota)).thenReturn(mascotaGuardada);

        Mascota resultado = mascotaService.guardarMascota(mascota);
        assertThat(resultado.getId()).isEqualTo(1L);
        verify(mascotaRepository).save(mascota);
    }
}
```
## Revisemos ahora nuestra prueba unitaria `MascotaServiceTest`

---

## 1. Definición de la Clase

```java
class MascotaServiceTest {
```
Define la clase de prueba unitaria para el servicio `MascotaService`. Por convención, las clases de prueba terminan en `Test`.

---

## 2. Declaración de Mocks y el Servicio

```java
@Mock
private MascotaRepository mascotaRepository;

@InjectMocks
private MascotaService mascotaService;
```

* `@Mock`: Crea un mock del repositorio `MascotaRepository` usando Mockito. Un mock es un objeto simulado para pruebas.
* `@InjectMocks`: Crea una instancia real de `MascotaService` e **inyecta** en ella el mock del repositorio. Así, el servicio utilizará el mock en lugar del repositorio real.

---

## 3. Inicialización de los Mocks

```java
@BeforeEach
void setUp() {
    MockitoAnnotations.openMocks(this);
}
```

* `@BeforeEach`: Indica que este método se ejecuta **antes de cada prueba**.
* `MockitoAnnotations.openMocks(this)`: Inicializa los mocks y realiza la inyección de dependencias. Es esencial para que Mockito funcione correctamente con JUnit 5.

---

## 4. Prueba Unitaria: `testGuardarMascota`

```java
@Test
void testGuardarMascota() {
    Mascota mascota = new Mascota(null, "Rex", "Perro", 5);
    Mascota mascotaGuardada = new Mascota(1L, "Rex", "Perro", 5);
    when(mascotaRepository.save(mascota)).thenReturn(mascotaGuardada);

    Mascota resultado = mascotaService.guardarMascota(mascota);
    assertThat(resultado.getId()).isEqualTo(1L);
    verify(mascotaRepository).save(mascota);
}
```

### Explicación paso a paso:

1. **Preparación de Datos**

   ```java
   Mascota mascota = new Mascota(null, "Rex", "Perro", 5); // Se crea una mascota sin ID (simulando que es nueva).
   Mascota mascotaGuardada = new Mascota(1L, "Rex", "Perro", 5);// Se crea una mascota con ID (simulando que ha sido guardada).
   ```

2. **Configuración del Mock**

   ```java
   when(mascotaRepository.save(mascota)).thenReturn(mascotaGuardada);
   ```

   * Se define el comportamiento del mock: **cuando** se llame a `save()` con la mascota, **devolverá** la mascota guardada con ID.

3. **Llamada al Método a Probar**

   ```java
   Mascota resultado = mascotaService.guardarMascota(mascota);
   ```

   * Se invoca el método real del servicio, que debería llamar al repositorio.

4. **Verificación del Resultado**

   ```java
   assertThat(resultado.getId()).isEqualTo(1L);
   ```

   * Se comprueba que el método retorne una mascota con ID igual a 1.

5. **Verificación de la Interacción**

   ```java
   verify(mascotaRepository).save(mascota);
   ```

   * Se verifica que el método `save()` del repositorio fue realmente llamado con la mascota.

---

## 5. Resumen de la Estructura

| Bloque                | Descripción                                                  |
| --------------------- | ------------------------------------------------------------ |
| `@Mock`               | Mock del repositorio para simular la capa de datos           |
| `@InjectMocks`        | Servicio real con el mock inyectado                          |
| `@BeforeEach`         | Inicializa los mocks antes de cada test                      |
| Crear datos de prueba | Instancia de mascota nueva y mascota guardada simulada       |
| `when...thenReturn`   | Define cómo responderá el mock al guardar                    |
| Llamada a método      | Ejecuta el método a probar                                   |
| `assertThat...`       | Valida que el resultado es el esperado                       |
| `verify`              | Confirma que el método del repositorio fue realmente llamado |

---

## 6. ¿Por qué es importante esta prueba?

Esta prueba unitaria permite asegurar que el método `guardarMascota()` de `MascotaService`:

* Utiliza correctamente el repositorio.
* Devuelve la mascota guardada con el ID asignado.
* No requiere una base de datos real, ya que todo se simula con mocks.

---

## ¿Y quién me asegura que los datos guardados son realmente los guardados?

Para ello se pueden agreagar más aserciones en la verificación de resultado 
```java
assertThat(resultado.getNombre()).isEqualTo("Rex");
```
