### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW

## Escalamiento en Azure con Maquinas Virtuales, Sacale Sets y Service Plans

### Dependencias
* Cree una cuenta gratuita dentro de Azure. Para hacerlo puede guiarse de esta [documentación](https://azure.microsoft.com/en-us/free/search/?&ef_id=Cj0KCQiA2ITuBRDkARIsAMK9Q7MuvuTqIfK15LWfaM7bLL_QsBbC5XhJJezUbcfx-qAnfPjH568chTMaAkAsEALw_wcB:G:s&OCID=AID2000068_SEM_alOkB9ZE&MarinID=alOkB9ZE_368060503322_%2Bazure_b_c__79187603991_kwd-23159435208&lnkd=Google_Azure_Brand&dclid=CjgKEAiA2ITuBRDchty8lqPlzS4SJAC3x4k1mAxU7XNhWdOSESfffUnMNjLWcAIuikQnj3C4U8xRG_D_BwE). Al hacerlo usted contará con $200 USD para gastar durante 1 mes.

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

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM.

    `ssh scalability_lab@xxx.xxx.xxx.xxx`
       

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).
4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    `npm install forever -g`

    `forever start FibinacciApp.js`

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

![](images/part1/part1-vm-3000InboudRule.png)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000
    
    ![100000](https://github.com/mariahv9/ARSWLab8/blob/main/resources/1000000.png)
    
    * 1010000
    
    ![1010000](https://github.com/mariahv9/ARSWLab8/blob/main/resources/1010000.png)
    
    * 1020000
    
    ![1020000](https://github.com/mariahv9/ARSWLab8/blob/main/resources/1020000.png)
    
    * 1030000
    
    ![1030000](https://github.com/mariahv9/ARSWLab8/blob/main/resources/1030000.png)
    
    * 1040000
    
    ![1040000](https://github.com/mariahv9/ARSWLab8/blob/main/resources/1040000.png)
    
    * 1050000
    
    ![1050000](https://github.com/mariahv9/ARSWLab8/blob/main/resources/1050000.png)
    
    * 1060000
    
    ![1060000](https://github.com/mariahv9/ARSWLab8/blob/main/resources/1060000.png)
    
    * 1070000
    
    ![1070000](https://github.com/mariahv9/ARSWLab8/blob/main/resources/1070000.png)
    
    * 1080000
    
    ![1080000](https://github.com/mariahv9/ARSWLab8/blob/main/resources/1080000.png)
    
    * 1090000    
    
    ![1090000](https://github.com/mariahv9/ARSWLab8/blob/main/resources/1090000.png)

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)

![](https://github.com/mariahv9/ARSWLab8/blob/main/resources/pruebauso1.png)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```
    
10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

![](https://github.com/mariahv9/ARSWLab8/blob/main/resources/resize.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.

12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.

El consumo de la CPU se redujo pero los tiempos de espera no cambiaron.

13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?
- Virtual Network (VNet).
- Direcciones IP Privadas.
- Direcciones IP Públicas.
- Grupos de seguridad de Red (NSG).

2. ¿Brevemente describa para qué sirve cada recurso?
- Virtual Network (VNet): Es una división lógica de redes dentro de nuestra suscripción a Azure. Por defecto, máquinas virtuales que estén en diferentes VNets no podrán conectarse entre sí. Podemos tener más de una VNet con el mismo rango de IPs, aunque no es una buena idea si después queremos conectarlas. Vemos que podemos crear otra VNet con el mismo rango de IPs y solo nos muestra un aviso.

- Direcciones IP Privadas: Se usan internamente en la VNet para comunicar entre sí máquinas virtuales que pertenecen a la misma subnet, máquinas que están en VNets diferentes mediante VNet peering, para conectar máquinas en Azure con máquinas on-premises mediante una VPN, para conectar máquinas virtuales en dos regiones diferentes de Azure mediante una VPN y también para conectar máquinas virtuales con su balanceador de carga o su gateway. Estas direcciones IP privadas NO se deben cambiar desde el sistema operativo de la máquina virtual.

- Direcciones IP Públicas: Se utilizan para la conexión desde Internet a la máquina virtual. La primera NIC que creamos en una VM se denomina Primary y tiene asignada la IP pública, el resto de NICs son secundarias y no tienen IP Pública.

- Grupos de seguridad de Red (NSG): Conjuntos de reglas que permiten controlar los paquetes entrantes y salientes a las máquinas virtuales. Se pueden asignar tanto a nivel de VNets y subnets como a nivel de NICs. Entraremos en detalle en otro post.

3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?

Al conectarnos con la máqina virtual creada por SSH y realizar conexion por diferentes servicios, realizamos un proceso de metaprocesos, por lo que al cerrar la conexión padre SSH se van a cerrar todas las conexiones realizadas a partir de este.

La regla de puerto de entrada se configura para poder hacer conexion tipo UDP a diferentes servicios que queramos instanciar en nuestra máquina virtual permitiendo el acceso público de nuestro servicio, en este caso publicamos la aplicación de Fibonacci.

4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.

![](https://github.com/mariahv9/ARSWLab8/blob/main/resources/t1.png)

![](https://github.com/mariahv9/ARSWLab8/blob/main/resources/t2.png)

5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.

![](https://github.com/mariahv9/ARSWLab8/blob/main/resources/pruebasuso2.png)

Consume mucha CPU ya que la implementación no es eficaz.

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    
* B1ls

![](https://github.com/mariahv9/ARSWLab8/blob/main/resources/pman1.png)

* B2ms

![](https://github.com/mariahv9/ARSWLab8/blob/main/resources/pman11.png)

Los tiempos son altos y hay fallos de desconexión.

7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?

El disco B1ls tiene de memoria 0.5 GiB y la B2ms tiene 8 GiB, esto permite que el almacenamiento interno sea mayor, el rendimiento de la CPU del disco B2ms es el doble al del disco B1ls. Las mejoras del disco B2ms son mayores y eso se ve reflejado en el consumo de créditos consumidos en comparación a la B1ls.
 
8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?

Al aumentar el tamaño del disco se hace un consumo menor de recursos, pero en las pruebas realizadas y en las tablas se puede observar que los tiempo no varian considerablemente, para que esto funcione y mejore en rendimiento tocaría mejorar la solución del fibonacci a nivel de implementación, agregando hilos sería una buena solución.

9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?

Al hacer el cambio de infraestructura el servicio requiere un reinicio, lo cual afecta la disponibilidad y podría tener consecuencias impactantes para los negocios.

10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?

Si hubo mejora en la CPU, ya que, al hacer el cambio de infraestructuras se redujo el porcentaje de uso a la mitad.

11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

No hay mejoras y los fallos se mantienen en el porcentaje.

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

![](https://github.com/mariahv9/ARSWLab8/blob/main/resources/hello.png)

![](https://github.com/mariahv9/ARSWLab8/blob/main/resources/pr1.png)

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

```
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
```
* VM1

![](https://github.com/mariahv9/ARSWLab8/blob/main/resources/1_4.png)

* VM2

![](https://github.com/mariahv9/ARSWLab8/blob/main/resources/2_4.png)

* VM3

![](https://github.com/mariahv9/ARSWLab8/blob/main/resources/3_4.png)

* VM4

![](https://github.com/mariahv9/ARSWLab8/blob/main/resources/4_4.png)

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?, ¿Qué es SKU, qué tipos hay y en qué se diferencian?, ¿Por qué el balanceador de carga necesita una IP pública?

Un **equilibrador de carga público** puede proporcionar conexiones de salida para máquinas virtuales dentro de la red virtual. Estas conexiones se realizan mediante la traducción de sus direcciones IP privadas a direcciones IP públicas. Las instancias públicas de Load Balancer se usan para equilibrar la carga del tráfico de Internet en las máquinas virtuales.

Un **equilibrador de carga interno (o privado)** se usa cuando se necesitan direcciones IP privadas solo en el front-end. Los equilibradores de carga internos se usan para equilibrar la carga del tráfico dentro de una red virtual. También se puede acceder a un servidor front-end del equilibrador de carga desde una red local en un escenario híbrido.

![](https://github.com/mariahv9/ARSWLab8/blob/main/resources/lbpregunta.png)

*Ilustración: Equilibrar las aplicaciones de niveles múltiples mediante Load Balancer público e interno*

* **Sku** Representa una unidad de mantenimiento de existencias (SKU) que se puede comprar debajo de un producto. Estos representan las diferentes formas del producto.

* ¿Cuál es el propósito del *Backend Pool*?

Define el grupo de recursos que brindarán tráfico para una regla de equilibrio de carga determinada.

* ¿Cuál es el propósito del *Health Probe*?

Configura una sonda de estado que su balanceador de carga puede usar para determinar si su instancia está en buen estado. Si su instancia falla su prueba de estado suficientes veces, dejará de recibir tráfico hasta que comience a pasar las pruebas de estado nuevamente.

* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.

Abre un puerto orientado a Internet y reenvía el tráfico al puerto del nodo interno que usa su aplicación. Si no tiene un equilibrador de carga, consulte Configurar un equilibrador de carga orientado a Internet.

* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?

- Azure Virtual Network (VNet) es el bloque de construcción fundamental para su red privada en Azure. VNet permite que muchos tipos de recursos de Azure, como Azure Virtual Machines (VM), se comuniquen de forma segura entre sí, con Internet y con las redes locales. VNet es similar a una red tradicional que operaría en su propio centro de datos, pero trae consigo beneficios adicionales de la infraestructura de Azure, como escala, disponibilidad y aislamiento.

- Subnet: La migración de cargas de trabajo a la nube pública requiere una planificación y coordinación cuidadosas. Una de las consideraciones clave puede ser la capacidad de conservar sus direcciones IP. Lo cual puede ser importante, especialmente si sus aplicaciones dependen de la dirección IP o si tiene requisitos de cumplimiento para usar direcciones IP específicas. Azure Virtual Network resuelve este problema al permitirle crear redes virtuales y subredes utilizando un rango de direcciones IP de su elección.

- Address space: El espacio de direcciones puede referirse a un rango de direcciones físicas o virtuales accesibles a un procesador o reservadas para un proceso .

- Address range: Las direcciones IP se pueden clasificar en cinco clases A, B, C, D y E. Cada clase consta de un subconjunto contiguo del rango general de direcciones IPv4.
 
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?

- Availability Zone: Las zonas de disponibilidad son ubicaciones físicas únicas con alimentación, red y refrigeración independientes. Cada zona de disponibilidad se compone de uno o más centros de datos y alberga infraestructura para admitir aplicaciones de misión crítica de alta disponibilidad. Las zonas de disponibilidad son tolerantes a las fallas del centro de datos mediante la redundancia y el aislamiento lógico de los servicios.

Se eligieron 3 zonas distintas para tener mejor disponibilidad y mayor tolerancia a fallos, en caso de que falle alguna zona otra zona se hará cargo del servicio.

- Las zonas de disponibilidad proporcionan aislamiento de fallas mediante la separación física. Cada zona consta de uno o más centros de datos con alimentación, red y refrigeración independientes. Considere el almacenamiento con redundancia de zona para aplicaciones donde se requiere acceso de lectura y escritura de alta disponibilidad en una región de Azure.

* ¿Cuál es el propósito del *Network Security Group*?

Le permiten configurar la seguridad de la red como una extensión natural de la estructura de una aplicación, lo que le permite agrupar máquinas virtuales y definir políticas de seguridad de red basadas en esos grupos. 

* Informe de newman 1 (Punto 2)

Se puede ver que los tiempos disminuyeron y a su vez los fallos, esto se da por que al tener el balanceador de carga los números de peticiones se equilibran en el servidor.

* VM1

![](https://github.com/mariahv9/ARSWLab8/blob/main/resources/vm1lb.png)

* VM2

![](https://github.com/mariahv9/ARSWLab8/blob/main/resources/vm2lb.png)

* VM3

![](https://github.com/mariahv9/ARSWLab8/blob/main/resources/vm3lb.png)


* Presente el Diagrama de Despliegue de la solución.

![](https://github.com/mariahv9/ARSWLab8/blob/main/resources/diagrama.png)


## Referencias

* [Balanceador de Carga](https://docs.microsoft.com/es-es/azure/load-balancer/load-balancer-overview)
* [Sku](https://docs.microsoft.com/en-us/partner-center/develop/product-resources#sku)
* [Backend pool](https://docs.microsoft.com/en-us/azure/load-balancer/backend-pool-management)
* [Health probe](https://www.bluematador.com/docs/troubleshooting/azure-load-balancer-health-probe)
* [Rule LB](https://docs.microsoft.com/en-us/azure/service-fabric/create-load-balancer-rule)
* [Virtual network](https://docs.microsoft.com/en-us/azure/virtual-network/virtual-networks-overview)
* [Subnet](https://docs.microsoft.com/en-us/azure/virtual-network/subnet-extension)
* [Address space](https://searchstorage.techtarget.com/definition/address-space)
* [Address range](http://techiebird.com/networkclass.html)
* [Availibility zone](https://azure.microsoft.com/en-us/global-infrastructure/availability-zones/)
* [Zone redundant](https://azure.microsoft.com/en-gb/updates/azure-zrs/)
* [Zone redundant]()

## Construido con 

* [Javascript](https://www.javascript.com/)
* [Postman](https://www.postman.com/) 
* [Azure Microsoft](https://azure.microsoft.com/) 

## Reviewed

Diego Alfonso Prieto Torres

## Authors

* **Alan Yesid Marin Mendez** - [PurpleBooth](https://github.com/Elan-MarMEn)
* **Maria Fernanda Hernandez Vargas** - [PurpleBooth](https://github.com/mariahv9)


Students of Systems Engineering of Escuela Colombiana de Ingenieria Julio Garavito 
