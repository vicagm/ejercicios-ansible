# Ejercicios curso ansible thinktic

## Ejercicio 1 / Primer despliegue

└── ansible.cfg\
└── inventory-dns\
└── main.yaml\
└── tasks\
    └── configure_network.yaml\
    └── install_mysql.yaml\
    └── install_nginx.yaml\
└── templates\
    └── rocky8_network.j2\
    └── ubuntu2204_network.j2\
└── vars\
    └── databases\
        └── vars.yaml\

En este ejercicio, cuando se lanza el playbook 'main.yaml' ubicado en la raíz del directorio se importan los playbooks ubicados en la carpeta 'tasks' en el siguiente orden.

### tasks/configure_network.yaml
Se ejecuta para todos los hosts y configura la red en los hosts haciendo uso de condiciones según su distribución.

En los hosts con distribución Debian se carga la plantilla 'templates/ubuntu2204_network.j2' haciendo el uso del módulo 'template' de ansible, en el caso de que lo haya realizado lanza el handler 'Restart network ubuntu' utilizando el módulo shell.

De igual manera, si la distribución es Rocky, hace lo propio con la plantilla 'templates/rocky8_network.j2' y en el caso de que la hubiese cambiado lanza el handler 'Restart network rocky'

### tasks/install_mysql.yaml
Se ejecutará solamente en los hosts configurados en el grupo databases del inventory-dns

Primeramente se crean las variables donde se guardan los valores de los paquetes requeridos para instalar mariadb, la url de la gpc key, el repositorio y los paquetes propios de mariadb. Se importa un fichero con las variables de la configuración del servidor. Se configura un handler para lanzar un 'FLUSH PRIVILEGES' cuando se hagan cambios en las bases de datos.

Se lanzan las tareas según la distribución utilizando condicionales, para la familia Debian se utiliza el módulo 'apt' en la instalación de paquetes y 'ufw' para las excepciones en el firewall.

Me faltaría realizar esto mismo para la familia Rocky que no estoy muy habituado a ella.

Una vez configurado el servidor y comprobado que está ejecutándose se realiza la configuración para el usuario root y el usuario configurado en el archivo de variables (vars/databases/vars.yaml) ejecutando el handler cuando sea necesario.

### tasks/install_nginx.yaml
Se ejecutará para los hosts configurados en el grupo 'webservers' del inventory-dns.

Configuramos una lista con los paquetes a instalar, lanzamos la instalación de esos paquetes con el módulo 'package' y verificamos que el servicio está arrancado con el módulo 'service'.

Después habilitamos los puertos en el firewall según su distribución. De momento sólo he realizado las tareas para ufw, no estoy familiarizado con Rockylinux.

## Ejercicio 2 / Despliegue de un docker-compose.yaml

└── ansible.cfg\
└── files\
    └── docker-compose.yaml\
└── inventory-dns\
└── tasks\
    └── wordpress_stack.yaml\

En este ejercicio se carga un fichero docker-compose.yaml con la configuración para lanzar un servidor mysql y wordpress. Cuando se lanza el playbook 'tasks/wordpress_stack.yaml' afectando solamente a los host agrupados en el grupo 'docker' se crea el directorio '/home/alumno/deploy' con el módulo 'file' y después se realiza un bucle con el módulo 'copy' para la copia de todos los ficheros que se necesiten, en este caso de la carpeta 'files' solo se cogerá 'docker-compose.yaml' y se copia en  '/home/alumno/deploy/docker-compose.yaml'

Seguidamente, se lanza con el módulo 'docker_compose' la ejecución del fichero compose anteriormente copiado generando así los contenedores, tras unos segundos después de levantar los servicios se podrá configurar wordpress accediendo con el navegador a la dirección de la máquina en el puerto expuesto en el fichero compose.

## Ejercicio 3 / Manejando vaults

└── 50-cloud-init.yaml\
└── ansible.cfg\
└── inventory\
└── inventory-dns\
└── tasks.yaml\
└── vars.yaml\

En este ejercicio, se hace uso de vaults para encriptar el fichero de variables (vars.yaml) que se le pasa al playbook (tasks.yaml).

Primeramente se configura en el fichero de configuración 'ansible.cfg' con el parámetro 'vault_password_file' pasándole el archivo '.vault_pass.txt' con la contraseña para encriptar/desencriptar el vault.

Escribimos el fichero de variables

```yaml
ruta_src: 50-cloud-init.yaml
ruta_dest: /home/alumno/50-cloud-init.yaml
user: alumno
group: alumno

#ruta_dest = /etc/netplan/50-cloud-init.yaml
```

Ejecutamos un ansible-vault con el parámetro encrypt y encriptamos el fichero.

```bash
ansible-vault encrypt vars.yaml
```
Con el parámetro view comprobamos que lo puede desencriptar.

Lanzamos el playbook 'tasks.yaml' pasando el fichero de variables encriptado y utilizando esas mismas variables para configurar el módulo de copia en los hosts agrupados en 'webservers'. En ese mismo playbook antes de hacer la copia se utilizan facts y sacamos por pantalla con el módulo 'debug' sus valores, con el módulo 'hostname' también se realiza el cambio de nombre de host. Finalmente se configura un handler para lanzar un reinicio del servicio de red, aunque en este caso no se ejecuta porque la ruta en la que se copia el fichero '50-cloud-init.yaml' no es '/etc/netplan/'.