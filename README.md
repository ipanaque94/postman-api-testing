# API Testing con Postman + Newman + Jenkins + Docker

> Mi primer proyecto de QA Automation. assertions automatizadas
> contra dos APIs públicas, con data-driven testing, Mock Server
> y pipeline CI/CD corriendo en Jenkins dentro de Docker.

---

## Este fue mi punto de partida

Antes de este proyecto, sabía que QA existía como rol pero no tenía
claro qué hacía un QA en el día a día. Pensaba que era alguien que
"clickeaba botones para ver si algo se rompía".

El curso de Postman de Free Range Testers me mostró algo distinto:
que QA es analizar qué debería hacer un sistema, diseñar cómo verificarlo,
y asegurarte de que esa verificación corra sola — sin que tengas que
estar presente cada vez.

Pero cuando terminé el curso y tenía mis colecciones funcionando en
Postman, me quedé con la misma pregunta que me acompaña en cada proyecto:
¿cómo hago para que esto no dependa de que yo lo ejecute manualmente?

La respuesta fue Jenkins + Docker. Y aprender a responder esa pregunta
es lo que convirtió este proyecto de "ejercicio del curso" a algo que
puedo mostrar en un portafolio.

---

## ¿Qué se prueba?

**The Dog API** y **The Cat API** — dos APIs públicas con autenticación
por API key que manejan razas, imágenes, votos y favoritos.

Tienen la misma estructura de endpoints
pero son sistemas independientes. Eso me permitió practicar el mismo patrón
de pruebas dos veces, en contextos distintos, y entender qué cambiaba y qué
era común entre ambas.

### Colección Dog API

| Endpoint | Método | Qué valida |
|---|---|---|
| `/breeds` | GET | Lista completa de razas retorna 200 y un array no vacío |
| `/breeds/{id}` | GET | Una raza específica tiene los campos esperados |
| `/breeds/search?q={name}` | GET | La búsqueda por nombre retorna razas que contienen el término buscado |
| `/images/random` | GET | Una imagen aleatoria tiene URL válida y type correcto |
| `/votes` | POST | Crear un voto retorna 200 con el ID del voto creado |
| `/votes/{id}` | DELETE | Eliminar un voto retorna 200 |
| `/favourites` | POST | Agregar un favorito retorna 200 con el ID del favorito |
| `/favourites/{id}` | DELETE | Eliminar un favorito retorna 200 |
| Mock Server | GET | La respuesta simulada coincide con el contrato esperado |

La colección de Cat API sigue exactamente la misma estructura — mismo
patrón, mismos tipos de assertions, diferente sistema.

---

## Data-driven testing con CSV — por qué importa

Una de las cosas que más me llamó la atención cuando aprendí esto
es que el mismo test puede correr contra múltiples datos sin duplicar
código.

Para la búsqueda por nombre de raza, en lugar de crear 10 tests
separados para "Labrador", "Bulldog", "Beagle"... creé uno solo
y lo alimenté con un CSV:

```
name,expected_count
Labrador,1
Bulldog,3
Poodle,2
Golden,2
Beagle,1
```

Newman corre ese test en cada fila del CSV — 10 iteraciones, mismo
test, datos distintos. En total el proyecto ejecuta **270 assertions
en 10 iteraciones** por colección.

Esto refleja la técnica de **partición de equivalencia** del ISTQB:
en lugar de probar 100 razas distintas, identifico los casos
representativos y los automatizo.

---

## Mock Server — probar sin depender de la API real

Esta fue la parte más interesante de aprender. El Mock Server de
Postman simula respuestas de la API sin hacer llamadas reales.

¿Para qué sirve eso?

Si la API real está caída por mantenimiento, tus tests siguen
corriendo. Si el equipo de backend todavía no terminó de desarrollar
un endpoint, puedes diseñar y validar los tests contra el mock
mientras ellos terminan.

Configuré el mock para que devuelva la misma estructura que la
API real — y el test valida que el contrato (la estructura de la
respuesta) sea correcto, independientemente de si es la API real
o el mock quien responde.

---

## Por qué Jenkins + Docker y no solo Newman desde terminal

Cuando terminé las colecciones, podía ejecutarlas con un comando
de Newman desde mi terminal. Funcionaba. Pero tenía un problema:
solo funcionaba en mi máquina, con mi configuración, con mis
variables de entorno.

Docker resuelve eso. El `Dockerfile.jenkins` define exactamente
qué versiones de Node.js y Newman se instalan, y Jenkins corre
dentro de ese contenedor — el mismo entorno en cualquier máquina
que tenga Docker instalado.

