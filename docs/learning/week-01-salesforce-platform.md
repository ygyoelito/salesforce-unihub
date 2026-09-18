# Week 01 — Salesforce Platform & Data Model

## Objetivo

Construir el mapa mental fundamental de Salesforce antes de comenzar a implementar el modelo de datos de UniHub.

El objetivo de esta etapa no es memorizar definiciones aisladas, sino comprender cómo se relacionan los principales conceptos de la plataforma.

---

# 1. Org

Una **Org (Organization)** puede entenderse como un entorno Salesforce aislado.

Una Org tiene sus propios:

- Usuarios.
- Datos.
- Configuración.
- Metadata.
- Reglas de acceso.
- Aplicaciones.
- Automatizaciones.
- Personalizaciones.

### Analogía con Odoo

Conceptualmente puede compararse con una instancia de Odoo.

> La analogía sirve para facilitar el aprendizaje, pero no implica equivalencia técnica exacta.

---

# 2. Object

Un **Object** representa una entidad sobre la que Salesforce puede almacenar y gestionar información.

Puede ser:

- **Standard Object**: proporcionado por Salesforce.
- **Custom Object**: creado para una necesidad específica del negocio.

Ejemplos de objetos estándar:

```text
Account
Contact
Lead
Opportunity
Case
User
```

Ejemplos de objetos que utilizaremos en UniHub:

```text
University__c
Student__c
Professor__c
Course__c
Enrollment__c
```

### Analogía con Odoo

Conceptualmente:

```text
Odoo Model
    ≈
Salesforce Object
```

No son equivalentes técnicamente.

---

# 3. Record

Un **Record** es una instancia concreta de un Object.

Si:

```text
Student__c
```

es la definición del objeto, entonces:

```text
Student:
    Name: Yoel Gonzalez
    Email: yoel@example.com
```

representa un registro concreto de ese objeto.

### Analogía con programación

```text
Class
   ↓
Object / Instance
```

Conceptualmente:

```text
Object
   ↓
Record
```

### Analogía con Odoo

```text
Odoo Record
    ≈
Salesforce Record
```

---

# 4. Field

Un **Field** representa una propiedad o atributo de un Object.

Por ejemplo:

```text
Student__c
├── Name
├── Email
├── BirthDate
└── University
```

Cada Field define qué información puede almacenarse en un Record.

Los campos pueden tener diferentes tipos:

```text
Text
Number
Currency
Date
Date/Time
Checkbox
Picklist
Email
Phone
Formula
Relationship
```

---

# 5. Data vs Metadata

Una distinción fundamental en Salesforce es separar **Data** de **Metadata**.

## Data

Es la información real almacenada en los registros.

Ejemplo:

```text
Student:
    Name  = Yoel Gonzalez
    Email = yoel@example.com
```

Eso es **Data**.

## Metadata

Es la definición de cómo está construido el sistema.

Ejemplo:

> `Student__c` debe tener un campo `Email` de tipo Email.

Eso es **Metadata**.

Otro ejemplo:

> `Student__c` tiene una relación con `University__c`.

También es Metadata.

### Regla mental

```text
METADATA
    ↓
Define la estructura

DATA
    ↓
Representa los valores reales
```

O de forma más general:

```text
SALESFORCE ORG
│
├── METADATA
│   ├── Objects
│   ├── Fields
│   ├── Relationships
│   ├── Apps
│   └── Configuración
│
└── DATA
    └── Records
```

---

# 6. Schema

El **Schema** representa la estructura de nuestro modelo de datos.

Podemos pensar en él como la **“osamenta” del proyecto**.

Define conceptualmente:

- Qué entidades existen.
- Qué información contiene cada entidad.
- Qué campos tiene cada entidad.
- Cómo se relacionan las entidades.
- Qué dependencias existen entre ellas.

Por ejemplo:

```text
University__c
       │
       │ relationship
       ▼
Student__c
       │
       │ relationship
       ▼
Enrollment__c
```

El schema nos permite entender la estructura global de los datos sin necesidad de observar cada registro individual.

### Distinción importante

```text
Metadata
    ↓
Describe / define la estructura de Salesforce

Schema
    ↓
Representa la estructura y relaciones del modelo de datos
```

