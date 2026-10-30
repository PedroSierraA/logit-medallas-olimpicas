# ¿Qué predice ganar una medalla olímpica en atletismo?

Modelo de regresión **logística (logit)** que estima la probabilidad de que un atleta de atletismo olímpico gane medalla (oro, plata o bronce), controlando por sexo, estatura, peso y tipo de prueba. Proyecto de nota de seguimiento del curso de Econometría.

## Pregunta de investigación

Controlando por sexo, estatura, peso y tipo de prueba, ¿qué características de un atleta de atletismo olímpico aumentan la probabilidad de ganar medalla?

## Dataset

[**120 years of Olympic history: athletes and results**](https://github.com/rgriff23/Olympic_history) — 271,116 participaciones olímpicas individuales (Atenas 1896 – Río 2016), con edad, estatura, peso, sexo, país, deporte, evento y medalla. Compilado originalmente por sports-reference.com; es uno de los datasets deportivos más usados en ciencia de datos.

El CSV crudo está incluido en `Data/athlete_events.csv` para que el notebook corra sin pasos adicionales.

## De dónde salen las decisiones de limpieza

Antes de filtrar nada, el notebook inspecciona el dataset crudo con código (no solo texto) y respalda cada decisión con evidencia:

- **Atletismo**: el deporte individual con más observaciones (38,624), más del triple que Tiro o Ciclismo.
- **`Year >= 1960`**: antes de 1960 falta Estatura o Peso en el 55.5% de las filas en promedio (algunos años por encima del 70%); desde 1960 ese faltante cae a ~2%. Además, antes de 1960 competían muy pocos países (9 en 1896, 11 en 1904, con EEUU acaparando el 79% de las filas y el 91% de las medallas de 1904 por simple ausencia de rivales); desde 1960 compiten 72+ países.
- **Excluir relevos, pruebas por equipo y "all-around"**: la medalla ahí depende de varios atletas a la vez, no de las características físicas del individuo que se está modelando.

Tras aplicar los filtros y eliminar faltantes residuales, la muestra final usada para modelar queda en **21,844 observaciones**, con **~7.3% de medallistas** (variable dependiente desbalanceada).

## Qué hace el notebook, en orden

| Sección | Contenido |
|---|---|
| 0 | Revisión del dataset crudo (evidencia antes de decisiones) |
| 1 | Limpieza y filtrado final |
| 2 | Análisis exploratorio (tasa de medalla por categoría, boxplots Estatura/Peso) |
| 3 | Multicolinealidad (VIF) — se revisa *antes* de ajustar el modelo |
| 4 | Ajuste del modelo logit completo |
| 5 | Prueba global (LR test) y eliminación hacia atrás |
| 6 | Interpretación de odds ratios |
| 7 | Bondad de ajuste (pseudo R² de McFadden, Hosmer-Lemeshow) |
| 8 | Supuesto de linealidad en el logit (prueba de Box-Tidwell) |
| 9 | Evaluación predictiva fuera de muestra (train/test, ROC/AUC, umbral óptimo de Youden) + 9.1 balanceo de clases (oversampling) |
| 10 | Predicciones: atleta hipotético, atletas famosos reales, mapa de calor Estatura×Peso, interacción con el tiempo |
| 11 | Conclusiones generales |
| — | Fundamentos teóricos del logit (función logística, odds, MLE, matriz de confusión, ROC/AUC, Hosmer-Lemeshow, supuestos) |

## Resultados principales

Tras la eliminación hacia atrás, el modelo final retiene **Estatura, Peso, Sexo y Categoría de prueba** (Lanzamientos y Velocidad se diferencian de la categoría base; Edad y las demás categorías no aportan de forma significativa).

- Por cada cm adicional de estatura, las odds de medalla se multiplican por **1.032** (+3.2%).
- Ser hombre multiplica las odds de medalla por **0.502** — no por menor capacidad atlética, sino por **densidad de competencia**: en la muestra hay más participaciones masculinas (13,643) que femeninas (8,201) compitiendo por el mismo número de medallas por prueba (siempre 3).
- El modelo es globalmente significativo (LR test, p≈0), pero el pseudo R² de McFadden es bajo (~0.0185): esperado, porque solo se usan rasgos físicos, sin ninguna medida directa de desempeño deportivo (marca, tiempo, ranking).
- No hay violaciones graves de supuestos: VIF < 5, se cumple linealidad en el logit (Box-Tidwell) para Estatura y Peso.
- Poder predictivo fuera de muestra discreto pero mejor que el azar: **AUC ≈ 0.585**.
- Se probó balancear las clases con oversampling y da prácticamente el mismo resultado que mover el umbral de decisión (AUC ≈ 0.584): balancear no mejora la discriminación del modelo, solo desplaza el punto de corte.
- La interacción Estatura×Año y Peso×Año **no es significativa**: el cuerpo "ideal" para ganar medalla no ha cambiado entre 1960 y 2016. Lo que sí cambió es cuánta gente compite por el mismo número de medallas.

## Cómo correrlo

```bash
pip install pandas numpy matplotlib statsmodels scikit-learn scipy jupyter
jupyter nbconvert --to notebook --execute --inplace Guia_Logit_Olimpico.ipynb
```

O simplemente abrir `Guia_Logit_Olimpico.ipynb` en Jupyter/VS Code y correr las celdas en orden; ya vienen con sus outputs guardados.

## Estructura del repo

```
.
├── Data/
│   └── athlete_events.csv       # dataset crudo (120 years of Olympic history)
├── Guia_Logit_Olimpico.ipynb    # notebook completo
└── README.md
```
