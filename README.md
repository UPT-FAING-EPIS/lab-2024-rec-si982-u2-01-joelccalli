**SESION DE LABORATORIO N° 02: Construyendo una Aplicación Web con ASP.NET y Entity Framework** 

**OBJETIVOS** 

- Comprender el desarrollo una Aplicación Web utilizando ASP.NET y Entity Framework 

**REQUERIMIENTOS** 

- Conocimientos: 
  - Conocimientos básicos de SQL. 
  - Conocimientos shell y comandos en modo terminal. 
- Hardware: 
  - Virtualization activada en el BIOS. 
  - CPU SLAT-capable feature. 
  - Al menos 4GB de RAM. 
- Software: 
  - Windows 10 64bit: Pro, Enterprise o Education (1607 Anniversary Update, Build 14393 o Superior) 
  - Docker Desktop 
  - Powershell versión 7.x 
  - .Net 8 
  - Azure CLI 

**CONSIDERACIONES INICIALES** 



|￿ |Tener una cuenta en Infracost [(https://www.infracost.io/),](https://www.infracost.io/) sino utilizar su |
| - | - |
||cuenta de github para generar su cuenta y generar un token. |
|￿ |Tener una cuenta en SonarCloud [(https://sonarcloud.io/)](https://sonarcloud.io/), sino utilizar su |
||cuenta de github para generar su cuenta y generar un token. El token |
||debera estar registrado en su repositorio de Github con el nombre de |
||SONAR\_TOKEN. |
|￿ |Tener una cuenta con suscripción en Azure [(https://portal.azure.com/).](https://portal.azure.com/) |
||Tener el ID de la Suscripción, que se utilizará en el laboratorio |

- Clonar el repositorio mediante git para tener los recursos necesarios en una ubicación que no sea restringida del sistema. 

**DESARROLLO** 

**PREPARACION DE LA INFRAESTRUCTURA** 

1. Iniciar la aplicación Powershell o Windows Terminal en modo administrador, ubicarse en ua ruta donde se ha realizado la clonación del repositorio 

md infra 

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.001.jpeg)

2. Abrir Visual Studio Code, seguidamente abrir la carpeta del repositorio clonado del laboratorio, en el folder Infra, crear el archivo main.tf con el siguiente contenido 

terraform { 

`  `required\_providers { 

`    `azurerm = { 

`      `source  = "hashicorp/azurerm"       version = "~> 4.0.0" 

`    `} 

`  `} 

`  `required\_version = ">= 0.14.9" 

} 

variable "suscription\_id" { 

`    `type = string 

`    `description = "Azure subscription id" } 

variable "sqladmin\_username" {     type = string 

`    `description = "Administrator username for server" } 

variable "sqladmin\_password" { 

`    `type = string 

`    `description = "Administrator password for server" } 

provider "azurerm" { 

`  `features {} 

`  `subscription\_id = var.suscription\_id } 

- Generate a random integer to create a globally unique name resource "random\_integer" "ri" { 

  `  `min = 100 

  `  `max = 999 

  } 

- Create the resource group 

resource "azurerm\_resource\_group" "rg" { 

`  `name     = "upt-arg-${random\_integer.ri.result}"   location = "eastus" 

} 

- Create the Linux App Service Plan 

resource "azurerm\_service\_plan" "appserviceplan" { 

`  `name                = "upt-asp-${random\_integer.ri.result}"   location            = azurerm\_resource\_group.rg.location 

`  `resource\_group\_name = azurerm\_resource\_group.rg.name 

`  `os\_type             = "Linux" 

`  `sku\_name            = "F1" 

} 

- Create the web app, pass in the App Service Plan ID 

resource "azurerm\_linux\_web\_app" "webapp" { 

`  `name                  = "upt-awa-${random\_integer.ri.result}"   location              = azurerm\_resource\_group.rg.location 

`  `resource\_group\_name   = azurerm\_resource\_group.rg.name 

`  `service\_plan\_id       = azurerm\_service\_plan.appserviceplan.id   depends\_on            = [azurerm\_service\_plan.appserviceplan]   //https\_only            = true 

`  `site\_config { 

`    `minimum\_tls\_version = "1.2" 

`    `always\_on = false 

`    `application\_stack { 

`      `docker\_image\_name = "patrickcuadros/shorten:latest" 

`      `docker\_registry\_url = "https://index.docker.io"       

`    `} 

`  `} 

} 

resource "azurerm\_mssql\_server" "sqlsrv" { 

`  `name                         = "upt-dbs-${random\_integer.ri.result}"   resource\_group\_name          = azurerm\_resource\_group.rg.name 

`  `location                     = azurerm\_resource\_group.rg.location 

`  `version                      = "12.0" 

`  `administrator\_login          = var.sqladmin\_username 

`  `administrator\_login\_password = var.sqladmin\_password 

} 

resource "azurerm\_mssql\_firewall\_rule" "sqlaccessrule" {   name             = "PublicAccess" 

`  `server\_id        = azurerm\_mssql\_server.sqlsrv.id 

`  `start\_ip\_address = "0.0.0.0" 

`  `end\_ip\_address   = "255.255.255.255" 

} 

resource "azurerm\_mssql\_database" "sqldb" {   name      = "shorten" 

`  `server\_id = azurerm\_mssql\_server.sqlsrv.id   sku\_name = "Free" 

} 

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.002.jpeg)

3. Abrir un navegador de internet y dirigirse a su repositorio en Github, en la sección *Settings*, buscar la opción *Secrets and Variables* y seleccionar la opción *Actions*. Dentro de esta crear los siguientes secretos 

AZURE\_USERNAME: Correo o usuario de cuenta de Azure AZURE\_PASSWORD: Password de cuenta de Azure SUSCRIPTION\_ID: ID de la Suscripción de cuenta de Azure SQL\_USER: Usuario administrador de la base de datos, ejm: adminsql SQL\_PASS: Password del usuario administrador de la base de datos, ejm: upt.2025 

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.003.jpeg)

5. En el Visual Studio Code, crear la carpeta .github/workflows en la raiz del proyecto, seguidamente crear el archivo deploy.yml con el siguiente contenido 

Click to expand: deploy.yml 

6. En el Visual Studio Code, guardar los cambios y subir los cambios al repositorio. Revisar los logs de la ejeuciòn de automatizaciòn y anotar el numero de identificaciòn de Grupo de Recursos y Aplicación Web creados 

azurerm\_linux\_web\_app.webapp: Creation complete after 53s [id=/subscriptions/1f57de72-50fd-4271-8ab9- 3fc129f02bc0/resourceGroups/upt-arg- XXX/providers/Microsoft.Web/sites/upt-awa-XXX] 

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.004.jpeg)

**CONSTRUCCION DE LA APLICACION** 

1. En el terminal, ubicarse en un ruta que no sea del sistema y ejecutar los siguientes comandos. 

dotnet new webapp -o src -n Shorten 

cd src 

dotnet tool install -g dotnet-aspnet-codegenerator --version 8.0.0 dotnet add package Microsoft.AspNetCore.Identity.UI --version 8.0.0 dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore - -version 8.0.0 

dotnet add package Microsoft.EntityFrameworkCore.Design -- version=8.0.0 

dotnet add package Microsoft.EntityFrameworkCore.SqlServer -- version=8.0.0 

dotnet add package Microsoft.EntityFrameworkCore.Tools --version=8.0.0 dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design -- version=8.0.0 

dotnet add package Microsoft.AspNetCore.Components.QuickGrid -- version=8.0.0 

dotnet add package Microsoft.AspNetCore.Components.QuickGrid.EntityFrameworkAdapter -- version=8.0.0 

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.005.jpeg)

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.006.jpeg)

