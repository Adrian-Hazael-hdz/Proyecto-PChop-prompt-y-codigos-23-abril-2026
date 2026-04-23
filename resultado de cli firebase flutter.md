Para llevar a cabo el proyecto **CRUD PCHOP**, es fundamental preparar el entorno de desarrollo. Aquí tienes la guía técnica paso a paso para configurar las herramientas de Firebase desde Windows.

---

## 1. Software Necesario: Node.js y NPM
Para utilizar las herramientas de línea de comandos de Firebase (**Firebase CLI**), necesitas **Node.js**, ya que NPM (Node Package Manager) es el motor que descarga e instala estas utilerías.

### Cómo verificar si Node.js y NPM están instalados
Abre una terminal (PowerShell o CMD) y escribe:
* `node -v`
* `npm -v`

Si recibes un número de versión (ej. `v20.10.0`), ya está instalado. De lo contrario, verás un error de "comando no reconocido".

### Procedimiento de instalación (Paso a Paso)
Si no lo tienes, sigue estos pasos para instalarlo de manera global:
1.  **Descarga:** Ve al sitio oficial [nodejs.org](https://nodejs.org/) y descarga la versión **LTS** (Long Term Support).
2.  **Instalador:** Ejecuta el archivo `.msi`. 
3.  **Configuración:** Durante la instalación, asegúrate de que la opción **"Add to PATH"** esté seleccionada (esto es lo que permite que sea "global").
4.  **Herramientas adicionales:** Si el instalador pregunta por "Tools for Native Modules", puedes omitirlo para este proyecto básico de Flutter.
5.  **Finalizar:** Reinicia tu computadora para que los cambios en las variables de entorno surtan efecto.

---

## 2. Instalación de Firebase CLI
Una vez que `npm` funciona, instalaremos las **firebase-tools** de forma global. Esto permite usar el comando `firebase` en cualquier carpeta de tu computadora.

### Comando de instalación global:
```bash
npm install -g firebase-tools
```
> **Nota:** El parámetro `-g` indica que la instalación es **Global**, requisito indispensable para que tu proyecto en la carpeta `crudpchop` pueda reconocer los comandos de Firebase.

---

## 3. Comandos Esenciales de Firebase

### Cómo acceder con tu Cuenta de Google
Antes de vincular Flutter con Firebase, debes autenticarte.
* **Comando:** `firebase login`
* **Proceso:** Se abrirá una ventana en tu navegador predeterminado. Selecciona tu cuenta de Google, otorga los permisos y verás un mensaje de éxito: *"Firebase CLI login successful"*.

### Cómo usar firebase-tools para Flutter
Para integrar Firebase en tu aplicación de forma automatizada, utilizamos un ejecutable adicional llamado `flutterfire`.

1.  **Instalar el activador de FlutterFire:**
    ```bash
    dart pub global activate flutterfire_cli
    ```
2.  **Vincular el proyecto:**
    Dentro de tu carpeta `crudpchop`, ejecuta:
    ```bash
    flutterfire configure
    ```
    Este comando (que forma parte de las herramientas de Firebase) te permitirá seleccionar tu proyecto de la consola y creará automáticamente el archivo `firebase_options.dart` necesario para el CRUD de computadoras.



---

## 4. Resumen de flujo de trabajo para el Estudiante

| Acción | Comando |
| :--- | :--- |
| **Verificar NPM** | `npm -v` |
| **Instalar CLI** | `npm install -g firebase-tools` |
| **Iniciar Sesión** | `firebase login` |
| **Configurar Flutter** | `flutterfire configure` |

Esta metodología asegura que el **Agente de Datos** que definimos anteriormente tenga un "túnel" de comunicación sólido hacia la base de datos Firestore en la nube.

¿Deseas que profundicemos en cómo realizar el despliegue (hosting) de este CRUD una vez terminado el código?
