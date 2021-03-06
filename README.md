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
   ![Imágen 1](images/instalacion/version de nodejs.png)
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
      ![Imágen 1](images/instalacion/1000000.png)
    * 1010000
      ![Imágen 1](images/instalacion/1010000.png)
    * 1020000
      ![Imágen 1](images/instalacion/1020000.png)
    * 1030000
      ![Imágen 1](images/instalacion/1030000.png)
    * 1040000
      ![Imágen 1](images/instalacion/1040000.png)
    * 1050000
      ![Imágen 1](images/instalacion/1050000.png)
    * 1060000
      ![Imágen 1](images/instalacion/1060000.png)
    * 1070000
      ![Imágen 1](images/instalacion/1070000.png)
    * 1080000
      ![Imágen 1](images/instalacion/1080000.png)
    * 1090000    
      ![Imágen 1](images/instalacion/1090000.png)

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

![Imágen 2](images/part1/part1-vm-cpu.png)

![Imágen 1](images/instalacion/estadisticas.png)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```
   ![Imágen 1](images/instalacion/newman.png)

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

![Imágen 3](images/part1/part1-vm-resize.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.
12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.
13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?
   Hay 5 recursos que se crean con la maquina virtual
   - Interfaz de red
   - Disco Duro
   - Clave SSH
   - Dirección IP Publica
   - Grupo de seguridad de red
2. ¿Brevemente describa para qué sirve cada recurso?
   - Ip publica: Es aquella que nos ofrece el proveedor de acceso a internet y se asigna a cualquier equipo o dispositivo conectado de forma directa a internet.
   - Una interfaz de red: Es el software específico de red que se comunica con el controlador de dispositivo específico de red y la capa IP a fin de proporcionar a la capa IP una interfaz coherente con todos los adaptadores de red que puedan estar presentes.
   - Disco Duro: Son dispositivos de almacenamiento de datos en los que podemos almacenar cualquier tipo de información digital. Ya sean fotografías, vídeos, archivos de texto o programas informáticos, el disco duro es una de las partes más importantes de cualquier sistema informático.
   - Clave SSH: Es uno de los dos archivos utilizados en un método de autenticación conocido como autenticación de clave pública SSH. En este método de autenticación, un archivo (conocido como la clave privada) generalmente se mantiene en el lado del cliente y el otro archivo (conocido como la clave pública) se almacena en el lado del servidor.
   - Grupo de seguridad: El grupo de seguridad de red de Azure para filtrar el tráfico de red hacia y desde los recursos de Azure de una red virtual de Azure. Un grupo de seguridad de red contiene reglas de seguridad que permiten o deniegan el tráfico de red entrante o el tráfico de red saliente de varios tipos de recursos de Azure.
3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?
   Al momento de realizar la conexión con la aplicación se crea un proceso que inicia la aplicación, por lo tanto cuando detenemos la maquina el proceso finaliza  de ejecutar la aplicación. La inbound port rule se realiza por el puerto 3000 para desplegar la aplicación de NODEJS y asi poderle hacer peticiones a la aplicación.
4. Adjunte tabla de tiempos e interprete por qué la función tarda tando tiempo.
   
   ![Imágen 1](images/instalacion/tabla.png)
   - El cambio que se hace al hardware no consideramos que es directamente propocional al rendimiento que evidenciamos en los datos.
5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.
   
6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.
    * Si hubo fallos documentelos y explique.
7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?
   ![Imágen 1](images/instalacion/TablaVs.png)
   
   -El tamaño B1ls tiene mucha menos capacidad y es mucho mas barato que el B2ms por lo que podemos concluir que estos tamaños son usados para un ambiente de pruebas a bajo costo.
8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?
   
   -Es que cuando se le hacen dichos cambios de hardware o se detiene la maquina toca volver a hacer el despliege para que el aplicativo siga estando disponible para recibir peticiones.
9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?
10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?
11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

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
  Balanceadores de carga Interno: Este balanceador de carga funciona dentro de una red privada la cual solo utilizan las ip de la misma. Su funcionalidad es de equilibrar la carga del tráfico.

- Balanceador de carga público: A diferencia del anterior las ip son publicas y el puerto de entrado son asignados a la dirección privada.

- SKU Son las unidades de mantenimiento de existencias, son un código único asignado a un servicio de azure. Tenemos varios tipos como:

- Estándar: Son productos estándar y se pueden vender individualmente o en paquetes conjuntos.
Componente: Son productos incluidos en los paquetes, ensamblajes y colecciones, no pueden venderse individualmente
- Paquete: No es necesario ensamblar antes del envío, debe haber disponibilidad completa y diferentes fuentes de cumplimiento.
- Virtual: Son productos que no necesitan instalación física, y no requieren un nivel de inventario.
- Ensamblaje: Se refieren a productos que se deben ensamblar antes del envío, todos los SKU deben estar dentro de la misma instalación esta debe ser local o de un proveedor Dropship/JIT/3PL.
- Paquete: No es necesario ensamblar antes del envío, debe haber disponibilidad completa y diferentes fuentes de cumplimiento

* ¿Cuál es el propósito del *Backend Pool*?
  - Es un componente del balanceador de carga el cual es el encargado de agrupar todas nuestras instancias de la aplicación y nos sirve para saber el estado de nuestras de las mismas
* ¿Cuál es el propósito del *Health Probe*?
  - Es un componente del balanceador de carga el cual es el encargado de saber si las instancias dentro del Backend Pool están en correcto estado, si alguna de ellas llega a fallar un numero determinado de veces entonces el Health Probe dejara de redirigir el trafico hacia ella hasta que vuelva el funcionamiento correcto.
* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?.
  - Un Load Balancing Rule se utiliza par definir el trafico a nuestras instancias los tipos de sesión presistente son:
  - None (hash-based): Esta regla tarta que una vez se establece la conexión con el cliente la instancia no cambiara hasta que se desconecte y se vuelva a conectar.
  - Client IP (source IP affinity 2-tuple o 3-tuple): En esta regla hace que las peticiones sucesivas de una misma ip serán gestionadas por la misma instancia.
* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?
  - Subnet: Es una segmentación de la red virtual (o cualquier red en general), permiten asignar una o varias subredes a la misma, estas subredes cuentan con un rango de direcciones apropiadas para una organización adecuada.
  - Virtual Network: Es una tecnología de red que permite extender la red de área local sobre una red pública o no controlada (Internet). permite enviar y recibir datos sobre redes compartidas o públicas comportandose como una red privada aprovechando la funcionalidad, seguridad y políticas de gestión de una red privada. se implementan realizando conexiones dedicadas y/o cifrado.
  - Address space: Cuando se crea una red virtual, se debe especificar un rango de direcciones ip privadas personalizadas (RFC 1918). Azure asigna a los recursos de una red virtual una dirección IP privada desde el espacio de direcciones que asigne. Por ejemplo, si implementa una máquina virtual en una red virtual con espacio de direcciones, 10.0.0.0/16, a la máquina virtual se le asignará una dirección IP privada como 10.0.0.4.
  - Address range: Indica cuantas direcciones se tienen en un address space y dependiendo de la cantidad de recursos que se necesiten en la red virtual, el rango aumentará o disminuirá. 
* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
  - Availability Zone: Son Ubicaciones geográficas únicas dentro de una región. Cada zona se compone de uno o más centros de datos equipados con alimentación, refrigeración y redes independientes. Seleccionamos 3 zonas de disponibilidad diferentes para poder tener una mejor disponibiliad y tolerancia a fallos dentro del sistema. En caso de que falle alguno de los centros de datos anteriormente mencionados, el loadbalancer utilizará otro nodo de la red que se encontrará ubicado en otra ubicación geográfica, de esta manera se garantiza resiliencia y se disminuye la probabilidad de que el sistema se encuentre no disponible.
  - IP zone-redundant: aporta resistencia, escalabilidad y disponibilidad a nuestro sistema, cuando utilizamos una ip zone-redundant azure separa física y lógicamente el gateway dentro de una region, lo cual permite mejorar la conectividad de la red privada y disminuye fallos a nivel de zona de disponibilidad.
   
* ¿Cuál es el propósito del *Network Security Group*?
  - Permite filtrar el tráfico hacia y desde los recursos en una red virtual de Azure, un grupo de seguridad permite definir reglas de entrada y/o salida que permitan o denieguen el tráfico de red entrante o saliente de varios tipos de recursos de Azure.
   
* Informe de newman 1 (Punto 2)

![Imágen 1](images/instalacion/newmanpunto2.PNG)


* Presente el Diagrama de Despliegue de la solución.


![Imágen 1](images/instalacion/esquema.png)