2. En el terminal, ejecutar el siguiente comando para crear los modelos de autenticación de identidad dentro de la aplicación. 

dotnet aspnet-codegenerator identity –useDefaultUI 

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.007.jpeg)

3. En el VS Code, modificar la cadena de conexión de la base de datos en el archivo appsettings.json, de la siguiente manera: 

"ShortenIdentityDbContextConnection": "Server=tcp:upt-dbs- XXX.database.windows.net,1433;Initial Catalog=shorten;Persist Security Info=False;User ID=YYY;Password=ZZZ;MultipleActiveResultSets=False;Encrypt=True;TrustS erverCertificate=False;Connection Timeout=30;" 

Donde: XXX, id de su servidor de base de datos YYY, usuario administrador de base de datos ZZZ, password del usuario de base de datos 

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.008.jpeg)

4. En el terminal, ejecutar el siguiente comando para crear las tablas de base de datos de identidad. 

dotnet ef migrations add CreateIdentitySchema dotnet ef database update 

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.009.jpeg)

5. En el Visual Studio Code, en la carpeta src/Areas/Domain, crear el archivo UrlMapping.cs con el siguiente contenido: 

namespace Shorten.Areas.Domain; 

/// <summary> 

/// Clase de dominio que representa una acortaciòn de url /// </summary> 

