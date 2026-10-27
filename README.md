# ⚙️ Config Server

Servicio de **configuración centralizada** de la plataforma LogiTrack. Expone por HTTP la configuración de los demás microservicios, leyéndola desde un repositorio Git, de modo que ningún servicio necesita llevar sus propiedades empaquetadas.

---

## 📖 Descripción

El Config Server es un servicio basado en **Spring Cloud Config** que cumple los siguientes aportes:

- Centraliza la configuración de todos los microservicios de LogiTrack.
- Obtiene los archivos de configuración desde un repositorio Git versionado.
- Permite tener configuraciones distintas por **aplicación**, **perfil** (`dev`, `prod`, …) y **rama/etiqueta**.
- Facilita cambiar configuración sin recompilar ni reempaquetar los servicios.

## 🧰 Tecnologías

| Tecnología           | Versión        |
| -------------------- | -------------- |
| Java                 | 25             |
| Spring Boot          | 4.1.2          |
| Spring Cloud         | 2025.1.4       |
| Spring Cloud Config  | Server         |
| Spring Boot Actuator | Incluido       |
| Maven                | Wrapper (`mvnw`) |

## 🔄 Cómo funciona

```
┌──────────────────┐   1. pide config    ┌──────────────────┐   2. clona/lee   ┌─────────────────────┐
│  Microservicio   │ ──────────────────► │  Config Server   │ ───────────────► │  Repositorio Git    │
│  (cliente)       │ ◄────────────────── │  (puerto 8085)   │ ◄─────────────── │  config-server      │
└──────────────────┘   3. devuelve       └──────────────────┘                  └─────────────────────┘
```

1. Al arrancar, cada microservicio solicita su configuración al Config Server.
2. El Config Server la busca en el repositorio Git configurado.
3. Devuelve las propiedades correspondientes a `{aplicación}` + `{perfil}` + `{rama}`.

Con `clone-on-start: true`, el repositorio se clona al iniciar el servicio, por lo que cualquier problema de acceso al repo se detecta de inmediato.

## ✅ Requisitos previos

- **JDK 25**
- **Maven 3.9+** (o usar el wrapper `./mvnw` incluido)
- Acceso a internet para clonar el repositorio de configuración
- Puerto **8085** libre

## 🔧 Configuración

Archivo `src/main/resources/application.yml`:

```yaml
server:
  port: 8085

spring:
  application:
    name: configserver
  profiles:
    active: git
  cloud:
    config:
      server:
        git:
          uri: https://github.com/Logitrack-Penguin/config-server
          clone-on-start: true
          default-label: main
```

| Propiedad                                      | Descripción                                             |
| ---------------------------------------------- | ------------------------------------------------------- |
| `server.port`                                  | Puerto donde escucha el servicio (`8085`)               |
| `spring.application.name`                      | Nombre de la aplicación                                 |
| `spring.profiles.active`                       | Backend de almacenamiento activo (`git`)                |
| `spring.cloud.config.server.git.uri`           | Repositorio Git que contiene los archivos de configuración |
| `spring.cloud.config.server.git.clone-on-start`| Clona el repositorio al arrancar                        |
| `spring.cloud.config.server.git.default-label` | Rama por defecto (`main`)                               |

### Repositorio de configuración

Los archivos de configuración viven en un repositorio Git y siguen la convención de nombres de Spring Cloud Config:

```
config-server/
├── application.yml              # Propiedades comunes a todos los servicios
├── <servicio>.yml               # Propiedades de un servicio
└── <servicio>-<perfil>.yml      # Propiedades de un servicio para un perfil (dev, prod…)
```

## Ejecución

### Con Maven Wrapper

```bash
./mvnw spring-boot:run
```

### Compilando el JAR

```bash
./mvnw clean package
java -jar target/configserver-0.0.1-SNAPSHOT.jar
```

El servicio quedará disponible en `http://localhost:8085`.

### Orden de arranque recomendado

El Config Server debe iniciarse **antes** que el resto de los microservicios, ya que estos dependen de él para obtener su configuración.-
## 🗂️ Estructura del proyecto

```
configserver/
├── src/
│   ├── main/
│   │   ├── java/ar/edu/unju/fi/arquitecturas/logitrack/configserver/
│   │   │   └── ConfigserverApplication.java   # @EnableConfigServer
│   │   └── resources/
│   │       └── application.yml
│   └── test/
├── pom.xml
└── README.md
```