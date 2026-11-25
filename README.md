# Laboratorio 6 · Banco de pruebas para selección de personal

Banco de pruebas que mide el porcentaje de acierto de un asistente que decide si un CV es **APTO**, **REVISAR** o **DESCARTAR** para el puesto de *AI Engineer*. Compara dos modelos (`fast`, `smart`) con dos versiones de prompt (`v1` vaga, `v2` con criterios claros).

## Criterios del puesto

| Decisión | Criterio |
|---|---|
| APTO | 2 o más años de experiencia. |
| DESCARTAR | Menos de 2 años. |
| REVISAR | Falta información, o le faltan menos de 3 meses para los 2 años. |

## Estructura

| Archivo | Contenido |
|---|---|
| `app.ts` | Banco de pruebas con `createAgent`, `responseFormat` y middleware (guardrail + reintentos). |
| `app-with-structured-output.ts` | Misma comparación con `withStructuredOutput`, para contrastar. |
| `dataset.json` | `dev-set` (15 casos) y `holdout-set` (6 casos), con la decisión esperada. |
| `docs/laboratory-assigment.md` | Enunciado. |

Los prompts están en `app.ts` (`prompt1` y `prompt2`).

## Cómo ejecutarlo

1. Instalar dependencias:

   ```bash
   npm install
   ```

2. Crear un archivo `.env` con la clave del curso:

   ```env
   OPENAI_API_KEY=<tu clave del curso>
   OPENAI_BASE_URL=https://ai.commitacademy.io/v1
   ```

3. Ejecutar:

   ```bash
   npm start                  # versión con createAgent
   npm run start:structured   # versión con withStructuredOutput
   ```

El resultado es una tabla con una fila por combinación de modelo, prompt y conjunto:

```text
modelo  prompt  conjunto  acierto (mín/medio/máx)  tiempo medio  umbral
fast    v1      dev       40% / 47% / 53%          1.1 s         ✗
```

- Cada combinación se ejecuta `RUNS` veces (5 en la comparación final).
- Una respuesta que no sea una de las tres etiquetas cuenta como fallo.
- El umbral es del 75 % sobre el acierto medio.
- `dev` y `holdout` se informan por separado.

> Una ejecución completa son unas 420 llamadas. Para iterar un prompt, reduce temporalmente `MODELS`, `RUNS` o los casos usados en `app.ts`.

## Cómo añadir un modelo

En `app.ts`, añade su nombre a la lista `MODELS`:

```ts
const MODELS = ["fast", "smart", "otro-modelo"];
```

## Cómo añadir una versión de prompt

1. Crear la función que recibe el CV y devuelve el prompt:

   ```ts
   const prompt3: PromptBuilder = (cv) => `... Analiza este CV: ${cv}`;
   ```

2. Registrarla en `PROMPTS`:

   ```ts
   const PROMPTS: Record<string, PromptBuilder> = { v1: prompt1, v2: prompt2, v3: prompt3 };
   ```

## Guardrail y reintentos

- **Guardrail por discriminación:** si el CV contiene alguna palabra de `FORBIDDEN_WORDS`, no se llama al modelo y se devuelve `FALLBACK_MESSAGE`. Ese caso cuenta como fallo.
- **Reintentos:** `modelRetryMiddleware` reintenta con backoff exponencial ante errores como el rate limit (429).

## Notas

- No subir el `.env` ni `node_modules` al `.zip`.
- Todos los CV del dataset son inventados.
