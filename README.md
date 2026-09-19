# Sports Team Manager

## 0. Propósito del documento

Este documento define la visión funcional general del sistema.

Es la fuente de verdad inicial para los handoffs posteriores y debe utilizarse como referencia antes de implementar cualquier módulo.

Los documentos posteriores definirán arquitectura, modelo de datos, UI, reglas de negocio e implementación.

---

# 1. Visión del producto

Sports Team Manager es una aplicación web para administrar equipos deportivos amateur.

El sistema debe permitir administrar múltiples equipos y múltiples temporadas por equipo.

La aplicación estará centrada inicialmente en fútbol, pero la arquitectura debería evitar dependencias innecesarias que impidan soportar otros deportes en el futuro.

Cada equipo tendrá acceso a cuatro módulos principales:

**Planilla**

**Formación**

**Estadísticas**

**Finanzas**

Toda la información operativa de estos módulos estará asociada a una temporada.

---

# 2. Conceptos principales

## Team

Representa un equipo deportivo.

Un usuario puede administrar uno o más equipos.

Ejemplo:

```text
Chilopi FC
```

---

## Season

Representa una temporada perteneciente a un equipo.

Ejemplo:

```text
Chilopi FC
├── Temporada 2026
├── Apertura 2027
└── Clausura 2027
```

Un equipo puede tener múltiples temporadas.

También podrá existir más de una temporada activa simultáneamente.

Cada temporada mantiene de forma independiente:

```text
Roster
Formation
Statistics
Finances
```

La aplicación siempre debe conocer:

```text
currentTeam
currentSeason
```

Todos los módulos trabajan sobre ese contexto.

---

# 3. Home

El Home será el punto de entrada a la aplicación.

Permitirá visualizar y seleccionar los equipos disponibles.

Al ingresar a un equipo se deberá seleccionar o utilizar una temporada activa.

Desde ese momento la aplicación funcionará bajo el contexto:

```text
Team
   ↓
Season
   ↓
Modules
```

El usuario podrá cambiar de temporada sin abandonar el equipo.

---

# 4. Navegación principal

Una vez seleccionado un equipo y una temporada existirán cuatro módulos principales.

```text
TEAM
│
├── Planilla
├── Formación
├── Estadísticas
└── Finanzas
```

El layout principal debería mantener siempre visible el equipo y la temporada actualmente seleccionados.

---

# 5. Planilla

La planilla representa los jugadores pertenecientes a una temporada.

Un jugador global puede participar en diferentes temporadas.

La pertenencia del jugador a una temporada debe tratarse independientemente de la existencia del jugador.

Ejemplo:

```text
Player
Nicolás Cristino
```

puede pertenecer a:

```text
Temporada 2026
Temporada 2027
```

sin necesidad de crear dos jugadores distintos.

## Funciones iniciales

La planilla permitirá:

* agregar jugadores;
* remover jugadores de la temporada;
* editar información del jugador;
* activar o desactivar jugadores;
* importar la planilla desde otra temporada.

Importar una planilla no debe modificar la temporada de origen.

Debe crear nuevas asociaciones entre los jugadores seleccionados y la temporada destino.

---

# 6. Formación

El módulo Formación será una pizarra táctica interactiva similar visualmente a los sistemas utilizados en juegos como FIFA / EA FC.

La cancha será el elemento principal de la pantalla.

Los jugadores pertenecientes a la planilla de la temporada podrán colocarse sobre la cancha.

## Modo desbloqueado

Cuando la formación se encuentra desbloqueada:

```text
formation.locked = false
```

el usuario puede arrastrar jugadores libremente por la cancha.

La posición no estará restringida a posiciones tácticas predefinidas.

Un jugador puede colocarse en cualquier coordenada válida dentro de la cancha.

Ejemplo conceptual:

```text
playerPosition
{
    playerId,
    x,
    y
}
```

Las coordenadas deberían almacenarse de manera relativa y no en píxeles.

Ejemplo:

```text
x: 0.52
y: 0.78
```

Esto permitirá que la formación sea responsive.

## Modo bloqueado

Cuando:

```text
formation.locked = true
```

los jugadores permanecen en sus posiciones actuales.

No pueden ser arrastrados accidentalmente.

## Side menu

El módulo tendrá un panel lateral.

Desde este panel se podrá acceder a:

```text
Jugadores disponibles

Formaciones predefinidas

4-3-3
4-4-2
4-2-3-1
3-5-2
etc.
```

