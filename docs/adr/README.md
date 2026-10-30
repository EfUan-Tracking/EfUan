# Architectural Decision Records

Este directorio conserva las decisiones arquitectónicas relevantes de EfUan en
formato MADR. Los ADRs son la fuente de contexto para cambios técnicos, tickets
y agentes de IA.

## Convención

- Nombrar cada archivo como NNNN-titulo-en-kebab-case.md.
- No reescribir una decisión aceptada para cambiar su significado. Crear un ADR
  nuevo que la reemplace y marcar el anterior como Superseded.
- Un ADR debe explicar el contexto, la decisión, las consecuencias y las
  alternativas descartadas.
- Todo ticket que modifique límites de módulos, dependencias, persistencia o
  integración externa debe enlazar el ADR aplicable y actualizarlo o proponer
  uno nuevo si cambia una decisión.
- Un cambio de código no debe contradecir un ADR aceptado sin una decisión
  posterior explícita.

## Índice

- [0001 - Monolito modular y estructura inicial del backend](0001-monolito-modular-y-estructura-inicial-del-backend.md)
- [0002 - Arquitectura y configuración inicial del frontend](0002-arquitectura-y-configuracion-inicial-del-frontend.md)

