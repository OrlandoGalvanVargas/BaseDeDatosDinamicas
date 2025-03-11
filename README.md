# Documentación Técnica: Formulario para Crear una Base de Datos Dinámica

## Descripción General
El formulario permite a los usuarios crear una base de datos dinámica en SQL Server utilizando un procedimiento almacenado. Los usuarios ingresan los detalles de la base de datos (nombre, rutas de archivos, tamaños iniciales, crecimiento, etc.), y la aplicación ejecuta el procedimiento almacenado para crear la base de datos.

---

## Requisitos
- **ASP.NET Core**: La aplicación está desarrollada en ASP.NET Core (Razor Pages).
- **SQL Server**: Se requiere una instancia de SQL Server con acceso a la base de datos `DBAdmin`.
- **Procedimiento Almacenado**: El procedimiento almacenado `CrearBaseDeDatosDinamica` debe estar creado en la base de datos `DBAdmin`.
- **Bootstrap**: Se utiliza para el diseño del formulario.

---

## Estructura del Proyecto
El proyecto tiene la siguiente estructura de archivos:

* **Pages/**
    * `CreateDatabase.cshtml` // Vista del formulario
    * `CreateDatabase.cshtml.cs` // Lógica del formulario
* **Shared/**
    * `_ValidationScriptsPartial.cshtml` // Scripts de validación
* `appsettings.json` // Configuración de la cadena de conexión
* `Program.cs` // Configuración de la aplicación

---

## Componentes del Formulario

### Modelo (`DatabaseCreationModel`)
El modelo representa los datos que el usuario ingresa en el formulario. Está definido en `Models/DatabaseCreationModel.cs`.

```csharp
public class DatabaseCreationModel
{
    [Required(ErrorMessage = "El nombre de la base de datos es obligatorio.")]
    public string NombreDB { get; set; }

    [Required(ErrorMessage = "La ruta del archivo MDF es obligatoria.")]
    public string RutaMDF { get; set; }

    [Required(ErrorMessage = "El tamaño inicial del archivo MDF es obligatorio.")]
    [Range(1, int.MaxValue, ErrorMessage = "El tamaño inicial debe ser mayor que 0.")]
    public int TamanioInicialMDF { get; set; }

    [Required(ErrorMessage = "El crecimiento del archivo MDF es obligatorio.")]
    [Range(1, int.MaxValue, ErrorMessage = "El crecimiento debe ser mayor que 0.")]
    public int CrecimientoMDF { get; set; }

    [Required(ErrorMessage = "El tamaño máximo del archivo MDF es obligatorio.")]
    [Range(1, int.MaxValue, ErrorMessage = "El tamaño máximo debe ser mayor que 0.")]
    public int TamanioMaximoMDF { get; set; }

    [Required(ErrorMessage = "La ruta del archivo LDF es obligatoria.")]
    public string RutaLDF { get; set; }

    [Required(ErrorMessage = "El tamaño inicial del archivo LDF es obligatorio.")]
    [Range(1, int.MaxValue, ErrorMessage = "El tamaño inicial debe ser mayor que 0.")]
    public int TamanioInicialLDF { get; set; }

    [Required(ErrorMessage = "El crecimiento del archivo LDF es obligatorio.")]
    [Range(1, int.MaxValue, ErrorMessage = "El crecimiento debe ser mayor que 0.")]
    public int CrecimientoLDF { get; set; }
}
```

### Vista (CreateDatabase.cshtml)
La vista es la interfaz de usuario donde los usuarios ingresan los datos. Está diseñada con Bootstrap para un aspecto moderno y responsivo.

```csharp
@page
@model BaseDeDatosDinamicas.Pages.CreateDatabaseModel
@{
    ViewData["Title"] = "Crear Base de Datos";
}

<div class="container mt-5">
    <h2 class="text-center mb-4">Crear Base de Datos</h2>

    <form method="post" class="card p-4 shadow">
        <!-- Campos del formulario -->
        <div class="mb-3">
            <label asp-for="DatabaseModel.NombreDB" class="form-label">Nombre de la Base de Datos</label>
            <input asp-for="DatabaseModel.NombreDB" class="form-control" placeholder="Ingrese el nombre de la base de datos" />
            <span asp-validation-for="DatabaseModel.NombreDB" class="text-danger"></span>
        </div>
        <!-- Más campos aquí -->
        <div class="d-grid">
            <button type="submit" class="btn btn-primary btn-lg">Crear Base de Datos</button>
        </div>
    </form>
</div>

@section Scripts {
    <partial name="_ValidationScriptsPartial" />
}
```

### Lógica del Formulario (CreateDatabase.cshtml.cs)
La lógica del formulario maneja la validación de los datos y la ejecución del procedimiento almacenado.

```csharp
using BaseDeDatosDinamicas.Models;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;
using Microsoft.Data.SqlClient;

namespace BaseDeDatosDinamicas.Pages
{
    public class CreateDatabaseModel : PageModel
    {
        private readonly IConfiguration _configuration;

        [BindProperty]
        public DatabaseCreationModel DatabaseModel { get; set; }

        public CreateDatabaseModel(IConfiguration configuration)
        {
            _configuration = configuration;
        }

        public void OnGet()
        {
        }

        public IActionResult OnPost()
        {
            if (!ModelState.IsValid)
            {
                return Page();
            }

            var connectionString = _configuration.GetConnectionString("DefaultConnection");

            using (SqlConnection connection = new SqlConnection(connectionString))
            {
                connection.Open();
                using (SqlCommand command = new SqlCommand("CrearBaseDeDatosDinamica", connection))
                {
                    command.CommandType = System.Data.CommandType.StoredProcedure;
                    command.Parameters.AddWithValue("@NombreDB", DatabaseModel.NombreDB);
                    command.Parameters.AddWithValue("@RutaMDF", DatabaseModel.RutaMDF);
                    command.Parameters.AddWithValue("@TamanioInicialMDF", DatabaseModel.TamanioInicialMDF);
                    command.Parameters.AddWithValue("@CrecimientoMDF", DatabaseModel.CrecimientoMDF);
                    command.Parameters.AddWithValue("@RutaLDF", DatabaseModel.RutaLDF);
                    command.Parameters.AddWithValue("@TamanioInicialLDF", DatabaseModel.TamanioInicialLDF);
                    command.Parameters.AddWithValue("@CrecimientoLDF", DatabaseModel.CrecimientoLDF);

                    command.ExecuteNonQuery();
                }
            }

            TempData["Message"] = "Base de datos creada exitosamente.";
            return RedirectToPage("CreateDatabase");
        }
    }
}
```

### Configuración de la Aplicación (Program.cs)
La aplicación está configurada para redirigir automáticamente a la página CreateDatabase al iniciar.

```csharp
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
builder.Services.AddRazorPages();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Error");
    app.UseHsts();
}

app.UseHttpsRedirection();
app.UseStaticFiles();

app.UseRouting();

app.UseAuthorization();

app.MapRazorPages();
app.MapGet("/", context =>
{
    context.Response.Redirect("/CreateDatabase");
    return Task.CompletedTask;
});

app.Run();
```

### Procedimiento Almacenado
El procedimiento almacenado CrearBaseDeDatosDinamica se encarga de crear la base de datos con los parámetros proporcionados.

```sql
use DBAdmin
go
CREATE PROCEDURE CrearBaseDeDatosDinamica
    @NombreDB NVARCHAR(128),
    @RutaMDF NVARCHAR(260),
    @TamanoInicialMDF INT,
    @CrecimientoMDF INT,
    @RutaLDF NVARCHAR(260),
    @TamanoInicialLDF INT,
    @CrecimientoLDF INT,
    @FilegroupsSecundarios NVARCHAR(MAX) = NULL,  -- Ejemplo: 'FG1, FG2'
    @Mensaje NVARCHAR(MAX) OUTPUT,
    @CodigoError INT OUTPUT
AS
BEGIN
    SET NOCOUNT ON;

    BEGIN TRY
        -- Validar que el nombre de la base de datos no esté vacío
        IF @NombreDB IS NULL OR @NombreDB = ''
        BEGIN
            SET @Mensaje = 'El nombre de la base de datos no puede estar vacío.';
            SET @CodigoError = 1;
            RETURN;
        END

        -- Validar que las rutas no estén vacías
        IF @RutaMDF IS NULL OR @RutaMDF = '' OR @RutaLDF IS NULL OR @RutaLDF = ''
        BEGIN
            SET @Mensaje = 'Las rutas de los archivos MDF y LDF no pueden estar vacías.';
            SET @CodigoError = 2;
            RETURN;
        END

        -- Validar que los tamaños y crecimientos sean mayores que 0
        IF @TamanoInicialMDF <= 0 OR @CrecimientoMDF <= 0 OR @TamanoInicialLDF <= 0 OR @CrecimientoLDF <= 0
        BEGIN
            SET @Mensaje = 'Los tamaños y crecimientos deben ser mayores que 0.';
            SET @CodigoError = 3;
            RETURN;
        END

        -- Verificar si la base de datos ya existe
        IF DB_ID(@NombreDB) IS NOT NULL
        BEGIN
            SET @Mensaje = 'La base de datos ' + @NombreDB + ' ya existe.';
            SET @CodigoError = 4;
            RETURN;
        END

        -- Construir la sentencia SQL dinámica
        DECLARE @SQL NVARCHAR(MAX)
        SET @SQL = '
        CREATE DATABASE ' + QUOTENAME(@NombreDB) + '
        ON PRIMARY 
        (
            NAME = ' + QUOTENAME(@NombreDB + '_Data') + ',
            FILENAME = ' + QUOTENAME(@RutaMDF, '''') + ',
            SIZE = ' + CAST(@TamanoInicialMDF AS NVARCHAR) + 'MB,
            FILEGROWTH = ' + CAST(@CrecimientoMDF AS NVARCHAR) + 'MB
        )
        LOG ON 
        (
            NAME = ' + QUOTENAME(@NombreDB + '_Log') + ',
            FILENAME = ' + QUOTENAME(@RutaLDF, '''') + ',
            SIZE = ' + CAST(@TamanoInicialLDF AS NVARCHAR) + 'MB,
            FILEGROWTH = ' + CAST(@CrecimientoLDF AS NVARCHAR) + 'MB
        )'

        -- Agregar filegroups secundarios si se especifican
        IF @FilegroupsSecundarios IS NOT NULL
        BEGIN
            DECLARE @Filegroup NVARCHAR(128)
            DECLARE @FilegroupSQL NVARCHAR(MAX)

            DECLARE FilegroupCursor CURSOR FOR
            SELECT value FROM STRING_SPLIT(@FilegroupsSecundarios, ',')

            OPEN FilegroupCursor
            FETCH NEXT FROM FilegroupCursor INTO @Filegroup

            WHILE @@FETCH_STATUS = 0
            BEGIN
                SET @FilegroupSQL = '
                ALTER DATABASE ' + QUOTENAME(@NombreDB) + '
                ADD FILEGROUP ' + QUOTENAME(@Filegroup) + ';'
                EXEC sp_executesql @FilegroupSQL

                FETCH NEXT FROM FilegroupCursor INTO @Filegroup
            END

            CLOSE FilegroupCursor
            DEALLOCATE FilegroupCursor
        END

        -- Ejecutar la sentencia SQL para crear la base de datos
        EXEC sp_executesql @SQL

        -- Mensaje de éxito
        SET @Mensaje = 'Base de datos ' + @NombreDB + ' creada exitosamente.';
        SET @CodigoError = 0;
    END TRY
    BEGIN CATCH
        -- Capturar el error
        SET @Mensaje = 'Error: ' + ERROR_MESSAGE();
        SET @CodigoError = ERROR_NUMBER();
    END CATCH
END;
```

---

## Instrucciones de Uso

### Ejecutar la Aplicación:
Inicia la aplicación ASP.NET Core.

### Acceder al Formulario:
La aplicación redirige automáticamente a `CreateDatabase`.

### Ingresar Datos:
Completa el formulario con los detalles de la base de datos.

### Enviar el Formulario:
Haz clic en "Crear Base de Datos" para ejecutar el procedimiento almacenado.

---

---

## Pruebas

### Validación del Formulario:
Asegúrate de que todos los campos sean obligatorios y que los valores numéricos sean mayores que 0.

### Conexión a SQL Server:
Verifica que la cadena de conexión en `appsettings.json` sea correcta.

### Ejecución del Procedimiento Almacenado:
Confirma que la base de datos se crea correctamente.

---
