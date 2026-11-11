# ERP Frescura Total

Sistema de consola para administrar una bodega de alimentos perecederos. Permite registrar mercancía, revisar los lotes, vender productos, generar facturas de prueba y organizar las entregas.

**Autor:** Kamilo Andres Jimenez  
**Programa o ficha:** [Completar]  
**Trainer:** Cristian Diaz

## Sobre el proyecto

Frescura Total distribuye alimentos a restaurantes y tiendas. El problema es que llevar el inventario, las ventas y las entregas por separado hace difícil saber cuánto queda de un producto, qué lote vence primero o a qué cliente se le entregó.

La idea del sistema es reunir esa información en una misma base de datos. Cada venta queda relacionada con los lotes utilizados, y cada lote conserva el proveedor del que salió y sus movimientos.

Los objetivos del proyecto son:

- Mantener actualizado el inventario por lotes.
- Dar salida primero a los productos que están más cerca de vencer.
- Registrar las pérdidas y las revisiones de calidad.
- Evitar ventas sin stock o de productos que no sean aptos.
- Seguir los pedidos desde su preparación hasta la entrega.

## Herramientas utilizadas

| Herramienta | Uso |
|---|---|
| Java 17 | Lenguaje de la aplicación |
| Maven | Dependencias, compilación y pruebas |
| MySQL 8 e InnoDB | Base de datos y transacciones |
| MySQL Connector/J 8.4.0 | Conexión JDBC |
| JUnit 5.11.4 | Pruebas automatizadas |
| H2 2.3.232 | Base en memoria para las pruebas rápidas |
| PDFBox 2.0.32 | Generación de facturas PDF |
| ZXing 3.5.3 | Generación del QR de prueba |
| Graphviz | Diagramas de clases y de base de datos |
| Git y GitHub | Control de versiones y entrega del código |

El proyecto se abre en NetBeans como una aplicación Java Maven.

## Qué se puede hacer

### Inventario y calidad

Registrar una recepción con producto, proveedor, cantidad, precios, fechas, ubicación y código de lote. También se pueden consultar existencias, poner un lote en cuarentena, resolver una inspección y registrar mermas.

Al iniciar la aplicación se revisan los vencimientos. Esa revisión también se puede ejecutar desde el menú. Los lotes en cuarentena conservan su estado hasta que un inspector los revise.

### Proveedores y compras

Registrar, consultar y editar proveedores, desactivarlos y asociarles productos. El historial muestra las recepciones que se les han comprado. La desactivación conserva los registros anteriores.

En esta versión, cada recepción se registra como una compra de una línea.

### Clientes y ventas

Registrar clientes mayoristas o minoristas y crear pedidos con varios productos. La aplicación comprueba las existencias y elige los lotes por FEFO: primero se utilizan los que vencen antes.

Si el pedido necesita más de un lote, se guarda el detalle de cuánto salió de cada uno. Los lotes próximos a vencer reciben un descuento configurable del 50 % por defecto.

### Facturación

Consultar las facturas y generar su PDF. Cada documento incluye cliente, productos, cantidades, precios, descuentos, impuestos y total, además de un QR y un CUFE simulados.

Volver a generar un PDF no registra otra venta ni descuenta inventario de nuevo.

### Logística

Registrar vehículos y conductores, consultar pedidos pendientes y crear rutas con uno o varios pedidos. Después se puede registrar la salida y la entrega de cada pedido, incluyendo quién lo recibió.

### Usuarios y trazabilidad

El acceso depende del rol del usuario. La consulta de trazabilidad muestra el origen del lote, sus fechas, stock, ventas, clientes y movimientos.

## Antes de ejecutar

Se necesita un JDK 17, Maven y MySQL Server 8 en funcionamiento. NetBeans es el IDE previsto para trabajar con el proyecto; también se puede ejecutar desde una terminal.

### Instalar las herramientas

