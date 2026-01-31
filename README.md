# Microservices with Docker Compose

## 1.- Iniciar bases de datos en Docker

### Desde la carpeta del microservices
```
docker-compose up -d
```

### Verificar que están corriendo
```
docker ps
```

## 2.- Ejecutar Script de BD desde un cliente de BD

### Para microservicio user-service
- database/V1__CREATE_TABLES.sql
- database/V2__ADD_INDEXES.sql
- database/V3__INSERT_DATA.sql

### Para microservicio product-service
- database/V1__CREATE_TABLES.sql
- database/V2__ADD_INDEXES.sql
- database/V3__INSERT_DATA.sql

## 3.- Ejecutar pruebas con JSON

### Para microservicio user-service

#### Postman : Obtener todos los usuarios

```
http://localhost:8081/api/users

```
