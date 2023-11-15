### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

### **Angie Natalia Mojica - Daniel Antonio Santanilla**

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/es-es/free/students/). Al hacerlo usted contará con $100 USD para gastar durante 12 meses.

### Parte 0 - Entendiendo el escenario de calidad

Adjunto a este laboratorio usted podrá encontrar una aplicación totalmente desarrollada que tiene como objetivo calcular el enésimo valor de la secuencia de Fibonnaci.

**Escalabilidad**
Cuando un conjunto de usuarios consulta un enésimo número (superior a 1000000) de la secuencia de Fibonacci de forma concurrente y el sistema se encuentra bajo condiciones normales de operación, todas las peticiones deben ser respondidas y el consumo de CPU del sistema no puede superar el 70%.

### Parte 1 - Escalabilidad vertical

1. Diríjase a el [Portal de Azure](https://portal.azure.com/) y a continuación cree una maquina virtual con las características básicas descritas en la imágen 1 y que corresponden a las siguientes:
    * Resource Group = SCALABILITY_LAB
    * Virtual machine name = VERTICAL-SCALABILITY
    * Image = Ubuntu Server 
    * Size = Standard B1ls
    * Username = scalability_lab
    * SSH publi key = Su llave ssh publica

    ![Imágen 1](images/part1/part1-vm-basic-config.png)

    **Lo realizado:**

    ![labVMVERTICAL1](images/lab/1.png)

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM (Revise la sección "Connect" de la virtual machine creada para tener una guía más detallada).

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

    **Lo realizado:**

    Se ejecuta el siguiente comando para conectarse a la VM:

    ```bash
    ssh -i VERTICAL-SCALABILITY_key.pem scalability_lab@172.200.71.211
    ```

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).

    **Lo realizado:**

    Se ejecuta los siguientes comandos para instalar node:

    ```bash
    sudo apt update
    sudo apt install nodejs
    ```

4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    ```bash
    git clone https://github.com/An6ie02/ARSW_LAB9.git1
    cd ARSW_LAB9/FibonacciApp
    npm install
    ```

