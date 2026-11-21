# Guion de exposición de Noticias

**16 diapositivas · 11 minutos 0 segundos estimados · tres integrantes.**

La presentación conserva el alcance técnico y aligera la experiencia: una idea principal por pantalla, ejemplos y cambios de recurso visual. Las diapositivas acompañan la explicación; las fórmulas completas y las preguntas difíciles quedan en los documentos de apoyo. Este guion figura también en las notas del PowerPoint.

- Persona 1: diapositivas 1–5 · 3 min 20 s.
- Persona 2: diapositivas 6–11 · 4 min 00 s.
- Persona 3: diapositivas 12–16 · 3 min 40 s.

## Cómo mantener la atención

Abrir con la pregunta y dejar una pausa breve. Señalar el elemento que se explica; evitar leer títulos y cifras dos veces. En WALMEX, explicar qué permite comprobar el dato. La demo es opcional: si se hace en vivo, sustituye la explicación de la diapositiva 15 y se ensaya para mantener el tiempo. Dejar detalles de fórmulas, hashes y rutas completas para las preguntas.

## 1. ¿Qué hay detrás de un indicador de noticias?

**Persona 1 · 35 segundos**

Antes de comenzar, una pregunta: si un sistema nos dice que las noticias de una empresa son positivas, ¿cómo comprobamos ese resultado? Nuestro módulo busca responderla. Parte de noticias financieras, las organiza, calcula indicadores y conserva los artículos que los respaldan. Vamos a seguir ese recorrido con un caso real de WALMEX. La idea central es sencilla: cada respuesta debe poder regresar a sus datos y a sus fuentes.

**Señalar:** Señalar la pregunta y las tres etapas.

**Transición:** Primero, veamos por qué una noticia es un dato difícil de procesar.

## 2. El reto está en ordenar la información

**Persona 1 · 35 segundos**

Las fuentes entregan información de maneras diferentes: APIs, RSS y extractos. Una misma noticia puede aparecer en varias capturas; además, algunas no tienen una fecha de publicación fiable. Si agregáramos todo directamente, podríamos contar información repetida o atribuirle una fecha incorrecta. Por eso, antes de interpretar noticias necesitamos integración, limpieza y trazabilidad. Es ahí donde comienza el trabajo de ingeniería y minería de datos.

**Señalar:** Recorrer formato, repetición y fechas.

**Transición:** Nuestra arquitectura separa estas responsabilidades.

## 3. De las fuentes a una consulta con evidencia

**Persona 1 · 40 segundos**

Este es el recorrido completo. Los conectores obtienen noticias y guardan capturas. Después, el procesamiento construye tablas e indicadores. Finalmente, el servicio semántico permite consultar esos resultados desde el panel o las herramientas de los agentes. Separar procesamiento y consulta tiene una consecuencia práctica: hacer una pregunta no vuelve a descargar noticias. Las APIs y RSS sirven para capturar; JSON y Parquet conservan datos; el servicio permite interpretarlos con reglas comunes.

**Señalar:** Seguir primero la fila de procesamiento y luego la de consulta.

**Transición:** Ahora acompañemos una noticia real por las capas.

## 4. Una noticia real, tres transformaciones

**Persona 1 · 45 segundos**

El titular de Walmart México que mostramos está en los archivos de DataBursátil. Bronce conserva el registro y su lote de captura. Silver añade una identidad estable, la asociación con WALMEX y la disponibilidad temporal. Como no tenemos una publicación fiable, el campo queda nulo. Gold utiliza esa noticia junto con otras para calcular indicadores de la empresa y la fecha. Los identificadores permiten recorrer el camino en sentido contrario: del agregado a los artículos y a las capturas. JSON conserva el respaldo y Parquet las tablas con sus tipos.

**Señalar:** Señalar titular, identidad Silver y conjunto Gold.

**Transición:** Hay dos implementaciones del procesamiento y conviene distinguirlas desde ahora.

## 5. Dos recorridos, resultados diferentes

**Persona 1 · 45 segundos**

La práctica PySpark implementa Bronce, Silver y Golden y calcula medias sin ponderación. El medallion que alimenta las consultas conserva el histórico y calcula características ponderadas. El agente consume esta segunda salida. No intercambiamos ambas como si sus fórmulas fueran iguales: conectarlas requiere adaptar esquemas y validar resultados. También detectamos un problema concreto en la revisión: el ejecutor y la prueba Spark importan una ruta inexistente. Lo documentamos; no afirmamos que ese recorrido haya pasado las pruebas.

