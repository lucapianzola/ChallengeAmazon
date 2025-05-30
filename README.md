
# Automatización ChallengeAmazon

**obtencion de precios automatizada de productos de Amazon con UiPath y Robotic Enterprise Framework (REFramework)**

---

## Objetivo de Negocio
El objetivo principal de esta solución RPA asistida es **obtener precios automáticamente productos en Amazon** para zonas de envío específicas, extrayendo precio, costo de entrega, fecha estimada de llegada y valoraciones de clientes. Esto reduce el esfuerzo manual, asegura consistencia de datos y permite comparar rápidamente ofertas por región.

## Descripción del Proyecto
Esta solución está desarrollada en **UiPath Studio** usando **REFramework** y contempla componentes personalizados:
- **FormatInput**: Normaliza los datos de entrada desde Excel/PDF.
- **InitAmazonWeb**: Inicializa la sesión en Amazon, ajusta idioma y moneda.
- **FiltradoProductos**: Aplica filtros de precio, fecha y rating tras la extracción.

## Configuración Detallada

Todos los parámetros están en `Data/Config.xlsx` (o Assets de Orchestrator):

| Hoja         | Descripción                                     |
| ------------ | ----------------------------------------------- |
| Settings     | Nombres de colas, timeouts, rutas de archivos   |
| Credentials  | Claves para Windows Credential Manager o Assets |
| Applications | URL base, códigos de idioma/moneda              |

## Arquitectura del Flujo de Trabajo

Se utiliza el patrón **REFramework** extendido con componentes propios:

1. **InitAllSettings.xaml** – Carga Configuración y Credenciales
2. **FormatInput.xaml** – Limpieza y normalización de datos de `datos_buscar.xlsx`
3. **InitAllApplications.xaml** – Llama a **InitAmazonWeb** para abrir Amazon y configurar idioma/moneda
4. **GetTransactionData.xaml** – Lee Excel o elementos de Cola
5. **Process.xaml** – Para cada producto:

   * Validación de idioma/moneda
   * Búsqueda y selección del primer resultado
   * Extracción de URL, precio, costo de entrega y fecha de llegada (dd/MM/yyyy)
   * Cambio de zona de entrega; en zonas no soportadas lanza BusinessRuleException
   * Extracción de tabla de valoraciones vía Data Scraping
6. **FiltradoProductos.xaml** – Aplica filtros de precio, fecha de llegada y rating
7. **SetTransactionStatus.xaml** – Marca Success, BusinessException o SystemException
8. **CloseAllApplications.xaml** – Cierra aplicaciones de forma ordenada

## Estructura del Proyecto

```text
ChallengeAmazon/
├─ Data/
│  └─ Config.xlsx            # Settings, Credentials, Applications
├─ Documentation/            # PDD, mapas de proceso, plantillas
├─ Exceptions_Screenshots/   # Capturas de error automáticas
├─ Framework/
│  ├─ InitAllSettings.xaml
│  ├─ FormatInput.xaml       # Normalización de datos
│  ├─ InitAllApplications.xaml
│  ├─ InitAmazonWeb.xaml     # Inicialización de Amazon
│  ├─ GetTransactionData.xaml
│  ├─ Process.xaml
│  ├─ FiltradoProductos.xaml # Filtros de negocio
│  ├─ SetTransactionStatus.xaml
│  └─ CloseAllApplications.xaml
├─ Input/
│  └─ datos_buscar.xlsx      # Generado desde PDF fuente
├─ Output/
│  └─ output_datos_buscar.xlsx # Resultados con hojas filtradas
├─ Tests/                    # Pruebas unitarias de workflows
├─ Main.xaml                 # Punto de entrada REFramework
└─ project.json              # Descriptor del proyecto UiPath
```

## Especificaciones de Entrada y Salida

* **Archivo de Entrada**: `Productos.pdf`

  * Columnas: `Nombre producto`, `Zona de envio`
* **Archivo de Salida**: `ProductosFormateados.xlsx`

  * Columnas: `Nombre producto`, `URL`, `Precio`, `Precio delivery`, `Precio total`, `Fecha de llegada`, `Rating`

## Filtrado e Informes

Tras procesar todos los productos, el bot aplica filtros con **FiltradoProductos**:

1. **Filtro de Precio**: Productos con `Precio Producto` entre umbrales configurados.
2. **Filtro Fecha de Entrega & Rating**: Ordena por porcentaje de valoración para fecha/estrella dada.

Los resultados se guardan en pestañas separadas del libro de salida.

## Ejecución de la Automatización

1. Actualizar `Data/Config.xlsx` con valores de tu entorno.
2. Colocar `datos_buscar.xlsx` en la carpeta `Input/`.
3. Abrir `Main.xaml` en UiPath Studio y hacer clic en **Run**.
4. Ver resultados en `Output/output_datos_buscar.xlsx` y logs en `%LOCALAPPDATA%\UiPath\Logs`.

## Pruebas

* Ejecutar flujos en `Tests/` desde UiPath Test Explorer.
* Verificar escenarios clave (zonas no disponibles, extracción de datos).

## Solución de Problemas y Excepciones

| Paso       | Excepción                    | Acción                                           |
| ---------- | ---------------------------- | ------------------------------------------------ |
| 2          | Producto no disponible       | Registrar datos con `-`                          |
| 3.1        | Zona de entrega no soportada | marcar BusinessRuleException                     |
| Cualquiera | Error de sistema inesperado  | Enviar captura y notificación por correo         |