1. Descarga un JDK 17 desde [Eclipse Temurin](https://adoptium.net/temurin/releases/?version=17). Configura `JAVA_HOME` con la carpeta del JDK y añade su carpeta `bin` al `PATH`.
2. Instala [Maven](https://maven.apache.org/download.cgi) y añade también su carpeta `bin` al `PATH`.
3. Instala [NetBeans](https://netbeans.apache.org/front/main/download/) con soporte para Java 17.
4. Instala [MySQL Community Server 8](https://dev.mysql.com/downloads/mysql/8.0.html). Durante la instalación configura la contraseña de `root` e inicia el servicio. Puedes instalar Workbench para administrar la base de datos.
5. Instala [Git](https://git-scm.com/downloads) para subir el proyecto al repositorio.

Abre una terminal nueva y revisa las versiones:

```bash
java -version
javac -version
mvn -version
git --version
```

Java y `javac` deben indicar la versión 17. Revisa también que Maven esté usando ese JDK. La primera compilación necesita Internet para descargar las dependencias.

Graphviz solo hace falta si se quieren volver a generar los diagramas; no es necesario para usar el ERP.

## Preparar la base de datos

La base parte del SQL entregado por el docente. La copia original está en `docs/original.sql`, y el análisis de sus tablas y de los cambios está en `docs/database-analysis.md`.

Se conservaron las 22 tablas originales con sus nombres en español. Se añadieron cuatro tablas para usuarios, compras, recepciones y movimientos, además de algunas restricciones y campos para conservar los datos históricos de las facturas.

Para instalar el proyecto utiliza **`docs/database.sql` en una base vacía**. El archivo contiene una sección DDL para la estructura y otra DML para los datos de prueba. No debe importarse otra vez sobre una instalación que ya tenga datos.

Desde una terminal:

```bash
mysql -u root -p
```

Dentro del cliente MySQL, ejecuta lo siguiente ajustando la ruta a tu equipo:

```sql
SOURCE C:/proyectos/frescura-total-erp/docs/database.sql;

CREATE USER 'frescura_app'@'localhost' IDENTIFIED BY 'CAMBIA_ESTA_CLAVE';
GRANT SELECT, INSERT, UPDATE ON erp_frescura_total.* TO 'frescura_app'@'localhost';
```

En Linux, la ruta podría ser `/home/tuusuario/proyectos/frescura-total-erp/docs/database.sql`.

Si usas Workbench, abre `docs/database.sql`, selecciona la conexión al servidor y ejecuta el archivo completo.

**No uses `original.sql` para instalar la aplicación.** Se conserva como referencia: tiene el `DROP DATABASE` del docente y no incluye las tablas complementarias.

### Datos incluidos

El script carga 50 proveedores, 50 productos, 50 clientes, 200 lotes y 50 ventas con sus detalles y facturas. También incluye compras, inspecciones, mermas, movimientos y 24 rutas con pedidos en distintos estados.

Los lotes empiezan con 100 unidades. En los próximos a vencer se registra una venta de 2, por lo que quedan 98; en los vencidos se registra una merma de 5 y quedan 95. Las fechas se calculan según el día de importación.

Por requisito de la consigna, el proveedor con ID 33 tiene el identificador `SYN-FT-900427-AG`. Es un dato ficticio de prueba, no un NIT colombiano. Al registrar proveedores desde la aplicación sí se valida el formato y el dígito de verificación del NIT.

## Configurar la conexión

Los valores generales están en `src/main/resources/application.properties`. Para escribir las credenciales de tu equipo, crea un archivo llamado `application-local.properties` en la raíz del proyecto:

```properties
db.url=jdbc:mysql://localhost:3306/erp_frescura_total?serverTimezone=America/Bogota&characterEncoding=UTF-8
db.user=frescura_app
db.password=CAMBIA_ESTA_CLAVE
sales.taxRate=0.19
sales.nearExpiryDiscount=0.50
invoices.directory=output/invoices
```

Cambia la contraseña por la que asignaste al usuario de MySQL. Este archivo está incluido en `.gitignore` para evitar subir las credenciales al repositorio.

También se pueden usar variables de entorno. Tienen prioridad sobre los archivos de propiedades:

| Propiedad | Variable de entorno |
|---|---|
| `db.url` | `DB_URL` |
| `db.user` | `DB_USER` |
| `db.password` | `DB_PASSWORD` |
| `sales.taxRate` | `SALES_TAXRATE` |
| `sales.nearExpiryDiscount` | `SALES_NEAREXPIRYDISCOUNT` |
| `invoices.directory` | `INVOICES_DIRECTORY` |

El 19 % es una tasa de impuesto general usada para este ejercicio. No representa el tratamiento tributario de cada alimento. La tasa y el descuento afectan a las ventas nuevas; las facturas anteriores conservan sus valores.

La tabla `promociones` del modelo original se conserva con un registro de referencia. En esta aplicación el descuento se configura en las propiedades.

## Ejecutar desde NetBeans

1. Selecciona **File → Open Project** y abre la carpeta que contiene `pom.xml`.
2. Espera a que Maven descargue las dependencias.
3. Configura el proyecto para usar JDK 17. Si no aparece, agrégalo en **Tools → Java Platforms**.
4. Ejecuta **Clean and Build**.
5. Ejecuta la clase `com.frescuratotal.erp.Main`.
6. Introduce los datos en la consola del IDE.

En la consola de NetBeans la contraseña puede verse mientras se escribe. Al ejecutar desde una terminal compatible se utiliza entrada oculta.

El proyecto se compiló y ejecutó con Maven y Java 17. La apertura en NetBeans y la ejecución en Windows quedan por comprobar en el equipo donde se vaya a presentar.

## Ejecutar desde la terminal

Ubícate en la carpeta donde está `pom.xml`:

```bash
mvn clean test
mvn clean package
java -jar target/frescura-total-erp-1.0.0.jar
```

También se puede iniciar con:

```bash
mvn exec:java
```

El ZIP incluye una copia compilada en `dist/`. Después de importar la base y configurar la conexión, puedes ejecutarla con:

```bash
java -jar dist/frescura-total-erp-1.0.0.jar
```

Ejecuta los comandos desde la raíz del proyecto para que la aplicación encuentre la configuración local.

## Usuarios de prueba

La contraseña de los cinco usuarios es **`Frescura2026!`**. Estas cuentas son únicamente para desarrollo.

| Usuario | Rol | Funciones principales |
|---|---|---|
| `admin` | ADMIN | Administración y acceso a los módulos operativos |
| `warehouse` | WAREHOUSE_OPERATOR | Proveedores, recepción, inventario y mermas |
| `sales` | SALES_AGENT | Clientes, ventas y facturas |
| `inspector` | INSPECTOR | Inspecciones y resolución de cuarentenas |
| `logistics` | LOGISTICS_MANAGER | Vehículos, conductores, rutas y entregas |

Solo `INSPECTOR` puede aprobar o rechazar un lote en cuarentena. Esta regla también se aplica al administrador, quien puede crear una cuenta con el rol de inspector si hace falta.

Las contraseñas se guardan con PBKDF2-HMAC-SHA256, 210000 iteraciones y una sal aleatoria al crear cada usuario.

## Cómo usar los menús

El menú muestra las opciones disponibles para el usuario que inició sesión. Mantiene la numeración general de la consigna. Cada operación regresa al menú principal; `0` sirve para volver o salir. Las cantidades decimales se escriben con punto, por ejemplo `2.50`.

Para probar el flujo completo:

1. Entra con `warehouse`. Consulta productos y proveedores en la opción **2**. Para recibir mercancía usa **1 → 2** e indica un producto asociado al proveedor.
2. Si el lote necesita revisión, ponlo en cuarentena con **1 → 4**. Entra con `inspector` y usa **1 → 5** para aprobarlo o rechazarlo. Un rechazo registra la baja de las unidades restantes.
3. Entra con `sales`. Consulta los clientes en **3 → 1**. Para crear una venta usa **3 → 3**, selecciona cliente y agrega los productos con sus cantidades. Escribe `0` al terminar y luego `CONFIRMAR`.
4. Revisa el PDF generado en `output/invoices/`. Desde la opción **4** puedes consultar las facturas y volver a generar una usando el ID de venta.
5. Entra con `logistics`. En la opción **5** registra los recursos, crea una ruta y asigna pedidos. Avanza cada entrega a `IN_TRANSIT` y después a `DELIVERED`, indicando el receptor.
6. Usa la opción **6** para consultar el origen y los movimientos de un lote.

Con los datos recién importados, el producto 1 tiene un lote próximo a vencer con ID 2 y 98 unidades, y otro disponible con ID 1 y 100 unidades. Una venta de 100 utiliza ambos lotes; una de 199 se rechaza si no se han registrado nuevas recepciones.

## Reglas que conviene tener presentes

Un lote que vence hoy o dentro de los próximos siete días queda en `NEAR_EXPIRY`. Si venció antes de hoy, queda en `EXPIRED`. La actualización de fechas no saca un lote de cuarentena.

Para vender se aceptan los estados `AVAILABLE` y `NEAR_EXPIRY`. Además del estado, se comprueba la fecha de vencimiento. Así se bloquea un lote vencido aunque todavía no se haya actualizado su estado.

FEFO ordena por vencimiento y usa el ID de lote para desempatar. El descuento se calcula por cada línea: primero el valor bruto, después el descuento y finalmente el impuesto sobre el valor neto. Los cálculos usan `BigDecimal` y se redondean a dos decimales con `HALF_UP`.

Al confirmar una venta se bloquean los productos por ID ordenado y los lotes con `SELECT ... FOR UPDATE`. El registro de la venta, sus detalles, el descuento de existencias, los movimientos, la factura y el paso a preparación se realizan dentro de la misma transacción. Si falta stock o falla el PDF, se hace rollback.

Las entregas siguen esta secuencia:

```text
CONFIRMED → IN_PREPARATION → IN_TRANSIT → DELIVERED
```

Una venta nueva queda en preparación al terminar su confirmación. Para salir a tránsito necesita una ruta. Una ruta solo se completa cuando se entregaron todos sus pedidos, y no se permite asignar un vehículo o conductor que ya esté ocupado en otra ruta activa.

## Sobre las facturas

Los PDF quedan en `output/invoices/`. El número tiene el formato `FT-00000001` y se forma con el ID de venta. Puede haber saltos en la numeración cuando una transacción consume un autonumérico y luego se revierte.

La factura conserva una copia de los datos del cliente y del nombre de los productos. Por eso, editar esos registros después no cambia los textos de una factura anterior. En `docs/sample-invoice.pdf` hay un ejemplo generado durante la comprobación del programa.

El NIT de la empresa está identificado como dato de demostración. El QR y el CUFE son simulados: **la factura no está validada por la DIAN ni tiene validez fiscal**.

## Organización del código

Los paquetes principales son `common`, `inventory`, `suppliers`, `sales`, `logistics` y `users`. Dentro de los módulos se separan los datos, las vistas, los controladores y los servicios. Los DAO específicos están en `repository`.

La separación MVC se usa de esta forma:

- **Modelo:** datos del dominio y servicios que aplican las reglas del negocio.
- **Vista:** menús y entrada/salida de consola, apoyados en `ConsoleView`.
- **Controlador:** recoge los datos de la vista y llama al servicio correspondiente.

`Main` crea los objetos, inicia la sesión y dirige la navegación. Las consultas SQL no están en los menús.

Para acceder a la base se usa el patrón DAO. `JdbcDao` reúne el código repetido de JDBC; `SupplierDao` e `InventoryDao` contienen operaciones propias de esos módulos. Algunos servicios pequeños usan directamente el DAO común dentro de su transacción.

Los datos de entrada se agrupan en `record`, y parte de los resultados se lee como mapas. En una versión más grande convendría convertir más consultas a entidades tipadas para reducir errores con los nombres de columnas.

### SOLID en el proyecto

| Principio | Ejemplo en el código |
|---|---|
| Responsabilidad única | `PasswordHasher` se ocupa de contraseñas, `PricingService` de los cálculos y `PdfInvoiceService` del documento |
| Abierto/cerrado | Otro generador puede implementar `InvoiceRenderer` sin cambiar el algoritmo de venta |
| Sustitución | El generador real y el de prueba cumplen el contrato de `InvoiceRenderer` |
| Segregación de interfaces | `InvoiceRenderer` tiene una sola operación: `render` |
| Inversión de dependencias | `SaleService` recibe `InvoiceRenderer` en lugar de crear directamente un generador PDF |

No se implementa Singleton. Los objetos se crean de forma explícita y comparten `Database`. Su `ConnectionFactory` permite usar una conexión distinta en las pruebas.

## Diagramas

Los archivos están en `docs/`:

- `diagrama_er.png`: las 26 tablas finales con sus atributos, PK, FK y cardinalidades.
- `diagrama_clases.png`: principales datos, servicios y controladores, con atributos, métodos y relaciones.
- `diagrama_er.dot` y `diagrama_clases.dot`: fuentes editables.

Los diagramas muestran las relaciones que existen en la implementación. Los campos originales que guardan el nombre de un usuario como texto no se representan como claves foráneas. En el diagrama de clases se incluye la implementación de `InvoiceRenderer`; no hay una jerarquía de herencia de dominio.

Para exportar de nuevo las imágenes, con Graphviz instalado:

```bash
dot -Tpng docs/diagrama_er.dot -o docs/diagrama_er.png
dot -Tpng docs/diagrama_clases.dot -o docs/diagrama_clases.png
```

## Pruebas

Se ejecutaron 31 pruebas JUnit con JDK 17 y Maven 3.9.9. La misma batería se probó primero con H2 y después con MySQL Community Server 8.0.46.

| Comprobación | Resultado |
|---|---|
| 31 pruebas con H2 | 0 fallidas, 0 errores, 0 omitidas |
| Las mismas 31 pruebas con MySQL 8 | 0 fallidas, 0 errores, 0 omitidas |
| `mvn clean test` | Correcto |
| `mvn clean package` | Correcto; se generó el JAR con dependencias |

Las pruebas cubren recepción, proveedores, códigos de lote únicos, vencimientos, cuarentena, permisos, inspecciones, mermas, FEFO, descuentos, ventas con varios lotes, trazabilidad, facturas, entregas e integridad referencial.

También se comprueba que una venta se revierta si falta un producto o falla el PDF, y que dos ventas simultáneas no puedan consumir las mismas unidades. Las pruebas de factura revisan importes, paginación y reimpresión sin volver a descontar stock.

Se hicieron además seis recorridos del JAR por consola sobre MySQL: recepción y cuarentena; aprobación; cliente y venta; ruta y entrega; factura y trazabilidad; y denegación de una opción a un usuario sin permiso. Los informes y registros de ejecución están en `docs/validation/`.

Estos resultados corresponden a los casos probados. La ejecución en NetBeans y Windows no se realizó en este entorno.

### Volver a ejecutar las pruebas

Para las pruebas rápidas no hace falta iniciar MySQL:

```bash
mvn clean test
```

Se usa una base H2 en memoria con las mismas tablas y datos, adaptando la sintaxis necesaria. El archivo es `src/test/resources/h2-fixture.sql`.

Para comprobar contra MySQL, crea una base separada llamada `ft_test` y usa una cuenta con permisos para crear y borrar tablas. **La batería borra y recrea las tablas de la base indicada antes de cada prueba. No uses la base de trabajo.**

En PowerShell:

```powershell
$env:FT_TEST_MYSQL_URL='jdbc:mysql://localhost:3306/ft_test?serverTimezone=America/Bogota'
$env:FT_TEST_MYSQL_USER='tu_usuario_de_pruebas'
$env:FT_TEST_MYSQL_PASSWORD='tu_clave_de_pruebas'
$env:FT_TEST_MYSQL_RESET='YES'
mvn test
Remove-Item Env:FT_TEST_MYSQL_URL, Env:FT_TEST_MYSQL_USER, Env:FT_TEST_MYSQL_PASSWORD, Env:FT_TEST_MYSQL_RESET
```

En Linux o macOS se pueden definir las mismas variables con `export` y retirarlas con `unset`. El reinicio de tablas solo se habilita si `FT_TEST_MYSQL_RESET` tiene el valor `YES`.

## Alcance y pendientes

Es un proyecto académico de consola. Las compras se registran al recibir mercancía; no se manejan órdenes de compra pendientes ni cuentas por pagar. El impuesto es general y el descuento se cambia en la configuración.

La logística controla las asignaciones y los estados, pero no calcula recorridos geográficos ni compara la capacidad del vehículo con productos que tengan unidades de medida diferentes.

Hay otros puntos que se podrían ampliar:

- Recuperación de contraseñas, administración avanzada de usuarios y copias de seguridad automáticas.
- Más entidades tipadas para los resultados de las consultas.
- Reintentos controlados cuando ocurra un deadlock o se pierda la conexión.
- Mejor distribución de textos largos en el PDF, donde algunos nombres pueden abreviarse.

MySQL y los archivos PDF no comparten una transacción distribuida. El programa revierte la venta si falla el PDF y elimina el archivo si falla el commit, pero una caída brusca puede dejar un PDF sin una venta confirmada. Las facturas guardadas en la base se pueden regenerar.

Un lote rechazado conserva la inspección y la merma y queda sin unidades. Al revisar las fechas puede cambiar su estado de vencimiento, pero eso no recupera su stock. Los permisos de la aplicación tampoco sustituyen los permisos de MySQL para quien acceda directamente a la base.

## Subir a GitHub

El repositorio debe ser privado. Antes de subirlo, completa los datos del autor al inicio del README. El archivo `.gitignore` excluye la configuración local, los compilados y las facturas generadas.

Desde la raíz del proyecto, configura tus datos reales:

```bash
git init -b main
git config user.name "TU NOMBRE REAL"
git config user.email "TU CORREO REAL"
git add .
git commit -m "Implement Frescura Total ERP"
```

Crea un repositorio privado vacío en GitHub y sustituye los datos de esta URL por los de tu cuenta y repositorio:

```bash
git remote add origin https://github.com/TU_USUARIO/TU_REPOSITORIO.git
git push -u origin main
git rev-parse HEAD
```

`git rev-parse HEAD` muestra el hash del commit que acabas de subir. No se incluye una URL ni un hash de entrega porque la publicación en la cuenta del estudiante todavía no se ha realizado.

Para dar acceso al trainer, entra al repositorio y abre **Settings → Collaborators → Add people**. Busca su usuario o correo confirmado y envía la invitación. Debe aceptarla para poder entrar al repositorio privado.

Para estudiar el flujo del código y practicar una presentación, consulta `docs/learning-guide.md`.
