## ANÁLISIS DE WANNACRY

Descargar la imagen forense desde https://drive.google.com/file/d/1VT3CiQIz8mig2b7PJ1oQmfwUEOfFCU6G/view?usp=sharing

**Determinar el profile**

Para determinar el profile (Sistema operativo y su versión) es necesario conocer sobre que profile se realizará la inspección, por ello ejecutar lo siguiente:

```
./volatility -f images/wcry.raw imageinfo
```
![image](https://user-images.githubusercontent.com/50930193/142294685-05601dc2-2527-40a0-9b69-5626d8a17e6a.png)

Esta imagen de memoria es ejecutada sobre un sistema de 32 bits de Windows XP Con Service Pack 2. El equipo tiene un procesador basado en PAE (Physical Address Extension). La imagen fue adquirida el 10 de octubre del 2011.

 
**Listar los Procesos**
 

Conocido el profile, para listar los procesos en memoria abrir el terminal y ejecutar lo siguiente:
```
./volatility -f images/wcry.raw --profile=WinXPSP2x86 pslist
./volatility -f images/wcry.raw --profile=WinXPSP2x86 pstree
```
![image](https://user-images.githubusercontent.com/50930193/142294974-aa646d39-ae2b-4d8f-a5d3-58e7891ebb2c.png)

![image](https://user-images.githubusercontent.com/50930193/142295201-70ff0bc4-5fe4-4a42-81d7-ec45db604a0e.png)

Los **PID 1940** y **PID 740** son procesos que son extraños y vemos que uno depende de otro


Con el plugin **psscan**, hace uso del direccionamiento de memoria física con el beneficio del uso de psscan es que algunas veces muestra información de procesos que no aparecen con otros comandos, este plugin **psscan** también enumerará todos los procesos, incluidos los procesos terminados, lo que puede ayudarnos a identificar la jerarquía de procesos y la línea de tiempo de creación.

```
./volatility -f images/wcry.raw --profile=WinXPSP2x86 psscan
```
![image](https://user-images.githubusercontent.com/50930193/142295605-9baad08b-0c30-4c39-abae-322d20aae493.png)

Como podemos ver los procesos terminados **taskdl.exe**, **taskse.exe** junto con el proceso padre **PID 1940**

```
./volatility -f images/wcry.raw --profile=WinXPSP2x86 psscan | grep 1940
./volatility -f images/wcry.raw --profile=WinXPSP2x86 psscan | grep 1940 > data.txt

```

![image](https://user-images.githubusercontent.com/50930193/142296744-190c701d-0100-4061-ac26-f998e88d90b5.png)

y si clasificamos el tiempo de creación del proceso usando sort  Sería fácil entender la línea de tiempo de la creación del proceso Los siguientes procesos desconocidos pueden considerarse sospechosos

```
.sort -k 7 data.txt
```

![image](https://user-images.githubusercontent.com/50930193/142298272-3ae1e9e9-689d-4a44-a0f7-6fa4cc5738d9.png)


mirando el orden de creación del proceso, el proceso **taskse.exe** se creó antes que el proceso **taskdl.exe**, pero todavía no tengo idea de lo que hacen estos procesos. A continuación se muestran los resultados de motores de búsqueda famosos sobre estos procesos.

![image](https://user-images.githubusercontent.com/50930193/142299318-7f6d518b-b58a-4d2a-aa23-c3ae5ce77818.png)


Estas muestras ya fueron analizados por los vendedores de inteligencia y AV amenaza gigante, pero, en muchos realidad de nuevos indicadores puede ser descubierto en poco tiempo cuando se trata de amenazas desconocidas
Run **DLLList** plugin para identificar los archivos DLL de proceso y ruta en la que el proceso se ejecuta desde, esto puede dar una comprensión clara de los procesos maliciosos si se ejecutan mediante archivos binarios descartados en carpetas poco comunes.

```
./volatility -f images/wcry.raw --profile=WinXPSP2x86  dlllist -p 1940

```
![image](https://user-images.githubusercontent.com/50930193/142299555-9fed61c1-9765-4eda-abdc-4942b0f531c3.png)

Identifique la ruta del binario para el proceso tasksche.exe que claramente parece poco común y sospechoso. Se recomienda mirar las DLL cargadas para comprender las características del proceso, como el cifrado, la modificación de registros y la creación de sockets, etc.

```
./volatility -f images/wcry.raw --profile=WinXPSP2x86  dlllist -p 740

```

![image](https://user-images.githubusercontent.com/50930193/142299641-7ca1f857-1a05-45c3-9f5a-390a647599f7.png)

El proceso **@ WanaDecryptor @** con PID 740 también usa la misma ruta de proceso tasksche.exe. Basado en archivos DLL cargados por el proceso @ WanaDecryptor @, puede realizar la creación de sockets (Ws2_32.dll), comunicaciones de red de alto nivel (WININET.DLL), consultar el registro (ADVAPI32.DLL), cifrar (SECURE32.DLL) e interactuar con los navegadores ( URLMON.DLL) como Internet Explorer, etc.

En cuanto a los identificadores de PID 1940, ha creado un mutex (los autores de malware han utilizado durante mucho tiempo los mutex para evitar que más de una instancia del malware se ejecute en la misma máquina. Un viejo truco anti-malware consiste en la creación de un mutex, para evitar la ejecución de un malware específico) llamado '' MsWinZonesCacheCounterMutexA ''

```
./volatility -f images/wcry.raw --profile=WinXPSP2x86 handles -p 1940 -t Key
./volatility -f images/wcry.raw --profile=WinXPSP2x86 handles -p 1940 -t Mutant

```
![image](https://user-images.githubusercontent.com/50930193/142299832-a7291836-4e25-426a-b05e-3862b11def27.png)


Una búsqueda rápida de este mutex en Google da lo siguiente:

![image](https://user-images.githubusercontent.com/50930193/142299931-e5b5ba14-802d-49ce-9114-69b1021af5ed.png)

Mutex "MsWinZonesCacheCounterMutexA" puede ser uno de los IOC para identificar sistemas infectados. Al igual que mutex como uno de los tipos de asas para cualquier proceso, la volatilidad de los mangos plugin también puede identificar archivos, Llave, Evento, hilos y tipo de puerto de asas para cualquier proceso. Un vistazo rápido a los archivos accedidos por PID 1940

```
./volatility -f images/wcry.raw --profile=WinXPSP2x86 handles -p 1940 -t File

```
![image](https://user-images.githubusercontent.com/50930193/142299993-db9e0c22-0783-4f6b-8f31-ad9927dfdc30.png)

Se recomienda mirar el tipo de identificador de clave para cualquier proceso que pueda brindar información sobre los cambios de registro realizados por ese proceso. A continuación se muestra el tipo de mango clave para el proceso PID 740

```
./volatility -f images/wcry.raw --profile=WinXPSP2x86 handles -p 740 -t Key

```
![image](https://user-images.githubusercontent.com/50930193/142300075-1684e77f-a27e-4402-9acb-365424c64ce0.png)

Aún no se encontró ningún mecanismo persistente, se puede identificar mediante el complemento printkey accediendo a Run, Runonce, Winlogonkeys, BootExcuteKey, carpetas de inicio y clave de servicios

```
./volatility -f images/wcry.raw --profile=WinXPSP2x86 printkey -K "Microsoft\Windows\CurrentVersion\Run"

```
![image](https://user-images.githubusercontent.com/50930193/142300288-3e42cd5f-9110-4b6d-b8b6-f0040beadd52.png)

Los artefactos relacionados con la red se pueden identificar mediante el complemento de conexiones para conexiones activas y el complemento connscan para conexiones terminadas
```
./volatility -f images/wcry.raw --profile=WinXPSP2x86 connections
./volatility -f images/wcry.raw --profile=WinXPSP2x86 connscan
```

![image](https://user-images.githubusercontent.com/50930193/142300436-3d3fd0c3-b0de-45d9-a4f4-623d0f0874a2.png)


Lamentablemente, no se encontraron conexiones. Dado que el volcado de memoria también puede contener algunas conexiones de red, podemos usar la herramienta de talla de datos bulk_extractor para extraer las conexiones de red de la memoria. El complemento ethscan de volatilidad también puede extraer pcap del volcado de memoria
```
bulk_extractor -E net -o DUMP/ images/wcry.raw
```
![image](https://user-images.githubusercontent.com/50930193/142300582-46354c3a-efae-4e27-a34d-ae0ce8b99744.png)


El pcap extraído se abrió en Wireshark para ver cualquier nombre de dominio relacionado con Killswitch y otras conexiones netowrk. Desafortunadamente, no se encontró ningún interruptor en este pcap (extraído de la memoria), excepto algunas direcciones IP remotas desconocidas.

![image](https://user-images.githubusercontent.com/50930193/142301171-7a92201a-618a-487d-99a7-69aefe430ecd.png)

Usando tshark, todas las direcciones IP de pcap se extraen en un archivo de texto y, además, se pueden usar como indicadores de compromiso.
```
tshark -T fields -e ip.src -r packets.pcap  | sort -u
```

![image](https://user-images.githubusercontent.com/50930193/142300966-501ddb92-3b0f-46ad-8bb2-82f762bd379a.png)

El killswitch se encontró en pcap que se capturó mientras wannacry infectaba el sistema y el enlace de descarga está disponible debajo de

Pcap:  https://mega.nz/#!h6oCBbYS!TV46RntkpyZaPZYaSpir3iutOQLBZvm4xf4t84enuHM

sha256sum: 8808a801c2c2d6d6d6cbd2cbd1

![image](https://user-images.githubusercontent.com/50930193/142301069-336f6d97-d43f-4e5c-a49e-a2a40d13d6a9.png)


Según el mecanismo de killswitch del autor de wannacry, el sistema se infectó aún más ya que el dominio no se resolvió y no se pudo acceder a él. En este pcap, se encontró un número de hosts desconocidos

![image](https://user-images.githubusercontent.com/50930193/142301113-87184402-e94d-45e6-9e21-6189a90f0c50.png)


 Todas las direcciones IP se copiaron en un archivo de texto usando tshark y se pueden tratar y usar como indicadores automáticos de compromiso.
```
tshark -T campos -e ip.src -r dump.pcap | sort -u 
```

Los archivos residentes en la memoria se pueden buscar usando el complemento de escaneo de archivos y se pueden descargar usando el complemento de archivos de volcado . Mientras buscaba una carpeta específica de tasksche.exe, curiosamente todos los archivos relacionados con ransomeware se encontraron en una ubicación en la carpeta **ivecuqmanpnirkt615**
```
./volatility -f images/wcry.raw --profile=WinXPSP2x86 filescan  | grep ivecuqmanpnirkt615
```
![image](https://user-images.githubusercontent.com/50930193/142332273-e6aa1cec-69c8-4856-86dc-dcea10cce118.png)


Estos archivos pueden volcarse usando la dirección física respectiva del archivo usando el complemento **dumpfiles** especificando la opción **-Q**.

```
./volatility -f images/wcry.raw --profile=WinXPSP2x86 dumpfiles -Q 0x00000000022ec718  -D DUMP/
./volatility -f images/wcry.raw --profile=WinXPSP2x86 dumpfiles -Q 0x00000000021dc028  -D DUMP/
./volatility -f images/wcry.raw --profile=WinXPSP2x86 dumpfiles -Q 0x0000000001fb2278  -D DUMP/
./volatility -f images/wcry.raw --profile=WinXPSP2x86 dumpfiles -Q 0x0000000001f871a0  -D DUMP/
``` 
![image](https://user-images.githubusercontent.com/50930193/142332920-6d430902-8c70-4346-af5c-fc99cc6db608.png)


Un análisis más detallado, como la ingeniería estática, dinámica o inversa de estos binarios extraídos, puede brindar mucha información sobre el mecanismo del ransomware. También podemos volcar un archivo deseado para un análisis más detallado y los hash de estos archivos se pueden usar como indicadores de compromiso para otros motores de detección.
Se encontraron cadenas interesantes en **@ WanaDecryptor @ .exe binary likes.wnry, f.wnry, c.wnry,** messages relacionados con el pago, cómo usar bitcoins, API relacionadas con el cifrado y la eliminación de instantáneas de volumen de la víctima.

![image](https://user-images.githubusercontent.com/50930193/142333562-d528141d-01f4-4ef1-a489-450d7271e6ea.png)

 Siempre se recomienda volcar el espacio de direcciones de memoria de los procesos para verificar si hay entradas sospechosas en la memoria del proceso, en lugar de centrarse únicamente en el binario.
 
 ```
./volatility -f images/wcry.raw --profile=WinXPSP2x86 memdump -p1940,740  -D DUMPA/
``` 

 ![image](https://user-images.githubusercontent.com/50930193/142334351-803896bc-2c7a-4be9-ae30-a6d675f6bf63.png)

El complemento de volatilidad memdump se utilizó para volcar el espacio de direcciones de los procesos @ WanaDecryptor @ y taskssche.exe para cualquier indicador
Al observar las picaduras del proceso tasksche.exe (PID 1940), se encontró que tasksche.exe inició el proceso @ WanaDecryptor @ con argumentos de línea de comando

 ```
strings 1940.dmp | head -n 100
strings 740.dmp | head -n 100

``` 

![image](https://user-images.githubusercontent.com/50930193/142334643-af1817de-9c17-45d3-b808-3373356f40ea.png)


Al observar las cadenas del volcado del proceso @ WanaDecryptor @ (PID 740), se encontró que el malware usa servicios ocultos TOR para comando y control. La lista de dominios .onion en el interior es la siguiente

![image](https://user-images.githubusercontent.com/50930193/142334833-538abf2d-7b08-4e0f-b9af-dccd3ac47a2b.png)

Los dominios de cebolla son los siguientes
gx7ekbenv2riucmf.onion
gx7ekbenv2riucmf.onion
57g7spgrzlojinas.onion
xxlvbrloxvriy2c5.onion
76jdd2ir2embyv47.onion
cwwnhwhlz52maqm7.onion

La dirección de bitcoin es 
12t9YDPgwueZ9NyMgw519p7AA8isjr6SMw

El enlace para la dirección de bitcoin se encontró como 
http://www.btcfrog.com/qr/bitcoinPNG.php?address=12t9YDPgwueZ9NyMgw519p7AA8isjr6SMw

Todos los archivos caídos se volcaron utilizando el complemento dumpfiles y estos hashes de archivos pueden usarse como indicadores de compromiso o ingerirse en los motores de búsqueda. Las reglas de YARA también son útiles para escribir sus propias reglas para prevenir o identificar rápidamente infecciones de ransomware 

 ```
./volatility -f images/wcry.raw --profile=WinXPSP2x86 yarascan -y "rules/malware/crime_wannacry.yar"
``` 
![image](https://user-images.githubusercontent.com/50930193/142335632-7f7a8545-dd5c-4782-a3fe-f62578f168f6.png)
