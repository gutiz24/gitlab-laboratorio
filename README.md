## Ejercicios GitLab

```diff
+ ## Propuesta Ejercicio 1
```

1. La propuesta del ejercicio se encuentra en el fichero `.gitlab-ci.yml`
2. Se han definido 4 stages: `build, test, docker, deploy`
```yaml
stages:
  - build
  - test
  - docker
  - deploy

```

3. En el job de `maven-build` se trata de compilar el cídog del proyecto que da como resultado un artefacto `tarject/<nombre-artefacto>.jar` como ejecutable. Este se guardará para los siguientes stages con la opción `artifacs`.
```yaml
maven-build:
  image: maven:3.9-eclipse-temurin-21
  stage: build
  script:
    - mvn clean package
  artifacts:
    when: on_success
    paths:
      - "target/*.jar"
```

4. En el job `maven-test` se trata de correr los test definidios del proyecto con `mvn verify` y generará un reporte `report.xml`. Este se lo guardará Gitlab como artefacto especial de reportes de test `reports`. NOTA: al no necesitar el artefacto anterior se usa la anotación `dependencies: []`
```yaml
maven-test:
  image: maven:3.9-eclipse-temurin-21
  stage: test
  dependencies: []
  script:
    - mvn verify
  artifacts:
    when: on_success
    reports:
      junit:
        - "target/surefire-reports/TEST-*.xml"
```
5. En el job `docker-build` se hace la generación y subida del código en una imagen docker en base a un `Dockerfile`. Primero se registra contra el container registry de gitlab `$CI_REGISTRY_NAME` en `before-script` para ello se usan varibales `$CI_REGISTRY_USER` con un usuario temporal del job para escribir y leer el registru y `$CI_RESGISTRY_PASSWORD` como un token temporal con solo el spoce para el registry. Y como último se genera la imagen y sube al registry en `script`

> [!CAUTION]
> Se puede hacer uso de comandos docker porque en la configuración de desarrollo del runner tiene mapeado el socket de docker con la maquina anfitriona. En entornos de producción se hace uso de `docker-in-docker` u herramientas alternativas como `kanico` (generador de imágenes)
```yaml
docker-build:
  stage: docker
  dependencies:
    - maven-build
  before_script:
    - echo $CI_REGISTRY_PASSWORD | docker login -u $CI_REGISTRY_USER --password-stdin $CI_REGISTRY_IMAGE
  script:
    - docker build -t $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA .
    - docker push $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```

6. En el job `deploy` se hace un despligue de la aplicación. Primero se autentica con el registry y se comprueba que no exista la aplicación previamente si no se elimina `docker rm`. Y como último se corre la aplicación descargando la imagen directametne del registry. Ahora la aplicación se puede usar visualizar en el puerto 8080 `-p 8082:8080` visualizano `http://gitlab.local:8082/`. De nuevo se reitera que se esto funciona porque se comparte el socket de docker.
```yaml
deploy:
  stage: deploy
  dependencies: []
  before_script:
    - docker login -u $CI_REGISTRY_USER -p $CI_JOB_TOKEN $CI_REGISTRY_IMAGE
    - if [[ "$(docker ps -a --filter "name=springapp" --format '{{.Names}}')" == "springapp" ]]; then  docker rm -f springapp; else echo "No existe";  fi
  script:
    - docker run --name "springapp" -d -p 8082:8080 $CI_REGISTRY_IMAGE:$CI_COMMIT_SHA
```
Resultado Final:
<p aling="center">
<img src="./img/image5.png" style="width:15%; height:auto;">
</p>

```diff
+ ## Propuesta Ejercicio 2
```

1. Para dar de alta un nuevo usuario y cierto rol. Se debe acceder a la consola como usuario `root`
2. Una vez accedido se debe ir a la parte de  `admin > userss` donde podremos crear un nuevo usuario con en la parte de `New User`
3. Se deben de rellenar los datos necesario y se debe dar a `Create New User`
<p aling="center">
<img src="./img/image1.png" style="width:20%; height:auto;">
</p>

4. Una vez creado se vuelve a editar el usario nuevo como root y se le crea una contraseña
<p aling="center">
<img src="./img/image2.png" style="width:20%; height:auto;">
</p>

5. Para poder pobar el nivel de permisos de debe de acceder al repo `ejercicios-gitlab`. Se debe invitar a un nuevo usuario con la cuenta original `developer1`. Para ello situado en el repo se debe acceder a `manage > Members > Invite Members`
<p aling="center">
<img src="./img/image3.png" style="width:20%; height:auto;">
</p>