El Schema forma parte de la visión estructural que construimos a partir de la Metadata.

---

# 7. Standard Objects vs Custom Objects

Salesforce proporciona objetos estándar para necesidades comunes.

Ejemplos:

```text
Account
Contact
Lead
Opportunity
Case
```

Cuando el dominio de negocio requiere una entidad específica que no está representada adecuadamente por un objeto estándar, podemos crear un **Custom Object**.

Para UniHub:

```text
University__c
Student__c
Professor__c
Course__c
Enrollment__c
```

El sufijo:

```text
__c
```

identifica convencionalmente un objeto personalizado.

### Decisión de diseño

No debemos adoptar la regla:

> “Si no existe un objeto estándar exactamente igual a mi entidad, creo un Custom Object.”

Antes debemos considerar:

- Funcionalidad existente.
- Relaciones.
- Seguridad.
- Automatización.
- Reporting.
- Mantenimiento.
- Requisitos reales del negocio.

---

# 8. Relationships

Una relación permite conectar registros de diferentes objetos.

Por ejemplo:

```text
University__c
       │
       │
       ▼
Student__c
```

Si un estudiante tiene un campo:

```text
University
```

ese campo puede utilizarse para relacionar un `Student__c` con una `University__c` concreta.

### Punto conceptual importante

El campo de relación no apunta a la definición del objeto como tal.

Apunta a un **registro concreto**.

Ejemplo:

```text
University__c

Id: U001
Name: Universidad de Valparaíso
```

y:

```text
Student__c

Name: Yoel
Email: yoel@example.com
University: U001
```

Por tanto:

> El Object define la estructura; el campo de relación permite conectar un registro con otro registro.

---

# 9. Lookup y Master-Detail

Al diseñar una relación no basta con preguntar:

> “¿Qué objetos necesito conectar?”

También debemos preguntarnos:

> “¿Qué grado de dependencia existe entre ellos?”

En UniHub podemos tener:

```text
University
    │
    └── Student
          │
          └── Enrollment
```

Podemos considerar diferentes niveles de dependencia.

Por ejemplo:

- Un Student pertenece a una University.
- Un Enrollment no tiene sentido sin un Student.

Esto introduce una consideración de **integridad referencial y dependencia de existencia**.

## Lookup

Una relación Lookup puede utilizarse cuando queremos relacionar registros sin expresar necesariamente una dependencia fuerte de existencia.

Conceptualmente puede ser una relación más flexible.

## Master-Detail

Master-Detail representa una relación más fuerte entre los registros y permite expresar una dependencia más estrecha entre padre y detalle.

La elección entre Lookup y Master-Detail no debe hacerse únicamente por cardinalidad.

También debemos considerar:

- Dependencia.
- Comportamiento ante eliminación.
- Seguridad.
- Propiedad del registro.
- Roll-Up Summary.
- Reglas de negocio.

### Idea fundamental

> Una relación no solamente conecta datos; también puede expresar reglas de dependencia y comportamiento.

---

# 10. App

Una **App** organiza la experiencia de usuario alrededor de funcionalidades y datos existentes.

Una App puede proporcionar:

- Navegación.
- Objetos visibles.
- Tabs.
- Funcionalidades relevantes.
- Una experiencia adaptada a determinados usuarios o procesos.

Una Org puede contener varias Apps.

Por ejemplo:

```text
ORG
│
├── UniHub Admin App
│   ├── Universities
│   ├── Students
│   ├── Professors
│   └── Courses
│
└── UniHub Professor App
    ├── Courses
    ├── Students
    └── Enrollments
```

Una App nueva **no implica necesariamente crear nuevos objetos o datos**.

Varias Apps pueden trabajar con los mismos objetos y registros.

### Distinción importante

```text
Objects
    ↓
Definen las entidades y estructura de datos

Apps
    ↓
Organizan la experiencia de usuario
```

### Analogía con Odoo

La analogía con un módulo de Odoo puede ayudar inicialmente, pero no debe tratarse como equivalencia técnica.

Un módulo de Odoo normalmente introduce o extiende funcionalidad y modelos.

Una App de Salesforce se centra principalmente en organizar la experiencia de usuario sobre funcionalidades y datos.