```
docker-compose up --build -d
# Abre http://localhost:9091
# Corre el pipeline: postman-api-pipeline
```

El Jenkinsfile tiene 4 stages:

1. **Checkout** — clona el repo
2. **Cat API Tests** — Newman corre la colección con el CSV de gatos
3. **Dog API Tests** — Newman corre la colección con el CSV de perros
4. **Publish Reports** — Los reportes HTML quedan publicados en Jenkins

Los reportes HTML generados por `newman-reporter-htmlextra` son
navegables — muestran cada request, cada assertion, qué pasó y qué
falló, con tiempos de respuesta incluidos.

---

## Lo que aprendí

**Las variables de entorno de Postman son el equivalente al `config.properties`**
de mis proyectos en Java. Separar la URL base, la API key y los datos de
prueba en un environment file permite cambiar entre ambientes (dev, staging,
prod) sin tocar las colecciones.

**El orden de los tests en una colección importa**
Si el test de DELETE corre antes del test de POST, falla porque intenta
eliminar algo que no existe. Aprendí a estructurar las colecciones en el
orden lógico del flujo de negocio: crear → consultar → modificar → eliminar.

**Newman y Postman no son exactamente lo mismo**
Algunas cosas que funcionan en la interfaz de Postman fallan cuando
las corres con Newman desde terminal — especialmente el manejo de
variables dinámicas que se establecen en scripts de `pm.test`. Tuve
que ajustar varios scripts para que funcionaran correctamente en ambos
entornos.

**Un reporte HTML no es solo estético**
El reporte de Newman muestra exactamente qué assertions fallaron, qué
valor esperabas y qué valor obtuviste. Esa información es la que un QA
comunica al equipo de desarrollo cuando reporta un defecto — no "falló
el test", sino "el endpoint `/breeds/search` devolvió 0 resultados para
'Labrador' cuando se esperaba al menos 1".

---

## Dónde encaja este proyecto en mi aprendizaje

Este proyecto fue el primero. Me enseñó los fundamentos:
qué es una colección, qué es un environment, cómo escribir
assertions en JavaScript dentro de Postman, qué es Newman,
y cómo conectar todo con un pipeline.

Los proyectos que vinieron después construyeron sobre esto:

- **[RestAssured API Testing](https://github.com/ipanaque94/RestAssured3APIS)** —
  el mismo concepto de pruebas de API pero en código Java, con suites
  organizadas, clases base reutilizables y reporte Allure publicado en GitHub Pages

- **[Playwright E2E Framework](https://github.com/ipanaque94/playwright-professional-framework)** —
  llevé la misma mentalidad de organización a pruebas de UI con TypeScript

La progresión no fue planificada desde el inicio. Fue orgánica: cada
proyecto me dejó con una pregunta nueva, y la siguiente herramienta
fue la que mejor la respondía.

---

## Cómo ejecutar

**Con Docker (recomendado):**

```bash
git clone https://github.com/ipanaque94/postman-api-testing
cd postman-api-testing
docker-compose up --build -d
# Abre http://localhost:9091
# Pipeline: postman-api-pipeline
```

**Sin Docker:**

```bash
npm install -g newman newman-reporter-htmlextra

# Cat API
newman run cat-api/TheCatAPI.json \
  --environment cat-api/ApiCatEnvironments.json \
  --iteration-data cat-api/cat_breeds.csv \
  -r htmlextra --reporter-htmlextra-export reports/reporte-cat.html

# Dog API
newman run dog-api/TheDogAPI.json \
  --environment dog-api/ApiDogEnvironments.json \
  --iteration-data dog-api/dog_breeds.csv \
  -r htmlextra --reporter-htmlextra-export reports/reporte-dog.html
```

---

## Stack

| Herramienta | Uso |
|---|---|
| Postman | Diseño de colecciones, environments, Mock Server y assertions |
| Newman | Ejecución de colecciones desde terminal y en CI |
| newman-reporter-htmlextra | Reportes HTML navegables con detalle por request |
| Jenkins | Pipeline CI/CD con stages por colección |
| Docker | Entorno reproducible — Jenkins + Newman en contenedor |

---

## Autor

**Enoc Ipanaque** — Lima, Perú

QA Automation Engineer en formación, estudiando para ISTQB Foundation Level.

[LinkedIn](www.linkedin.com/in/enoc-isaac-ipanaque-rodas-b3729a283) · [GitHub](https://github.com/ipanaque94)
