# Wpp-noticias

Sistema de extracción, deduplicación, clasificación y priorización de noticias.
El flujo trabaja en Colab y envía las noticias priorizadas a Zapier.

Estructura general
------------------
1. Scraping básico de titulares:
   - Cada fuente está en el diccionario `FUENTES` con su URL base y clase (nacional/internacional).
   - `extraer_links(base)` recolecta URLs absolutas y aplica reglas por dominio para aislar artículos.

2. Descarga y parsing:
   - Se usa `newspaper3k` con `Config` personalizado para el user-agent.
   - `obtener_noticia(url)` devuelve título y texto.

3. Clasificación temática:
   - Pipeline zero-shot `facebook/bart-large-mnli`.
   - Etiquetas: economía, política, policial, deportes, otros.

4. Detección de duplicados:
   - Embeddings E5-base.
   - Búsqueda FAISS con IP.

5. Unión de duplicados:
   - Union-Find para agrupar transitivamente.
   - Fusión de textos.
   - Fusión de medios.

6. Conteo de palabras clave:
   - Lista estática de términos críticos.
   - Se cuenta con expresiones regulares palabra por palabra.
   - Campo generado: `palabras_claves`.

7. Análisis de sentimiento:
   - `pipeline("sentiment-analysis")`.
   - Se evalúan los primeros 512 caracteres.

8. Ranking de noticias:
   - `score()` combina:
        tipo (nacional/internacional)
        etiqueta
        sentimiento
        palabras clave
   - Se eligen:
        top 10 nacionales
        top 5 internacionales

9. Formato de salida:
   - Se construye un string concatenado:
        Titulo
        Texto
        Medio
        URL
        -------
10. Envío a Zapier:
   - Webhook POST.
   - Función: `enviar_resumen_por_zapier()`.


Uso
---
1. Ejecutar todas las celdas secuencialmente.
2. Revisar `df_news` para ver los artículos crudos.
3. Revisar `df_fusionado` para ver duplicados unificados.
4. Revisar `df_final` para los seleccionados y priorizados.
5. `noticias_string` es el payload final.
6. `enviar_resumen_por_zapier(noticias_string)` envía el resultado.

Notas
-----
- Los modelos corren en GPU si está disponible.
- El scraping es frágil por cambios en HTML de cada medio.
- Ajustar reglas de extracción según cada dominio.
- Ajustar umbral FAISS para calibrar sensibilidad a duplicados.
- Ajustar el scoring según preferencia del usuario final.

