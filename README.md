# API Testing con Postman + Jenkins + Docker

Automaticé las pruebas de dos APIs públicas usando Postman y Newman,
integradas en un pipeline CI/CD con Jenkins corriendo en Docker.

Este proyecto nació del curso de Postman de Free Range Testers y lo
llevé un paso más allá integrando todo con Jenkins y Docker para que
los tests corran automáticamente.

## ¿Qué probé?

**The Dog API** y **The Cat API** — dos APIs públicas que manejan
información e imágenes de perros y gatos.

Para cada API automaticé:

- Obtener listado completo de razas
- Buscar una raza por ID
- Buscar razas por nombre (data-driven con CSV)
- Obtener imágenes aleatorias
- Crear y eliminar votos
- Agregar y eliminar favoritos
- Validar respuestas contra un Mock Server

En total: **270 assertions ejecutadas en 10 iteraciones**.

## Stack

- Postman — colecciones, environments, Mock Server
- Newman — ejecución desde terminal con reportes HTML
- Jenkins — pipeline que corre todo automáticamente
- Docker — Jenkins corre en su propio contenedor, aislado

## Cómo correrlo

Necesitas Docker instalado. Luego:

```bash
git clone https://github.com/ipanaque94/postman-api-testing
cd postman-api-testing
docker-compose up --build -d
```

Abre `http://localhost:9091` y corre el pipeline `postman-api-pipeline`.

Los reportes HTML quedan publicados directamente en Jenkins.

## Para correrlo sin Docker

```bash
npm install -g newman newman-reporter-htmlextra

# CAT API
newman run "D:\PROYECTO POSTMAN API CAT\TheCatAPI.json" --environment "D:\PROYECTO POSTMAN API CAT\ApiCatEnvironments.json" --iteration-data "D:\PROYECTO POSTMAN API CAT\cat_breeds.csv"
* Generar el HTML EN LA CARPETA
newman run "D:\PROYECTO POSTMAN API CAT\TheCatAPI.json" --environment "D:\PROYECTO POSTMAN API CAT\ApiCatEnvironments.json" --iteration-data "D:\PROYECTO POSTMAN API CAT\cat_breeds.csv" -r htmlextra --reporter-htmlextra-export "D:\PROYECTO POSTMAN API CAT\reporte-cat-final.html"

# DOG API
PS D:\PROYECTO CICD-DOCKER-POSTMAN\postman-api-testing> newman run "D:\PROYECTO POSTMAN DOG API\TheDogAPI.json" --environment "D:\PROYECTO POSTMAN DOG API\ApiDogEnvironments.json" --iteration-data "D:\PROYECTO POSTMAN DOG API\dog_breeds.csv"
* Generar HTML
newman run "D:\PROYECTO POSTMAN DOG API\TheDogAPI.json" --environment "D:\PROYECTO POSTMAN DOG API\ApiDogEnvironments.json" --iteration-data "D:\PROYECTO POSTMAN DOG API\dog_breeds.csv" -r htmlextra --reporter-htmlextra-export "D:\PROYECTO POSTMAN DOG API\reporte-dog-final.html"
```

## Lo que aprendí

Automatizar no es solo escribir tests — es asegurarte de que esos
tests corran solos, en cualquier máquina, sin depender de
configuraciones locales. Docker resolvió exactamente eso.

---

**Enoc Ipanaque** — QA Automation Engineer
