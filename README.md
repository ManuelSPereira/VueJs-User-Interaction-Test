# Instalacion

## Requisitos
- Tener instalado Node js
https://nodejs.org/en/download

- Tener instalado Visual Studio Code
https://code.visualstudio.com/



## Pasos
1. Descargar el Repositorio

![](image.png)

2. Abrir la carpeta en Visual Studio Code

![](image-1.png)

3. Ejecutar "npm install" en la terminal

![](image-2.png)
![](image-3.png)

3.A En caso de error en la ejecucion del comando en relacion con "Execution Policy", se debe abrir Powershell como administrador e introducir el comando "Set-ExecutionPolicy Unrestricted" y confirmamos ingresando "s"

![](image-6.png)
![](image-7.png)

3.B Para revertir los cambios, se realiza lo mismo que en el paso 3.A, con la diferencia que se ingresa el comando "Set-ExecutionPolicy Restricted" en vez del anterior. Se puede ver el valor actual con el comando "Get-ExecutionPolicy"

![](image-8.png)


4. Ejecutar "npm run dev" en la terminal

![](image-4.png)

5. Abrir el enlace en el navegador

![](image-5.png)
