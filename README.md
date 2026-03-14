# postman-api-testing

Automated API testing with Postman, Newman and Jenkins CI/CD

# Postman API Testing - CI/CD con Jenkins y Docker

## Descripción

Proyecto de testing automatizado de APIs usando Postman, Newman y Jenkins CI/CD.

## APIs Probadas

- **The Dog API** → https://thedogapi.com
- **The Cat API** → https://thecatapi.com

## Tech Stack

- Postman — diseño de colecciones y tests
- Newman — ejecución desde línea de comandos
- Jenkins — pipeline CI/CD
- Docker — contenedor de Jenkins
- newman-reporter-htmlextra — reportes HTML interactivos

## Estructura

```
postman-api-testing/
  dog-api/          → Colección, environment y CSV de Dog API
  cat-api/          → Colección, environment y CSV de Cat API
  reports/          → Reportes HTML generados por Newman
  Jenkinsfile       → Pipeline de Jenkins
```

## Cómo ejecutar localmente

```bash
# Dog API
newman run dog-api/TheDogAPI.json \
  --environment dog-api/ApiDogEnvironments.json \
  --iteration-data dog-api/dog_breeds.csv \
  -r htmlextra \
  --reporter-htmlextra-export reports/reporte-dog.html

# Cat API
newman run cat-api/TheCatAPI.json \
  --environment cat-api/ApiCatEnvironments.json \
  --iteration-data cat-api/cat_breeds.csv \
  -r htmlextra \
  --reporter-htmlextra-export reports/reporte-cat.html
```

Pipeline Jenkins
El pipeline ejecuta automáticamente:

1. Instala Newman y reporters
2. Corre tests de Dog API con CSV (5 iteraciones)
3. Corre tests de Cat API con CSV (5 iteraciones)
4. Publica reportes HTML en Jenkins

Cobertura de tests
| API | Endpoints | Assertions |
|-----|-----------|------------|
| Dog API | 10 requests | 110 assertions |
| Cat API | 10 requests | 160 assertions |
| **Total** | **20 requests** | **270 assertions** |
