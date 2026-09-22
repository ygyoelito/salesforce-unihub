# UniHub — Salesforce CLI para Tabs y Lightning App

Este documento recopila exclusivamente los comandos de **Salesforce CLI (`sf`)** que hemos utilizado o aplicado durante la creación de los Custom Object Tabs y la Lightning App de UniHub.

> Nota: los archivos `*.tab-meta.xml` pueden generarse mediante Salesforce CLI. La definición `UniHub.app-meta.xml` la creamos manualmente como metadata y después la validamos/desplegamos con Salesforce CLI.

---

## 1. Generar un Custom Object Tab

Patrón general:

```powershell
sf schema generate tab `
  --object <OBJECT_API_NAME> `
  --icon <ICON_NUMBER> `
  --directory force-app/main/default/tabs
```

Ejemplo utilizado para `University__c`:

```powershell
sf schema generate tab `
  --object University__c `
  --icon 54 `
  --directory force-app/main/default/tabs
```

Resultado esperado:

```text
force-app/main/default/tabs/University__c.tab-meta.xml
```

---

## 2. Generar los Tabs de UniHub

### University

```powershell
sf schema generate tab `
  --object University__c `
  --icon 54 `
  --directory force-app/main/default/tabs
```

### Career

```powershell
sf schema generate tab `
  --object Career__c `
  --icon 35 `
  --directory force-app/main/default/tabs
```

### University Career

```powershell
sf schema generate tab `
  --object UniversityCareer__c `
  --icon 40 `
  --directory force-app/main/default/tabs
```

### Student

```powershell
sf schema generate tab `
  --object Student__c `
  --icon 38 `
  --directory force-app/main/default/tabs
```

### Course

```powershell
sf schema generate tab `
  --object Course__c `
  --icon 57 `
  --directory force-app/main/default/tabs
```

---

## 3. Estructura generada

```text
force-app/main/default/tabs/
├── University__c.tab-meta.xml
├── Career__c.tab-meta.xml
├── UniversityCareer__c.tab-meta.xml
├── Student__c.tab-meta.xml
└── Course__c.tab-meta.xml
```

---

## 4. Revisar un archivo generado desde PowerShell

Aunque no es un comando de Salesforce CLI, lo usamos para inspeccionar rápidamente el metadata generado:

```powershell
Get-Content force-app/main/default/tabs/University__c.tab-meta.xml
```

---

## 5. Lightning App `UniHub`

La aplicación se definió manualmente como metadata en:

```text
force-app/main/default/applications/UniHub.app-meta.xml
```

No utilizamos un comando `sf ... generate app` para crearla.

La aplicación referencia los tabs:

```text
University__c
Career__c
UniversityCareer__c
Student__c
Course__c
```

---

## 6. Dry-run de Tabs + App + Permission Set

```powershell
sf project deploy start `
  --dry-run `
  --source-dir force-app/main/default/tabs `
  --source-dir force-app/main/default/applications/UniHub.app-meta.xml `
  --source-dir force-app/main/default/permissionsets/UniHub_Developer.permissionset-meta.xml `
  --target-org siloe
```

---

## 7. Deploy real de Tabs + App + Permission Set

```powershell
sf project deploy start `
  --source-dir force-app/main/default/tabs `
  --source-dir force-app/main/default/applications/UniHub.app-meta.xml `
  --source-dir force-app/main/default/permissionsets/UniHub_Developer.permissionset-meta.xml `
  --target-org siloe
```

---

## 8. Dry-run de una List View

```powershell
sf project deploy start `
  --dry-run `
  --source-dir force-app/main/default/objects/University__c/listViews `
  --target-org siloe
```

---

## 9. Deploy real de una List View

```powershell
sf project deploy start `
  --source-dir force-app/main/default/objects/University__c/listViews `
  --target-org siloe
```

---

## 10. Patrón de trabajo aprendido

```text
Generar Tab con sf
        ↓
Revisar metadata XML
        ↓
Crear/editar Lightning App metadata
        ↓
Actualizar Permission Set
        ↓
sf project deploy start --dry-run
        ↓
Corregir errores si existen
        ↓
sf project deploy start
        ↓
Verificar en App Launcher → UniHub
```

---

## 11. Recordatorio de shell

En **PowerShell**, la continuación de línea se hace con:

```text
`
```

Ejemplo:

```powershell
sf project deploy start `
  --dry-run `
  --source-dir force-app/main/default/tabs `
  --target-org siloe
```

En **Bash/Git Bash**, se utiliza `\\`:

```bash
sf project deploy start \\
  --dry-run \\
  --source-dir force-app/main/default/tabs \\
  --target-org siloe
```
