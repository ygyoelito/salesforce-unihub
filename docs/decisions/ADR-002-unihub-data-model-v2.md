# ADR-002 — Modelo de Datos UniHub v2

**Estado:** Aceptado
**Supersedes:** ADR-001 — Diseño conceptual inicial de UniHub  
**Fecha:** 2026-09-18  
**Proyecto:** UniHub  
**Alcance:** Semana 1 — Platform & Data Model  
**Sustituye a:** ADR-001 — Diseño conceptual inicial de UniHub

---

## 1. Contexto

UniHub es la aplicación práctica sobre Salesforce utilizada a lo largo del roadmap de mentoría de Salesforce Developer.

El modelo conceptual inicial contenía:

- `University__c`
- `Student__c`
- `Career__c`
- `Course__c`
- `Professor__c`

con las siguientes relaciones iniciales:

```text
University 1 ───── N Student
University 1 ───── N Career
Career     1 ───── N Student
Career     N ───── N Course
Professor  N ───── N Course
```

Durante la revisión conceptual de **Lookup vs Master-Detail**, el modelo fue reevaluado desde la perspectiva de:

- dependencia del ciclo de vida;
- eliminación en cascada;
- ownership;
- sharing;
- implicaciones de Roll-Up Summary;
- relaciones muchos-a-muchos;
- Junction Objects;
- integridad de datos;
- prevención de relaciones redundantes;
- mantener el proyecto de aprendizaje realista, pero intencionalmente limitado en alcance.

La revisión mostró que varias relaciones del modelo conceptual inicial eran demasiado simples para representar correctamente las reglas de negocio previstas para UniHub.

Este ADR establece la nueva baseline conceptual que se utilizará antes de implementar metadata de Salesforce.

---

## 2. Principio central de modelado

El tipo de relación debe derivarse de la semántica del negocio, no de la conveniencia técnica.

```text
N:N no implica automáticamente dos relaciones Master-Detail.
```

Para cada relación, se debe evaluar primero:

1. Dependencia del ciclo de vida.
2. Qué ocurre cuando se elimina un registro relacionado.
3. Si el hijo necesita ownership independiente.
4. Si el hijo necesita sharing independiente.
5. Si la propia relación contiene datos de negocio.

Solo después debe elegirse entre Lookup y Master-Detail.

---

## 3. Modelo conceptual final

```text
University__c
      1
      │ Master-Detail primario
      N
UniversityCareer__c
      N
      │ Master-Detail secundario
      1
Career__c

UniversityCareer__c
      1
      │ Lookup
      N
Student__c

UniversityCareer__c
      1
      │ Master-Detail primario
      N
UniversityCareerCourse__c
      N
      │ Master-Detail secundario
      1
Course__c

Professor__c
      1
      │ Master-Detail
      N
ProfessorUniversity__c
      N
      │ Master-Detail
      1
University__c

ProfessorUniversity__c
      1
      │ Master-Detail primario
      N
ProfessorUniversityCourse__c
      N
      │ Master-Detail secundario
      1
Course__c

ProfessorUniversity__c
      1
      │ Lookup desde UniversityCareer__c.CareerDirector
      N
UniversityCareer__c
```

> **Nota:** el primary master de `ProfessorUniversity__c` todavía no ha sido seleccionado explícitamente. Esta decisión se pospone de forma intencional hasta la implementación de metadata.

---

## 4. `University__c`

Representa una universidad como entidad de negocio independiente.

### Campos

| Campo | Significado |
|---|---|
| `Name` | Nombre de la universidad |
| `Address` | Dirección de la universidad |

### Significado de negocio

Una universidad puede:

- ofrecer múltiples carreras;
- compartir carreras genéricas con otras universidades;
- tener múltiples profesores;
- compartir profesores con otras universidades.

Por tanto, ni las carreras ni los profesores pertenecen directamente a una universidad mediante una relación simple 1:N.

---

## 5. `Career__c`

Representa una **carrera o programa académico genérico y reutilizable**.

Ejemplos:

```text
Ingeniería Informática
Ingeniería Industrial
Telecomunicaciones
Mecánica
```

### Campos

| Campo | Significado |
|---|---|
| `Name` | Nombre genérico de la carrera |

### Eliminado del modelo inicial

Los siguientes campos dejan de pertenecer directamente a `Career__c`:

```text
duration
university
```

Razón:

- una carrera puede ser ofrecida por múltiples universidades;
- la duración puede variar según la universidad;
- la oferta específica de una universidad se representa mediante `UniversityCareer__c`.

---

## 6. `UniversityCareer__c`

Representa:

> Una universidad concreta ofreciendo una carrera concreta.

Es el Junction Object de:

