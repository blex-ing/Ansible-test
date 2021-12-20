# README.MD

### __Desafío__
Los servidores SSH permiten diversos métodos de autenticación, la mayoría basada en contraseña. Si algún usuario del sistema tiene una contraseña débil, un atacante puede resolver fácilmente la contraseña y crear una conexión SSH no autorizada.

### __Solución__ 
Instalación de vault-ssh-helper, permite al cliente que se conecta a través de la conexión SSH a un host de destino usar  la clave de un solo uso (OTP) que le ha proporcionado el servidor de Vault. Si el cliente está autorizado utiliza esta OTP durante la autenticación SSH para conectarse al host de destino deseado, Vault SSH Agent del servidor recibe la OTP y valida esta OTP con el servidor de Vault. El servidor de Vault elimina esta OTP, asegurándose que solo se conecte una vez.

## Estructura del proyecto
El proyecto se divide en 4 partes.

1. En el directorio principal puede encontrar 2 playbook padres principales, el __install_vault_ssh_helper.yml__ y __uninstall_vault_ssh_helper.yml__ que llamaran a los playbooks hijos correspondientes en función del tipo de sistema operativo y versión del host remoto al que se esté atacando desde Ansible tower.

2. En el directorio __[/os](./os)__ puede encontrar los playbooks de cada uno de los sistemas operativos y versiones en el que este proyecto es capaz de instalar vault-ssh-helper, los sistemas operativos  y versiones son:
    1. Amazon linux 1, 2.
    2. RedHat 6, 7, 8.
    3. Centos 6, 7, 8.
    4. Ubuntu 14, 16, 18, 20.
    5. Debian 9, 10.
    6. Suse 11, 12, 15.

3. En el directorio __[/packages](./packages)__ puede encontrar los diferentes paquetes, archivos, módulos que necesita para la correcta instalación de vault-ssh-helper.
    1. __certificate:__ contiene el certificado que permite la comunicación con el servidor de vault.
    2. __module:__ contiene los distintos módulos que se usarán cuando el host remoto este activado __selinux__. Los módulos que se han creado, funcionan en los sistemas operativos __RedHat__ y __Centos__ versión [7,8], __amazon linux__ versión [2].
    3. __unzip__ contiene el paquete de intalación de cada sistema operativo soportado.
    4. __vault_ssh__ almacena el paquete de instalación de vault-ssh-helper.

4. En el directorio __[/parameters](./parameters)__ se encuentran dos ficheros .json, uno con las variables de configuración para el proceso de instalación y otro con las variables de configuración para el proceso de desinstalación.

