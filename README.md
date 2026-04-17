# Practica volúmenes PostgreSQL

## 1. Titulo

Creación de volúmenes para persistir base de datos con PostgreSQL usando contenedores Docker

## 2. Tiempo de duración

60 minutos

## 3. Fundamentos:

Docker es una plataforma que permite crear, desplegar y ejecutar aplicaciones dentro de contenedores. Un contenedor es una unidad ligera que incluye todo lo necesario para que una aplicación funcione, como el código, las librerías y las dependencias. A diferencia de las máquinas virtuales, los contenedores comparten el sistema operativo del host, lo que los hace más rápidos y eficientes.

PostgreSQL es un sistema de gestión de bases de datos relacional de código abierto, reconocido por su robustez, extensibilidad y cumplimiento de estándares SQL. En esta práctica se utiliza la imagen oficial de PostgreSQL para desplegar una base de datos dentro de un contenedor Docker.

Un volumen en Docker es un mecanismo de almacenamiento persistente que existe fuera del ciclo de vida del contenedor. Esto significa que, aunque el contenedor sea eliminado, los datos almacenados en el volumen se conservan. Sin un volumen, los datos se guardan en la capa de escritura del contenedor, la cual se destruye junto con él.

En esta práctica se demuestra la diferencia entre ambos escenarios: primero se crea un contenedor PostgreSQL sin volumen para verificar que los datos se pierden al eliminarlo, y luego se repite el proceso utilizando un volumen nombrado (`pgdata`) para comprobar que los datos persisten correctamente.

Durante el desarrollo también se presentaron errores comunes relacionados con la ruta del volumen y la reutilización de contenedores, los cuales fueron identificados y corregidos usando comandos como `docker volume inspect` y ajustando la ruta de montaje a `/var/lib/postgresql/data`.

## 4. Conocimientos previos.

Para realizar esta práctica el estudiante necesita tener claro los siguientes temas:

* Comandos básicos de Linux
* Manejo de terminal (WSL o nativa)
* Conceptos básicos de bases de datos relacionales
* Uso básico de SQL (CREATE, INSERT, SELECT)
* Conceptos básicos de Docker (contenedores, imágenes, puertos)
* Concepto de volúmenes y persistencia de datos

## 5. Objetivos a alcanzar

* Crear contenedores PostgreSQL usando Docker
* Comprender la pérdida de datos al eliminar un contenedor sin volumen
* Crear y asociar volúmenes Docker para persistir datos
* Verificar la persistencia de datos al recrear un contenedor con volumen
* Identificar y corregir errores comunes en el montaje de volúmenes
* Exportar historial de comandos en Linux

## 6. Equipo necesario:

* Computador con sistema operativo Windows
* WSL (Windows Subsystem for Linux)
* Docker instalado
* Terminal de comandos (bash)
* Cliente psql (incluido en la imagen de PostgreSQL)

## 7. Material de apoyo.

* Documentación oficial de Docker
* Documentación oficial de PostgreSQL
* Guía de la asignatura

---

## 8. Procedimiento

### Parte 1: Base de datos sin volumen

**Paso 1:** Crear el contenedor `server_db1` sin volumen

```bash
docker run -d \
  --name server_db1 \
  -e POSTGRES_PASSWORD=1234 \
  -e POSTGRES_USER=admin \
  -p 5432:5432 \
  postgres
```

---

**Paso 2:** Conectarse al contenedor y crear la base de datos, tabla e insertar datos

```bash
docker exec -it server_db1 psql -U admin
```

```sql
CREATE DATABASE test;
\c test

CREATE TABLE customer (
  id SERIAL PRIMARY KEY,
  fullname VARCHAR(100),
  status VARCHAR(20)
);

INSERT INTO customer (fullname, status) VALUES ('Daniela Abad', 'activo');
SELECT * FROM customer;
\q
```

---

**Paso 3:** Detener y eliminar el contenedor

```bash
docker stop server_db1
docker rm server_db1
```

---

**Paso 4:** Recrear el contenedor con el mismo nombre