```text
University N ───── N Career
```

implementado conceptualmente como:

```text
University 1 ───── N UniversityCareer
Career     1 ───── N UniversityCareer
```

### Relaciones

```text
University → UniversityCareer = Master-Detail
Career     → UniversityCareer = Master-Detail
```

`University__c` se selecciona conceptualmente como **primary master**.

### Campos

| Campo | Significado |
|---|---|
| `University` | Universidad que ofrece la carrera |
| `Career` | Carrera genérica ofrecida |
| `Duration Years` | Duración de la carrera en esa universidad, expresada en años |
| `Active` | Indica si la universidad ofrece actualmente la carrera |
| `Start Date` | Fecha en que la universidad comenzó a ofrecer la carrera |
| `Career Director` | Director actual de esa oferta específica de carrera |

### Regla de integridad

La pareja debe ser única:

```text
UNIQUE(University, Career)
```

Solo puede existir un registro para una combinación específica universidad/carrera.

---

## 7. `Student__c`

Representa a un estudiante y únicamente su **situación académica actual**.

Los cambios académicos históricos quedan fuera del alcance actual.

### Campos

| Campo | Significado |
|---|---|
| `Name` | Nombre del estudiante |
| `Gender` | Género |
| `Address` | Dirección |
| `RUT` | Identificación |
| `Academic Year` | Año académico actual |
| `University Career` | Oferta universidad/carrera actual |

### Relación

```text
UniversityCareer 1 ───── N Student
                     Lookup
```

### Por qué Lookup

`Student__c`:

- tiene ciclo de vida independiente;
- no debe eliminarse si desaparece una oferta académica;
- necesita ownership independiente;
- puede necesitar reglas de sharing independientes de la oferta académica.

### Eliminado del modelo inicial

Los campos directos:

```text
Student.University
Student.Career
```

se eliminan conceptualmente.

La universidad y la carrera se obtienen mediante:

```text
Student
   │
   ▼
UniversityCareer
├── University
└── Career
```

Esto elimina el problema de integridad redundante por el cual `Student.University` y `Student.Career` podían quedar inconsistentes entre sí.

### Regla de año académico

Todos los valores de duración se expresan en años.

Debe cumplirse:

```text
1 <= Student.AcademicYear
     <= Student.UniversityCareer.DurationYears
```

---

## 8. `Course__c`

Representa una **asignatura genérica y reutilizable**.

Ejemplos:

```text
Matemáticas I
Programación I
Bases de Datos
Física
```

### Campos

| Campo | Significado |
|---|---|
| `Name` | Nombre genérico de la asignatura |

### Eliminado del modelo inicial

```text
number_hours
```

deja de pertenecer directamente a `Course__c`.

Razón:

El número de horas puede depender del plan de estudios concreto universidad/carrera en el que participe la asignatura.

---

## 9. `UniversityCareerCourse__c`

Representa:

> Una asignatura concreta como parte del plan de estudios de una carrera concreta en una universidad concreta.

Sustituye al modelo `CareerCourse__c` considerado previamente, porque los datos curriculares no dependen únicamente de Career + Course, sino de University + Career + Course.

### Relación representada

```text
UniversityCareer N ───── N Course
```

implementada como:

```text
UniversityCareer
        1
        │ Master-Detail primario
        N
UniversityCareerCourse
        N
        │ Master-Detail secundario
        1
Course
```

### Campos

| Campo | Significado |
|---|---|
| `University Career` | Oferta de carrera específica de una universidad |
| `Course` | Asignatura genérica |
| `Number Hours` | Número de horas dentro de este plan |
| `Credits` | Créditos otorgados dentro de este plan |
| `Mandatory / Optional` | Indica si la asignatura es obligatoria u optativa dentro de este plan |

### Regla de integridad

```text
UNIQUE(UniversityCareer, Course)
```

Una asignatura solo puede aparecer una vez dentro del mismo plan universidad/carrera.

### Decisión de alcance

El historial y versionado de planes de estudio todavía no se modelan.

Si una asignatura deja de formar parte del plan, se elimina su registro `UniversityCareerCourse__c`.

---

## 10. `Professor__c`

Representa a un profesor como persona, independientemente de las universidades donde trabaje.

### Campos

| Campo | Significado |
|---|---|
| `Name` | Nombre del profesor |
| `Gender` | Género |
| `RUT` | Identificación |

Un profesor puede pertenecer simultáneamente a múltiples universidades.

---

## 11. `ProfessorUniversity__c`

Representa:

> La relación actual entre un profesor y una universidad.

Materializa:

```text
Professor N ───── N University
```

### Relaciones

Ambos lados se modelan conceptualmente como Master-Detail porque:

