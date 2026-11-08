---
name: csharp-vertical-slice-cqrs
description: Contexto para proyectos desarrollados en C# y .NET. Reglas de arquitectura, CQRS y prevencion de antipatrones (servicios anémicos).
---
# C# Vertical Slice & CQRS

## Contexto
Proyectos desarrollados en C# y .NET.

## Reglas Principales

### Estructura Organizacional
Impone la agrupación de código por *Feature* (Vertical Slices) en lugar de utilizar capas técnicas globales y genéricas.

### Patrón CQRS
Exige la aplicación del patrón Command Query Responsibility Segregation utilizando librerías como MediatR, imponiendo una estricta separación de `Command` y `Query` de sus `Handler` respectivos.

### Prevención de Anti-patrones
Prohíbe la creación de servicios anémicos o controladores inflados. Se promueve firmemente el uso de Minimal APIs y Endpoints delgados (Thin Endpoints).