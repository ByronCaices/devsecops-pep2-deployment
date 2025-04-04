
# 🛠️ Guía de Configuración del Proyecto

## 📥 1. Clonar el repositorio con sus submódulos

Clona el repositorio principal junto con sus submódulos ejecutando:

```bash
git clone --recurse-submodules https://github.com/usuario/repositorio.git
```

---

## ⚙️ 2. Crear archivos `.env`

Es necesario que cada uno de los tres repositorios tenga un archivo `.env`.  
Por ahora, basta con copiar el contenido de `.env.example` en un nuevo archivo llamado `.env`.

> 📄 **Nota:** Este paso debe realizarse en los tres repositorios.

---

## 🏗️ 3. Generar el build del backend

Puedes compilar el backend usando el siguiente comando:

```bash
mvn clean package
```

O bien, realizar el proceso desde la interfaz de **IntelliJ IDEA** (recomendado).  
Se aconseja **desactivar los tests** para agilizar la compilación.

---

## 🐳 4. Iniciar contenedores

Levanta los contenedores principales (sin incluir el del frontend) ejecutando:

```bash
docker compose up --build -d postgres pgadmin4 backend jenkins
```

> ⚠️ **Importante:**  
> No se debe iniciar el contenedor del frontend manualmente.  
> Si ya está corriendo, el pipeline fallará al intentar desplegarlo, ya que detectará un contenedor activo con la misma imagen.  
> Para evitar conflictos, deja que Jenkins lo maneje.

---

## 🔧 5. Instalar comandos Docker en Jenkins

Una vez que los contenedores estén activos, accede al contenedor de Jenkins e instala Docker y Docker Compose:

```bash
docker exec -it jenkins bash
```

Dentro del contenedor:

```bash
apt-get update && apt-get install -y docker.io docker-compose
```

---

## 🔑 6. Acceder a Jenkins

Accede a Jenkins en tu navegador a través del puerto **8082**.  
Se solicitará una contraseña, la cual puedes obtener con el siguiente comando:

```bash
docker logs -f jenkins
```

Copia la contraseña y procede con la configuración inicial.  
Puedes crear las credenciales que desees, pero asegúrate de recordarlas.

---

## 🧩 7. Crear un pipeline en Jenkins

1. Haz clic en **"Create a job"**.
    
2. Asigna un nombre y selecciona el tipo **Pipeline**.
    
3. En la sección **"Build Triggers"**, activa la opción **"Poll SCM"**.
    
4. Configura el cron con el siguiente valor:
    
    ```
    H/5 * * * *
    ```
    
    Esto hará que el pipeline revise automáticamente, cada 5 minutos, si hay cambios en la rama `main` del repositorio `deployment` o `frontend`.
5. Pegar script del Jenkinsfile del frontend cuando se te solicite

---

## 🚀 8. Ejecutar el pipeline

- Ejecuta el pipeline manualmente por primera vez para verificar que no existan errores.
    
- Ejecuta una segunda vez para confirmar el correcto funcionamiento.
    
- Finalmente, realiza un _push_ con algún cambio en el proyecto `frontend`.  
    Como el pipeline se ejecuta cada 5 minutos, solo deberás esperar a que se active automáticamente y verifiques que se despliegue correctamente.
    

---