- la asociación no tiene sentido sin el profesor;
- la asociación no tiene sentido sin la universidad;
- no necesita ownership independiente.

El primary master exacto se mantiene intencionalmente pendiente hasta la implementación de metadata.

### Campos

| Campo | Significado |
|---|---|
| `Professor` | Profesor |
| `University` | Universidad |
| `Active` | Indica si el profesor pertenece actualmente al claustro de la universidad |
| `Incorporation Date` | Fecha de la incorporación activa más reciente |
| `University Role` | Rol funcional dentro de la universidad |

### Valores de `University Role`

```text
Professor
Professor & Director
```

`Professor & Director` significa que la persona sigue siendo profesor y, adicionalmente, puede ejercer funciones directivas.

### Regla de integridad

```text
UNIQUE(Professor, University)
```

Se mantiene una única asociación lógica profesor/universidad.

Si un profesor abandona una universidad y posteriormente regresa, se reactiva el registro `ProfessorUniversity__c` existente en lugar de crear un duplicado.

`Incorporation Date` se actualiza con la fecha de incorporación más reciente.

---

## 12. Director de carrera

`UniversityCareer__c.CareerDirector` referencia a:

```text
ProfessorUniversity__c
```

y **no** directamente a `Professor__c`.

Razón:

El significado de negocio no es simplemente:

```text
Pedro
```

sino:

```text
Pedro @ Universidad A
```

### Cardinalidad

```text
ProfessorUniversity 1 ───── N UniversityCareer
```

Una relación profesor/universidad puede dirigir múltiples carreras.

Cada `UniversityCareer__c` tiene exactamente un director de carrera actual.

### Tipo de relación

```text
UniversityCareer.CareerDirector → ProfessorUniversity = Lookup
```

Es Lookup porque eliminar o desactivar una relación profesor/universidad no debe eliminar la propia oferta de carrera de la universidad.

### Relación obligatoria

`Career Director` es obligatorio.

### Reglas de integridad

Un director de carrera válido debe cumplir todas las condiciones siguientes:

```text
CareerDirector.University
=
UniversityCareer.University
```

```text
CareerDirector.Active = True
```

```text
CareerDirector.UniversityRole = "Professor & Director"
```

### Reglas de transición de estado

Un registro `ProfessorUniversity__c` no puede desactivarse mientras continúe siendo director de carrera de algún `UniversityCareer__c` activo.

`ProfessorUniversity__c.UniversityRole` no puede cambiar de:

```text
Professor & Director
```

a:

```text
Professor
```

mientras el registro continúe asignado como director de carrera de alguna oferta activa.

Primero deben reasignarse las relaciones de dirección de carrera afectadas.

---

## 13. `ProfessorUniversityCourse__c`

Representa:

> Un profesor, dentro de una universidad concreta, habilitado/asociado para impartir una asignatura genérica concreta.

La relación conceptual original:

```text
Professor N ───── N Course
```

se sustituye porque carecía de contexto universitario.

La relación revisada es:

```text
ProfessorUniversity N ───── N Course
```

implementada como:

```text
ProfessorUniversity
        1
        │ Master-Detail primario
        N
ProfessorUniversityCourse
        N
        │ Master-Detail secundario
        1
Course
```

### Campos

Para el alcance actual no se requieren campos de negocio adicionales, más allá de las dos relaciones.

### Significado de negocio

El registro significa:

```text
Este profesor, dentro de esta universidad,
está habilitado/asociado para impartir esta asignatura.
```

Todavía no representa una asignación docente concreta dentro de un semestre, sección, horario o aula.

---

## 14. Resumen de relaciones

| Desde | Hacia | Cardinalidad | Concepto de relación Salesforce |
|---|---|---:|---|
| `University__c` | `UniversityCareer__c` | 1:N | Master-Detail |
| `Career__c` | `UniversityCareer__c` | 1:N | Master-Detail |
| `UniversityCareer__c` | `Student__c` | 1:N | Lookup |
| `UniversityCareer__c` | `UniversityCareerCourse__c` | 1:N | Master-Detail primario |
| `Course__c` | `UniversityCareerCourse__c` | 1:N | Master-Detail secundario |
| `Professor__c` | `ProfessorUniversity__c` | 1:N | Master-Detail |
| `University__c` | `ProfessorUniversity__c` | 1:N | Master-Detail |
| `ProfessorUniversity__c` | `UniversityCareer__c` | 1:N | Lookup (`Career Director`) |
| `ProfessorUniversity__c` | `ProfessorUniversityCourse__c` | 1:N | Master-Detail primario |
| `Course__c` | `ProfessorUniversityCourse__c` | 1:N | Master-Detail secundario |

