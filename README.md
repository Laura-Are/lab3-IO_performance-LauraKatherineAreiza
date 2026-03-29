# laboratorio # 3 - IO performance
Laura KAtherine Areiza Henao - 1042150762

## Preparación del entorno experimental

### Identificación de la Tecnología de Almacenamiento

Para identificar el tipo de unidad física del sistema, se utilizó PowerShell en Windows ejecutado como administrador con el siguiente comando:

```
<Get-PhysicalDisk | Select-Object FriendlyName, MediaType, BusType>
```

A continuación, se muestra la evidencia del resultado:

<p style="text-align: center;">
  <img src="images/Identificación_de_la_Tecnología_de_Almacenamiento.png" width="600"/>
</p>

De acuerdo con los valores obtenidos:

MediaType: SSD
BusType: NVMe

Esto indica que mi equipo cuenta con una unidad de estado sólido (SSD) que utiliza la tecnología NVMe.

### Registro de Especificaciones del Sistema

Antes de iniciar las pruebas, se registran las siguientes características del entorno de ejecución. Esta información permite contextualizar posibles variaciones en los tiempos de respuesta.

| Parámetro                 | Valor de Referencia |
|---------------------------|---------------------|
| Sistema Operativo         | Windows 11          |
| CPU (Modelo y Núcleos)    | 12th Gen Intel Core i5-1235U / 10 núcleos |
| Memoria RAM Total         | 8 GB                |
| Tipo de Disco             | SSD NVMe            |
| Carga de CPU en Reposo    | 4% - 11%            |


```
CPU : Get-CimInstance Win32_Processor | Select-Object Name, NumberOfCores

RAM : Get-CimInstance Win32_ComputerSystem | Select-Object TotalPhysicalMemory
```

<p style="text-align: center;">
  <img src="images/Paso_2_Info_CPU_RAM_Tabla.png" width="600"/>
</p>


Referencias de Rendimiento Teórico
Utilice estos valores como línea base para validar si sus resultados son coherentes con la teoría:

|Tecnología |Latencia Promedio|	Throughput Típico|IOPS Típico (4 KB aleatorio)|Escala de Tiempo|
|-----------|-----------------|------------------|----------------------------|----------------|
|HDD        |	10 ms	      |100 - 150 MB/s    |75 – 300	                  |Milisegundos    |
|SSD (SATA) |	100 µs        |	500 - 550 MB/s	 |50,000 – 100,000            |	Microsegundos  |
|SSD NVMe   |	10 - 20 µs    |	2 - 7 GB/s	     |500,000 – 1,000,000+        |	Microsegundos  |


> [!WARNING]
> **Preparación antes del benchmark:**
> 
> - Cerrar aplicaciones de alto consumo (navegadores, IDEs, etc.).
> - Verificar que no haya actualizaciones en segundo plano.
> - Evitar el uso de máquinas virtuales o contenedores.
> - Utilizar archivos grandes para evitar lecturas desde caché.
> - Realizar accesos dispersos para reducir la pre-lectura.
> - Ejecutar cada prueba de forma independiente.

## Actividad del laboratorio.

### Etapa 1

En esta etapa se registran las características del equipo utilizado para la ejecución del experimento. Esta información permite contextualizar los resultados obtenidos y entender posibles variaciones en el rendimiento.

| Parámetro                        | Valor Observado |
|----------------------------------|-----------------|
| Sistema Operativo                | Windows 11      |
| CPU (Modelo y Frecuencia)        | Intel Core i5-1235U |
| Arquitectura y Núcleos           | x64 / 10 núcleos |
| Memoria RAM Total                | 8 GB            |
| Tecnología de Almacenamiento     | SSD NVMe        |
| Carga de CPU en Reposo (%)       | < 10%           |

```
CPU en reposo : (Get-Counter '\Processor(_Total)\% Processor Time').CounterSamples.CookedValue
```

> [!NOTE]
> La carga de CPU fue medida en estado de reposo estable, después de unos segundos de haber abierto el Administrador de tareas, observando valores inferiores al 10%.

### Etapa 2

* Realizar el portafolio y copiarlo.
* Descargar el archivo disk_io_lab_guided.ipynb
* Seguir las instrucciones del archivo.

#### Punto de control 1
Antes de continuar, responda brevemente:

1. **¿Qué representa la latencia en este laboratorio?:**
La latencia representa el tiempo que tarda el sistema en comenzar a leer un bloque de datos desde el disco. Es como el "tiempo de reacción" antes de que empiece realmente la tranferencia, y incluye el tiempo de retraso que implica encontrar la informacion fisicamente. Es el tiempo que se necesita para tratar de acceder a los datos.