Seleccionar una formación predefinida reposicionará automáticamente los jugadores.

Posteriormente el usuario podrá desbloquear la pizarra y personalizar las posiciones.

---

# 7. Estadísticas

El módulo de estadísticas comenzará deliberadamente simple.

La información será ingresada manualmente por los administradores del equipo.

No existirá inicialmente integración automática con plataformas deportivas.

## Primera versión

Se administrarán:

```text
Partidos
Goleadores
```

## Partido

Un partido deberá permitir almacenar inicialmente:

```text
fecha
rival
competición
local / visita
goles equipo
goles rival
estado
```

Estados posibles inicialmente:

```text
scheduled
played
cancelled
```

## Goleadores

Para cada partido jugado podrá registrarse qué jugadores marcaron goles.

La aplicación calculará automáticamente el ranking de goleadores de la temporada.

Ejemplo:

```text
1. Player A — 12
2. Player B — 8
3. Player C — 4
```

Las estadísticas deben calcularse desde los eventos registrados y no almacenarse innecesariamente como valores duplicados.

---

# 8. Finanzas

Finanzas será uno de los módulos con mayor flexibilidad del sistema.

Su objetivo principal será responder claramente:

```text
¿Cuánto dinero debería haber recaudado el equipo?

¿Cuánto dinero se ha pagado?

¿Cuánto dinero falta por pagar?
```

La precisión del total tiene prioridad sobre imponer reglas rígidas de cuotas.

---

# 9. Jugadores y cuotas

Los jugadores activos de una temporada podrán participar del cálculo financiero.

Sin embargo, no todos los jugadores están obligados a pagar.

Ejemplo:

```text
Player
financialStatus = exempt
```

Casos posibles:

```text
Jugador normal
Jugador parche
Jugador lesionado
Jugador suspendido temporalmente
Jugador exento
```

No debemos eliminar jugadores de la planilla solamente porque temporalmente no pagan.

La participación deportiva y la participación financiera son conceptos separados.

---

# 10. Estado financiero del jugador

La participación de un jugador en las cuotas debe poder activarse y desactivarse.

Ejemplo:

```text
financialActive = true
```

Si un jugador se lesiona:

```text
financialActive = false
```

El sistema deberá recalcular las cuotas futuras según las reglas financieras de la temporada.

Los pagos históricos nunca deben modificarse como consecuencia de este cambio.

---

# 11. Modelo flexible de cuotas

La aplicación no debe asumir exclusivamente:

```text
jugadores × cuota fija
```

El sistema debe estar preparado para diferentes modelos.

Ejemplos:

```text
Monto mensual fijo del equipo

Monto fijo por jugador

Gasto extraordinario dividido entre jugadores

Cuota especial

Pago único

Jugador exento

Jugador activo sólo parte del período
```

El objetivo será construir posteriormente un motor de cálculo simple pero extensible.

---

# 12. Pagos

El usuario registrará cuánto dinero entregó cada jugador.

Ejemplo:

```text
Player A
deuda: $20.000

pago registrado:
$10.000

saldo:
$10.000
```

Los pagos deben almacenarse como movimientos independientes.

Nunca se debe almacenar únicamente el saldo actual.

Ejemplo:

```text
payments

id
playerId
seasonId
amount
date
description
```

El saldo deberá calcularse desde:

```text
obligaciones - pagos
```

Esto permitirá mantener historial y auditoría.

---

# 13. Principio financiero fundamental

Los valores históricos no deben cambiar cuando cambian las condiciones actuales.

Ejemplo:

Un jugador estuvo activo durante enero y febrero.

Pagó:

```text
Enero
Febrero
```

En marzo se lesiona y deja de pagar.

La deuda de enero y febrero debe permanecer intacta.

Sólo deben cambiar las obligaciones posteriores al cambio.

Por esta razón el sistema financiero deberá trabajar eventualmente con períodos u obligaciones registradas.

---

# 14. Persistencia

La aplicación utilizará:

```text
Supabase
```

como backend persistente.

Supabase será responsable inicialmente de:

```text
PostgreSQL
Authentication
Database API
Storage cuando sea necesario
```

---

# 15. Desarrollo con mocks

Durante las primeras fases no es obligatorio conectarse inmediatamente a Supabase.

La arquitectura deberá permitir trabajar con datos mock.

Ejemplo:

