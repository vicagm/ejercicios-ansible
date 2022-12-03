# Ejercicios curso ansible thinktic

## Ejercicio 1 / Primer despliegue

├── ansible.cfg
├── inventory-dns
├── main.yaml
├── tasks
│   ├── configure_network.yaml
│   ├── install_mysql.yaml
│   └── install_nginx.yaml
├── templates
│   ├── rocky8_network.j2
│   └── ubuntu2204_network.j2
└── vars
    └── databases
        └── vars.yaml

En este ejercicio, cuando se lanza el playbook main.yaml ubicado en la raíz del directorio se importan los playbooks ubicados en la carpeta 'tasks' en el siguiente orden.

### tasks/configure_network.yaml
Se ejecuta para todos los hosts y configura la red en los hosts haciendo uso de condiciones según su distribución.

En los hosts con distribución Debian se carga la plantilla 'templates/ubuntu2204_network.j2' haciendo el uso del módulo template de ansible, en el caso de que lo haya realizado lanza el handler 'Restart network ubuntu' haciendo uso del modulo shell.

De igual manera, si la distribución es Rocky, hace lo propio con la plantilla 'templates/rocky8_network.j2' y en el caso de que la hubiese cambiado lanza el handler 'Restart network rocky'

### tasks/install_mysql.yaml
Se ejecutará solamente en los hosts configurados en el grupo databases del inventory-dns

Primeramente se crean las variables donde se guardan los valores de los paquetes requeridos para instalar mariadb, la url de la gpc key, el repositorio y los paquetes propios de mariadb. Se importa un fichero con las variables de la configuración del servidor. Se configura un handler para lanzar un 'FLUSH PRIVILEGES' cuando se hagan cambios en las bases de datos.

Se lanzan las tareas según la distribución utilizando condicionales, para la familia Debian se utiliza el módulo apt en la instalación de paquetes y ufw para las excepciones en el firewall.

Me faltaría realizar esto mismo para la familia Rocky que no estoy muy habituado a ella.

Una vez configurado el servidor y comprobado que está ejecutándose se realiza la configuración para el usuario root y el usuario configurado en el archivo de variables (vars/databases/vars.yaml) ejecutando el handler cuando sea necesario.

### tasks/install_nginx.yaml
Se ejecutará para los hosts configurados en el grupo 'webservers' del inventory-dns.

Configuramos una lista con los paquetes a instalar, lanzamos la instalación de esos paquetes con el módulo 'package' y verificamos que el servicio está arrancado con el módulo 'service'.

Después habilitamos los puertos en el firewall según su distribución.