public class UrlMapping 

{ 

`    `/// <summary> 

`    `/// Identificador del mapeo de url 

`    `/// </summary> 

`    `/// <value>Entero</value> 

`    `public int Id { get; set; } 

`    `/// <summary> 

`    `/// Valor original de la url 

`    `/// </summary> 

`    `/// <value>Cadena</value> 

`    `public string OriginalUrl { get; set; } = string.Empty;     /// <summary> 

`    `/// Valor corto de la url 

`    `/// </summary> 

`    `/// <value>Cadena</value> 

`    `public string ShortenedUrl { get; set; } = string.Empty; } 

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.010.jpeg)

6. En el Visual Studio Code, en la carpeta src/Areas/Domain, crear el archivo ShortenContext.cs con el siguiente contenido: 

using Microsoft.EntityFrameworkCore; 

namespace Shorten.Models; 

/// <summary> 

/// Clase de infraestructura que representa el contexto de la base de datos 

/// </summary> 

using Microsoft.EntityFrameworkCore; 

namespace Shorten.Areas.Domain; 

/// <summary> 

/// Clase de infraestructura que representa el contexto de la base de datos 

/// </summary> 

public class ShortenContext : DbContext 

{ 

`    `/// <summary> 

`    `/// Constructor de la clase 

`    `/// </summary> 

`    `/// <param name="options">opciones de conexiòn de BD</param> 

`    `public ShortenContext(DbContextOptions<ShortenContext> options) : base(options) 

`    `{ 

`    `} 

`    `/// <summary> 

`    `/// Propiedad que representa la tabla de mapeo de urls     /// </summary> 

`    `/// <value>Conjunto de UrlMapping</value> 

`    `public DbSet<UrlMapping> UrlMappings { get; set; } 

} 

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.011.jpeg)

7. En el Visual Studio Code, en la carpeta src, modificar el archivo Program.cs con el siguiente contenido al inicio: 

using Microsoft.AspNetCore.Identity; 

using Microsoft.EntityFrameworkCore; 

using Shorten.Areas.Identity.Data; 

using Shorten.Areas.Domain; 

var builder = WebApplication.CreateBuilder(args); 

var connectionString = builder.Configuration.GetConnectionString("ShortenIdentityDbContextCon nection") ?? throw new InvalidOperationException("Connection string 'ShortenIdentityDbContextConnection' not found."); 

builder.Services.AddDbContext<ShortenIdentityDbContext>(options => options.UseSqlServer(connectionString)); 

builder.Services.AddDefaultIdentity<IdentityUser>(options => options.SignIn.RequireConfirmedAccount = true).AddEntityFrameworkStores<ShortenIdentityDbContext>(); 

