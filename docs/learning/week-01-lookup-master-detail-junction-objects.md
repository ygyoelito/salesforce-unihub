# Week 1 — Lookup, Master-Detail y Junction Objects

## Objetivo

Este documento resume los conceptos aprendidos sobre relaciones en Salesforce durante la Week 1.

El objetivo no es documentar decisiones concretas del proyecto UniHub, sino conservar el modelo mental general necesario para diseñar relaciones correctamente en Salesforce.

---

## 1. Lookup vs Master-Detail

Tanto **Lookup** como **Master-Detail** permiten relacionar registros.

La diferencia principal no está en la cardinalidad, sino en el **grado de dependencia entre los registros**.

Una relación puede ser `1:N` tanto con Lookup como con Master-Detail.

La pregunta principal debe ser:

> ¿El registro hijo tiene identidad y ciclo de vida propios, o su existencia depende conceptualmente del padre?

---

## 2. Lookup Relationship

Lookup representa una relación entre registros que conservan mayor independencia.

Modelo mental:

```text
A ───── Lookup ─────> B

A está relacionado con B,
pero ambos conservan identidad propia.
```

Características principales:

- El registro relacionado puede tener un ciclo de vida independiente.
- El hijo puede conservar ownership independiente.
- El sharing puede gestionarse de forma independiente.
- La desaparición del registro padre no implica necesariamente la eliminación del hijo.
- La relación puede ser obligatoria sin convertirse por ello en Master-Detail.
- Los Roll-Up Summary Fields estándar no están disponibles directamente sobre relaciones Lookup.

Lookup es una buena opción cuando el hijo sigue teniendo valor de negocio aunque desaparezca o cambie el padre.

---

## 3. Master-Detail Relationship

Master-Detail representa una dependencia estructural fuerte.

Modelo mental:

```text
MASTER
   │
   └──── DETAIL
```

El Detail forma parte conceptualmente del Master.

Características principales:

- El Detail depende del Master para su existencia.
- El Detail no tiene ownership independiente.
- El ownership está controlado por el Master.
- El sharing del Detail está controlado por el Master.
- La eliminación del Master elimina también sus Details.
- Permite Roll-Up Summary Fields declarativos estándar.

Master-Detail es apropiado cuando el hijo pierde su significado de negocio si desaparece el padre.

---

## 4. Ciclo de vida

El ciclo de vida es uno de los criterios más importantes para decidir entre Lookup y Master-Detail.

### Independencia

Si el hijo puede continuar existiendo aunque desaparezca el padre:

```text
Padre eliminado
      ↓
Hijo sigue teniendo sentido
```

la relación apunta conceptualmente hacia **Lookup**.

### Dependencia fuerte

Si el hijo deja de tener sentido sin el padre:

```text
Padre eliminado
      ↓
Hijo pierde significado
```

la relación apunta conceptualmente hacia **Master-Detail**.

---

## 5. Eliminación

No debe elegirse Master-Detail simplemente porque se desee una relación "más fuerte".

La eliminación en cascada debe reflejar una regla real del negocio.

Regla mental:

> No debería existir eliminación en cascada cuando el supuesto hijo conserva significado y valor de negocio independientemente del padre.

Un hijo puede estar relacionado con otros objetos y aun así ser Detail. Lo importante es determinar si su existencia sigue teniendo sentido sin el Master.

---

## 6. Ownership

En una relación Lookup, el registro hijo puede mantener ownership independiente.

```text
Lookup
Parent Owner ≠ Child Owner
```

En Master-Detail, el Detail no mantiene ownership propio independiente:

```text
Master Owner
     ↓
Detail
```

Por tanto, una pregunta útil durante el diseño es:

> ¿Tiene sentido que el hijo tenga un propietario distinto del padre?

Si la respuesta es sí, Lookup suele ser más coherente.

Si la respuesta es no porque el hijo forma parte estructuralmente del padre, Master-Detail puede ser más apropiado.

---

## 7. Sharing

Ownership y sharing están relacionados.

### Lookup

El registro relacionado puede tener reglas de acceso propias.

```text
Parent
  └── seguridad propia

Child
  └── seguridad propia
```

### Master-Detail

El acceso al Detail queda subordinado al Master.

```text
MASTER
   ↓
controla acceso
   ↓
DETAIL
```

Si un usuario no debería poder acceder al hijo sin tener acceso al padre, esto es una señal a favor de Master-Detail.

---

## 8. Roll-Up Summary

Los Roll-Up Summary Fields permiten que un Master agregue información proveniente de sus Details.

Operaciones comunes:

