# ADR-0002: Arquitectura y configuración inicial del frontend

- Estado: Aceptado
- Fecha: 2026-10-30
- Decisores: Equipo de EfUan

## Contexto

El frontend de EfUan debe presentar estadísticas, comparaciones y valoraciones de
Fórmula 1 de forma rápida, responsive y fácil de evolucionar. Debe favorecer la
traducción de diseños desde Figma, la reutilización de componentes, el tipado
estricto y una estructura cuyo contexto sea fácil de encontrar por personas y
agentes de IA.

El backend de EfUan es un monolito modular en .NET. El frontend no debe duplicar
sus reglas de negocio, cálculos de rendimiento ni sincronización de OpenF1.

## Decisión

Se inicializa el frontend como una aplicación de Next.js con App Router, React,
TypeScript, Tailwind CSS, ESLint y Prettier.

La aplicación se ubica en el directorio frontend y usa npm como gestor de
paquetes. El runtime mínimo es Node.js 20.9; se recomienda la versión fijada en
.nvmrc para el desarrollo local reproducible.

### Next.js y Turbopack

Se adopta el App Router de Next.js. Las rutas se definen por archivos y
carpetas bajo src/app. El archivo layout.tsx raíz contiene el HTML común, y cada
page.tsx representa una pantalla/ruta.

Next.js 16 usa Turbopack por defecto tanto para next dev como para next build.
Por ello, los scripts no incluyen una bandera --turbopack explícita ni existe
una configuración personalizada de Turbopack al inicio:

~~~json
{
  "dev": "next dev",
  "build": "next build",
  "start": "next start"
}
~~~

next.config.ts se mantiene mínimo hasta que exista una necesidad concreta de
configuración del framework. No se agrega una configuración de Webpack ni se
cambia de bundler preventivamente. Si una dependencia exige Webpack, se
evaluará como excepción documentada antes de usar next dev --webpack o
next build --webpack.

### Estructura de código

~~~text
frontend/
  src/
    app/          Rutas, layouts y UI propia del ciclo de navegación
    components/   Componentes de presentación reutilizables
    modules/      Capacidades funcionales del producto
    hooks/        Hooks reutilizables de React
    services/     Clientes tipados hacia la API .NET
    lib/          Utilidades puras y agnósticas del framework
    types/        Tipos compartidos entre módulos
  public/         Recursos estáticos
~~~

app compone rutas y módulos; no debe contener reglas de negocio ni repetir
lógica HTTP. Las capacidades de producto se crearán dentro de modules sólo al
existir un caso de uso real. Por ejemplo, se podrá crear
modules/comparison cuando se implemente la comparación de pilotos, pero no se
crean subcarpetas de features vacías durante la inicialización.

components contiene UI transversal, por ejemplo tablas, tarjetas, gráficos,
botones y layouts visuales. Un componente específico de una capacidad debe
permanecer cerca de esa capacidad en modules, no moverse a components sólo por
ser un componente de React.

El alias @/ resuelve a src/ y debe utilizarse para imports internos. Esto evita
rutas relativas extensas y mejora la legibilidad al trabajar entre módulos.

### Renderizado y frontera con el backend

Los componentes de App Router son Server Components por defecto. Se utilizarán
para estructura de páginas y obtención inicial de datos cuando corresponda. Un
componente se marca con "use client" únicamente si necesita interacción,
estado local, efectos del navegador o APIs del navegador, por ejemplo filtros,
tabs, tooltips o gráficos interactivos.

El frontend consume las capacidades publicadas por la API .NET. No calcula
ratings, comparaciones, resultados deportivos ni sincroniza OpenF1. Los
clientes HTTP centralizados viven en services y los tipos de respuesta se
mantienen en el módulo propietario o, cuando son transversales, en types.

Las variables que llegan al navegador deben usar el prefijo NEXT_PUBLIC_. Las
credenciales, secretos de servidor y cadenas de conexión no pueden estar en
variables públicas del frontend.

### Estilos y calidad

Tailwind CSS es el sistema de estilos inicial. Favorece la implementación
rápida de diseños responsive y la traducción de Figma a componentes sin crear
un CSS global por cada pantalla. Los estilos globales deben limitarse a tokens,
reset, tipografía y reglas base; los estilos de componentes viven junto a sus
componentes mediante utilidades de Tailwind.

ESLint valida problemas de código y convenciones. Prettier es el formateador
único. Sus scripts son:

~~~json
{
  "lint": "eslint .",
  "typecheck": "tsc --noEmit",
  "format": "prettier --write .",
  "format:check": "prettier --check ."
}
~~~

Antes de integrar cambios se deben ejecutar lint, typecheck, format:check y
build. Los ajustes concretos de formato viven en prettier.config.mjs; los
artefactos generados y el lockfile están excluidos en .prettierignore.

No se incorporan inicialmente bibliotecas de componentes, gráficos, manejo de
estado global ni data fetching. Se elegirán cuando el primer caso de uso real
permita comparar requisitos de accesibilidad, SSR, caché, tamaño de bundle y
mantenimiento.

## Consecuencias

### Positivas

- La aplicación tiene rutas claras, tipado estricto y un build de producción
  integrado.
- Turbopack reduce el tiempo de feedback durante desarrollo sin configuración
  adicional.
- La estructura por módulos agrupa el contexto funcional y simplifica el paso
  de Figma a frontend.
- El frontend conserva una frontera nítida con el backend .NET.
- Los scripts de lint, tipos y formato permiten llevar controles equivalentes a
  la calidad configurada en el backend.

### Costos y restricciones

- App Router requiere distinguir con criterio los componentes de servidor de
  los componentes cliente.
- La decisión de librerías de gráficas, componentes o datos se pospone; cada
  primera incorporación debe justificar su necesidad.
- Una configuración personalizada de bundler puede reducir la compatibilidad
  con Turbopack y sólo se introducirá cuando sea imprescindible.

## Alternativas consideradas

### React sin framework

Descartada. Requeriría escoger e integrar manualmente routing, renderizado,
optimización, build y convenciones de aplicación.

### Pages Router

Descartada para código nuevo. App Router se alinea con las capacidades actuales
de React y con el modelo de layouts, Server Components y rutas basado en
archivos.

### Webpack como bundler inicial

Descartada. Turbopack es el bundler predeterminado de Next.js 16 y cubre el
caso de uso inicial. Webpack queda como mecanismo de compatibilidad ante una
restricción demostrada.

### Crear desde el inicio módulos vacíos por cada pantalla

Descartada. Las carpetas de primer nivel establecen la convención; los módulos
de producto aparecen cuando exista una funcionalidad concreta que los justifique.