builder.Services.AddDbContext<ShortenContext>(options => options.UseSqlServer(connectionString)); builder.Services.AddQuickGridEntityFrameworkAdapter(); 

// Add services to the container. builder.Services.AddRazorPages(); 

var app = builder.Build(); 

// Configure the HTTP request pipeline. 

if (!app.Environment.IsDevelopment()) 

{ 

`    `app.UseExceptionHandler("/Error"); 

`    `// The default HSTS value is 30 days. You may want to change this for production scenarios, see https://aka.ms/aspnetcore-hsts. 

`    `app.UseHsts(); 

} 

app.UseHttpsRedirection(); app.UseStaticFiles(); 

app.UseRouting(); app.UseAuthorization(); app.MapRazorPages(); app.Run(); 

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.012.jpeg)

8. En el terminal, ejecutar los siguientes comandos para realizar la migración de la entidad UrlMapping 

dotnet ef migrations add DomainModel --context ShortenContext dotnet ef database update --context ShortenContext 

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.013.jpeg)

9. En el terminal, ejecutar el siguiente comando para crear nu nuevo controlador y sus vistas asociadas. 

dotnet aspnet-codegenerator razorpage Index List -m UrlMapping -dc ShortenContext -outDir Pages/UrlMapping -udl 

dotnet aspnet-codegenerator razorpage Create Create -m UrlMapping -dc ShortenContext -outDir Pages/UrlMapping -udl 

dotnet aspnet-codegenerator razorpage Edit Edit -m UrlMapping -dc ShortenContext -outDir Pages/UrlMapping -udl 

dotnet aspnet-codegenerator razorpage Delete Delete -m UrlMapping -dc ShortenContext -outDir Pages/UrlMapping -udl 

dotnet aspnet-codegenerator razorpage Details Details -m UrlMapping - dc ShortenContext -outDir Pages/UrlMapping -udl 

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.014.jpeg)

10. En el Visual Studio Code, en la carpeta src, modificar el archivo \_Layout.cshtml, Adicionando la siguiente opciòn dentro del navegador: 

<!DOCTYPE html> 

<html lang="en"> 

<head> 

`    `<meta charset="utf-8" /> 

`    `<meta name="viewport" content="width=device-width, initial- scale=1.0" /> 

`    `<title>@ViewData["Title"] - Shorten</title> 

`    `<link rel="stylesheet" href="~/lib/bootstrap/dist/css/bootstrap.min.css" /> 

`    `<link rel="stylesheet" href="~/css/site.css" asp-append- version="true" /> 

`    `<link rel="stylesheet" href="~/Shorten.styles.css" asp-append- version="true" /> 

</head> 

<body> 

`    `<header> 

`        `<nav class="navbar navbar-expand-sm navbar-toggleable-sm navbar-light bg-white border-bottom box-shadow mb-3"> 

`            `<div class="container"> 

`                `<a class="navbar-brand" asp-area="" asp- page="/Index">Shorten</a> 

`                `<button class="navbar-toggler" type="button" data-bs- toggle="collapse" data-bs-target=".navbar-collapse" aria- controls="navbarSupportedContent" 

`                        `aria-expanded="false" aria-label="Toggle navigation"> 

`                    `<span class="navbar-toggler-icon"></span> 

`                `</button> 

`                `<div class="navbar-collapse collapse d-sm-inline-flex justify-content-between"> 

`                    `<ul class="navbar-nav flex-grow-1"> 

`                        `<li class="nav-item"> 

`                            `<a class="nav-link text-dark" asp-area="" asp-page="/Index">Home</a> 

`                        `</li> 

`                        `<li class="nav-item"> 

`                            `<a class="nav-link text-dark" asp-area="" asp-page="/Privacy">Privacy</a> 

`                        `</li> 

`                        `<li class="nav-item"> 

`                            `<a class="nav-link text-dark" asp-area="" asp-page="/UrlMapping/Index">Shorten</a> 

`                        `</li>                     

`                    `</ul> 