2. **¿Qué representa el throughput?:**
El throughput representa la velocidad a la que se puede procesar los datos una vez que se empezo la tranferencia. osea, que tan rapido es el tiempo de el viaje del disco a la memoria.

3. **¿Por qué en acceso secuencial normalmente se asume que $M \approx 1$?:**
En el accceso secuencial normalmente se asume que $M \approx 1$ porque al estar los datos organizados de una forma continua el disco solo necesita ubicarse una vez y a partir de ahy puede seguir leyendo sin interrupciones. Se podria decir que no tendria que "buscar" cada bloque por separado.

4. **¿Por qué en acceso aleatorio $M$ tiende a ser mayor?:**
En el acceso aleatorio, $M$ tiende a ser mayor porque cada bloque de datos puede estar en ubicaciones diferentes del disco. Esto oabliga al sistema a ubicarse constantemente, generando muchos accesos independientes al disco.

#### Punto de control 2

Se configuro el experimento.

 **(1) Tamaño del archivo:** ¿Es suficiente para superar la caché RAM de
   su equipo? Compare con los valores de RAM registrados en la Etapa 1
   de la guía.

    El tamaño del archivo es de 256 MB, lo que es mucho menor que la memoria de RAM de mi equipo. Por lo tanto, no es suficiente para superar el caché completamente, ya que el sistema podria cargar gran parte del archivo en memoria y reducir los accesos al disco.

**(2) Tamaño de bloque:** Los tamaños evaluados (4 KB, 16 KB, 64 KB,
   256 KB) corresponden a tamaños típicos de páginas en sistemas
   operativos y motores de bases de datos. ¿Cuál esperaría que tuviera
   mejor rendimiento en acceso aleatorio y por qué?

    En el acceso aleatorio se espera que los bloques grandes (como 256 KB) tengan mejor rendimiento, porque permiten leer mas datos en cada acceso y asi poder reducir la cantidad de accesos totales ($M$).

**(3) Entorno de ejecución:** ¿Está ejecutando en local o en Google Colab?
   Recuerde que en Colab los tiempos medidos corresponden al hardware de
   Google, no al suyo.

    Estoy ejecutando este experimento en un entorno local, por lo que los tiempos medidos cooresponden al hardware real de mi equipo.

#### Punto de control 3

Este espacio pertenece a l celda de reinicio del experimento.

#### Punto de control 4

Después de crear el archivo, responda:

**(1) ¿Qué papel cumple este archivo dentro del experimento?:**
El archivo cumple el papel de representar los datos almacenados en disco sobre los que se realizaran las lecturas. Es la base del experimiento, ya que permite simular accesos reales para medir la latencia y el throughput.

**(2) ¿Por qué es útil trabajar con un archivo relativamente grande?**
Es util trabajar con un archivo grande porque es lo mas parecdio al comportamiendo real del almacenamiento, ya que es mas probable que los datos no quepan completamente en memoria y el sistema tenga que acceder al disco.

**(3) ¿Qué cree que ocurriría si el archivo fuera demasiado pequeño?**
Si el archivo fuera demasiado pequeño, probablemente el sistema lo cargaria totalmente en memoria y no abrian accesos reales al disco. Esto seria que los tiempos medidos fueran realativamente bajos y no reflejarian el costo real de las operaciones de I/O

## Análisis de resultados empíricos

Observe la tabla generada y responda:

1. ¿Cuál patrón de acceso fue más rápido para cada tamaño de bloque?
2. ¿El throughput cambió al aumentar el tamaño de bloque?
3. ¿En qué caso observó la mayor diferencia entre secuencial y aleatorio?

> **Criterio mínimo:** la respuesta 3 debe incluir valores numéricos
> concretos obtenidos de la tabla (throughput en MiB/s o tiempo en s).

### Respuesta

En los resultados obtenidos, se observó que para bloques de 4 KB y 16 KB el acceso aleatorio fue más rápido que el secuencial, lo cual no corresponde al comportamiento esperado y puede explicarse por efectos de caché del sistema operativo. Sin embargo, para bloques de 64 KB y 256 KB, el acceso secuencial fue más rápido, lo cual es coherente con la teoría, ya que permite aprovechar la lectura continua de datos. Además, el throughput aumentó al incrementar el tamaño del bloque en ambos patrones de acceso; por ejemplo, en acceso secuencial pasó de aproximadamente 1204.09 MiB/s con bloques de 4 KB a más de 3976.41 MiB/s con bloques de 256 KB, evidenciando una mejora significativa en el rendimiento. Finalmente, la mayor diferencia entre acceso secuencial y aleatorio se presentó en bloques de 256 KB, donde el acceso secuencial alcanzó un throughput de 3976.41 MiB/s con un tiempo de 0.0644 s, mientras que el acceso aleatorio obtuvo 3296.25 MiB/s y un tiempo de 0.3034 s.