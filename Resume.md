## Arquitectura del proyecto

![](Attachments/Pasted%20image%2020250528090023.png)
## IAM (Identity and Access Management)

Maneja el acceso a los recursos en AWS

![](Attachments/Pasted%20image%2020250528031503.png)

### Creamos un usuario

![](Attachments/Pasted%20image%2020250528031645.png)

![](Attachments/Pasted%20image%2020250528033153.png)

Luego de crearlo, debemos de asignarle los permisos necesarios para el proyecto

Para S3:

![](Attachments/Pasted%20image%2020250528033446.png)

Para Glue:

![](Attachments/Pasted%20image%2020250528035041.png)

Para Athena:

![](Attachments/Pasted%20image%2020250528035120.png)

Para QuickSight:

![](Attachments/Pasted%20image%2020250528035256.png)

Resumen de la creación:

![](Attachments/Pasted%20image%2020250528035340.png)

Una vez confirmada la creación nos aparece esta ventana:

![](Attachments/Pasted%20image%2020250528035510.png)

Tenemos la siguiente información:
- Console sign-in URL: https://063219793319.signin.aws.amazon.com/console
- User name: user280525-1
- Console password: ughQ+28|

Cuando intento entrar me pide un cambio de contraseña:

![](Attachments/Pasted%20image%2020250528040250.png)

Esta es la nueva: oI!xrag5o8Qr66

Notamos las restricciones de nuestro usuario:

![](Attachments/Pasted%20image%2020250528040448.png)

## Creación de S3 bucket

![](Attachments/Pasted%20image%2020250528041158.png)


El nombre del bucket tiene que ser único, si existe otro llamado igual nos dará error

![](Attachments/Pasted%20image%2020250528041520.png)

Con lo demás por default le damos a crear

![](Attachments/Pasted%20image%2020250528041645.png)

Dentro del bucket creamos los folders de staging y DW

![](Attachments/Pasted%20image%2020250528041804.png)

![](Attachments/Pasted%20image%2020250528041928.png)

Normalmente los datos que entran en staging provienen de una base de datos, pero se subirá un archivo manualmente

![](Attachments/Pasted%20image%2020250528042544.png)

![](Attachments/Pasted%20image%2020250528042700.png)

En la siguiente parte se usará Glue para transformar y transferir los datos de `staging` a `datawarehouse`

## Glue

`Glue` ofrece `Visual ETL` lo que nos permite crear código `pyspark` de manera visual

Aquí estamos escribiendo las 3 fuentes que son de tipo Amazon S3, refiriendonos a las 3 tablas

![](Attachments/Pasted%20image%2020250528044845.png)

Personalizamos y buscamos la fuente con browse

![](Attachments/Pasted%20image%2020250528045048.png)


![](Attachments/Pasted%20image%2020250528045109.png)

![](Attachments/Pasted%20image%2020250528045154.png)

Seleccionamos el formato en el que está nuestra fuente, que es `CSV`

![](Attachments/Pasted%20image%2020250528045357.png)

Hacemos lo mismo para la otras fuentes

![](Attachments/Pasted%20image%2020250528045650.png)

Ahora realizamos un JOIN de `albums` y `artists`

![](Attachments/Pasted%20image%2020250528045848.png)

Especificamos la condición del JOIN

![](Attachments/Pasted%20image%2020250528050141.png)

Se puede personalizar el nombre de la transformación

![](Attachments/Pasted%20image%2020250528050242.png)


Creamos otro JOIN para `track`

![](Attachments/Pasted%20image%2020250528050406.png)

![](Attachments/Pasted%20image%2020250528050540.png)


Finalmente hacemos DROP de las columnas que no necesitamos

Borramos `.track_id` porque es duplicado de `track_id`, y borramos `id` porque es duplicado de `artist_id` 

![](Attachments/Pasted%20image%2020250528050853.png)

Finalmente establecemos el target

![](Attachments/Pasted%20image%2020250528051215.png)

Está listo el pipeline:

![](Attachments/Pasted%20image%2020250528051347.png)

Si vamos a `Script` podemos ver el código `pyspark` generado

![](Attachments/Pasted%20image%2020250528051435.png)

No puedo guardar el job por un problema de permisos

![](Attachments/Pasted%20image%2020250528053617.png)

Así que cambio a la cuenta root y me asigno permisos para el servicio Glue

![](Attachments/Pasted%20image%2020250528054456.png)

![](Attachments/Pasted%20image%2020250528054602.png)