---

# 11. Modelo mental general

Los conceptos estudiados pueden visualizarse así:

```text
                         ORG
                          │
             ┌────────────┴────────────┐
             │                         │
          METADATA                    DATA
             │                         │
       ┌─────┴──────┐                  │
       │            │                  │
    Objects      Schema             Records
       │            │                  │
    ┌──┴──┐      relaciones        ┌───┴────┐
    │     │                        │        │
 Fields Relationships          Student  University
                                  │
                                  │
                               Fields
```

Y sobre esa estructura:

```text
                    ORG
                     │
              ┌──────┴──────┐
              │             │
          Admin App     Professor App
              │             │
              └──────┬──────┘
                     │
                mismos datos
                y objetos
                distinta
                experiencia
```

---

# 12. Modelo UniHub inicial

El dominio que estamos comenzando a modelar es:

```text
University
Student
Professor
Course
Enrollment
```

Una primera visión conceptual:

```text
University
     │
     ├── Students
     │
     └── Courses

Student
     │
     └── Enrollment
            │
            └── Course
```

Este modelo todavía no está implementado.

Primero debemos diseñarlo y justificar las decisiones.

---

# 13. Analogías con Odoo

Las siguientes equivalencias son únicamente conceptuales:

| Odoo | Salesforce |
|---|---|
| Model | Object |
| Record | Record |
| Field | Field |
| Relational field | Relationship |
| Database/instance | Org (aproximación conceptual) |

No deben interpretarse como equivalencias técnicas directas.

El objetivo de estas analogías es aprovechar conocimientos previos para construir el modelo mental de Salesforce más rápidamente.

---

# 14. Conceptos consolidados

Después de esta etapa podemos expresar los conceptos principales de la siguiente manera:

### Org

> Entorno Salesforce aislado que contiene datos, metadata, configuración, usuarios y funcionalidades.

### Object

> Entidad sobre la que Salesforce define y gestiona información.

### Record

> Instancia concreta de un Object.

### Field

> Propiedad o atributo utilizado para almacenar información de un Object.

### Metadata

> Información que define la estructura y configuración de Salesforce.

### Schema

> Estructura del modelo de datos: entidades, campos, relaciones y dependencias.

### Relationship

> Mecanismo que permite conectar registros de diferentes objetos y expresar determinadas reglas de dependencia y comportamiento.

### App

> Forma de organizar la experiencia de usuario alrededor de funcionalidades y datos existentes.

---

# 15. Preguntas que debemos ser capaces de responder

Antes de pasar a la implementación, deberíamos poder razonar sobre preguntas como:

1. ¿Qué diferencia existe entre Object y Record?
2. ¿Qué diferencia existe entre Data y Metadata?
3. ¿Qué representa el Schema?
4. ¿Cuándo utilizaríamos un Standard Object y cuándo un Custom Object?
5. ¿Qué representa un campo de relación?
6. ¿Qué diferencia conceptual existe entre Lookup y Master-Detail?
7. ¿Qué significa que un registro dependa de otro?
8. ¿Una App necesita crear nuevos objetos?
9. ¿Pueden varias Apps utilizar los mismos objetos?
10. ¿Qué decisiones del modelo de datos pertenecen al Schema?

---

# 16. Estado de aprendizaje

```text
Org                 ✓ Understand
Object              ✓ Understand
Record              ✓ Understand
Field               ✓ Understand
Data vs Metadata    ✓ Understand
Schema              ✓ Understand
Standard/Custom     ✓ Understand
Relationships       ✓ Understand
Lookup              ✓ Initial understanding
Master-Detail       ✓ Initial understanding
App                 ✓ Understand
```

El siguiente objetivo es pasar de comprender los conceptos a **diseñar el modelo de datos de UniHub antes de implementarlo en Salesforce**.

---

## Próximo ejercicio

Diseñar las cuatro entidades iniciales:

```text
University
Student
Professor
Course
```

Para cada una:

1. Definir el Custom Object.
2. Proponer 3–5 Fields.
3. Identificar las Relationships.
4. Justificar las decisiones de diseño.

No se implementará todavía en Salesforce.

Primero se diseñará el modelo.
