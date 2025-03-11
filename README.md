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

Pages/
├── CreateDatabase.cshtml // Vista del formulario
├── CreateDatabase.cshtml.cs // Lógica del formulario
Shared/
├── _ValidationScriptsPartial.cshtml // Scripts de validación
appsettings.json // Configuración de la cadena de conexión
Program.cs // Configuración de la aplicación


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