6. Una vez hecho se da la posibilidad de seleccionar el tipo de rol que tendrá el nuevo usuario invitado.
<p aling="center">
<img src="./img/image4.png" style="width:20%; height:auto;">
</p>

6. Tabla de diferencias

<style type="text/css">
.tg  {border-collapse:collapse;border-color:#ccc;border-spacing:0;}
.tg td{background-color:#fff;border-color:#ccc;border-style:solid;border-width:1px;color:#333;
  font-family:Arial, sans-serif;font-size:14px;overflow:hidden;padding:10px 5px;word-break:normal;}
.tg th{background-color:#f0f0f0;border-color:#ccc;border-style:solid;border-width:1px;color:#333;
  font-family:Arial, sans-serif;font-size:14px;font-weight:normal;overflow:hidden;padding:10px 5px;word-break:normal;}
.tg .tg-6v43{background-color:#ffffff;font-weight:bold;text-align:left;vertical-align:top}
.tg .tg-b3sw{background-color:#efefef;font-weight:bold;text-align:left;vertical-align:top}
</style>
<table class="tg"><thead>
  <tr>
    <th class="tg-b3sw"></th>
    <th class="tg-b3sw">Guest</th>
    <th class="tg-b3sw">Reporter</th>
    <th class="tg-b3sw">Developer</th>
    <th class="tg-b3sw">Maintainer</th>
  </tr></thead>
<tbody>
  <tr>
    <td class="tg-b3sw">Commit</td>
    <td class="tg-6v43">NO</td>
    <td class="tg-6v43">SI</td>
    <td class="tg-6v43">SI</td>
    <td class="tg-6v43">SI</td>
  </tr>
  <tr>
    <td class="tg-b3sw">Pipeline<br>Manual</td>
    <td class="tg-6v43">NO</td>
    <td class="tg-6v43">NO</td>
    <td class="tg-6v43">SI</td>
    <td class="tg-6v43">SI</td>
  </tr>
  <tr>
    <td class="tg-b3sw">Push and Pull<br>Repo</td>
    <td class="tg-6v43">NO</td>
    <td class="tg-6v43">NO</td>
    <td class="tg-6v43">SI</td>
    <td class="tg-6v43">SI</td>
  </tr>
  <tr>
    <td class="tg-b3sw">Merge Request</td>
    <td class="tg-6v43">NO</td>
    <td class="tg-6v43">NO</td>
    <td class="tg-6v43">SI</td>
    <td class="tg-6v43">SI</td>
  </tr>
  <tr>
    <td class="tg-b3sw">Administración<br>del repo</td>
    <td class="tg-6v43">Solo<br>Visualizar</td>
    <td class="tg-6v43">Solo<br>Visualizar</td>
    <td class="tg-6v43">Solo<br>Visualizar</td>
    <td class="tg-6v43">SI</td>
  </tr>
</tbody></table>

```diff
+ ## Propuesta Ejercicio 3
```
### Con el método de CI job token
Al hacer el git clone con otro repositorio externo, por defecto no deja hacerlo con la variable especial `${CI_JOB_TOKEN}` aunque se sea miembro del repositorio clonando, Para ello hace falta una configurción adicional `CI/CD > Job Token Permissions` donde se da los permisos de autorización a grupos o poryectos externos.

* **Configuración por defecto y resultado de git clone**
<p aling="center">
<img src="./img/image7.png" style="width:15%; height:auto;">
</p>
<p aling="center">
<img src="./img/image8.png" style="width:15%; height:auto;">
</p>

* **Configuración añadiendo autorizción externa y resultado de git clone**
<p aling="center">
<img src="./img/image6.png" style="width:15%; height:auto;">
</p>
<p aling="center">
<img src="./img/image9.png" style="width:15%; height:auto;">
</p>

Cabe resaltar que los permisos del `CI_JOB_TOKEN` son los mismo que del usuario que los ejecuta y por tanto se aplican los scopes que tenga ese usuario en la instacia de gitlab.

* **Código Pipeline**
```yaml
stages:
  - clone-repo
clone-repo:
    stage: clone-repo
    image: alpine:3.20
    before_script:
      - apk add git
    script:
        - git clone http://gitlab-ci-token:${CI_JOB_TOKEN}@gitlab.local:8888/bootcamp/ejercicios-gitlab.git
```
### Con el método deploy keys
Para el uso de deploys keys en el pipelines CI/CD se debe generar y configurar primero en la configuración de repositorio `Settings > Repository > Deploy Keys`

1. Se debe generar la llave SSH con la herramienta `ssh-keygen`
```bash
ssh-keygen -t rsa -b 2048 -C "springapp-key"
```
2. En la parte de `Deploy keys` se selecciona la opción `Add new key` y se le pega la llave pública generado del paso previo `.pub`
<p aling="center">
<img src="./img/image10.png" style="width:15%; height:auto;">
</p>

3. En el otro repositorio externo que se quiere clonar usando el deploy key. Se debe añadir la llave privada en una variable como forma de archivo (File)
<p aling="center">
<img src="./img/image11.png" style="width:15%; height:auto;">
</p>

4. Luego para hacer que SSH confíe en el host (gitlab.local). Se debe configurar el archivo `know_hosts` para ello se puede crear una nueva variable de tipo archivo `SSH_KNOW_HOSTS` que como valor tendrá el resultado del comando `ssh-keyscan gitlab.local`

<p aling="center">
<img src="./img/image12.png" style="width:15%; height:auto;">
</p>

5. El resultado de la operación es que se puede hacer correctamente el clonado del repositorio.

<p aling="center">
<img src="./img/image13.png" style="width:15%; height:auto;">
</p>

6. Cabe resaltar que aunque no se sea miembro del repositorioa a clonar. Se puede efectuar la opreación porque los deploy-keys están definidos para una interacción máquina-máquina que manejarán los permisos de lectura o escritura definidos.

* **Código Pipeline**
```yaml
stages:
  - clone-repo

clone-repo:
    image: ubuntu:24.04
    stage: clone-repo
    before_script:
      - 'command -v ssh-agent >/dev/null || ( apt-get update -y && apt-get install openssh-client git -y )'
      - eval $(ssh-agent -s)
      - chmod 400 "$SSH_PRIVATE_KEY"
      - ssh-add "$SSH_PRIVATE_KEY"
      - mkdir -p ~/.ssh
      - chmod 700 ~/.ssh
      - cp "$SSH_KNOWN_HOSTS" ~/.ssh/known_hosts
      - chmod 644 ~/.ssh/known_hosts
    script:
        - git clone git@gitlab.local:bootcamp/ejercicios-gitlab.git
```

### 1. CI/CD de una aplicación spring - OBLIGATORIO

* Crea un nuevo proyecto en GitLab y un repositorio en el mismo, para la aplicación `springapp`. El código fuente de la misma lo puedes encontrar en este [enlace](../02-gitlab/springapp).
* Sube el código al repositorio recientemente creado en GitLab.
* Crea una pipeline con los siguientes stages:
  * `maven:build` - En este `stage` el código de la aplicación se compila con [maven](https://maven.apache.org/).
  * `maven:test` - En este `stage` ejecutamos los tests utilizando [maven](https://maven.apache.org/).
  * `docker:build` - En este `stage` generamos una nueva imagen de Docker a partir del Dockerfile suministrado en el raíz del proyecto.
  * `deploy` - En este `stage` utilizamos la imagen anteriormente creada, y la hacemos correr en nuestro local.

* **Pistas**:
  * Utiliza la versión de maven 3.6.3
  * El comando para realizar una `build` con maven: `mvn clean package`
  * El comando para realizar los tests con maven: `mvn verify`
  * Cuando despleguemos la aplicación en local, podemos comprobar su ejecución en: `http://localhost:8080`

En resumen, la `pipeline` de `CI/CD`, debe hacer la build de la aplicación generando los ficheros jar, hacer los tests de maven y finalmente dockerizar la app (el dockerfile ya se proporciona en el repo) y hacer un deploy en local.

### 2. Crear un usuario nuevo y probar que no puede acceder al proyecto anteriormente creado - OBLIGATORIO

* Añadirlo con el role `guest`, comprobar que acciones puede hacer.
* Cambiar a role `reporter`, comprobar que acciones puede hacer.
* Cambiar a role `developer`, comprobar que acciones puede hacer.
* Cambiar a role `maintainer`, comprobar que acciones puede hacer.

Acciones a probar:

* Commit
* Ejecutar pipeline manualmente
* Push and pull del repo
* Merge request
* Acceder a la administración del repo

### 3. Crear un nuevo repositorio, que contenga una pipeline, que clone otro proyecto, springapp anteriormente creado - OPCIONAL

Relizar de las siguientes maneras:
  
* **Con el método de CI job token**
  * ¿Qué ocurre si el repo que estoy clonando no estoy cómo miembro?

> Pista: https://docs.gitlab.com/ee/ci/jobs/ci_job_token.html

* **Con el método deploy keys**
  * Crear deploy key en el repo springapp y poner solo lectura
  * Crear pipeline que use la deploy key para clonar el repo

 > Pista: https://docs.gitlab.com/ee/ci/ssh_keys/