**Señalar:** Comparar motores, agregación y consumidor; señalar el aviso.

**Transición:** Mi compañero explicará cómo convertimos los artículos en información cuantitativa.

## 6. Primero: limpiar y reconocer la empresa

**Persona 2 · 35 segundos**

La limpieza elimina HTML, entidades y espacios innecesarios. Después vinculamos noticias con empresas usando sus nombres y alias. También distinguimos entre un titular y un extracto para no guardar el mismo texto dos veces. Estas decisiones parecen pequeñas, pero afectan qué analizamos y cómo agrupamos los registros. La asociación por alias es léxica: puede equivocarse y no demuestra por sí sola una relación económica con la empresa.

**Señalar:** Recorrer limpieza, alias e identidad.

**Transición:** El siguiente reto es evitar que la repetición distorsione el resultado.

## 7. Repetir una noticia no debe multiplicar su señal

**Persona 2 · 40 segundos**

Aplicamos dos tratamientos diferentes. Silver elimina duplicados exactos de la misma clave dentro de una captura, pero conserva observaciones entre capturas para auditoría. Para Gold se selecciona la última observación disponible de cada artículo y asociación. Además, TF-IDF compara titulares de la misma empresa y día. Si la similitud coseno supera cero punto cinco, reduce el peso de los similares posteriores. Esa similitud no prueba que describan el mismo evento: es una regla para limitar la repetición textual.

**Señalar:** Señalar eliminación exacta frente a reducción de peso.

**Transición:** Además de la repetición, importa la antigüedad.

## 8. El peso cambia con la antigüedad y la relevancia

**Persona 2 · 40 segundos**

El peso temporal disminuye de forma exponencial. Este ejemplo matemático muestra cómo, manteniendo los demás factores en uno, una noticia pasa de peso uno a aproximadamente cero punto setenta y cuatro después de un día, y cero punto cuarenta y uno después de tres. El cálculo también reduce el peso cuando la relevancia es menor que cero punto cuatro. Son reglas explícitas, no parámetros aprendidos. El detalle técnico de su referencia temporal está en la revisión: el agregador utiliza el fin del día UTC para el decaimiento.

**Señalar:** Señalar las tres barras y la condición de relevancia.

**Transición:** Con los pesos y el sentimiento calculamos seis características.

## 9. Seis indicadores resumen el contexto

**Persona 2 · 45 segundos**

Los seis indicadores responden preguntas distintas. El sentimiento resume el balance de palabras positivas y negativas mediante una media ponderada. El volumen es el logaritmo de uno más la suma de pesos, por eso no equivale al número de artículos. El cambio compara ventanas reciente y anterior. La dispersión mide cuánto difieren los sentimientos; la relevancia resume la asociación, y ESG considera titulares de esa temática. Las fórmulas completas están en el material de defensa. Ninguna de estas variables demuestra por sí sola cómo se moverá el precio de una acción.

**Señalar:** Señalar sentimiento y volumen; recorrer brevemente los cuatro restantes.

**Transición:** Para un análisis financiero correcto, también debemos saber cuándo se conocía cada dato.

## 10. El pasado se consulta con un corte temporal

**Persona 2 · 40 segundos**

Distinguimos publicación, captura, disponibilidad y cálculo. La disponibilidad es el máximo entre publicación y captura; sin publicación usamos la captura. En este ejemplo, una consulta con corte a mediodía excluye una captura de las tres de la tarde. También debe excluir un cálculo hecho después del corte. Esto ayuda a prevenir el sesgo de anticipación: usar en el pasado información que todavía no se conocía. Las ventanas permiten comparar periodos, pero sus conteos no se suman si se superponen.

**Señalar:** Señalar el corte y la captura excluida.

**Transición:** Estas reglas se reúnen en la capa semántica.

## 11. Una respuesta también debe explicar sus límites

**Persona 2 · 40 segundos**

La capa semántica centraliza el significado de las métricas y devuelve fecha, ventana, método y procedencia. Permite consultar empresa, tema, historial, cobertura y evidencia. También distingue datos antiguos, entidades sin resultado, archivos no disponibles y problemas de integridad. Esto evita interpretar cualquier ausencia como un sentimiento neutral. Cuando se pide una fecha histórica exacta, no se sustituye silenciosamente por otra. En el caso de WALMEX veremos un resultado antiguo con su fecha real, no presentado como información actual.

