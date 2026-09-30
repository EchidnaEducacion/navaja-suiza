# Puesta a punto de Ubuntu para desarrollo

Esta guía describe cómo preparar una instalación de Ubuntu para trabajar con:

- Git
- GitHub CLI (`gh`)
- GitHub mediante SSH
- Claude Code
- Visual Studio Code

---

## 1. Actualizar el sistema

Antes de instalar las herramientas, actualizar la información de los repositorios y los paquetes instalados:

```bash
sudo apt update
sudo apt upgrade -y
```

Instalar algunas utilidades básicas:

```bash
sudo apt install -y \
    curl \
    wget \
    ca-certificates \
    gnupg \
    openssh-client
```

---

# 2. Git

## Instalar Git

```bash
sudo apt install -y git
```

Comprobar la instalación:

```bash
git --version
```

## Configurar usuario

Configurar el nombre y correo electrónico que aparecerán en los commits:

```bash
git config --global user.name "Nombre Apellidos"
git config --global user.email "correo@example.com"
```

Comprobar la configuración:

```bash
git config --global --list
```

Opcionalmente, establecer `main` como nombre por defecto para la rama inicial:

```bash
git config --global init.defaultBranch main
```

---

# 3. GitHub CLI (`gh`)

GitHub CLI permite trabajar con GitHub directamente desde la terminal.

## Instalar

Añadir el repositorio oficial:

```bash
sudo mkdir -p -m 755 /etc/apt/keyrings

wget -nv -O- https://cli.github.com/packages/githubcli-archive-keyring.gpg \
    | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null

sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg
```

Añadir el repositorio:

```bash
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
    | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
```

Actualizar e instalar:

```bash
sudo apt update
sudo apt install -y gh
```

Comprobar:

```bash
gh --version
```

---

# 4. Crear un par de claves SSH para GitHub

GitHub recomienda actualmente utilizar claves **Ed25519**.

Comprobar primero si ya existen claves:

```bash
ls -la ~/.ssh
```

Crear una nueva:

```bash
ssh-keygen -t ed25519 -C "correo@example.com"
```

Cuando aparezca:

```text
Enter file in which to save the key (/home/usuario/.ssh/id_ed25519):
```

pulsar `Enter` para aceptar la ubicación por defecto.

Se crearán dos archivos:

```text
~/.ssh/id_ed25519
~/.ssh/id_ed25519.pub
```

El primero es la **clave privada**:

```text
~/.ssh/id_ed25519
```

**Nunca debe compartirse ni subirse a GitHub.**

El segundo es la **clave pública**:

```text
~/.ssh/id_ed25519.pub
```

Esta es la que se proporciona a GitHub.

---

# 5. Añadir la clave al SSH Agent

Iniciar el agente:

```bash
eval "$(ssh-agent -s)"
```

Añadir la clave:

```bash
ssh-add ~/.ssh/id_ed25519
```

Comprobar las claves cargadas:

```bash
ssh-add -l
```

---

# 6. Añadir la clave pública a GitHub

Hay dos formas de hacerlo.

## Opción A: mediante GitHub CLI

Primero autenticarse:

```bash
gh auth login
```

Seleccionar:

```text
GitHub.com
```

y seguir el proceso de autenticación mediante el navegador.

Después añadir la clave:

```bash
gh ssh-key add ~/.ssh/id_ed25519.pub --title "$(hostname)"
```

Esta es probablemente la forma más cómoda si ya hemos instalado `gh`.

## Opción B: manualmente desde la web

Mostrar la clave pública:

```bash
cat ~/.ssh/id_ed25519.pub
```

Copiar **toda la línea**, que tendrá aproximadamente este aspecto:

```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAI........ usuario@example.com
```

Entrar en GitHub:

```text
Settings
 └── SSH and GPG keys
      └── New SSH key
```

Introducir un nombre descriptivo para el ordenador y pegar la clave pública.

---

# 7. Comprobar la conexión SSH con GitHub

