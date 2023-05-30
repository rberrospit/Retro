



# ADQUISICIÓN DE IMÁGENES DESDE UN FOLDER O ARCHIVOS ESPECIFICO DE WINDOWS

Conectar un USB Stick (Memoria USB). 

Ejecutar el programa AccessData FTK Imager. 

Añadir un ítem de evidencia  (File > Add Evidence Item”)

 
![image](https://user-images.githubusercontent.com/50930193/135372779-9a7dffb6-a544-4606-b9de-05bd83c9d262.png)


luego seleccionar “Physical Drive”. 

 ![image](https://user-images.githubusercontent.com/50930193/135372792-dc4599b0-5c84-47a5-b5b6-e314c5ac9b0a.png)


En la siguiente ventana seleccionar la Unidad USB, y para finalizar se debe presionar el botón “Finish”, ubicado en la parte inferior.

 ![image](https://user-images.githubusercontent.com/50930193/135372822-5cef04d1-7e60-4807-9164-bc4c6599391b.png)

 
En la sección definida como “Evidence Tree” se visualizará el contenido de la Unidad USB. 

 ![image](https://user-images.githubusercontent.com/50930193/135372834-4a310f3d-0751-4732-8b3b-e2fdcaaa2be4.png)


Hacer clic derecho sobre alguna de las carpetas expuestas, ante lo cual se presentará 
un menú conteniendo cuatro opciones. 

Hacer clic en la opción “Add to Custom Content Image (AD1)”, para luego repetir el mismo procedimiento con otro archivo. Se visualizará el resultado de estas acciones en la ventana de nombre “Custom Contect Sources”.

 ![image](https://user-images.githubusercontent.com/50930193/135372848-7a9b9966-6c32-4371-8a38-74b8cc62b2e0.png)


Hacer clic en el botón “Create Image” ubicado en la parte inferior de esta ventana. Luego de esta acción, se presentará una nueva ventana donde se solicitará definir el destino en el cual se copiará la evidencia forense. 

![image](https://user-images.githubusercontent.com/50930193/135372864-a5ffe6c2-255d-4873-80d7-4fc24358064f.png)

 
Hacer clic en el botón “Add”, completar la información solicitada sobre la evidencia, hacer clic en el botón “Siguiente”, 

Seleccionar una carpeta donde se copiará la evidencia, utilizando el botón “Browse” y definir un nombre de archivo) en el campo (Image FileName).  Y luego dar click en “Finish”

![image](https://user-images.githubusercontent.com/50930193/135372873-57e66b3b-7a85-4989-b8f1-467df2083e06.png)

 


Para iniciar el proceso de captura, hacer clic en el botón “Start”. Con este procedimiento se generará una imagen forense, la cual únicamente contiene como evidencia, la carpeta y el archivo seleccionado desde la memoria USB.

  ![image](https://user-images.githubusercontent.com/50930193/135372887-791ce602-1c8d-4c7c-9f1f-ce5da789e2ba.png)
  
![image](https://user-images.githubusercontent.com/50930193/135372923-c090c2a8-7f76-4649-ba55-037b61f7e6e7.png)

 
# ADQUISICIÓN DE IMÁGENES DE ARCHIVOS PROTEGIDOS DE UN SISTEMA EN FUNCIONAMIENTO EN WINDOWS


Hacer clic en “File -> Obtain Protected Files”

 ![image](https://user-images.githubusercontent.com/50930193/135372932-1e753308-883f-4ae3-8552-c2bf81bad09c.png)

Luego definir el directorio en el cual se copiarán los archivos capturados,utilizando el botón “Browse”. En la parte inferior de la ventana se mostrarán dos opciones, seleccionar la opción “Password recovery and all Registry Files”. 


 ![image](https://user-images.githubusercontent.com/50930193/135372943-4c796e1f-576f-4639-b9bc-1d197868bb92.png)



Luego hacer clic en el botón “OK”, para iniciar el proceso de copia.
 
Este procedimiento capturará en una carpeta definida los archivos “Users”, “System”, “SAM”,
“NTUSER.DA”T, “Default”, “Security”, “Software”, entre otros. En clases posteriores, para visualizar (ANALIZAR) algunos de estos archivos se puede utilizar AccessData Registry Viewer, o RegRipper

 
![image](https://user-images.githubusercontent.com/50930193/135372958-a0958a95-3f5f-410a-8d37-8e51978ee9f3.png)

![image](https://user-images.githubusercontent.com/50930193/135372966-6336c10a-bb23-48cc-9d7d-c6b8dca26b2c.png)


 
