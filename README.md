# Guía Paso a Paso: CRUD de Mascotas con Spring Boot

## 1. Creación del proyecto con Spring Initializr

**¿Por qué?**
Spring Initializr es una web que permite generar proyectos Spring Boot fácilmente y con las dependencias necesarias.

**Pasos:**

1. Ingresa a [https://start.spring.io/](https://start.spring.io/)
2. Completa los campos:

   * **Project:** Maven
   * **Language:** Java
   * **Group:** com.pruebas
   * **Artifact:** unitarias
   * **Java:** 17 (o superior)

![Captura de pantalla 2025-06-05 204631](https://github.com/user-attachments/assets/615d084e-8a82-43e4-8ef4-e7f2ea562147)

3. Haz clic en **Add dependencies** y agrega:

   * Spring Web
   * Spring Data JPA
   * MySQL Driver (o H2 para pruebas en memoria)
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
