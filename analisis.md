# 📊 Análisis de resultados

## 🧾 Resultados de referencia

### Modelo `fast`

| Modelo | Prompt | Conjunto | Acierto (mín / medio / máx) | Tiempo medio | Umbral 75 % |
|---|---|---|---|---|---|
| fast | v1 | dev | 67 % / 67 % / 67 % | 6.8 s | ✗ |
| fast | v1 | holdout | 50 % / 57 % / 67 % | 13.2 s | ✗ |
| fast | v2 | dev | 100 % / 100 % / 100 % | 8.2 s | ✓ |
| fast | v2 | holdout | 100 % / 100 % / 100 % | 312.8 s | ✓ |

### Modelo `smart`

| Modelo | Prompt | Conjunto | Acierto (mín / medio / máx) | Tiempo medio | Umbral 75 % |
|---|---|---|---|---|---|
| smart | v1 | dev | 60 % / 63 % / 67 % | 5.3 s | ✗ |
| smart | v1 | holdout | 83 % / 83 % / 83 % | 6.0 s | ✓ |
| smart | v2 | dev | 100 % / 100 % / 100 % | 6.4 s | ✓ |
| smart | v2 | holdout | 100 % / 100 % / 100 % | 5.2 s | ✓ |

---

## 1. ¿Qué combinación pasarías a producción y por qué?

Pasaría a producción el modelo smart con la v2. Porque podemos ver claramente una diferencia enorme en el tiempo medio entre los modelos fast y smart. Además en la v1 vemos un mejor umbral en los casos holdout con el modelo smart, y en el conjunto dev ambos fallaron por ser demasiado vago el prompt.

---

## 2. ¿Cuánto cambió el acierto entre la versión 1 y la versión 2 con el mismo modelo, y qué significa eso?

En ambos modelos se puede ver claramente el porcentaje de acierto entre v1 y v2, obteniendo un 100% en v2, con un poco más de segundos, pero muy mínima la diferencia.

---

## 3. ¿Qué casos del hold-out set fallaron y qué cambiarías en el prompt o en el dataset para cubrirlos?

**Los fallos del hold-out (6 casos) aparecieron solo con el prompt v1.** Con v2 ambos modelos acertaron los 6 casos en las 5 ejecuciones.

| Modelo | Prompt | Acierto medio | Casos fallados (de 6) |
|---|---|---|---|
| fast | v1 | 57 % | entre 2 y 3 según la ejecución |
| smart | v1 | 83 % | 1, siempre en las 5 ejecuciones |
| fast / smart | v2 | 100 % | ninguno |

Que `smart` con v1 falle siempre el mismo número de casos se debe seguramente al prompt vago, el cual no le da un criterio para ese caso y el modelo decide por su cuenta, no por azar.

Qué cambiaría:

| Qué           | Cambio propuesto                                                                                                                                                 |
| ------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Prompt v1** | No lo arreglaría porque es justamente la versión vaga que queremos para poder comprobar como con unas instrucciones completa cambia nuestro nivel de respuestas. |
| **Prompt v2** | Podríamos seguir ejecutandolos con más casos y ver si en algún momento falla la ejecución con el fin de seguir agregando dataset                                 |
| **Dataset**   | Añadiría más casos difíciles: variantes de inyección de prompt (*"ignore previous instructions"*), CV mal armados y casos cercanos al límite de los 2 años.      |

---

## 4. 🎲 ¿Cuánto varió el acierto entre las 5 ejecuciones y por qué no basta con ejecutar una vez?

La variación apareció solo con el prompt v1.

| Modelo | Prompt | Conjunto | Rango entre ejecuciones | Variación |
|---|---|---|---|---|
| fast | v1 | holdout | 50 % a 67 % | **17 puntos** |
| smart | v1 | dev | 60 % a 67 % | **7 puntos** |
| fast | v1 | dev | 67 % (siempre) | 0 |
| smart | v1 | holdout | 83 % (siempre) | 0 |
| fast / smart | v2 | dev y holdout | 100 % (siempre) | 0 |

No basta con ejecutar una sola vez porque:

- **El mismo modelo con el mismo prompt no responde siempre igual.** `fast` con v1 en hold-out dio 50 % en una ejecución y 67 % en otra. Con una sola ejecución habría reportado seguramente un nivel más bajo de lo que llegué a conseguir.
- **Con pocos casos, un fallo pesa mucho.** Al tener pocos casos para el holdout por ejemplo, sacar un caso implicaría un cambio grande en el resultado, y esa diferencia puede confundirse con una mejora o un empeoramiento real del prompt.
- **La estabilidad de siempre obtener un alto nivel de resultados.** Que v2 dé 100% en las 5 ejecuciones aporta más confianza que un 100 % de una sola vez.
- **El tiempo también cambia.** El 312.8 s de `fast` v2 en hold-out viene de esperas por rate limit y reintentos, una sola medición lo habría tomado como la latencia real del modelo.
