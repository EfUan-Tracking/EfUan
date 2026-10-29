# ADR-0001: Monolito modular y estructura inicial del backend

- Estado: Aceptado
- Fecha: 2026-10-29
- Decisores: Lucio

## Contexto

EfUan busca ser una fuente confiable y actualizada de estadísticas, estimaciones
y comparaciones de pilotos y equipos de Fórmula 1. El MVP debe importar datos de
OpenF1, persistirlos localmente y ofrecer estadísticas, comparaciones y
valoraciones explicables.

La arquitectura debe priorizar cohesión alta, bajo acoplamiento, seguridad al
integrar cambios, bajo costo operativo, legibilidad y un contexto fácil de
recuperar para personas y agentes de IA. Debe poder crecer hacia capacidades
sociales, búsqueda, visualizaciones compartibles o simulación sin incorporarlas
prematuramente.

## Decisión

El backend será un **monolito modular** en C#/.NET. Se desplegará como una sola
aplicación y podrá usar inicialmente una sola base PostgreSQL, pero las
capacidades de negocio conservarán dueños de datos y dependencias explícitas.

Cada módulo se compila inicialmente como un único proyecto .csproj. Las
carpetas Domain, Application, Infrastructure y Contracts, cuando sean
necesarias, son límites organizativos internos y no proyectos separados. Esto
mantiene cohesionada cada capacidad durante el MVP y evita multiplicar proyectos
vacíos.

La solución se llama EfUan.slnx; el SDK se fija en global.json. Las
configuraciones comunes de compilación y estilo viven en Directory.Build.props
y .editorconfig en la raíz.

### Estructura actual

~~~
backend/src/
  Api/
    Api.csproj
  Abstractions/
    Abstractions.csproj
  Modules/
    Catalog/
      EfUan.Catalog.csproj
    RaceData/
      EfUan.RaceData.csproj
    Statistics/
      EfUan.Statistics.csproj
    Comparison/
      EfUan.Comparison.csproj
    Performance/
      EfUan.Performance.csproj
~~~

Api es el host de ASP.NET Core y el punto de composición: recibe solicitudes
HTTP, configura middleware, autenticación, autorización, observabilidad y
dependencias, y publica endpoints. No contiene reglas de negocio, integración
con OpenF1 ni acceso directo a datos de los módulos.

Abstractions queda reservado para contratos técnicos mínimos realmente
compartidos, por ejemplo Result, IClock, ICurrentUser o contratos de eventos.
No puede contener entidades ni reglas de negocio de Fórmula 1.

### Límites y propiedad de datos

| Módulo | Responsabilidad y datos de los que es dueño |
| --- | --- |
| Catalog | Datos de referencia: pilotos, equipos/constructores, monoplazas o entradas por temporada, circuitos, temporadas, grandes premios, sesiones y participación piloto-equipo-temporada. |
| RaceData | Hechos deportivos obtenidos de fuentes externas: resultados, posiciones, vueltas, tiempos, stints, paradas, prácticas y clasificación. |
| Statistics | Métricas descriptivas: ritmo de carrera y qualy, gaps, consistencia, posiciones medias y degradación. |
| Comparison | Comparaciones normalizadas entre pilotos/equipos: H2H, compañeros comunes y cruces indirectos; reduce sesgos de coche, pista, temporada y contexto. |
| Performance | Valoraciones y ratings explicables. Decide la ponderación de estadísticas y conclusiones comparativas; no define el algoritmo comparativo. |

La cadena de dependencias de módulos es unidireccional:

~~~
Catalog <- RaceData <- Statistics <- Comparison <- Performance
~~~

RaceData puede usar capacidades públicas de Catalog; Statistics puede usar
RaceData; y así sucesivamente. Ningún módulo puede referenciar a un módulo
situado a su derecha ni a Api. Api puede referenciar los módulos para exponer
sus capacidades HTTP.

Las referencias se declaran mediante ProjectReference en los .csproj. Si un
requisito necesita una dirección nueva, se debe evaluar un contrato publicado,
una proyección de lectura o un ADR nuevo antes de añadir la dependencia.

### Persistencia, consultas y sincronización

Todos los módulos pueden persistir en la misma instancia PostgreSQL durante el
MVP, pero cada módulo es dueño de sus tablas, migraciones y repositorios. Un
módulo no accede directamente a tablas, entidades de persistencia o repositorios
internos de otro módulo.

Ejemplo: cuando Performance necesita todos los pilotos de una temporada,
solicita una proyección pública a Catalog; Catalog consulta sus tablas y
devuelve el resultado. Performance no consulta las tablas de pilotos por su
cuenta.

Los datos de OpenF1 se sincronizan de forma controlada y se sirven desde la
copia local, no mediante una llamada remota en cada petición HTTP:

~~~
OpenF1 -> sincronización del módulo dueño -> PostgreSQL local -> contratos públicos
~~~

Catalog sincroniza datos de referencia. RaceData sincroniza resultados y datos
de sesión. Las proyecciones pueden incluir SyncedAt para informar su frescura.
La frecuencia de actualización depende del tipo de dato: los perfiles históricos
cambian poco, mientras que sesiones y resultados se actualizan durante un fin de
semana de carrera.

Una actualización manual de administrador o una estrategia
stale-while-revalidate son opciones futuras, pero no deben bloquear una lectura
normal esperando a OpenF1.

## Consecuencias

### Positivas

- Cada capacidad principal tiene una ubicación y dueño claros.
- Se evitan llamadas duplicadas a OpenF1 y el acoplamiento al modelo externo.
- Comparison puede evolucionar independientemente de la fórmula de ratings, y
  Performance usa sus resultados como factor de valoración.
- La compilación impide referencias circulares entre los módulos actuales.
- El sistema conserva un despliegue y un costo operativo de monolito, sin
  impedir futuras extracciones si surge una necesidad real.

### Costos y restricciones

- Los límites se mantienen por contratos y disciplina de propiedad, aun cuando
  PostgreSQL sea compartida.
- Las fórmulas de comparación y rendimiento deben versionarse y explicarse para
  que las valoraciones sean reproducibles.
- Un proyecto único por capacidad no impone en compilación sus capas internas.
  Si el tamaño, equipos o dependencias externas lo justifican, un ADR futuro
  podrá dividir un módulo en proyectos internos.

## Alternativas consideradas

### Capas globales para todo el backend

Descartada para el MVP. Distribuye una funcionalidad entre muchos proyectos y
hace más difícil encontrar contexto y aislar fallos por capacidad de negocio.

### Un proyecto por cada capa interna de cada módulo desde el inicio

Pospuesta. Aporta límites de compilación más fuertes, pero multiplica
referencias, configuración y proyectos sin funcionalidad.

### Un Catalog que posee todos los datos, incluidos resultados

Descartada. Los resultados y tiempos son hechos deportivos y pertenecen a
RaceData. Mezclarlos con datos de referencia haría que Catalog creciera sin
límite.

### Consultar OpenF1 en cada lectura

Descartada. Añade latencia, fragilidad, límites de uso de la API externa y
resultados no reproducibles. La aplicación trabaja sobre datos locales
sincronizados.

### Crear módulos futuros desde el inicio

Descartada. Profiles/Identity, Social, Search, Forums, Sharing y Simulation se
crearán sólo cuando una funcionalidad real defina su límite, datos y contratos.

