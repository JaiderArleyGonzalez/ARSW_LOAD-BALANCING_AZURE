### Escuela Colombiana de Ingeniería
### Arquitecturas de Software - ARSW
### Realizado por Miguel Barrera y Jaider Gonzalez
#### Evidencias y Documentación en este README

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

2. Para conectarse a la VM use el siguiente comando, donde las `x` las debe remplazar por la IP de su propia VM (Revise la sección "Connect" de la virtual machine creada para tener una guía más detallada).

    `ssh scalability_lab@xxx.xxx.xxx.xxx`

3. Instale node, para ello siga la sección *Installing Node.js and npm using NVM* que encontrará en este [enlace](https://linuxize.com/post/how-to-install-node-js-on-ubuntu-18.04/).
4. Para instalar la aplicación adjunta al Laboratorio, suba la carpeta `FibonacciApp` a un repositorio al cual tenga acceso y ejecute estos comandos dentro de la VM:

    `git clone <your_repo>`

    `cd <your_repo>/FibonacciApp`

    `npm install`

5. Para ejecutar la aplicación puede usar el comando `npm FibinacciApp.js`, sin embargo una vez pierda la conexión ssh la aplicación dejará de funcionar. Para evitar ese compartamiento usaremos *forever*. Ejecute los siguientes comando dentro de la VM.

    ` node FibonacciApp.js`

    **Ejecución de la aplicación:**

    ![](images/solucion/ejecucion.png)

6. Antes de verificar si el endpoint funciona, en Azure vaya a la sección de *Networking* y cree una *Inbound port rule* tal como se muestra en la imágen. Para verificar que la aplicación funciona, use un browser y user el endpoint `http://xxx.xxx.xxx.xxx:3000/fibonacci/6`. La respuesta debe ser `The answer is 8`.

    ![](images/part1/part1-vm-3000InboudRule.png)

    **Configuración del puerto:**

    ![](images/solucion/puerto.png)

    **Prueba de ejecución:**

    ![](images/solucion/8.png)

7. La función que calcula en enésimo número de la secuencia de Fibonacci está muy mal construido y consume bastante CPU para obtener la respuesta. Usando la consola del Browser documente los tiempos de respuesta para dicho endpoint usando los siguintes valores:
    * 1000000

        ![](images/solucion/imagen.png)

    * 1010000

        ![](images/solucion/imagen1.png)

    * 1020000
        
        ![](images/solucion/imagen2.png)

    * 1030000

        ![](images/solucion/imagen3.png)

    * 1040000

        ![](images/solucion/imagen4.png)

    * 1050000

        ![](images/solucion/imagen5.png)

    * 1060000

        ![](images/solucion/imagen6.png)

    * 1070000

        ![](images/solucion/imagen7.png)

    * 1080000

        ![](images/solucion/imagen8.png)

    * 1090000

        ![](images/solucion/imagen9.png)    

8. Dírijase ahora a Azure y verifique el consumo de CPU para la VM. (Los resultados pueden tardar 5 minutos en aparecer).

    ![Imágen 2](images/part1/part1-vm-cpu.png)
    **Consumo de la CPU:**
    ![](images/solucion/cpu.png)

    ![](images/solucion/cpuh.png)

9. Ahora usaremos Postman para simular una carga concurrente a nuestro sistema. Siga estos pasos.
    * Instale newman con el comando `npm install newman -g`. Para conocer más de Newman consulte el siguiente [enlace](https://learning.getpostman.com/docs/postman/collection-runs/command-line-integration-with-newman/).
    * Diríjase hasta la ruta `FibonacciApp/postman` en una maquina diferente a la VM.
    * Para el archivo `[ARSW_LOAD-BALANCING_AZURE].postman_environment.json` cambie el valor del parámetro `VM1` para que coincida con la IP de su VM.
    * Ejecute el siguiente comando.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```
    **Instalación:**
    
    ![](images/solucion/insNewman.png)

    ![](images/solucion/dir.png)

    **Se asigna la IP correspondiente a la VM:**
    ![](images/solucion/vi1.png)

    ![](images/solucion/vi2.png)

    **Prueba de Ejecución:**
    ![](images/solucion/comando.png)

    **Consumo de CPU:**

    ![](images/solucion/comandocpu.png)

10. La cantidad de CPU consumida es bastante grande y un conjunto considerable de peticiones concurrentes pueden hacer fallar nuestro servicio. Para solucionarlo usaremos una estrategia de Escalamiento Vertical. En Azure diríjase a la sección *size* y a continuación seleccione el tamaño `B2ms`.

    ![Imágen 3](images/part1/part1-vm-resize.png)
    
    **Cambio del tamaño de la VM:**
    
    ![](images/solucion/B2ms.png)

11. Una vez el cambio se vea reflejado, repita el paso 7, 8 y 9.
    
    **Repetición paso 7:**
    
    ![](images/solucion/1000000.png)

    ![](images/solucion/1010000.png)

    ![](images/solucion/1020000.png)

    ![](images/solucion/1030000.png)

    ![](images/solucion/1040000.png)

    ![](images/solucion/1050000.png)

    ![](images/solucion/1060000.png)

    ![](images/solucion/1070000.png)

    ![](images/solucion/1080000.png)

    ![](images/solucion/1090000.png)

    **Repetición paso 8:**

    ![](images/solucion/ccpu.png)

    **Repetición paso 9:**

    ![](images/solucion/nuevoComan.png)

12. Evalue el escenario de calidad asociado al requerimiento no funcional de escalabilidad y concluya si usando este modelo de escalabilidad logramos cumplirlo.

![](images/solucion/reduc.png)

Dado que hubo una reducción en el tiempo de ejecución y una disminución en el uso de la CPU, podemos concluir que el modelo de escalabilidad ha tenido un impacto positivo en el cumplimiento del requerimiento no funcional de escalabilidad. Sin embargo, la presencia de solicitudes fallidas es un problema importante. Podría indicar que, aunque el sistema maneja mejor las cargas normales, puede estar alcanzando sus límites en situaciones de carga más alta, es algo que se ve reflejado en los picos de las gráficas de consumo de CPU. 

13. Vuelva a dejar la VM en el tamaño inicial para evitar cobros adicionales.

    ![](images/solucion/B2ms.png)

**Preguntas**

1. ¿Cuántos y cuáles recursos crea Azure junto con la VM?

    La creación de una máquina virtual (VM) en Microsoft Azure no solo implica la creación de la VM en sí, sino también de varios recursos asociados que son necesarios para el funcionamiento y la administración de la VM. Se crean 6 recursos:

    * Clave SSH

    * Dirección IP pública
    
    * Disco
    
    * Grupo de seguridad de red
    
    * Interfaz de red
    
    * Máquina virtual

2. ¿Brevemente describa para qué sirve cada recurso?
    
    * **Clave SSH:** La clave SSH se utiliza para autenticar y permitir el acceso seguro a la máquina virtual a través del protocolo SSH. Se utiliza en lugar de la autenticación basada en contraseña para una mayor seguridad.

    * **Dirección IP pública:** La dirección IP pública permite que la máquina virtual sea accesible desde Internet. Es la dirección a la que se pueden conectar servicios, aplicaciones o usuarios externos.
    
    * **Disco:** Un disco en Azure se utiliza para almacenar el sistema operativo, las aplicaciones y los datos de la máquina virtual. Puede haber discos para el sistema operativo (OS) y discos de datos adicionales.

    * **Grupo de seguridad de red:** Un grupo de seguridad de red (NSG) actúa como un cortafuegos virtual para controlar el tráfico de red hacia y desde una máquina virtual. Permite configurar reglas para permitir o denegar el tráfico según las necesidades de seguridad.

    * **Interfaz de red:** La interfaz de red está asociada a la máquina virtual y al grupo de seguridad de red. Facilita la comunicación de la máquina virtual con la red y ayuda a administrar las reglas de seguridad que se aplican a la máquina.

    * **Máquina virtual:** La máquina virtual en sí misma es el recurso principal. Representa la instancia de una computadora virtual que ejecuta un sistema operativo y puede ser configurada con recursos como CPU, memoria, almacenamiento, etc. para satisfacer los requisitos específicos de la aplicación.

3. ¿Al cerrar la conexión ssh con la VM, por qué se cae la aplicación que ejecutamos con el comando `npm FibonacciApp.js`? ¿Por qué debemos crear un *Inbound port rule* antes de acceder al servicio?

    La aplicación se cae al cerrar la conexión SSH porque el proceso en la VM está asociado a la sesión SSH activa. Cuando la conexión SSH se cierra, el proceso asociado también termina. La creación de una Inbound port rule es necesaria para permitir el tráfico al puerto específico (en este caso, el puerto 3000) en la VM. Sin esta regla, el tráfico no puede llegar al servicio que se ejecuta en la VM.

4. Adjunte tabla de tiempos e interprete por qué la función tarda tanto tiempo.

    **Tiempo base:**

    | Número de Fibonacci | Tiempo de Respuesta (segundos) | 
    | :------------: | :------------: |
    | 1000000    | 26.72   |
    | 1010000   | 26.79    | 
    | 1020000   | 27.74    |
    | 1030000    | 28.81    |
    | 1040000   | 28.66    | 
    | 1050000   | 29.02    |
    | 1060000    | 29.91    |
    | 1070000   | 30.92   | 
    | 1080000   | 31.97    |
    | 1090000    | 33   |
    
    **Tiempo tras estrategua de escalamiento vertical:**

    | Número de Fibonacci | Tiempo de Respuesta (segundos) | 
    | :------------: | :------------: |
    | 1000000    | 25.44   |
    | 1010000   | 26.36    | 
    | 1020000   | 27.32    |
    | 1030000    | 27.87    |
    | 1040000   | 28.24    | 
    | 1050000   | 28.46    |
    | 1060000    | 28.94    |
    | 1070000   | 29.63   | 
    | 1080000   | 30.81    |
    | 1090000    | 31.09   |

    Antes del escalamiento vertical, la función de Fibonacci tenía tiempos de respuesta más altos, indicando una carga de trabajo más pesada para la VM original (B1ls). Después de aplicar la estrategia de escalamiento vertical, al cambiar a una VM más grande (B2ms), se observa una mejora en los tiempos de respuesta. Esto sugiere que la VM con mayores recursos puede manejar la carga de trabajo de manera más eficiente, reduciendo el tiempo necesario para calcular los números de Fibonacci. Sin embargo, a pesar de la mejora, los tiempos de respuesta aún pueden considerarse altos debido a la naturaleza ineficiente de la implementación de la función de Fibonacci.

5. Adjunte imágen del consumo de CPU de la VM e interprete por qué la función consume esa cantidad de CPU.
    
    **Consumo de CPU base:**

    ![](images/solucion/comandocpu.png)

    **Consumo de CPU tras estrategia de escalamiento vertical:**

    ![](images/solucion/ccpu.png)

    La imagen muestra el consumo de CPU de la VM. Se observa que, después de cambiar el tamaño de la VM a B2ms, hay una mejora en el consumo de CPU en comparación con el tamaño original B1ls. La función de FibonacciApp consume menos recursos de CPU, lo que indica una mejor capacidad de la VM para manejar la carga de trabajo.

    La mejora en el consumo de CPU se debe a que B2ms proporciona más recursos de CPU y memoria en comparación con B1ls. Esto permite que la aplicación maneje las solicitudes concurrentes de manera más eficiente y reduce la presión sobre los recursos de la VM. En consecuencia, se logra una distribución más equitativa de la carga y se mejora el rendimiento general de la aplicación.

6. Adjunte la imagen del resumen de la ejecución de Postman. Interprete:
    * Tiempos de ejecución de cada petición.

        Los tiempos de ejecución varían, pero en general son altos debido a la ineficiencia de la función de Fibonacci. La aplicación no escala bien bajo carga concurrente, como se evidencia en los tiempos de respuesta.

    * Si hubo fallos documentelos y explique.
        
        Sí, hubo solicitudes fallidas. Esto puede deberse a que la carga concurrente excede la capacidad de la VM, causando tiempos de espera y posiblemente errores en algunas solicitudes.

7. ¿Cuál es la diferencia entre los tamaños `B2ms` y `B1ls` (no solo busque especificaciones de infraestructura)?

    La diferencia no solo radica en las especificaciones de infraestructura, sino también en el rendimiento y la capacidad de procesamiento. El tamaño B2ms es superior en términos de recursos de CPU y memoria en comparación con B1ls. A menudo, cambiar a un tamaño de VM más grande implica un mayor rendimiento, pero también puede tener un costo adicional.

8. ¿Aumentar el tamaño de la VM es una buena solución en este escenario?, ¿Qué pasa con la FibonacciApp cuando cambiamos el tamaño de la VM?

    Aumentar el tamaño de la VM puede ser una solución para mejorar el rendimiento en términos de capacidad de procesamiento y memoria. Sin embargo, en este escenario específico, la función de FibonacciApp aún es ineficiente y puede no escalar de manera óptima, incluso con un tamaño de VM más grande. Cambiar el tamaño de la VM puede mejorar el rendimiento, pero no aborda la raíz del problema, que es la ineficiencia en la implementación de la función.

9. ¿Qué pasa con la infraestructura cuando cambia el tamaño de la VM? ¿Qué efectos negativos implica?

    Cambiar el tamaño de la VM implica detener y reiniciar la VM con la nueva configuración. Esto puede resultar en un tiempo de inactividad temporal para la aplicación. Además, un tamaño de VM más grande puede implicar costos adicionales, ya que se le facturará según la capacidad de la VM. El aumento en los recursos de la VM puede mejorar el rendimiento, pero es esencial evaluar y equilibrar los costos y beneficios.

10. ¿Hubo mejora en el consumo de CPU o en los tiempos de respuesta? Si/No ¿Por qué?

    Sí, hubo una mejora en el consumo de CPU y los tiempos de respuesta después de cambiar el tamaño de la VM a B2ms. Esto se debe a que B2ms proporciona más recursos de CPU y memoria, lo que permite manejar mejor la carga de trabajo. Sin embargo, la mejora podría no ser suficiente para manejar cargas más grandes o mejorar significativamente la eficiencia de la función de Fibonacci.

11. Aumente la cantidad de ejecuciones paralelas del comando de postman a `4`. ¿El comportamiento del sistema es porcentualmente mejor?

    el aumento en la cantidad de ejecuciones paralelas pone más presión en el sistema y revela aún más las limitaciones de la implementación actual de la función de Fibonacci y la capacidad de la VM. Se experimenta un mayor consumo de CPU y más solicitudes fallidas bajo estas cargas más altas.

### Parte 2 - Escalabilidad horizontal

#### Crear el Balanceador de Carga

Antes de continuar puede eliminar el grupo de recursos anterior para evitar gastos adicionales y realizar la actividad en un grupo de recursos totalmente limpio.

1. El Balanceador de Carga es un recurso fundamental para habilitar la escalabilidad horizontal de nuestro sistema, por eso en este paso cree un balanceador de carga dentro de Azure tal cual como se muestra en la imágen adjunta.

    ![](images/part2/part2-lb-create.png)

    **Balanceador de carga creado:**

    ![](images/solucion/loadbalancer.png)

2. A continuación cree un *Backend Pool*, guiese con la siguiente imágen.

    ![](images/part2/part2-lb-bp-create.png)

    **Backend Pool:**

    ![](images/solucion/pool.png)

3. A continuación cree un *Health Probe*, guiese con la siguiente imágen.

    ![](images/part2/part2-lb-hp-create.png)

    **Health Probe:**

    ![](images/solucion/health.png)

4. A continuación cree un *Load Balancing Rule*, guiese con la siguiente imágen.

    ![](images/part2/part2-lb-lbr-create.png)

    **Load Balancing Rule:**

    ![](images/solucion/RULE.png)

5. Cree una *Virtual Network* dentro del grupo de recursos, guiese con la siguiente imágen.

    ![](images/part2/part2-vn-create.png)

    **Virtual Network creado:**

    ![](images/solucion/network.png)

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

    **Máquina virtual VM1:**

    ![](images/solucion/VM1.png)

    **Máquina virtual VM2:**

    ![](images/solucion/VM2.png)

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
    
    Se sigue el mismo proceso de instalación con la diferencia que se usa la carpeta "part2" dentro de postman:

    ![](images/solucion/install.png)

    ![](images/solucion/direc.png)

#### Probar el resultado final de nuestra infraestructura

1. Por supuesto el endpoint de acceso a nuestro sistema será la IP pública del balanceador de carga, primero verifiquemos que los servicios básicos están funcionando, consuma los siguientes recursos:

    ```
    http://52.155.223.248/
    http://52.155.223.248/fibonacci/1
    ```

    **Pruebas de ejecución:**

    ![](images/solucion/pruebaBalanceador.png)

    ![](images/solucion/balanceadorfibo.png)

2. Realice las pruebas de carga con `newman` que se realizaron en la parte 1 y haga un informe comparativo donde contraste: tiempos de respuesta, cantidad de peticiones respondidas con éxito, costos de las 2 infraestrucruras, es decir, la que desarrollamos con balanceo de carga horizontal y la que se hizo con una maquina virtual escalada.

    ![](images/solucion/config.png)

    ![](images/solucion/config2.png)

    **Máquina virtual VM1:** 

    ![](images/solucion/newManV1.png)

    **Máquina virtual VM2:**

    ![](images/solucion/newManV2.png)

    |Infraestructura/Criterio | Promedio de tiempo de respuesta | Peticiones Respondidas con Éxito | Costo por hora |
    | :------------: | :------------: | :------------: | :------------: |
    | **Con Enfoque Vertical**    | 4 min 44.4 seg  | 6 de 10   | 1,4 dólares  |
    | **Con Enfoque Horizontal**   | 3 min 53 seg   | 19 de 20    | 0,19 dólares    | 
    
    * Tiempos de Respuesta:

        La infraestructura con escalabilidad horizontal muestra un mejor rendimiento en términos de tiempos de respuesta promedio. Esto se debe a la distribución de la carga entre múltiples nodos, lo que permite manejar de manera más eficiente las solicitudes concurrentes.
        
        La infraestructura con escalabilidad vertical también muestra mejoras en comparación con la configuración original de una sola VM. Sin embargo, los tiempos de respuesta aún son más altos en comparación con la escalabilidad horizontal.
    
    * Cantidad de Peticiones Respondidas con Éxito:

        La escalabilidad horizontal logra responder exitosamente a un mayor porcentaje de solicitudes bajo carga concurrente en comparación con la escalabilidad vertical. Esto destaca la capacidad de la infraestructura distribuida para manejar de manera efectiva la carga y mantener la disponibilidad del servicio.
    
    * Costos:

        La escalabilidad horizontal presenta un costo por hora significativamente menor en comparación con la escalabilidad vertical. Esto se debe a que se utilizan varias VM pequeñas distribuidas, lo que permite un uso más eficiente de los recursos y una mayor flexibilidad en términos de escalabilidad.
        
        La escalabilidad vertical, al utilizar una única VM más grande, tiene un costo por hora más alto. Aunque puede ofrecer mejoras en el rendimiento, también puede resultar más costosa a medida que se aumentan los recursos.

    la escalabilidad horizontal con balanceo de carga ofrece un mejor equilibrio entre rendimiento, disponibilidad y costos, especialmente para aplicaciones que experimentan cargas variables y requieren una mayor capacidad de respuesta frente a solicitudes concurrentes.

3. Agregue una 4 maquina virtual y realice las pruebas de newman, pero esta vez no lance 2 peticiones en paralelo, sino que incrementelo a 4. Haga un informe donde presente el comportamiento de la CPU de las 4 VM y explique porque la tasa de éxito de las peticiones aumento con este estilo de escalabilidad.

    ```
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10 &
    newman run ARSW_LOAD-BALANCING_AZURE.postman_collection.json -e [ARSW_LOAD-BALANCING_AZURE].postman_environment.json -n 10
    ```

    ![](images/solucion/4CososVM1.png)

    ![](images/solucion/4CososVM2.png)

    ![](images/solucion/CPUVM1.png)

    ![](images/solucion/CPUVM2.png)

    En un entorno de balanceo de carga horizontal con múltiples VMs distribuidas, se observa un comportamiento más equilibrado en la carga de CPU entre las VMs.
    
    Al realizar más peticiones en paralelo, cada VM puede contribuir a manejar la carga total, evitando posibles cuellos de botella.
    Al distribuir las peticiones entre varias VMs, el balanceo de carga maneja una mayor cantidad de solicitudes simultáneas.

    La mejora en la tasa de éxito se atribuye a la capacidad del sistema para gestionar más solicitudes concurrentes al tener múltiples nodos trabajando en paralelo.
    Al agregar más VMs y distribuir la carga, se maneja un mayor número de solicitudes simultáneas.
    La escalabilidad horizontal permite crecer de manera más efectiva al agregar nodos adicionales según sea necesario.

    La tasa de éxito del 100 % en las peticiones, que se atribuye al incremento de la concurrencia a 4 peticiones en paralelo, indica una mejora significativa en la capacidad del sistema para manejar cargas de trabajo más intensivas. 
    
    Al realizar más peticiones en paralelo, el sistema pudo manejarlas con éxito, lo que sugiere una mayor escalabilidad.
    La adición de recursos, ya sea mediante el escalamiento vertical de una VM o mediante el escalamiento horizontal con varias VMs y un balanceador de carga, ha mejorado la capacidad del sistema para gestionar solicitudes concurrentes.

    La capacidad de manejar todas las solicitudes con éxito, incluso bajo una carga más alta, sugiere que el sistema ahora es más robusto y resistente.
    La resiliencia del sistema a mayores niveles de tráfico es esencial para garantizar un rendimiento constante y una experiencia del usuario sin interrupciones.

**Preguntas**

* ¿Cuáles son los tipos de balanceadores de carga en Azure y en qué se diferencian?
    
    Azure ofrece dos tipos principales de balanceadores de carga:

    * Balanceador de Carga Público: Distribuye el tráfico entrante desde Internet entre máquinas virtuales (VM) en un conjunto de disponibilidad o conjunto de máquinas virtuales. Se utiliza para aplicaciones de Internet y proporciona una IP pública para el acceso desde Internet.

    * Balanceador de Carga Interno: Distribuye el tráfico entre las VM en una red virtual o entre redes virtuales conectadas. No tiene una IP pública y se utiliza para cargas de trabajo internas.

    ¿Qué es SKU, qué tipos hay y en qué se diferencian?

    * SKU en Azure se refiere a la especificación del nivel de servicio o capacidad de un recurso. Puede haber diferentes SKU para el mismo tipo de recurso, diferenciándose en términos de rendimiento, características y precios.

        Tipos de SKU comunes incluyen:

        * Básico: Proporciona funcionalidades esenciales a un precio económico.

        * Estándar: Ofrece características avanzadas y mayor capacidad de rendimiento.

    ¿Por qué el balanceador de carga necesita una IP pública?

    * El balanceador de carga necesita una IP pública para ser accesible desde Internet. Proporciona un punto de entrada único para el tráfico entrante y distribuye ese tráfico entre las máquinas virtuales en el Backend Pool. La IP pública es la dirección a la que se dirigen las solicitudes desde Internet.

* ¿Cuál es el propósito del *Backend Pool*?

    El Backend Pool es un conjunto de recursos (máquinas virtuales) que recibirán el tráfico distribuido por el balanceador de carga. Especifica las máquinas virtuales que participarán en el equilibrio de carga.

* ¿Cuál es el propósito del *Health Probe*?

    La Health Probe (sonda de salud) se utiliza para monitorear la salud de las instancias del Backend Pool. Verifica si las instancias están respondiendo correctamente y, en caso contrario, las excluye temporalmente del tráfico para garantizar una distribución de carga efectiva solo entre las instancias saludables.

* ¿Cuál es el propósito de la *Load Balancing Rule*? ¿Qué tipos de sesión persistente existen, por qué esto es importante y cómo puede afectar la escalabilidad del sistema?
    
    * La Load Balancing Rule define cómo se distribuirá el tráfico entre las instancias del Backend Pool. Especifica el puerto en el que se recibirá el tráfico y el puerto al que se dirigirá en las instancias del Backend Pool.

    * Tipos de Sesión Persistente: Permite asignar una solicitud específica a la misma instancia en situaciones donde es necesario mantener la persistencia en las conexiones. Puede ser basada en IP o Cookie.

        * IP Affinity: Asigna una IP específica a la misma instancia.
        * Cookie Affinity: Asigna la misma instancia basada en cookies.

    * Importancia: La sesión persistente es crucial cuando las aplicaciones requieren mantener el estado de la sesión del usuario a través de múltiples solicitudes. Sin persistencia, las solicitudes podrían dirigirse a diferentes instancias, lo que podría afectar la experiencia del usuario.

    * Afectación en Escalabilidad: La sesión persistente puede afectar la escalabilidad al introducir la necesidad de mantener conexiones específicas con ciertas instancias. Esto puede limitar la capacidad de distribuir libremente el tráfico entre todas las instancias y, en consecuencia, afectar la escalabilidad horizontal.

* ¿Qué es una *Virtual Network*? ¿Qué es una *Subnet*? ¿Para qué sirven los *address space* y *address range*?

    * Virtual Network (Red Virtual): Es un recurso de red que permite a las máquinas virtuales comunicarse entre sí y con recursos externos. Proporciona aislamiento de red para los recursos de Azure.

    * Subnet (Subred): Es una división de una red virtual en segmentos más pequeños. Permite organizar y segmentar los recursos de red dentro de una red virtual.

    * Address Space y Address Range:

        * Address Space: Es el rango total de direcciones IP disponibles para la red virtual.

        * Address Range: Es el rango de direcciones IP específicas asignadas a una subred dentro de la red virtual.

* ¿Qué son las *Availability Zone* y por qué seleccionamos 3 diferentes zonas?. ¿Qué significa que una IP sea *zone-redundant*?
    
    * Availability Zone (Zona de Disponibilidad): Es una ubicación física única con energía, refrigeración y red independientes. Azure proporciona redundancia y disponibilidad mejoradas al distribuir recursos entre múltiples zonas de disponibilidad.

    * Zone-Redundant IP (IP Redundante por Zona): Significa que la dirección IP está asociada a recursos en múltiples zonas de disponibilidad, lo que mejora la redundancia y la disponibilidad.

    La elección de tres diferentes zonas de disponibilidad tiene el propósito de mejorar la redundancia y la disponibilidad del sistema. Al distribuir las máquinas virtuales en tres zonas distintas, se asegura que, en caso de una interrupción en una zona, las otras dos zonas puedan continuar operando. Este enfoque proporciona una mayor resiliencia contra posibles fallas en una ubicación física específica.

* ¿Cuál es el propósito del *Network Security Group*?
    
    * El NSG actúa como un cortafuegos virtual que controla el tráfico hacia los recursos de Azure. Permite o deniega el tráfico basado en reglas definidas, brindando un control detallado sobre la seguridad de red.

* Presente el Diagrama de Despliegue de la solución.

![](images/solucion/Diagrama.png)