```text
/data/mock/
    teams.json
    seasons.json
    players.json
    roster.json
    formations.json
    matches.json
    payments.json
```

La UI no debería acceder directamente a estos archivos.

Debe existir una capa intermedia.

Ejemplo:

```text
UI
 ↓
Services / Repository
 ↓
Data Source
```

Durante desarrollo:

```text
Data Source = JSON
```

Posteriormente:

```text
Data Source = Supabase
```

Esto debe permitir cambiar la fuente de datos sin reescribir los componentes.

---

# 16. Arquitectura tecnológica propuesta

Stack principal:

```text
Next.js
React
TypeScript
Tailwind CSS
Supabase
```

Herramientas complementarias podrán incorporarse según necesidad.

Probables candidatos:

```text
dnd-kit
Zod
React Hook Form
TanStack Query
Lucide
```

No deben agregarse dependencias hasta que exista una necesidad real.

---

# 17. Principios de desarrollo

La implementación deberá seguir estos principios:

```text
UI ≠ lógica de negocio

Componentes ≠ acceso directo a Supabase

Datos históricos ≠ estado actual

Player ≠ SeasonPlayer

Roster ≠ Player

Pago ≠ saldo

Formación ≠ posición fija

Temporada = contexto principal de los módulos
```

La lógica de negocio deberá mantenerse separada de los componentes visuales.

---

# 18. Entidades conceptuales iniciales

El modelo inicial probablemente requerirá conceptos equivalentes a:

```text
User

Team

Season

Player

SeasonPlayer

Formation

FormationPlayerPosition

Match

Goal

FinancialRule

FinancialPeriod

PlayerFinancialStatus

FinancialObligation

Payment
```

Estos nombres son conceptuales.

El modelo definitivo será diseñado en un handoff dedicado a base de datos y dominio.

---

# 19. Fuera de alcance inicial

La primera versión no buscará implementar todavía:

```text
Estadísticas deportivas avanzadas

Tracking GPS

Integración con federaciones

Integración bancaria

Pagos online

Chat interno

Notificaciones push

Marketplace

Scouting

Inteligencia artificial

Streaming

Gestión completa de torneos
```

Estas funcionalidades podrán evaluarse posteriormente.

---

# 20. Objetivo del MVP

El MVP debe permitir que una persona pueda:

```text
Crear un equipo

Crear una temporada

Agregar jugadores

Administrar la planilla

Crear una formación visual

Mover jugadores libremente

Guardar la formación

Registrar partidos

Registrar goleadores

Configurar jugadores que pagan o no pagan

Registrar pagos

Visualizar deuda individual

Visualizar deuda total

Visualizar dinero recaudado
```

Si estas funciones funcionan correctamente, tendremos la base sobre la cual desarrollar las siguientes versiones.

---

# 21. Estrategia de implementación

El proyecto será construido mediante handoffs independientes.

Cada handoff deberá poder entregarse a Codex como una unidad de trabajo.

Los handoffs deberán especificar:

```text
Objetivo

Contexto

Alcance

Reglas de negocio

Modelo involucrado

Componentes

Tareas

Criterios de aceptación

Checklist

Fuera de alcance
```

Codex debe implementar únicamente lo solicitado por el handoff actual.

No debe anticipar funcionalidades pertenecientes a handoffs futuros salvo que sea necesario establecer una interfaz o estructura mínima.

---

# 22. Roadmap de handoffs

La planificación detallada se dividirá posteriormente en documentos independientes.

Arquitectura general:

```text
00_PROJECT_OVERVIEW

01_PROJECT_ARCHITECTURE

02_DOMAIN_MODEL

03_DATABASE_DESIGN

04_APP_SHELL_AND_NAVIGATION

05_TEAMS

06_SEASONS

07_ROSTER

08_FORMATION

09_MATCHES

10_STATISTICS

11_FINANCE_DOMAIN

12_FINANCE_ENGINE

13_FINANCE_UI

14_SUPABASE_INTEGRATION

15_AUTHENTICATION

16_TESTING

17_MVP_POLISH
```

El orden exacto podrá ajustarse durante la planificación.

---

# 23. Regla principal del proyecto

Toda funcionalidad debe responder claramente a esta jerarquía:

```text
USER
 ↓
TEAM
 ↓
SEASON
 ↓
MODULE
 ↓
DATA
```

Cuando exista duda sobre dónde pertenece una información, esta jerarquía debe utilizarse como primera referencia.
