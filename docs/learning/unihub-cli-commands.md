# UniHub — Comandos CLI utilizados

## Git

```bash
git branch --show-current
```

```bash
git status --short
```

```bash
git diff
```

```bash
git diff --cached
```

```bash
git diff --staged
```

```bash
git diff -- force-app/main/default/objects/University__c/University__c.object-meta.xml
```

```bash
git diff -- force-app/main/default/objects/Career__c
```

```bash
git add force-app/main/default/objects/University__c
```

```bash
git add force-app/main/default/objects/Career__c
```

```bash
git add force-app/main/default/objects/UniversityCareer__c
```

```bash
git add force-app/main/default/flows/Set_University_Career_Key.flow-meta.xml
```

```bash
git add force-app/main/default/permissionsets/UniHub_Developer.permissionset-meta.xml
```

```bash
git commit -m "feat: add University data model"
```

```bash
git commit -m "feat: add Career data model"
```

```bash
git commit -m "feat: add University Career data model"
```

```bash
git push
```

```bash
git switch -c feature/week-01-data-model-implementation
```

```bash
git switch main
```

```bash
git pull
```

```bash
git merge test
```

```bash
git log --oneline --decorate -5
```

```bash
git diff main..test
```

## Salesforce CLI — Generación de metadata

```bash
sf schema generate sobject --label "University"
```

```bash
sf schema generate field --label "Address" --object force-app/main/default/objects/University__c
```

```bash
sf schema generate field \
  --label "Address" \
  --object force-app/main/default/objects/University__c
```

## Salesforce CLI — Deploy y dry-run

```bash
sf project deploy start \
  --dry-run \
  --source-dir force-app/main/default/objects/University__c \
  --target-org siloe
```

```bash
sf project deploy start \
  --source-dir force-app/main/default/objects/University__c \
  --target-org siloe
```

```bash
sf project deploy start \
  --dry-run \
  --source-dir force-app/main/default/objects/Career__c \
  --source-dir force-app/main/default/permissionsets/UniHub_Developer.permissionset-meta.xml \
  --target-org siloe
```

```bash
sf project deploy start \
  --source-dir force-app/main/default/objects/Career__c \
  --source-dir force-app/main/default/permissionsets/UniHub_Developer.permissionset-meta.xml \
  --target-org siloe
```

```bash
sf project deploy start \
  --dry-run \
  --source-dir force-app/main/default/objects/UniversityCareer__c \
  --source-dir force-app/main/default/flows/Set_University_Career_Key.flow-meta.xml \
  --target-org siloe
```

```bash
sf project deploy start \
  --dry-run \
  --source-dir force-app/main/default/objects/UniversityCareer__c \
  --source-dir force-app/main/default/flows/Set_University_Career_Key.flow-meta.xml \
  --source-dir force-app/main/default/permissionsets/UniHub_Developer.permissionset-meta.xml \
  --target-org siloe
```

```bash
sf project deploy start \
  --source-dir force-app/main/default/objects/UniversityCareer__c \
  --source-dir force-app/main/default/flows/Set_University_Career_Key.flow-meta.xml \
  --source-dir force-app/main/default/permissionsets/UniHub_Developer.permissionset-meta.xml \
  --target-org siloe
```

```powershell
sf project deploy start `
  --dry-run `
  --source-dir force-app/main/default/objects/UniversityCareer__c `
  --target-org siloe
```

```powershell
sf project deploy start `
  --source-dir force-app/main/default/objects/UniversityCareer__c `
  --target-org siloe
```

## Salesforce CLI — Permission Sets

```powershell
sf org assign permset --name UniHub_Developer --target-org siloe
```

## Salesforce CLI — Describe

```powershell
sf sobject describe --sobject University__c --target-org siloe
```

```powershell
$d = sf sobject describe --sobject University__c --target-org siloe | ConvertFrom-Json
```

```powershell
$d.fields |
    Where-Object { $_.name -like "*Address*" } |
    Select-Object name, type, createable, updateable
```

```powershell
$d.fields |
    Select-Object name, type |
    Where-Object { $_.name -match "Street|City|State|Country|Address" }