Ejecutar:

```bash
ssh -T git@github.com
```

La primera vez puede aparecer:

```text
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

Responder:

```text
yes
```

Si todo está correctamente configurado, GitHub mostrará un mensaje indicando que la autenticación se ha realizado correctamente.

A partir de este momento pueden clonarse repositorios mediante SSH:

```bash
git clone git@github.com:USUARIO/REPOSITORIO.git
```

---

# 8. Comprobar GitHub CLI

Consultar el estado de autenticación:

```bash
gh auth status
```

Algunos comandos útiles:

```bash
gh repo list
```

Clonar un repositorio:

```bash
gh repo clone USUARIO/REPOSITORIO
```

Crear un repositorio:

```bash
gh repo create
```

---

# 9. Instalar Claude Code

Claude Code es el agente de programación de Anthropic para terminal.

## Instalar Node.js

Claude Code requiere Node.js.

Una opción sencilla en Ubuntu es instalar Node.js y npm:

```bash
sudo apt install -y nodejs npm
```

Comprobar:

```bash
node --version
npm --version
```

> Para entornos de desarrollo en los que se necesiten distintas versiones de Node.js puede resultar preferible utilizar `nvm` en lugar de los paquetes de Ubuntu.

## Instalar Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

**No utilizar `sudo npm install -g`.**

Comprobar:

```bash
claude --version
```

Comprobar también el estado de la instalación:

```bash
claude doctor
```

## Iniciar Claude

Entrar en el directorio de un proyecto:

```bash
cd mi-proyecto
```

y ejecutar:

```bash
claude
```

La primera vez solicitará autenticación.

---

# 10. Instalar Visual Studio Code

Una forma sencilla de instalar la versión oficial en Ubuntu es utilizar el paquete `.deb` proporcionado por Microsoft.

Descargarlo:

```bash
wget -O /tmp/vscode.deb \
    "https://code.visualstudio.com/sha/download?build=stable&os=linux-deb-x64"
```

Instalar:

```bash
sudo apt install -y /tmp/vscode.deb
```

Eliminar el instalador:

```bash
rm /tmp/vscode.deb
```

La instalación mediante el paquete oficial configura además el repositorio de VS Code para recibir posteriormente las actualizaciones mediante APT.

Comprobar:

```bash
code --version
```

---

# 11. Abrir un proyecto con Visual Studio Code

Por ejemplo:

```bash
mkdir -p ~/proyectos/prueba
cd ~/proyectos/prueba
```

Abrir el directorio:

```bash
code .
```

---

# 12. Configuración recomendada de VS Code

Algunas extensiones pueden instalarse directamente desde la terminal.

Por ejemplo:

```bash
code --install-extension ms-python.python
```

Para listar las extensiones instaladas:

```bash
code --list-extensions
```

VS Code detectará automáticamente Git si está instalado.

---

# 13. Prueba completa de Git + GitHub

Crear un repositorio de prueba:

```bash
mkdir ~/proyectos/prueba-github
cd ~/proyectos/prueba-github
```

Inicializar Git:

```bash
git init
```

Crear un README:

```bash
echo "# Prueba GitHub" > README.md
```

Añadirlo:

```bash
git add README.md
```

Crear el primer commit:

```bash
git commit -m "Initial commit"
```

Crear directamente el repositorio en GitHub y subirlo:

```bash
gh repo create prueba-github \
    --private \
    --source=. \
    --remote=origin \
    --push
