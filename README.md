# HA-Hands-On

## NOTA: Esto es un laboratorio, no usar para fines productivos.

**1.** Empezar a realizar pasos del hands-on https://catalog.us-east-1.prod.workshops.aws/workshops/5ceb632a-c07f-44a5-a3bd-b8f616a631c0/en-US/application/lab7
   hasta módulo 8.
   
   
**2.** Mientras configure el cache dentro del sitio de Wordpress (paso 8 del link anterior), recibirá un error 502 bad gateway, para solucionar:

   &nbsp; &nbsp; **2.a** Seleccione una de las instancias creadas por el autoscalling group creado previamente y seleccione "launch more like this". 
   
   ![image](https://user-images.githubusercontent.com/102708806/194206344-3284a001-bc9d-4b39-819c-0510885c5ca6.png)
   
   &nbsp; &nbsp; **2.b** Seleccione un nombre como "CacheCorrector" 
   
   &nbsp; &nbsp; **2.c** Dirijase hasta "advanced details" en la sección "user data" y copie hasta el final la siguiente línea de código:
   
   
   ```
   sed -i '600s/esc_url_raw/filter_var/' /var/www/wordpress/wordpress/wp-content/plugins/w3-total-cache/DbCache_WpdbInjection_QueryCaching.php
   ```
   
   
   &nbsp; &nbsp; **2.d** Pruebe accediendo al URL del balanceador de carga ELB y debería funcionar el panel de administrador.
   
   &nbsp; &nbsp; **2.e** Elimine instancia "CacheCorrector"
   

   
**3.** Continue con los pasos descritos en https://catalog.us-east-1.prod.workshops.aws/workshops/5ceb632a-c07f-44a5-a3bd-b8f616a631c0/en-US/application/lab7
   hasta módulo 8 desde este paso:
   
   <img width="1090" alt="image" src="https://user-images.githubusercontent.com/102708806/194206957-2738d4bf-861c-41eb-ab53-91664e8704b8.png">

**4.** Cuando finalice "módulo 9. Add a Content Delivery Network" dirijase al servicio AWS IAM y cree los permisos acá descritos.
 https://chaos-engineering.workshop.aws/en/030_basic_content/030_basic_experiment/10-permissions.html
 
**5.** dirijase al servicio AWS FIS y cree una plantilla.

<img width="946" alt="image" src="https://user-images.githubusercontent.com/102708806/194207332-88abd32f-9a40-4911-88e0-04a041577ca5.png">

**6.** Agregue una descripción como "StopInstancesInAz".
**7.** Agregue un target con los siguientes atributos y guarde:


- **Name:** EC2AZ
- **Resource type:** aws:ec2:instance
- **Resource tags:** AppName=ha-web-app
- **Resource filters:** Placement.AvailabilityZone=us-east-1a , State.Name= running


<img width="443" alt="image" src="https://user-images.githubusercontent.com/102708806/194207617-9a5b300b-5b99-46a5-b3ee-c5e02ed53fdc.png">

**8.** Agregue un nuevo target con los siguientes parámetros y guarde:


- **Name:** AuroraAZ
- **Resource type:** aws:rds-cluster
- **Resource tags:** AppName=ha-web-app


**9.** Agregue una acción con los siguientes parámetros y guarde:


- **Name:** Stopinstance
- **Action type:** aws:ec2:stop-instances
- **Target:** EC2 AZ


<img width="772" alt="image" src="https://user-images.githubusercontent.com/102708806/194207790-781c8cee-f5a6-4ec1-8452-0a6baf3a0861.png">

**10.** Agregue otra acción con los siguientes parámetros y guarde:


- **Name:** FailOverAurora
- **Action type:** aws:rds:failover-db-cluster
- **Target:** AuroraAZ


**11.** Revisar que la página esté funcionando, que hayan 2 instancias en us-east-1a y 2 en us-east-1b y revisar como se encuentra actualmente cluster de Aurora RDS.

<img width="967" alt="image" src="https://user-images.githubusercontent.com/102708806/194208002-fea09984-39b0-4551-96b7-216836c27a96.png">

En este ejemplo la instancia que funciona como escritura se encuentra en la az us-east-1

**12.** Ir a servicio FIS, buscar en experiment templates la plantilla creada y seleccionar.

**13.** Start experiment.

**14.** Refrescar constantemente página Wordpress. Mirar cuantas instancias hay disponibles, ver RDS que cambiaron los endpoints

<img width="951" alt="image" src="https://user-images.githubusercontent.com/102708806/194208195-60e35b32-05fc-4c9f-81f0-d73fde30b1c7.png">

La instancia de escritura ahora se encuentra en us-east-1b

**15.** Limpiar recursos como muestra en: https://catalog.us-east-1.prod.workshops.aws/workshops/5ceb632a-c07f-44a5-a3bd-b8f616a631c0/en-US/cleanup-resources