![](Attachments/Pasted%20image%2020250528054640.png)

Con esto ya tenemos un nuevo rol

![](Attachments/Pasted%20image%2020250528054749.png)

Ahora apareció un error (https://repost.aws/knowledge-center/iam-passrole-error) sobre que nuestro usuario no podía asignarse un rol, lo corregimos así:

![](Attachments/Pasted%20image%2020250528062402.png)

Este sería el contenido de esa nueva política:

![](Attachments/Pasted%20image%2020250528062439.png)

Con esa última correción se pudo guardar el job:

![](Attachments/Pasted%20image%2020250528062627.png)

Corremos el job

![](Attachments/Pasted%20image%2020250528064207.png)

![](Attachments/Pasted%20image%2020250528064243.png)

Demoró un par de minutos y luego cambió al status de `succeeded`

![](Attachments/Pasted%20image%2020250528064456.png)

Finalmente, se crearon los archivos de tipo parquet en el datawarehouse

![](Attachments/Pasted%20image%2020250528064934.png)

## Glue Crawler

Un crawler crea la metadata que permite a Glue y a Athena ver la información que está en el S3 Bucket como un esquema de base de datos. **Glue Catalog**

![](Attachments/Pasted%20image%2020250528070001.png)

Creamos un nuevo crawler

![](Attachments/Pasted%20image%2020250528070529.png)

Añadimos como source al datawarehouse

![](Attachments/Pasted%20image%2020250528070640.png)

Utilizamos el mismo rol ya que seguimos utilizando el servicio de Glue

![](Attachments/Pasted%20image%2020250528070731.png)

Actualmente no tenemos una base de datos, necesitamos crearla

![](Attachments/Pasted%20image%2020250528070823.png)

![](Attachments/Pasted%20image%2020250528070856.png)

![](Attachments/Pasted%20image%2020250528070951.png)

Esta sería la configuración final del crawler

![](Attachments/Pasted%20image%2020250528071111.png)

Al inicio no tenemos ninguna tabla porque la base de datos `spotify` está vacía

![](Attachments/Pasted%20image%2020250528071237.png)


Corremos el crawler

![](Attachments/Pasted%20image%2020250528071320.png)

Apareció otro error, ## Crawler Error:

Service Principal: glue.amazonaws.com is not authorized to perform: logs:PutLogEvents on resource: arn:aws:logs:us-east-2:063219793319:log-group:/aws-glue/crawlers:log-stream:spotify_crawler because no identity-based policy allows the logs:PutLogEvents action (Service: AWSLogs; Status Code: 400; Error Code: AccessDeniedException; Request ID: 2ec6c397-4059-4fc8-a4ad-2af0c3c711d7; Proxy: null). For more information, see Setting up IAM Permissions in the Developer Guide (http://docs.aws.amazon.com/glue/latest/dg/getting-started-access.html).

![](Attachments/Pasted%20image%2020250528071916.png)

Agregamos esta política al rol de `glue_access_s3`

![](Attachments/Pasted%20image%2020250528072718.png)

Vemos que el crawler se ejecuta con normalidad

![](Attachments/Pasted%20image%2020250528072557.png)


Ahora aparece `datawarehouse` dentro de `Tables`

![](Attachments/Pasted%20image%2020250528072842.png)

![](Attachments/Pasted%20image%2020250528073314.png)

## AWS Athena

![](Attachments/Pasted%20image%2020250528073834.png)

Dentro del Query Editor podemos realizar consultas a la tabla anterior

![](Attachments/Pasted%20image%2020250528074307.png)


Creamos un S3 bucket para las consultas que haremos

![](Attachments/Pasted%20image%2020250528074613.png)

Ahora hacemos la consulta de los 8 primeros registros

![](Attachments/Pasted%20image%2020250528074749.png)

## QuickSight

No tenemos permisos para QuickSight

![](Attachments/Pasted%20image%2020250528075536.png)


Debemos entrar como root

![](Attachments/Pasted%20image%2020250528081337.png)

Creamos un nuevo dataset utilizando Athena

![](Attachments/Pasted%20image%2020250528082036.png)
![](Attachments/Pasted%20image%2020250528082149.png)

Esto se puede en Virginia

![](Attachments/Pasted%20image%2020250528083508.png)

Entonces debemos cambiar a Ohio

![](Attachments/Pasted%20image%2020250528083935.png)

![](Attachments/Pasted%20image%2020250528085531.png)

