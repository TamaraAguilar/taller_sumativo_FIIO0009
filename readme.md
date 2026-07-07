# Despliegue de entorno con Docker

## Descripción

Entorno web compuesto por dos servicios containerizados con Docker:

- **Frontend**: Angular servido con Nginx (angular-cc-1-client), accesible en http://localhost:8080
- **Backend**: Node.js (angular-cc-1-server), accesible en http://localhost:3000

## Uso de imágenes

- **node:24-alpine**: se eligió la variante `alpine` por ser significativamente más liviana
  (~40MB vs ~350MB de la imagen completa), reduciendo el tamaño final y la superficie de
  ataque del contenedor. Se fijó la versión `24` (LTS) en lugar de usar `latest`, para
  garantizar builds reproducibles y evitar romper la app ante actualizaciones inesperadas
  de Node.

- **nginx:1.27-alpine**: mismo criterio, imagen oficial, ligera y con versión fija.
  Nginx se usa para servir los archivos estáticos del build de Angular de forma eficiente
  y como proxy reverso hacia el backend.

## Dockerfile

- **Backend**: usa un único stage. Se copian primero `package*.json` para aprovechar la
  caché de capas de Docker (si el código cambia pero no las dependencias, no se reinstala
  `npm install`). Se instala solo con `--omit=dev` para no incluir dependencias de
  desarrollo en la imagen final. Se crea un usuario no root (`appuser`) para no ejecutar
  el proceso con privilegios de administrador dentro del contenedor.

- **Frontend**: usa **multi-stage build**. La primera etapa compila la app Angular con
  Node.js; la segunda etapa solo copia el resultado (`dist/`) a una imagen limpia de
  Nginx. Esto evita que las herramientas de build y `node_modules` (pesados y
  innecesarios en producción) terminen en la imagen final.

## Volúmenes y puertos

- El **frontend** expone el puerto `80` internamente, mapeado a `8080` en el host.
- El **backend** expone el puerto `3000`, mapeado 1:1 al host.
- Se configuró un volumen (`backend_logs`) para persistir los logs del backend fuera
  del ciclo de vida del contenedor. No se utiliza base de datos en este proyecto, por
  lo que no aplica persistencia de datos de negocio.

## Cómo levantar el proyecto

```bash
docker compose up -d --build
```

Verificar contenedores activos:
```bash
docker ps
```