**Señalar:** Señalar los estados y los metadatos que acompañan una respuesta.

**Transición:** Mi compañero mostrará cómo llegan estas consultas al usuario.

## 12. El agente pregunta a herramientas, las tools a los datos

**Persona 3 · 45 segundos**

El usuario puede preguntar al orquestador, que delega en el agente de Noticias. El especialista dispone de cinco herramientas: esquema, consulta, historial, evidencia y cobertura. Están registradas mediante un servidor MCP interno del SDK y llaman al servicio semántico. El modelo decide qué herramienta necesita y redacta la explicación; los indicadores provienen del procesamiento guardado. Esta interfaz conserva el contrato que utiliza el coordinador. Para ejecutar la conversación completa se necesita autenticación y acceso al modelo configurado.

**Señalar:** Seguir la ruta y señalar las cinco herramientas.

**Transición:** El Observatorio ofrece otra forma de acceder al mismo servicio.

## 13. Observatorio: explorar, comprobar y preguntar

**Persona 3 · 40 segundos**

El panel permite seleccionar empresa o tema y fecha, revisar indicadores, explorar el histórico y abrir titulares. Sus consultas pasan por Flask al servicio semántico. El chat tiene un recorrido distinto: NewsAgent utiliza un cliente MCP por entrada y salida estándar hacia un servidor separado. Ambos reutilizan la lógica de consulta, pero no son la misma integración. El panel puede leer datos locales; el chat requiere proveedor configurado. Aquí mostramos un diagrama funcional, porque no obtuvimos una captura de navegador durante la revisión.

**Señalar:** Señalar la exploración y después la ruta del chat; aclarar que es un diagrama.

**Transición:** Veamos qué devuelve una consulta sobre archivos reales.

## 14. WALMEX: un resultado que podemos comprobar

**Persona 3 · 45 segundos**

Este es el corte del treinta y uno de octubre de dos mil veintiséis, con ventana de siete días y método keywords. Encontramos siete artículos recientes, un sentimiento de menos cero punto cero cero cinco tres y relevancia media de cero punto siete siete cuatro. El volumen ponderado es uno punto doscientos catorce; no son uno punto dos artículos. El histórico disponible para WALMEX tiene solo dos observaciones, con el mismo score. Las mostramos sin inventar una tendencia. La consulta marca los datos como antiguos y el signo ligeramente negativo no constituye una predicción de precios.

**Señalar:** Señalar fecha, tres números y los dos puntos históricos.

**Transición:** Ahora comprobemos qué artículos respaldan ese agregado.

## 15. La evidencia permite revisar la respuesta

**Persona 3 · 45 segundos**

La consulta de evidencia recuperó titulares reales de DataBursátil, incluidos los de la conferencia de Walmart México y su reporte trimestral. En esa consulta se resolvieron los identificadores y lotes del resultado sin faltantes. La comprobación también descubre límites: otro titular menciona BanBajío y aparece asociado a WALMEX, por lo que merece revisión. Para demostrar el módulo seguimos esta secuencia: cobertura, empresa disponible, indicadores y artículos de respaldo. Después consultamos una empresa inexistente. En la revisión devolvió not_found, sin fabricar sentimiento.

**Señalar:** Señalar los titulares y recorrer los pasos de la demo.

**Transición:** Terminamos distinguiendo lo comprobado de lo pendiente.

## 16. De titulares a información trazable

**Persona 3 · 45 segundos**

El aporte del módulo es integrar noticias, transformarlas en características cuantitativas y permitir consultas que conserven fechas y evidencia. En la revisión aprobaron cincuenta y siete pruebas de datos e indicadores y seis del panel. Dos casos stdio se excluyeron; las suites MCP y dominio no completaron sus límites y Spark tuvo un error de colección. Esos resultados y sus límites están documentados. Quedan por resolver la importación Spark, revisar asociaciones y validar la conversación con un modelo habilitado. La conclusión es que cada indicador puede examinarse desde sus datos de origen. Gracias.

**Señalar:** Señalar verificado, pendiente y la frase de cierre.

**Transición:** Abrir espacio para preguntas.

## Referencias para preguntas

Consultar `PREGUNTAS_DEFENSA_NOTICIAS.md` para fórmulas y decisiones, `GUIA_DEMO_NOTICIAS.md` para comandos y `REVISION_TECNICA_NOTICIAS.md` para verificaciones y límites. Las pruebas mencionadas corresponden a la revisión documentada; no se volvieron a ejecutar al cambiar el diseño.
