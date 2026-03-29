# laboratorio # 3 - IO performance
Laura KAtherine Areiza Henao - 1042150762

## Identificación de la Tecnología de Almacenamiento

Para identificar el tipo de unidad física del sistema, se utilizó PowerShell en Windows ejecutado como administrador con el siguiente comando:

```<Get-PhysicalDisk | Select-Object FriendlyName, MediaType, BusType>```

A continuación, se muestra la evidencia del resultado:

![Resultado del comando](evidencia/Identificación_de_la_Tecnología_de_Almacenamiento.png)

De acuerdo con los valores obtenidos:

MediaType: SSD
BusType: NVMe

Esto indica que mi equipo cuenta con una unidad de estado sólido (SSD) que utiliza la tecnología NVMe.

[def]: dentificación_de_la_Tecnología_de_Almacenamiento.pn