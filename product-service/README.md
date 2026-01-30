# Product Service - Clean Architecture

Microservicio de Productos construido con **Clean Architecture** usando Spring Boot 3.5.6.

## 🏗️ Arquitectura

```
src/main/java/com/tecsup/app/micro/product/
├── ProductServiceApplication.java          # Clase principal
│
├── domain/                                  # CAPA DE DOMINIO (núcleo)
│   ├── model/
│   │   └── Product.java                    # Entidad de dominio
│   ├── repository/
│   │   └── ProductRepository.java          # Interface del repositorio (puerto)
│   └── exception/
│       ├── ProductNotFoundException.java
│       ├── InvalidProductDataException.java
│       └── UserServiceException.java
│
├── application/                             # CAPA DE APLICACIÓN
│   ├── usecase/                            # Casos de uso individuales
│   │   ├── GetAllProductsUseCase.java
│   │   ├── GetProductByIdUseCase.java
│   │   ├── GetAvailableProductsUseCase.java
│   │   ├── GetProductsByUserUseCase.java
│   │   ├── CreateProductUseCase.java
│   │   ├── UpdateProductUseCase.java
│   │   └── DeleteProductUseCase.java
│   └── service/
│       └── ProductApplicationService.java   # Orquestador de casos de uso
│
├── infrastructure/                          # CAPA DE INFRAESTRUCTURA
│   └── persistence/
│       ├── entity/
│       │   └── ProductEntity.java          # Entidad JPA
│       ├── mapper/
│       │   └── ProductPersistenceMapper.java # Mapper MapStruct
│       └── repository/
│           ├── JpaProductRepository.java    # Spring Data JPA
│           └── ProductRepositoryImpl.java   # Adaptador
│
└── presentation/                            # CAPA DE PRESENTACIÓN
    ├── controller/
    │   ├── ProductController.java          # REST Controller
    │   └── GlobalExceptionHandler.java     # Manejo de excepciones
    ├── dto/
    │   ├── CreateProductRequest.java
    │   ├── UpdateProductRequest.java
    │   └── ProductResponse.java
    └── mapper/
        └── ProductDtoMapper.java           # Mapper MapStruct
```

## 📊 Flujo de Arquitectura

```
┌─────────────────────────────────────────────────────────────────┐
│                    PRESENTATION LAYER                           │
│  ┌─────────────┐  ┌───────────────┐  ┌────────────────────────┐ │
│  │ Controller  │──│   DTOs        │──│  ProductDtoMapper      │ │
│  └──────┬──────┘  └───────────────┘  └────────────────────────┘ │
└─────────┼───────────────────────────────────────────────────────┘
          │
┌─────────▼───────────────────────────────────────────────────────┐
│                    APPLICATION LAYER                            │
│  ┌──────────────────────────┐  ┌──────────────────────────────┐ │
│  │ ProductApplicationService│──│   Use Cases                  │ │
│  │   (Orchestrator)         │  │   - GetAllProductsUseCase    │ │
│  └──────────┬───────────────┘  │   - CreateProductUseCase     │ │
│             │                  │   - UpdateProductUseCase     │ │
│             │                  │   - DeleteProductUseCase     │ │
│             │                  └──────────────────────────────┘ │
└─────────────┼───────────────────────────────────────────────────┘
              │
┌─────────────▼───────────────────────────────────────────────────┐
│                      DOMAIN LAYER                               │
│  ┌───────────────┐  ┌────────────────────┐  ┌───────────────┐   │
│  │    Product    │  │  ProductRepository │  │  Exceptions   │   │
│  │   (Entity)    │  │    (Interface)     │  │               │   │
│  └───────────────┘  └─────────┬──────────┘  └───────────────┘   │
└───────────────────────────────┼─────────────────────────────────┘
                                │
┌───────────────────────────────▼─────────────────────────────────┐
│                   INFRASTRUCTURE LAYER                          │
│  ┌───────────────────────┐  ┌────────────────────────────────┐  │
│  │ ProductRepositoryImpl │──│      JpaProductRepository      │  │
│  │     (Adapter)         │  │     (Spring Data JPA)          │  │
│  └───────────────────────┘  └────────────────────────────────┘  │
│  ┌──────────────────────┐                                       │
│  │   ProductEntity      │                                       │
│  │   (JPA Entity)       │                                       │
│  └──────────────────────┘                                       │
└─────────────────────────────────────────────────────────────────┘
```

## 🚀 Configuración

### Requisitos
- Java 21+
- PostgreSQL (Docker recomendado)
- Maven 3.8+

### Base de Datos (Docker)

```bash
# Crear red de microservicios (si no existe)
docker network create microservices-network

# Iniciar PostgreSQL
docker-compose up -d
```

O manualmente:

```bash
docker run -d \
  --name postgres-product \
  -e POSTGRES_DB=productdb \
  -e POSTGRES_USER=postgres \
  -e POSTGRES_PASSWORD=postgres \
  -p 5433:5432 \
  postgres:15
```

### Ejecutar la Aplicación

```bash
./mvnw spring-boot:run
```

## 📡 API Endpoints

| Método | Endpoint | Descripción |
|--------|----------|-------------|
| GET | `/api/products` | Obtener todos los productos |
| GET | `/api/products/{id}` | Obtener producto por ID |
| GET | `/api/products/available` | Obtener productos disponibles (stock > 0) |
| GET | `/api/products/user/{userId}` | Obtener productos por usuario |
| POST | `/api/products` | Crear producto |
| PUT | `/api/products/{id}` | Actualizar producto |
| DELETE | `/api/products/{id}` | Eliminar producto |
| GET | `/api/products/health` | Health check |

## 📝 Ejemplos de Peticiones

### Crear Producto
```bash
curl -X POST http://localhost:8082/api/products \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Laptop Gaming",
    "description": "Laptop para gaming de alta gama",
    "price": 1599.99,
    "stock": 10,
    "category": "Electronics",
    "createdBy": 1
  }'
```

### Obtener Todos los Productos
```bash
curl http://localhost:8082/api/products
```

### Obtener Producto por ID
```bash
curl http://localhost:8082/api/products/1
```

### Actualizar Producto
```bash
curl -X PUT http://localhost:8082/api/products/1 \
  -H "Content-Type: application/json" \
  -d '{
    "name": "Laptop Gaming Pro",
    "description": "Laptop para gaming de alta gama actualizada",
    "price": 1799.99,
    "stock": 15,
    "category": "Electronics"
  }'
```

### Eliminar Producto
```bash
curl -X DELETE http://localhost:8082/api/products/1
```

## 🧪 Testing

```bash
./mvnw test
```

## 📦 Tecnologías

- Spring Boot 3.5.6
- Spring Data JPA
- PostgreSQL
- MapStruct 1.5.5 (para mapeo de objetos)
- Lombok
- Jakarta Validation
- Java 21

## 🔑 Principios de Clean Architecture

1. **Independencia de Frameworks**: El dominio no depende de Spring
2. **Testeabilidad**: Cada capa puede probarse de forma aislada
3. **Independencia de UI**: El controlador puede cambiarse sin afectar la lógica
4. **Independencia de BD**: JPA puede cambiarse por otra tecnología
5. **Independencia de agentes externos**: Los servicios externos están detrás de interfaces
