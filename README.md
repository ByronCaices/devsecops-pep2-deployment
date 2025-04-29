# 🛠️ Guía de Configuración del Proyecto

## 📥 1. Clonar el repositorio con sus submódulos

Clona el repositorio principal junto con sus submódulos ejecutando:

```bash
git clone --recurse-submodules https://github.com/ByronCaices/devsecops-pep1-deployment.git
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
    

## 🟦 9. Integración de SonarQube en Frontend

### 9.1 Levantar el contenedor de SonarQube

```bash
docker compose up -d sonarqube
```

---

### 9.2 Configurar SonarQube (primer acceso y token)

```bash
# Abrir en el navegador
http://localhost:9000
```

- Usuario: `admin`
- Contraseña: `admin`
- Cambia la contraseña cuando lo solicite
- Ve a **My Account > Security** y genera un token.
- Guarda este token para Jenkins.

---

### 9.3 Agregar el token de SonarQube en Jenkins

```bash
# En Jenkins: Manage Jenkins > Manage Credentials > (global) > Add Credentials
# Tipo: Secret text
# ID: sonarqube-token
# Secret: <pega el token generado en sonarqube>
```

---

### 9.4 Configurar SonarQube en Jenkins

```bash
# En Jenkins: Manage Jenkins > Manage Plugins
# Instala el plugin: SonarQube Scanner

# En Jenkins: Manage Jenkins > Configure System
# Sección: SonarQube servers
# Name: sonarqube local
# Server URL: http://sonarqube:9000
# Server authentication token: selecciona el token creado
# Guarda los cambios
```

---

### 9.5 Configurar NodeJS en Jenkins

```bash
# En Jenkins: Manage Jenkins > Global Tool Configuration
# Sección: NodeJS
# Name: nodejs
# Marca "Install automatically"
# Guarda los cambios
```

---

### 9.6 Agregar el análisis SonarQube al Jenkinsfile

En la configuración del pipeline, en la sección de "pipeline script" se debe copiar el Jenkinsfile nuevo que está en la carpeta raíz del frontend, el cual contiene el stage "SonarQube Analysis" para realizar el análisis

```groovy
stage('SonarQube Analysis') {
    steps {
        dir('deployment/devsecops-prestabanco-frontend') {
            withCredentials([string(credentialsId: 'sonarqube-token', variable: 'SONARQUBE_TOKEN')]) {
                withSonarQubeEnv('sonarqube local') {
                    sh '''
                        npx sonar-scanner \
                          -Dsonar.projectKey=prestabanco-frontend \
                          -Dsonar.projectName="PrestaBanco Frontend" \
                          -Dsonar.sources=src \
                          -Dsonar.host.url=http://sonarqube:9000 \
                          -Dsonar.login=$SONARQUBE_TOKEN
                    '''
                }
            }
        }
    }
}
```

---


### 9.7 Ejecutar el pipeline

```bash
# Ejecuta el pipeline manualmente desde Jenkins
# Verifica el análisis en http://localhost:9000 que será creado con el projectName establecido en el stage de SonarQube Analysis
```