```bash
docker run -d \
  --name server_db1 \
  -e POSTGRES_PASSWORD=1234 \
  -e POSTGRES_USER=admin \
  -p 5432:5432 \
  postgres
```

---

**Paso 5:** Verificar que los datos no persisten

```bash
docker exec -it server_db1 psql -U admin
```

```sql
\l
\q
```

> La base de datos `test` ya no existe. Los datos se perdieron al eliminar el contenedor sin volumen.

---

### Parte 2: Base de datos con volumen

**Paso 6:** Crear el volumen `pgdata`

```bash
docker volume create pgdata
docker volume ls
```

---

**Paso 7:** Crear el contenedor `server_db2` asociado al volumen

```bash
docker run -d \
  --name server_db2 \
  -e POSTGRES_PASSWORD=1234 \
  -e POSTGRES_USER=admin \
  -p 5433:5432 \
  -v pgdata:/var/lib/postgresql/data \
  postgres
```

---

**Paso 8:** Conectarse y crear base de datos, tabla e insertar datos

```bash
docker exec -it server_db2 psql -U admin
```

```sql
CREATE DATABASE test;
\c test

CREATE TABLE customer (
  id SERIAL PRIMARY KEY,
  fullname VARCHAR(100),
  status VARCHAR(20)
);

INSERT INTO customer (fullname, status) VALUES ('Daniela Abad', 'activo');
SELECT * FROM customer;
\q
```

---

**Paso 9:** Detener y eliminar el contenedor `server_db2`

```bash
docker stop server_db2
docker rm server_db2
```

---

**Paso 10:** Recrear el contenedor `server_db2` reutilizando el mismo volumen

```bash
docker run -d \
  --name server_db2 \
  -e POSTGRES_PASSWORD=1234 \
  -e POSTGRES_USER=admin \
  -p 5433:5432 \
  -v pgdata:/var/lib/postgresql/data \
  postgres
```

---

**Paso 11:** Verificar que los datos persisten 

```bash
docker exec -it server_db2 psql -U admin
```
```sql
\l
\c test
SELECT * FROM customer;
\q
```
<img src="https://github.com/user-attachments/assets/e93bce21-b37f-48e0-aac9-c06f51a2e990" width="450" />

> La base de datos `test` y la tabla `customer` siguen existiendo con sus registros intactos.

---

### Errores encontrados y solución

Durante la práctica se presentó un problema de persistencia: el volumen existía pero los datos no se conservaban correctamente. La causa fue que la ruta de montaje estaba incompleta (`/var/lib/postgresql` en lugar de `/var/lib/postgresql/data`).

**Diagnóstico:**

```bash
docker volume ls
docker volume inspect pgdata
```

**Solución aplicada:** Eliminar el volumen con la ruta incorrecta y recrearlo especificando la ruta correcta:

```bash
docker rm -f server_db2
docker volume rm pgdata
docker volume create pgdata

docker run -d \
  --name server_db2 \
  -e POSTGRES_PASSWORD=1234 \
  -e POSTGRES_USER=admin \
  -p 5433:5432 \
  -v pgdata:/var/lib/postgresql/data \
  postgres
```

---

**Paso 12:** Guardar historial de comandos

```bash
history | tee tarea-s3-Daniela_Abad.txt
```

---

## 9. Resultados esperados:

Se logró demostrar el comportamiento de los datos en dos escenarios distintos con Docker y PostgreSQL.

En la **Parte 1**, al eliminar el contenedor `server_db1` sin volumen asociado, la base de datos `test` y sus registros desaparecieron por completo al recrear el contenedor, confirmando que los datos no persisten sin volumen.

En la **Parte 2**, al asociar el volumen `pgdata` al contenedor `server_db2`, los datos de la base de datos `test` y la tabla `customer` permanecieron intactos después de eliminar y recrear el contenedor, demostrando la persistencia que ofrece un volumen Docker.

Además, se identificó y corrigió un error común relacionado con la ruta de montaje del volumen, lo que refuerza la comprensión del funcionamiento interno de los volúmenes en Docker.

## 10. Bibliografía

- Daniela, A. (2026). Volúmenes Docker y persistencia de datos con PostgreSQL.
