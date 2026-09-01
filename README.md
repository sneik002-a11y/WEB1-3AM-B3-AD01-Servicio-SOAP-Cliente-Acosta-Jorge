# TiendaSOAP — Backend

Servicio web SOAP desarrollado en **.NET (ASP.NET Core + CoreWCF)** para la gestión de **Categorías** y **Productos**, como parte de la práctica de Programación Web 1.
---

## Tecnologías utilizadas

- **ASP.NET Core** (proyecto vacío, .NET 10.0)
- **CoreWCF.Http** — implementación de servicios SOAP sobre ASP.NET Core
- **Entity Framework Core** + **Microsoft.EntityFrameworkCore.SqlServer** — acceso a datos
- **SQL Server** — base de datos `TiendaSOAPDB`

---

## Modelo de datos

### Categoria

| Campo        | Tipo    |
|--------------|---------|
| IdCategoria  | int (PK)|
| Nombre       | string  |
| Descripcion  | string  |
| Estado       | bit     |

### Producto

| Campo        | Tipo             |
|--------------|------------------|
| IdProducto   | int (PK)         |
| Nombre       | string           |
| Descripcion  | string           |
| Precio       | decimal          |
| Stock        | int              |
| Estado       | bit              |
| IdCategoria  | int (FK → Categoria) |

---

## Servicios SOAP expuestos

Interfaz `IProductoService`, endpoint: `/ProductoService.svc`

| # | Operación | Descripción |
|---|-----------|-------------|
| 1 | `ObtenerCategorias()` | Lista todas las categorías |
| 2 | `ObtenerProductos()` | Lista todos los productos |
| 3 | `ObtenerProducto(int id)` | Obtiene un producto por su Id |
| 4 | `AgregarProducto(Producto p)` | Crea un nuevo producto |
| 5 | `ActualizarProducto(Producto p)` | Actualiza un producto existente |
| 6 | `EliminarProducto(int id)` | Elimina un producto por su Id |
| 7 | `ObtenerProductosPorPrecio(decimal precioMin, decimal precioMax)` | Filtra productos por rango de precio |
| 8 | `ObtenerProductosPorCategoria(int idCategoria)` | Filtra productos por categoría |

Cada método `Agregar`/`Actualizar`/`Eliminar` retorna un `bool` indicando si la operación tuvo éxito.

---

## Estructura del proyecto

```
TiendaSOAP/
├── Models/
│   ├── Categoria.cs
│   └── Producto.cs
├── Data/
│   └── TiendaDBContext.cs
├── Services/
│   ├── IProductoService.cs      (contrato SOAP)
│   └── ProductoService.cs       (implementación)
├── Program.cs
├── appsettings.json
└── script-tiendasoap.sql
```

---

## Configuración

### 1. Paquetes NuGet

```
Install-Package CoreWCF.Http
Install-Package Microsoft.EntityFrameworkCore.SqlServer
```

### 2. Cadena de conexión (`appsettings.json`)

```json
"ConnectionStrings": {
  "TiendaConnection": "Server=.\\SQLEXPRESS;Database=TiendaSOAPDB;Trusted_Connection=True;TrustServerCertificate=True;"
}
```

Ajustar el nombre de instancia de SQL Server según el entorno local.

### 3. Base de datos

Ejecutar `script-tiendasoap.sql`, que crea `TiendaSOAPDB` con las tablas `Categoria` y `Producto` (con la FK entre ambas) e inserta datos de prueba.

### 4. CORS

Habilitado para permitir el consumo desde el frontend Angular (`http://localhost:4200`):

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AngularPolicy", policy =>
    {
        policy
            .WithOrigins("http://localhost:4200")
            .AllowAnyHeader()
            .AllowAnyMethod();
    });
});
```

---

## Decisiones técnicas y problemas resueltos

Durante el desarrollo surgieron algunos detalles que vale la pena documentar:

- **Nombres de tabla:** el script SQL usa nombres en singular (`Categoria`, `Producto`), mientras que los `DbSet` en el contexto están en plural (`Categorias`, `Productos`). Se mapeó explícitamente con `modelBuilder.Entity<T>().ToTable("...")` en `OnModelCreating`.

- **Autoincremento de las PK:** como las columnas se llaman `IdProducto` / `IdCategoria` (no siguen la convención `Id` o `<Entidad>Id`), Entity Framework no las reconocía automáticamente como `IDENTITY`. Se configuró explícitamente con `.ValueGeneratedOnAdd()` para cada una.

- **Orden de serialización SOAP:** al no tener atributos `[DataContract]`/`[DataMember]`, WCF asumía un contrato implícito con orden **alfabético** de propiedades, distinto al orden declarado en la clase. Esto causaba que campos como `IdCategoria` llegaran en `0` aunque el XML enviado tuviera el valor correcto, rompiendo la relación con `Categoria`. Se resolvió agregando `[DataContract]` y `[DataMember(Order = n)]` a `Producto` y `Categoria`, fijando el orden esperado: `IdProducto, Nombre, Descripcion, Precio, Stock, Estado, IdCategoria`.

---

## Pruebas (Postman)

Se incluyen 8 archivos de ejemplo (`Postman *.txt`) con el sobre SOAP (envelope) listo para cada operación. Para todas las peticiones:

- Método: `POST`
- URL: `http://localhost:<puerto>/ProductoService.svc`
- Header `Content-Type`: `text/xml; charset=utf-8`
- Header `SOAPAction`: `http://tempuri.org/IProductoService/<NombreDelMetodo>`
- Body: `raw` → `XML`, con el contenido del archivo correspondiente

---

## Frontend

Este backend es consumido por un cliente Angular (`tienda-soap-frontend`) que implementa las peticiones SOAP directamente sobre `HttpClient`, con componentes separados para **Producto** (CRUD completo + filtros por precio y categoría) y **Categoria** (listado).

## Autor
Nombre: Jorge Acosta 
Asignatura: Programación Web1 
Paralelo: Tercero A Matutina 