```text
COUNT
SUM
MIN
MAX
```

Ejemplo conceptual:

```text
Invoice
├── Line = 100
├── Line = 80
└── Line = 25

Invoice.Total = 205
```

Una idea importante:

> Roll-Up Summary es una consecuencia útil de Master-Detail, no una razón suficiente por sí sola para elegir Master-Detail.

No debe deformarse el modelo de negocio únicamente para obtener un Roll-Up Summary.

Si la relación correcta es Lookup, la agregación debe resolverse mediante otro mecanismo.

---

## 9. Junction Objects

Salesforce utiliza normalmente un **Junction Object** para representar relaciones muchos-a-muchos.

Una relación conceptual:

```text
A N ───── N B
```

puede representarse mediante:

```text
A
1
│
N
Junction
N
│
1
B
```

Es decir, una relación `N:N` se transforma en dos relaciones `1:N`.

El Junction Object también puede contener datos propios de la relación.

Ejemplo conceptual:

```text
Student
       Membership
   /
Club
```

`Membership` podría contener:

```text
join_date
status
membership_number
```

---

## 10. N:N no implica automáticamente dos Master-Detail

Una relación muchos-a-muchos **no obliga** a utilizar dos relaciones Master-Detail.

Este es un principio importante:

```text
N:N ≠ dos Master-Detail obligatoriamente
```

Primero debe analizarse el ciclo de vida del objeto intermedio respecto a cada extremo.

Puede existir:

```text
A
│ Master-Detail
│
Association
│
│ Lookup
B
```

o incluso:

```text
A
│ Lookup
│
Association
│
│ Lookup
B
```

La decisión depende de:

- ciclo de vida;
- eliminación;
- ownership;
- sharing;
- necesidad de conservar histórico;
- valor de negocio independiente.

La cardinalidad y el tipo de relación son decisiones distintas.

---

## 11. Primary Master y Secondary Master

En un Junction Object construido con dos relaciones Master-Detail, una relación queda como **Primary Master** y la otra como **Secondary Master**.

Ambas siguen siendo necesarias para representar la dependencia del Junction Object, pero no desempeñan exactamente el mismo papel en Salesforce.

La elección del Primary Master debe realizarse considerando cuál de los dos Masters representa mejor el **contexto principal** del Junction Object.

Ejemplo conceptual:

```text
Career
   │ Primary Master
   ▼
CareerCourse
   ▲
   │ Secondary Master
Course
```

Cambiar cuál es Primary Master no cambia la cardinalidad `N:N` representada.

---

## 12. Método de decisión

Antes de elegir Lookup o Master-Detail conviene responder estas preguntas:

```text
1. ¿El hijo puede existir sin el padre?
2. ¿Debe eliminarse el hijo si desaparece el padre?
3. ¿Necesita ownership independiente?
4. ¿Necesita sharing independiente?
5. ¿Debe conservarse como histórico?
6. ¿Tiene valor de negocio por sí mismo?
7. ¿Necesitamos Roll-Up Summary?
```

Las primeras preguntas relacionadas con ciclo de vida, ownership y sharing deben pesar más que la comodidad técnica.

---

## 13. Regla de diseño aprendida

La relación debe decidirse a partir del significado del negocio.

No debe elegirse:

```text
Master-Detail
```

solo porque:

```text
- permita Roll-Up Summary;
- parezca una relación más fuerte;
- sea el patrón típico de un Junction Object.
```

Ni debe elegirse:

```text
Lookup
```

simplemente porque sea más flexible.

La secuencia correcta de razonamiento es:

```text
Modelo de negocio
      ↓
Ciclo de vida
      ↓
Eliminación
      ↓
Ownership
      ↓
Sharing
      ↓
Lookup / Master-Detail
```

---

## 14. Idea central

La pregunta más útil para distinguir ambas relaciones es:

> ¿El hijo es una entidad independiente relacionada con el padre, o forma parte estructuralmente de él?

Modelo mental final:

```text
LOOKUP
"Estos registros están relacionados,
pero conservan identidad propia."

MASTER-DETAIL
"Este Detail forma parte del Master
y su existencia está subordinada a él."
```

---

## 15. Principio aprendido sobre modelado

No debe introducirse complejidad solamente porque sea posible.

Una relación, objeto o atributo debe existir porque responde a una necesidad real del negocio.

```text
Comprender el negocio
        ↓
Modelar
        ↓
Elegir la relación Salesforce
        ↓
Implementar
```

Esto evita diseñar el modelo alrededor de las capacidades técnicas de Salesforce en lugar de diseñarlo alrededor del dominio que la aplicación debe representar.