```

## Salesforce CLI — Crear registros

```powershell
sf data create record --sobject University__c --values "Name='UniHub Validation Test'" --target-org siloe
```

```powershell
sf data create record --sobject University__c --values "Name='UniHub Validation Test' Address__Street__s='1-1 Chiyoda' Address__City__s='Tokyo' Address__StateCode__s='13' Address__CountryCode__s='JP'" --target-org siloe
```

```powershell
sf data create record --sobject Career__c --values "Name='Computer Science'" --target-org siloe
```

```powershell
sf data create record --sobject Career__c --values "Name='UniHub Duration Test'" --target-org siloe
```

```powershell
sf data create record --sobject Career__c --values "Name='UniHub Active Test'" --target-org siloe
```

```powershell
sf data create record --sobject Career__c --values "Name='UniHub Upper Bound Test'" --target-org siloe
```

```powershell
sf data create record --sobject UniversityCareer__c --values "University__c='<U_ID>' Career__c='<C1_ID>' DurationYears__c=5" --target-org siloe
```

```powershell
sf data create record --sobject UniversityCareer__c --values "University__c='<U_ID>' Career__c='<C1_ID>' DurationYears__c=5" --target-org siloe
```

```powershell
sf data create record --sobject UniversityCareer__c --values "University__c='<U_ID>' Career__c='<C2_ID>' DurationYears__c=3" --target-org siloe
```

```powershell
sf data create record --sobject UniversityCareer__c --values "University__c='<U_ID>' Career__c='<C4_ID>' DurationYears__c=10" --target-org siloe
```

```powershell
sf data create record --sobject UniversityCareer__c --values "University__c='<U_ID>' Career__c='<C3_ID>' DurationYears__c=5 Active__c=true" --target-org siloe
```

```powershell
sf data create record --sobject UniversityCareer__c --values "University__c='<U_ID>' Career__c='<C3_ID>' DurationYears__c=5 Active__c=true StartDate__c=2026-09-19" --target-org siloe
```

## Salesforce CLI — Consultas SOQL

```powershell
sf data query --query "SELECT Id, Name FROM University__c" --target-org siloe
```

```powershell
sf data query --query "SELECT Id, Name, Address__Street__s, Address__City__s, Address__StateCode__s, Address__CountryCode__s FROM University__c LIMIT 1" --target-org siloe
```

```powershell
sf data query --query "SELECT Id, Name, Address__Street__s, Address__City__s, Address__StateCode__s, Address__CountryCode__s FROM University__c WHERE Name = 'UniHub Validation Test'" --target-org siloe
```

```powershell
sf data query --query "SELECT Id, Name FROM Career__c" --target-org siloe
```

```powershell
sf data query --query "SELECT Id, Name FROM Career__c WHERE Name = 'Computer Science'" --target-org siloe
```

```powershell
sf data query --query "SELECT Id, Name, University__c, Career__c, DurationYears__c, Active__c, StartDate__c, UniversityCareerKey__c FROM UniversityCareer__c WHERE Id='<UC_ID>'" --target-org siloe
```

```powershell
sf data query --query "SELECT Name, DurationYears__c, Active__c, StartDate__c, UniversityCareerKey__c FROM UniversityCareer__c WHERE Career__c='<C3_ID>' AND University__c='<U_ID>'" --target-org siloe
```

## Salesforce CLI — Eliminar registros

```powershell
sf data delete record --sobject University__c --record-id <ID_DEL_REGISTRO> --target-org siloe
```

## Salesforce CLI — Actualización e instalación de plugins

```powershell
sf update stable-rc
```

```powershell
sf plugins install @salesforce/plugin-code-analyzer
```

```powershell
sf code-analyzer rules
```

```powershell
sf doctor
```

## Salesforce CLI — Obtener metadata desde Salesforce hacia el proyecto local en VS Code

### Listar los metadatos de forma general

```powershell
sf org list metadata `
  --metadata-type Layout `
  --target-org siloe
```

### Sin indicar metadata

```powershell
sf project retrieve start --target-org siloe
```

### Traer un Layout concreto

```powershell
sf project retrieve start `
  --metadata "Layout:University__c-University Layout" `
  --target-org siloe
```

### Traer un Permission Set

```powershell
sf project retrieve start `
  --metadata "PermissionSet:UniHub_Developer" `
  --target-org siloe
```

### Traer un objeto completo

```powershell
sf project retrieve start `
  --metadata "CustomObject:Student__c" `
  --target-org siloe
```

### Traer varios componentes a la vez

```powershell
sf project retrieve start `
  --metadata "CustomObject:Student__c" `
  --metadata "PermissionSet:UniHub_Developer" `
  --metadata "Layout:Student__c-Student Layout" `
  --target-org siloe
```

## npm

```powershell
npm.cmd view @salesforce/plugin-code-analyzer version
```

## Bash / PowerShell — Crear directorios

```bash
mkdir -p force-app/main/default/objects/University__c/validationRules
```

```powershell
New-Item -ItemType Directory -Force force-app/main/default/objects/University__c/validationRules
```