5. Para ejecutar la aplicación puede usar el comando `npm FibonacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    ```bash
    node FibonacciApp.js
    ```

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

    ![](images/part1/part1-vm-3000InboudRule.png)

    **Lo realizado:**

    Se verifica que este la regla de entrada

    ![labVMVERTICAL2](images/lab/19.png)

    Se prueba la aplicación en el navegador

    ![labVMVERTICAL3](images/lab/6.png)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000
    * 1010000
    * 1020000
    * 1030000
    * 1040000
    * 1050000
    * 1060000
    * 1070000
    * 1080000
    * 1090000

    **Lo realizado:**

    ![tiemposVMVERTICALantes](./images/lab/7.png)

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

    ![Imágen 2](images/part1/part1-vm-cpu.png)

    **Lo realizado:**

    ![consumoVMVERTICALantes](./images/lab/8.png)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```bash
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

    **Lo realizado:**

    Se instalo newman con el siguiente comando:

    ```bash
    npm install -g newman
    ```

    Se cambió el valor del parámetro VM1

    ```json
    "key": "VM1",
    "value": "172.200.71.211",
    "enabled": true
    ```

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

    ![Imágen 3](images/part1/part1-vm-resize.png)

    **Lo realizado:**

    Se cambió el tamaño de la VM a B2ms

    ![labVMVERTICAL4](images/lab/15.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.

    **Lo realizado:**

    ![tiemposVMVERTICALdespues](./images/lab/11.png)

    ![consumoVMVERTICALdespues](./images/lab/12.png)

    ![postmanVMVERTICALdespues](./images/lab/13.png)

12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.

    **Lo realizado:**

    Sí, ya que al aumentar el tamaño de la VM, se aumenta la cantidad de CPU y memoria RAM, por lo que la aplicación puede procesar más peticiones en menos tiempo.

13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?
   
    Azure junto con la VM crea 6 recursos adicionales que son:

    ![recursosVMVERTICAL](./images/lab/9.png)


2. ¿Brevemente describa para qué sirve cada recurso?
   
    **Public Ip address:** Las direcciones IP públicas permiten que los recursos de Azure se comuniquen con Internet y con los servicios públicos de Azure. Dedicas la dirección al recurso (VM) hasta que lo desasignas.

    **Network security group:** Puede usar un grupo de seguridad de red de Azure para filtrar el tráfico de red hacia y desde los recursos de Azure en una red virtual de Azure. Un grupo de seguridad de red contiene reglas de seguridad que permiten o deniegan el tráfico de red entrante o el tráfico de red saliente desde varios tipos de recursos de Azure. Para cada regla, puede especificar origen y destino, puerto y protocolo.

    **Virtual Network:** Azure Virtual Network es un servicio que proporciona el componente fundamental para su red privada en Azure. Una instancia del servicio (una red virtual) permite que muchos tipos de recursos de Azure se comuniquen de forma segura entre sí, con Internet y con las redes locales. Estos recursos de Azure incluyen máquinas virtuales (VM).

    Una red virtual es similar a una red tradicional que operaría en su propio centro de datos. Pero aporta beneficios adicionales de la infraestructura de Azure, como escala, disponibilidad y aislamiento.

    **Network Interface:** Una interfaz de red (NIC) permite que una máquina virtual (VM) de Azure se comunique con Internet, Azure y recursos locales. Una máquina virtual que crea en Azure Portal tiene una NIC con la configuración predeterminada.

    **SSH Key:** Las SSH key se usan para conectarse a máquinas virtuales (VM) en Azure.

    **Disk:** Los discos administrados de Azure son volúmenes de almacenamiento a nivel de bloque administrados por Azure y utilizados con Azure Virtual Machines. Los discos administrados son como un disco físico en un servidor local pero virtualizados. Con los discos administrados, todo lo que tiene que hacer es especificar el tamaño del disco, el tipo de disco y aprovisionarlo. Una vez que aprovisiona el disco, Azure se encarga del resto.

3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?

    El comando npm FibonacciApp.js inicia un proceso que solo funciona si hay una conexión activa. Si la conexión se cierra, la aplicación se cierra y deja de funcionar.

    Debemos crear un Inbound port rule para permitir el tráfico de entrada al puerto 3000, ya que por defecto Azure no permite el tráfico de entrada a ningún puerto, y este es necesario para que la aplicación funcione.

4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.

    La aplicación tarda tanto porque se calculan todos los números de la secuencia de Fibonacci hasta el número que se desea calcular, y esto toma mucho tiempo.

    **Antes del escalamiento:**

    ![tiemposVMVERTICALantes](./images/lab/7.png)

    **Después del escalamiento:**

    ![tiemposVMVERTICALdespues](./images/lab/11.png)

5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.

    La función consume esa cantidad de CPU porque se calculan todos los números de la secuencia de Fibonacci hasta el número que se desea calcular, y esto necesita muchos recursos de CPU.

    **Antes del escalamiento:**

    ![consumoVMVERTICALantes](./images/lab/8.png)

    **Después del escalamiento:**

    ![consumoVMVERTICALdespues](./images/lab/12.png)

    **Gráfica completa:** (El punto en la gráfica es el momento en el que se realizó el escalamiento)

    ![consumoVMVERTICAL](./images/lab/14.png)

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.

        **Antes del escalamiento:**

        ![postmanVMVERTICALantes](./images/lab/10.png)

        **Después del escalamiento:**

        ![postmanVMVERTICALdespues](./images/lab/13.png)

    * Si hubo fallos documentelos y explique.

        El error ECONNRESET significa que la conexión TCP en su cliente Postman fue cerrada inesperadamente por el servidor o algún intermediario como un proxy. En el contexto de su prueba de Postman, parece que la solicitud a la API "fibonacci" en la iteración 2 fue interrumpida antes de que pudiera completarse.

        Este error puede ser causado por varias razones, como un servidor que se cierra inesperadamente, un tiempo de espera de la conexión, problemas de red, etc. Para solucionarlo, puede necesitar investigar el servidor y la red en la que se está ejecutando.

        ![postmanVMVERTICALfallos](./images/lab/16.png)	

7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?

    Las VM de la serie B se pueden implementar en diversos tipos de hardware y procesadores, por lo que se proporciona una asignación de ancho de banda competitiva. La serie B se ejecuta en procesadores de 3.ª generación Intel® Xeon® (Ice Lake), Intel® Xeon® (Cascade Lake), Intel® Xeon® (Skylake), Intel® Xeon® (Broadwell) o Intel® Xeon® (Haswell). 
    
    Las VM de la serie B son idóneas para cargas de trabajo que no necesitan un rendimiento completo de la CPU de forma continua, como los servidores web, las pruebas de concepto, las bases de datos pequeñas y los entornos de compilación de desarrollo. Estas cargas de trabajo suelen necesitar unos requisitos de rendimiento ampliables.

    La serie B incluye los siguientes tamaños de máquina virtual:

     - Unidad de proceso de Azure (ACU): varía.*
     - Premium Storage: Compatible
     - Almacenamiento en caché de Premium Storage: No compatible
     - Migración en vivo: Compatible
     - Actualizaciones con conservación de memoria: Compatible
     - Compatibilidad con generación de VM: Generación 1 y 2
     - Redes aceleradas: Compatible \*\*
     - Discos de sistema operativo efímero: Compatible
     - Virtualización anidada: no compatible

    | Característica | Standard_B1ls | Standard_B1ms |
    |---|---|---|
    | **vCPU** | 1 | 1 |
    | **Memoria: GiB** | 0.5 | 2 |
    | **GiB de almacenamiento temporal (SSD)** | 4 | 4 |
    | **Rendimiento de CPU base de la máquina virtual (%)** | 10 | 40 |
    | **Créditos iniciales** | 30 | 30 |
    | **Créditos ingresados/hora** | 72 | 12 |
    | **Créditos máximos ingresados** | 72 | 288 |
    | **Discos de datos máx.** | 2 | 2 |
    | **Rendimiento máximo del disco sin almacenamiento en la caché: IOPS/Mbps** | 160/10 | 640/10 |
    | **Rendimiento máximo del disco sin almacenamiento en la caché expandido: IOPS/MBps1** | 4000/100 | 4000/100 |
    | **Nº máx. NIC** | 2 | 2 |

8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?

    No, porque el aumento del tamaño de la VM no es una solución escalable, ya que si se aumenta el número de peticiones, la VM no podrá procesarlas todas y se volverá a tener el mismo problema de consumo de CPU.

    Cuando se cambia el tamaño de la VM, la aplicación deja de funcionar y se debe volver a ejecutar.

9.  ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?

    ![infraestructuraVMVERTICAL](./images/lab/15.png)

    Al cambiar el tamaño de la VM, se debe reiniciar la máquina, por lo que se pierde la conexión ssh y la aplicación deja de funcionar. 

10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?

    Sí, porque al aumentar el tamaño de la VM, se aumenta la cantidad de CPU y memoria RAM, por lo que la aplicación puede procesar más peticiones en menos tiempo.

11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

    El comportamiento es porcentualmente mejor porque se pueden procesar más peticiones en menos tiempo. Ésto se puede ver en el punto de la siguiente gráfica de consumo de CPU.

    ![postmanVMVERTICAL4](./images/lab/17.png)

    ![consumoVMVERTICAL4](./images/lab/18.png)

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

![](images/part2/part2-lb-create.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

![](images/part2/part2-lb-bp-create.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

![](images/part2/part2-lb-hp-create.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

![](images/part2/part2-lb-lbr-create.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

![](images/part2/part2-vn-create.png)

#### Crear las maquinas virtuales (Nodos)

Ahora vamos a crear 3 VMs (VM1, VM2 y VM3) con direcciones IP públicas standar en 3 diferentes zonas de disponibilidad. Después las agregaremos al balanceador de carga.

1. En la configuración básica de la VM guíese por la siguiente imágen. Es importante que se fije en la "Avaiability Zone", donde la VM1 será 1, la VM2 será 2 y la VM3 será 3.

![](images/part2/part2-vm-create1.png)

2. En la configuración de networking, verifique que se ha seleccionado la *Virtual Network*  y la *Subnet* creadas anteriormente. Adicionalmente asigne una IP pública y no olvide habilitar la redundancia de zona.

![](images/part2/part2-vm-create2.png)

3. Para el Network Security Group seleccione "avanzado" y realice la siguiente configuración. No olvide crear un *Inbound Rule*, en el cual habilite el tráfico por el puerto 3000. Cuando cree la VM2 y la VM3, no necesita volver a crear el *Network Security Group*, sino que puede seleccionar el anteriormente creado.

![](images/part2/part2-vm-create3.png)

4. Ahora asignaremos esta VM a nuestro balanceador de carga, para ello siga la configuración de la siguiente imágen.

![](images/part2/part2-vm-create4.png)

5. Finalmente debemos instalar la aplicación de Fibonacci en la VM. para ello puede ejecutar el conjunto de los siguientes comandos, cambiando el nombre de la VM por el correcto

```
git clone https://github.com/daprieto1/ARSW_LOAD-BALANCING_AZURE.git

curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.34.0/install.sh | bash
source /home/vm1/.bashrc
nvm install node

cd ARSW_LOAD-BALANCING_AZURE/FibonacciApp
npm install

npm install forever -g
forever start FibonacciApp.js
```

Realice este proceso para las 3 VMs, por ahora lo haremos a mano una por una, sin embargo es importante que usted sepa que existen herramientas para aumatizar este proceso, entre ellas encontramos Azure Resource Manager, OsDisk Images, Terraform con Vagrant y Paker, Puppet, Ansible entre otras.

#### Probar el resultado final de nuestra infraestructura

1. Porsupuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

```
http://52.155.223.248/
http://52.155.223.248/fibonacci/1
```

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?
* ¿Cuál es el propósito del *Backend Pool*?
* ¿Cuál es el propósito del *Health Probe*?
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
* ¿Cuál es el propósito del *Network Security Group*?
* Informe de newman 1 (Punto 2)
* Presente el Diagrama de Despliegue de la solución.




