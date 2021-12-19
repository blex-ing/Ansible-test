# Ansible-test

## ¿Cómo ejecutar una colección usando ansible-test?
Un usuario puede ejecutar ansible-test dentro de una colección para probar la colección en si, se puede ejecutar desde la raíz de la 
colección o por debajo para que ansible-test ejecute las pruebas de colección.

## La estructura de directorios de una colección en Ansible es:
Al instalar una colección, la ubicación de instalación predeterminada es la ruta anterior, que ya satisface la convenció de directorio.
ansible-test proporciona formas diferentes de ejecutar diferentes tipos de pruebas en sus colecciones, los cuales son:

1.  Pruebas de integración.
    user@ubuntu
    ansible-test integration
2.  Pruebas unitarias.
    user@ubuntu
    ansible-test units
3.  Pruebas de cordura.
    user@ubuntu
    ansible-test sanity

## __Pruebas de integración__

Estas son pruebas de extremo a extremo para verificar las funciones de ruta de código como se esperaba y para detectar cambios 
importantes en el producto que está tratando de automatizar.
Para poder ejecutar todas las pruebas, se usa la herramienta ansible-test de Ansible, que realiza una amplia variedad de tareas, desde la 
documentación hasta la ejecución de pruebas unitarias y de integración. Pero cómo se ha mencionado antes, debemos preparar una 
estructura de directorios.

{…..}/ansible_collection/{namespace}/{collection}/

