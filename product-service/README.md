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
- PostgreSQL
- Maven 3.8+

### Base de Datos (Docker)

- Usar el siguiente `docker-compose.yml` para iniciar PostgreSQL:

localización: ../docker-compose.yml

```yaml

services:
  # PostgreSQL para User Service
  postgres-user:
    image: postgres:15-alpine
    container_name: postgres-user
    environment:
      POSTGRES_DB: userdb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_INITDB_ARGS: "--encoding=UTF8 --locale=en_US.UTF-8"
    ports:
      - "5432:5432"
    volumes:
      - postgres-user-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d userdb"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  # PostgreSQL para Product Service
  postgres-product:
    image: postgres:15-alpine
    container_name: postgres-product
    environment:
      POSTGRES_DB: productdb
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_INITDB_ARGS: "--encoding=UTF8 --locale=en_US.UTF-8"
    ports:
      - "5433:5432"  # Puerto externo 5433, interno 5432
    volumes:
      - postgres-product-data:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres -d productdb"]
      interval: 10s
      timeout: 5s
      retries: 5
    restart: unless-stopped

  # pgAdmin (Opcional - para visualizar DBs)
  pgadmin:
    image: dpage/pgadmin4:latest
    container_name: pgadmin
    environment:
      PGADMIN_DEFAULT_EMAIL: admin@admin.com
      PGADMIN_DEFAULT_PASSWORD: admin
      PGADMIN_CONFIG_SERVER_MODE: 'False'
    ports:
      - "5050:80"
    volumes:
      - pgadmin-data:/var/lib/pgadmin
    depends_on:
      - postgres-user
      - postgres-product
    restart: unless-stopped

volumes:
  postgres-user-data:
    driver: local
  postgres-product-data:
    driver: local
  pgadmin-data:
    driver: local

```

```bash
# Iniciar PostgreSQL
docker-compose up -d
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