`                    `<partial name="\_LoginPartial" /> 

`                `</div> 

`            `</div> 

`        `</nav> 

`    `</header> 

`    `<div class="container"> 

`        `<main role="main" class="pb-3"> 

`            `@RenderBody() 

`        `</main> 

`    `</div> 

`    `<footer class="border-top footer text-muted"> 

`        `<div class="container"> 

`            `&copy; 2025 - Shorten - <a asp-area="" asp- page="/Privacy">Privacy</a> 

`        `</div> 

`    `</footer> 

`    `<script src="~/lib/jquery/dist/jquery.min.js"></script> 

`    `<script src="~/lib/bootstrap/dist/js/bootstrap.bundle.min.js"></script> 

`    `<script src="~/js/site.js" asp-append-version="true"></script> 

`    `@await RenderSectionAsync("Scripts", required: false) </body> 

</html> 

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.015.jpeg)

11. En el Visual Studio Code, en la carpeta raiz del proyecto, crear un nuevo archivo Dockerfile con el siguiente contenido: 
- Utilizar la imagen base de .NET SDK 

FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build 

- Establecer el directorio de trabajo WORKDIR /app 
- Copiar el resto de la aplicación y compilar COPY src/. ./ 

  RUN dotnet restore 

  RUN dotnet publish -c Release -o out 

- Utilizar la imagen base de .NET Runtime 

FROM mcr.microsoft.com/dotnet/aspnet:8.0-alpine AS runtime LABEL org.opencontainers.image.source="https://github.com/p- cuadros/Shorten02" 

- Establecer el directorio de trabajo 

WORKDIR /app 

ENV ASPNETCORE\_URLS=http://+:80 

RUN apk add icu-libs 

ENV DOTNET\_SYSTEM\_GLOBALIZATION\_INVARIANT=false 

- Copiar los archivos compilados desde la etapa de construcción COPY --from=build /app/out . 
- Definir el comando de entrada para ejecutar la aplicación ENTRYPOINT ["dotnet", "Shorten.dll"] 

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.016.jpeg)

**DESPLIEGUE DE LA APLICACION** 

1. En el terminal, ejecutar el siguiente comando para obtener el perfil publico (Publish Profile) de la aplicación. Anotarlo porque se utilizara posteriormente. 

az webapp deployment list-publishing-profiles --name upt-awa-XXX -- resource-group upt-arg-XXX --xml 

Donde XXX; es el numero de identicación de la Aplicación Web creada en la primera sección 

2. Abrir un navegador de internet y dirigirse a su repositorio en Github, en la sección *Settings*, buscar la opción *Secrets and Variables* y seleccionar la opción *Actions*. Dentro de esta hacer click en el botón *New Repository Secret*. En el navegador, dentro de la ventana *New Secret*, colocar como nombre AZURE\_WEBAPP\_PUBLISH\_PROFILE y como valor el obtenido en el paso anterior. 
3. En el Visual Studio Code, dentro de la carpeta .github/workflows, crear el archivo ci-cd.yml con el siguiente contenido 

name: Construcción y despliegue de una aplicación MVC a Azure 

env: 

`  `AZURE\_WEBAPP\_NAME: upt-awa-XXX  # Aqui va el nombre de su aplicación   DOTNET\_VERSION: '8'                     # la versión de .NET 

on: 

`  `push: 

`    `branches: [ "main" ] 

`    `paths: 

- 'src/\*\*' 
- '.github/workflows/\*\*' 

`  `workflow\_dispatch: 

permissions: 

`  `contents: read 

`  `packages: write 

jobs: 

`  `build: 

`    `runs-on: ubuntu-latest 

`    `steps: 

- uses: actions/checkout@v4 
- name: 'Login to GitHub Container Registry' 

`        `uses: docker/login-action@v3 

`        `with: 

`            `registry: ghcr.io 

`            `username: ${{github.actor}} 

`            `password: ${{secrets.GITHUB\_TOKEN}} 