## Instalación
Para el proceso de instalación de vault-ssh-helper que se realiza a través del playbook en el host remoto, se sigue los pasos de instalación del tutorial que puede ver en el siguiente enlace  [ssh-otp](https://learn.hashicorp.com/tutorials/vault/ssh-otp).
En cada __host remoto__, se debe instalar [vault-ssh-helper](https://github.com/hashicorp/vault-ssh-helper).

### Tareas del playbook
El playbook crea un archivo de configuración __/etc/vault-ssh-helper.d/config.hcl__ y modifica el archivo de configuración sshd __/etc/pam.d/sshd__ del módulo de autenticación conectable (PAM) y el archivo de configuración sshd __/etc/ssh/sshd_config__ y finalmente, reinicia el servicio sshd.

### Playbook de instalacion de vault-ssh-helper
Todas las tareas del playbook antes de llevar a cabo una acción que modifique, añada o quite algo en el host remoto, se realiza la comprobación del estado en el que se encuentra al host remoto.

* Tarea 1.0: Comprueba si en el host remoto ya está instalado vault-ssh-helper
    * Tarea 1.1: Si la tarea anterior muestra que vault-ssh-helper está en el host remoto, se ejecuta
    ```bash
    /usr/local/bin/vault-ssh-helper -verify-only  -config /etc/vault-ssh-helper.d/config.hcl
    ```
Vault-ssh-helper viene comprimido en un archivo .zip, para ello necesitamos que en el host remoto este instalado la herramienta __unzip__, y si no está instalado el playbook lo instala.

* Tarea 2.0: Comprueba si está instalado el programa unzip y si no está instalado.
    * Tarea 2.1: Copia el paquete de instalación en el host remoto.
    * Tarea 2.2: Instala el Unzip.

* Tarea 3.0: Comprueba si el paquete de instalación de vault-ssh-helper está en el host remoto, y si no está.
    * Tarea 3.1: Copia el paquete __vault-ssh-helper_0.2.0_linux_amd64.zip__ en el host remoto.
    * Tarea 3.2: Descomprime el paquete en la ruta __/usr/local/bin__.

* Tarea 4.0: Crea el directorio __/etc/vault-ssh-helper.d__
    * Tarea 4.1: Crea el fiche de configuración __config.hcl__ en el directorio anterior.
    * Tarea 4.2: Edita el fichero __config.hcl__ (se ha añadido una línea nueva con el certificado que tiene que coger).

* Tarea 5.1: Realiza un Backup del archivo /etc/pam.d/sshd
    * Tarea 5.2: Teniendo en cuenta la particularidad del sistema operativo se comenta varias líneas del archivo __sshd__.
        * Amazon Linux, RedHat, Centos: se comenta.
            1. #auth       substack     password-auth
            2. #password   include      password-auth

        * Linux, Debian: se comenta.
            1. #@include common-auth

        * Suse: se comenta 
            1. #auth        include     common-auth
    
    * Tarea 5.3: Añade dos lineas al archivo.
        1. auth       optional       pam_unix.so not_set_pass use_first_pass nodelay
        2. auth       requisite      pam_exec.so quiet expose_authtok log=/var/log/vault-ssh.log /usr/local/bin/vault-ssh-helper -config=/etc/vault-ssh-helper.d/config.hcl

* Tarea 6.1: Realiza un Backup del archivo /etc/ssh/sshd_config
    * Tarea 6.2 y 6.3: Modifica el archivo de configuración __sshd_config__.
    
    * Comenta.
        1. #ChallengeResponseAuthentication no
        2. #UsePAM no
        3. #PasswordAuthentication yes
    * Des comenta.
        1. ChallengeResponseAuthentication yes
        2. UsePAM yes
        3. PasswordAuthentication no

* Tarea 7.0: Esta tarea se encarga de copiar el certificado __vault_portal.crt__ que permite la conexión con el servidor de vault mediante una variable definida previamente en un fichero .json.

* Tarea 8.0: Reinicia el servicio __sshd__.
    * Tarea 8.1: valida si la configuración es correcta.
    ```bash
    /usr/local/bin/vault-ssh-helper -verify-only  -config /etc/vault-ssh-helper.d/config.hcl
    ```

* Tareas Rv 0 al 5: Elimina todos los archivos que se han usado para instalar vault-ssh-helper que se han copiado en el directorio /tmp.

### Playbook de desinstalación.

El playbook de desinstalación, elimina todas los archivos modificados y restaura las backups de estos archivos. Además elimina los programas que se instalaron cuando se instaló vault-ssh-helper.

### Observaciones playbook instalación (debian y ubuntu)

En el playbook de instalación perteneciente a las distribuciones Debian y Ubuntu, se han comentado las tareas 9.0, 9.1 y rollback para evitar fallos a la hora de la instalación, de modo que Vault solo se instalará en dichas distribuciones si __selinux__ está desactivado.

### Observaciones playbook desinstalación

Se han comentado las tareas 6.0, 7.0, 7.1, 7.2, 8.0, 8.1, 9.0, 9.1 y 9.2 debido a un fallo que dejaba la instancia sobre la que se ejecutaba el playbook inservible, por el contrario, se agregan las tareas 5.2, 5.2.1, 5.2.2, 5.3  y en redhat, ami_linux y centos 5.4 que devuelven la configuración del fichero __sshd__ a su estado inicial.

## Particularidades del proyecto.

La instalación de Vault-ssh-helper es posible aun cuando el servicio de Selinux este activo en los sistemas operativos, Amazon linux [2], Redhat [7, 8], Centos [7, 8]. En los sistemas operativos Ubuntu, Debian, Suse el funcionamiento se interrumpe cuando Selinux está activo.

Para crear las políticas que permiten el funcionamiento de vault-ssh-helper cuando está activo __selinux__ en modo enforcing, los pasos a seguir lo puede ver en el video [módulos selinux](https://youtu.be/hbHu6QyED9I?t=1461).
