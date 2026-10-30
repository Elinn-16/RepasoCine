# Repaso Parcial I — Cine

Material de repaso para el recuperatorio de Programación II. Es el
mismo tipo de ejercicio que el Parcial I, con el dominio de un cine en
vez de agentes, y otros números en las reglas. **No hay entrega ni
plazo** — es para practicar.

## Qué es

Una API en .NET 8 para administrar salas de cine y sus funciones. El
proyecto base (modelos, `DbContext`, la interfaz del repositorio, el
controller con las rutas ya declaradas) **ya está armado** — falta
completar la lógica de dos archivos:

- `Repositories/SalaRepository.cs`
- `Controllers/SalasController.cs`

El enunciado completo está en **`Repaso Parcial Cine.pdf`** (o
`enunciado.md` si preferís leerlo en texto plano).

## Cómo levantar el proyecto, paso a paso

### 1. Crear la base de datos

Necesitás un SQL Server accesible (LocalDB en Windows, o un contenedor
si estás en Linux/Mac — ver más abajo). Corré el script
**`script_db.txt`** contra tu servidor (es un script T-SQL normal, se
puede pegar en SSMS / Azure Data Studio / `sqlcmd` tal cual). Crea la
base `db_cine`, las dos tablas y carga datos de ejemplo pensados para
poder probar todas las reglas del enunciado.

### 2. Revisar la cadena de conexión

Está en `RepasoCineWebAPI/appsettings.json`, bajo
`ConnectionStrings:CineDbConnection`. Por defecto apunta a LocalDB:

```
Server=(localdb)\MSSQLLocalDB;Database=db_cine;Trusted_Connection=True;TrustServerCertificate=True;MultipleActiveResultSets=true
```

Si no estás en Windows o no tenés LocalDB, dos opciones:

**a) SQL Server en un contenedor (Podman/Docker):**

```bash
podman run -d --name sql-repaso -e "ACCEPT_EULA=Y" -e "MSSQL_SA_PASSWORD=TuPassword123!" \
  -p 1433:1433 mcr.microsoft.com/mssql/server:2022-latest
```

y cambiá la cadena de conexión (en tu `appsettings.json` **local**,
nunca la subas así) a algo como:

```
Server=localhost,1433;Database=db_cine;User Id=sa;Password=TuPassword123!;TrustServerCertificate=True;MultipleActiveResultSets=true
```

**b) Contraseña por fuera del archivo** (recomendado si usás
usuario/contraseña en vez de autenticación integrada): usá
`dotnet user-secrets` o una variable de entorno en vez de escribir la
contraseña en `appsettings.json`:

```bash
cd RepasoCineWebAPI
dotnet user-secrets init
dotnet user-secrets set "ConnectionStrings:CineDbConnection" "Server=localhost,1433;Database=db_cine;User Id=sa;Password=TuPassword123!;TrustServerCertificate=True;MultipleActiveResultSets=true"
```

o, por variable de entorno al correr:

```bash
ConnectionStrings__CineDbConnection="Server=localhost,1433;Database=db_cine;User Id=sa;Password=TuPassword123!;TrustServerCertificate=True;MultipleActiveResultSets=true" dotnet run
```

### 3. Levantar el proyecto

Desde VS Code, con la carpeta `RepasoCineWebAPI - para alumnos`
abierta: apretá **F5** (ya están configurados `.vscode/launch.json` y
`tasks.json` para correr en modo Development y abrir Swagger solo).

O por consola:

```bash
cd "RepasoCineWebAPI - para alumnos/RepasoCineWebAPI"
dotnet run
```

y abrí `http://localhost:5191/swagger` a mano (Swagger solo está
disponible en modo Development).

## Qué hay que completar

1. **`Repositories/SalaRepository.cs`**: los tres métodos de
   `IRepository` (`GetById`, `GetActiveRoomsWithPendingScreenings`,
   `AssignScreeningToRoom`) hoy lanzan `NotImplementedException`.
2. **`Controllers/SalasController.cs`**: las tres acciones ya tienen
   su ruta, su verbo HTTP y su firma — falta el cuerpo de cada una
   (hoy también lanzan `NotImplementedException`).

No hace falta tocar los modelos, el `DbContext`, `Program.cs` ni
`appsettings.json` — ya están resueltos.

## Cómo probar

El archivo `RepasoCineWebAPI/RepasoCineWebAPI.http` tiene una
colección de casos de prueba, cada uno con el código de respuesta
esperado en un comentario. Si usás VS Code con la extensión **REST
Client** (o el cliente HTTP integrado de Visual Studio/Rider), lo
podés correr caso por caso directamente desde ahí. Si preferís,
también podés probar cada caso a mano desde Swagger.

Si algo no te cierra, revisá primero **`GUIA-ERRORES-FRECUENTES.md`**
— tiene pistas sobre los errores más comunes de este tipo de
ejercicio, sin dar la solución.