- name: 'Build Inventory Image' 

`        `run: | 

`            `docker build . --tag ghcr.io/${{github.actor}}/shorten:${{github.sha}}             docker push ghcr.io/${{github.actor}}/shorten:${{github.sha}} 

`  `deploy: 

`    `permissions: 

`      `contents: none 

`    `runs-on: ubuntu-latest 

`    `needs: build 

`    `environment: 

`      `name: 'Development' 

`      `url: ${{ steps.deploy-to-webapp.outputs.webapp-url }} 

`    `steps: 

- name: Desplegar a Azure Web App 

`        `id: deploy-to-webapp 

`        `uses: azure/webapps-deploy@v2 

`        `with: 

`          `app-name: ${{ env.AZURE\_WEBAPP\_NAME }} 

`          `publish-profile: ${{ secrets.AZURE\_WEBAPP\_PUBLISH\_PROFILE }}           images: ghcr.io/${{github.actor}}/shorten:${{github.sha}} 

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.017.jpeg)

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.018.jpeg)

4. En el Visual Studio Code o en el Terminal, confirmar los cambios con sistema de controlde versiones (git add ... git commit...) y luego subir esos cambios al repositorio remoto (git push ...). 

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.019.png)

5. En el Navegador de internet, dirigirse al repositorio de Github y revisar la seccion Actions, verificar que se esta ejecutando correctamente el Workflow. 

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.020.jpeg)

6. En el Navegador de internet, una vez finalizada la automatización, ingresar al sitio creado y navegar por el [(https://upt-awa- XXX.azurewebsites.net)](https://upt-awa-xxx.azurewebsites.net/). 
6. En el Terminal, revisar las metricas de navegacion con el siguiente comando. 

az monitor metrics list --resource "/subscriptions/XXXXXXXXXXXXXXX/resourceGroups/upt-arg- XXX/providers/Microsoft.Web/sites/upt-awa-XXXX" --metric "Requests" -- start-time 2025-01-07T18:00:00Z --end-time 2025-01-07T23:00:00Z -- output table 

Reemplazar los valores: 1. ID de suscripcion de Azure, 2. ID de creaciòn de infra y 3. El rango de fechas de uso de la aplicación. 

7. En el Terminal, ejecutar el siguiente comando para obtener la plantilla de los recursos creados de azure en el grupo de recursos UPT. 

az group export -n upt-arg-XXX > lab\_01.json 

![](img/Aspose.Words.d136312e-2d7e-4f7f-bfab-caead5710454.021.jpeg)

8. En el Visual Studio Code, instalar la extensión *ARM Template Viewer*, abrir el archivo lab\_02.json y hacer click en el icono de previsualizar ARM. 

**ACTIVIDADES ENCARGADAS** 

1. Subir el diagrama al repositorio como lab\_02.png y el reporte de metricas. 
1. Realizar el scanero del codigo de terraform utilizando TfSec o Trivy dentro del Github Action. 
1. En la aplicación completar el envio de correo para el registro de usuarios [(https://learn.microsoft.com/es-](https://learn.microsoft.com/es-es/aspnet/core/security/authentication/accconfirm?view=aspnetcore-9.0&tabs=visual-studio)



||[es/aspnet/core/security/authentication/accconfirm?view=aspnetcore-](https://learn.microsoft.com/es-es/aspnet/core/security/authentication/accconfirm?view=aspnetcore-9.0&tabs=visual-studio)|
| :- | - |
||[9.0&tabs=visual-studio)](https://learn.microsoft.com/es-es/aspnet/core/security/authentication/accconfirm?view=aspnetcore-9.0&tabs=visual-studio) |
|4\. |En la aplicación migrar la cadena de conexion a la base de datos a una |
||Configuración de aplicación de Azure, como una variable de ambiente. |
|5\. |Realizar el escaneo de vulnerabilidad con SonarCloud y Semgrep dentro |
||del Github Action correspondiente. |

