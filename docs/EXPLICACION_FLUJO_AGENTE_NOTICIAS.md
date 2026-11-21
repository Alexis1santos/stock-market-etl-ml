# Cómo funciona la consulta del agente de Noticias

Cuando una persona hace una pregunta sobre noticias, el agente no lee los
archivos de datos directamente. El modelo decide si necesita una herramienta
(tool) y cuál usar según la pregunta y las descripciones de las herramientas
disponibles.

## Flujo general

```text
Pregunta del usuario
        ↓
Agente de Noticias analiza la pregunta
        ↓
Elige una tool y genera sus parámetros
        ↓
La tool llama a la capa semántica
        ↓
La capa semántica consulta Gold o Silver
        ↓
El resultado vuelve al agente
        ↓
El agente redacta la respuesta
```

## Ejemplo: consultar métricas

Ante la pregunta:

> ¿Cuál es el sentimiento de WALMEX?

El agente puede elegir `query_noticias` y pasarle parámetros como:

```json
{
  "ticker": "WALMEX"
}
```

La tool pasa esos parámetros a `NewsDomainQueries`, que valida la consulta y
llama a `NewsSemanticService`. El servicio busca las métricas de WALMEX en
Gold y devuelve información estructurada, como el sentimiento, la fecha del
cálculo, las fuentes y las advertencias. La tool devuelve ese resultado al
agente para que responda en lenguaje natural.

## Ejemplo: consultar titulares como evidencia

Si la persona también pregunta qué titulares respaldan el resultado, el agente
puede elegir `evidencia_noticias`. Esa tool llama al servicio semántico para
buscar en Silver los artículos asociados al resultado, respetando la fecha y
el corte temporal solicitados.

Por tanto, una consulta de métricas no implica necesariamente consultar
Silver:

- **Gold** se consulta para obtener las métricas calculadas.
- **Silver** se consulta cuando se solicitan titulares o evidencia.

## Quién hace cada parte

- **Agente/modelo:** interpreta la pregunta y decide qué tool usar.
- **Tool MCP:** recibe los parámetros del agente y los pasa al código de
  consultas.
- **`NewsDomainQueries`:** adapta y valida los parámetros de las tools.
- **`NewsSemanticService`:** aplica las reglas semánticas y consulta los
  datasets correspondientes.
- **Gold y Silver:** contienen, respectivamente, métricas calculadas y
  observaciones de artículos que pueden servir como evidencia.

En una frase:

> El agente decide qué herramienta necesita; la herramienta llama a la capa
> semántica; y la capa semántica obtiene las métricas de Gold o la evidencia de
> Silver y devuelve el resultado al agente.
