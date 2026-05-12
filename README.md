# Productos Service — Análisis SonarQube

**Autor:** Calderon  
**Materia:** Patrones de Diseño de Software  
**Unidad:** 10 — Métricas de Calidad y SonarQube  
**Año:** 2026

---

## Descripción

Proyecto Spring Boot con código **intencionalmente imperfecto** para análisis estático
con SonarQube y reporte de cobertura con JaCoCo (Post-Contenido 1, Unidad 10).

---

## Pasos de ejecución

### 1. Levantar SonarQube con Docker

```bash
docker run -d \
  --name sonarqube \
  -p 9000:9000 \
  -e SONAR_ES_BOOTSTRAP_CHECKS_DISABLE=true \
  sonarqube:community
```

Esperar: `SonarQube is operational` con `docker logs -f sonarqube`  
Abrir http://localhost:9000 — credenciales: `admin / admin`

### 2. Crear proyecto en SonarQube

- Projects → Create Project → Manually  
- Project key: `com.universidad:productos-service`  
- Generar token y copiarlo

### 3. Compilar + generar cobertura JaCoCo

```bash
mvn clean verify
```

### 4. Ejecutar análisis SonarQube

```bash
mvn sonar:sonar -Dsonar.token=TU_TOKEN_AQUI
```

### 5. Ver resultados

http://localhost:9000/dashboard?id=com.universidad%3Aproductos-service

---

## Estado inicial del análisis

| Categoría        | Cantidad | Rating |
|------------------|----------|--------|
| Bugs             | 2        | C      |
| Vulnerabilidades | 0        | A      |
| Code Smells      | 6        | C      |
| Cobertura        | 5.0%     | —      |

---

## Hallazgos principales identificados

### Bug 1: Método `buscar()` retorna null
- **Archivo:** `ProductoService.java`, línea 37  
- **Descripción:** `repo.findById(id).orElse(null)` puede causar NullPointerException en el controlador.  
- **Severidad:** Major

### Bug 2: Campo `nombre` sin restricción NOT NULL
- **Archivo:** `Producto.java`, línea 11  
- **Descripción:** Falta `@Column(nullable=false)`, permite persistir nombre nulo.  
- **Severidad:** Minor

### Code Smell 1: Inyección por campo con @Autowired
- **Archivo:** `ProductoService.java`, línea 12  
- **Descripción:** Usar inyección por constructor es mejor práctica.

### Code Smell 2: Método con alta complejidad ciclomática
- **Archivo:** `ProductoService.java`, línea 15  
- **Descripción:** `procesarProducto()` mezcla validación, construcción y persistencia; CC > 10.

### Code Smell 3: Comparación con cadena vacía
- **Archivo:** `ProductoService.java`, línea 17  
- **Descripción:** `n.equals("")` debe reemplazarse por `n.isBlank()`.

### Code Smell 4: Lógica de negocio en entidad JPA
- **Archivo:** `Producto.java`, línea 16  
- **Descripción:** Método `getEstado()` viola el principio de responsabilidad única (SRP).

---

## Capturas del dashboard

![Dashboard SonarQube](docs/sonar-dashboard.png)
![Detalle Bugs](docs/sonar-bugs.png)

---

## Estructura del proyecto

```
calderon-post1-u10/
├── src/
│   ├── main/java/com/universidad/productosservice/
│   │   ├── domain/Producto.java
│   │   ├── repository/ProductoRepository.java
│   │   ├── service/ProductoService.java
│   │   └── ProductosServiceApplication.java
│   ├── main/resources/application.properties
│   └── test/java/.../ProductosServiceApplicationTests.java
├── docs/
│   ├── sonar-dashboard.png
│   └── sonar-bugs.png
├── sonar-project.properties
├── pom.xml
├── .gitignore
└── README.md
```