Relaciones muchos-a-muchos derivadas:

```text
University N:N Career
Professor N:N University
UniversityCareer N:N Course
ProfessorUniversity N:N Course
```

---

## 15. Junction / Associative Objects

Los objetos asociativos actuales son:

```text
UniversityCareer__c
ProfessorUniversity__c
UniversityCareerCourse__c
ProfessorUniversityCourse__c
```

Cada uno existe porque la relación subyacente es muchos-a-muchos y/o porque la relación posee significado o datos de negocio propios.

La elección de Master-Detail se realizó a partir del razonamiento sobre ciclo de vida y ownership, no simplemente por la cardinalidad N:N.

---

## 16. Decisiones sustituidas por este ADR

Las siguientes decisiones conceptuales iniciales quedan sustituidas:

| Modelo anterior | Modelo revisado |
|---|---|
| `University 1:N Career` | `University N:N Career` mediante `UniversityCareer__c` |
| `Career.duration` | `UniversityCareer.DurationYears` |
| `Career.university` | Eliminado |
| `Student.university` | Eliminado |
| `Student.career` | Eliminado |
| Sin campo de año académico | `Student.AcademicYear` |
| `Course.number_hours` | `UniversityCareerCourse.NumberHours` |
| `Career N:N Course` | `UniversityCareer N:N Course` |
| `CareerCourse__c` | Sustituido por `UniversityCareerCourse__c` |
| `Professor N:N Course` | Sustituido por `ProfessorUniversity N:N Course` |
| Sin relación profesor/universidad | `ProfessorUniversity__c` |
| Sin modelo explícito de director de carrera | `UniversityCareer.CareerDirector → ProfessorUniversity` |

---

## 17. Fuera de alcance de forma explícita

Para mantener UniHub alineado con los objetivos de aprendizaje de Salesforce Developer, se posponen intencionalmente:

```text
Historial académico del estudiante
Historial laboral del profesor
Versionado del plan de estudios
Períodos académicos / semestres
Secciones
Horarios
Aulas
Ejecuciones concretas de asignaturas
Asignación de un profesor a una ejecución concreta de una asignatura
Historial de directores de carrera
Historial curricular
Matrícula del estudiante en asignaturas específicas
Calificaciones
Asistencia
```

Un concepto futuro `Enrollment` continúa siendo válido para etapas posteriores, cuando UniHub necesite representar la matrícula de un estudiante en asignaturas concretas con atributos como:

```text
enrollment_date
semester
status
final_grade
```

---

## 18. Continuación hacia la implementación

Este ADR define únicamente la baseline conceptual.

La siguiente fase de implementación deberá traducir este modelo a metadata de Salesforce y decidir, objeto por objeto:

1. Tipos exactos de campos Salesforce.
2. Configuración obligatoria u opcional de las relaciones.
3. Orden de primary master donde todavía esté pendiente.
4. Implementación para prevenir duplicados en reglas de unicidad compuesta.
5. Validation Rules y/o Flow/Apex necesarios para la integridad entre registros.
6. Comportamiento de eliminación para relaciones Lookup.
7. Convenciones de nombres de registros.
8. API names definitivos de objetos y campos.
9. Configuración de seguridad, ownership y sharing.
10. Pruebas para los invariantes de negocio definidos.

No debe crearse metadata de Salesforce hasta que estas decisiones de implementación sean revisadas durante el proceso de mentoría.

---

## 19. Conclusiones de aprendizaje capturadas por esta decisión

El ejercicio de modelado estableció los siguientes principios sobre el modelo de datos de Salesforce:

- Lookup y Master-Detail pueden representar relaciones 1:N.
- La cardinalidad por sí sola no determina el tipo de relación.
- Master-Detail expresa dependencia de ciclo de vida, ownership y sharing.
- Lookup permite mayor independencia de ciclo de vida y seguridad.
- Una relación obligatoria puede seguir siendo Lookup.
- La disponibilidad de Roll-Up Summary es una consecuencia de Master-Detail, no una justificación suficiente para elegirlo.
- Las relaciones N:N se representan mediante un objeto asociativo.
- Un objeto asociativo no requiere automáticamente dos relaciones Master-Detail.
- Los atributos pertenecen a la entidad o relación que realmente determina su valor.
- Debe evitarse duplicar información que puede derivarse a través de una relación existente.
- Solo debe modelarse la complejidad requerida por el alcance actual del negocio.

---

## 20. Decisión

Este documento pasa a ser la **baseline conceptual del modelo de datos de UniHub v2**.

Las futuras implementaciones y ejercicios de aprendizaje deberán utilizar este modelo salvo que un ADR posterior sustituya explícitamente alguna parte.