```

Comprobar:

```bash
git remote -v
```

---

# 14. Crear un proyecto local con Git y subirlo a GitHub

Este apartado describe el flujo completo para empezar un proyecto nuevo en local, ponerlo bajo control de versiones y publicarlo en GitHub.

## Crear el directorio del proyecto

```bash
mkdir -p ~/proyectos/mi-proyecto
cd ~/proyectos/mi-proyecto
```

## Inicializar el repositorio

```bash
git init
```

Esto crea el directorio oculto `.git`, donde Git guarda todo el historial del proyecto.

Si no se configuró `init.defaultBranch` en el apartado 2, asegurarse de que la rama principal se llama `main`:

```bash
git branch -M main
```

## Crear los primeros archivos

Un `README.md` con la descripción del proyecto:

```bash
echo "# Mi proyecto" > README.md
```

Un `.gitignore` con los archivos y directorios que **no** deben versionarse (dependencias, entornos virtuales, ficheros de configuración local, secretos, etc.). Por ejemplo:

```bash
cat > .gitignore << 'EOF'
# Dependencias y entornos
node_modules/
.venv/
__pycache__/

# Configuración local y secretos
.env

# Editores y sistema
.vscode/
.idea/
.DS_Store
EOF
```

> En <https://github.com/github/gitignore> hay plantillas de `.gitignore` para la mayoría de lenguajes y frameworks.

## Hacer el primer commit

Ver el estado del repositorio:

```bash
git status
```

Añadir los archivos al área de preparación (*staging*):

```bash
git add .
```

Revisar qué se va a incluir en el commit:

```bash
git status
```

Crear el commit:

```bash
git commit -m "Commit inicial"
```

Consultar el historial:

```bash
git log --oneline
```

## Subir el proyecto a GitHub

Hay dos formas de hacerlo.

### Opción A: mediante GitHub CLI

Desde el directorio del proyecto:

```bash
gh repo create mi-proyecto \
    --private \
    --source=. \
    --remote=origin \
    --push
```

Este comando crea el repositorio en GitHub, añade el remoto `origin` y sube la rama actual. Utilizar `--public` en lugar de `--private` si el repositorio debe ser público.

### Opción B: creando el repositorio desde la web

Entrar en GitHub y crear un repositorio nuevo:

```text
+ (esquina superior derecha)
 └── New repository
```

Indicar el nombre (`mi-proyecto`) y la visibilidad. **No marcar** las opciones de añadir README, `.gitignore` ni licencia, ya que el repositorio local ya tiene su propio contenido y se producirían conflictos al subirlo.

Añadir el repositorio de GitHub como remoto usando la URL SSH:

```bash
git remote add origin git@github.com:USUARIO/mi-proyecto.git
```

Comprobar el remoto:

```bash
git remote -v
```

Subir la rama `main` y dejarla vinculada con la rama remota:

```bash
git push -u origin main
```

La opción `-u` solo es necesaria la primera vez; a partir de ahí basta con `git push`.

## Flujo de trabajo habitual

Una vez publicado el proyecto, el ciclo normal de trabajo es:

```bash
# Traer los cambios del remoto
git pull

# ... editar archivos ...

# Revisar y confirmar los cambios
git status
git diff
git add .
git commit -m "Descripción del cambio"

# Subirlos a GitHub
git push
```

Abrir el repositorio en el navegador:

```bash
gh repo view --web
```

---

# 15. Comprobación final

Ejecutar:

```bash
git --version
gh --version
ssh -V
node --version
npm --version
claude --version
code --version
```

Comprobar GitHub:

```bash
gh auth status
```

Comprobar SSH:

```bash
ssh -T git@github.com
```

Y comprobar la clave:

```bash
ssh-add -l
```

---

# Resultado

Una vez completados los pasos tendremos el entorno preparado con:

```text
Ubuntu
├── Git
│   └── usuario configurado
│
├── SSH
│   ├── id_ed25519       → clave privada
│   └── id_ed25519.pub   → clave pública
│
├── GitHub
│   ├── GitHub CLI (gh)
│   ├── autenticación
│   └── acceso mediante SSH
│
├── Claude Code
│   └── claude
│
└── Visual Studio Code
    └── code
```

El flujo normal de trabajo puede ser:

```bash
gh repo clone USUARIO/REPOSITORIO
cd REPOSITORIO
code .
```

o trabajar con Claude Code desde el mismo proyecto:

```bash
cd REPOSITORIO
claude
```