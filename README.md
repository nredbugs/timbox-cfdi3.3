# CFDI 3.3

Documentación para la integración del anexo 20 versión 3.3 (CFDI).

Aviso:

El 1 de julio de 2017 entra en vigor la versión 3.3 de la factura, no obstante podrás continuar emitiendo facturas en la versión 3.2 hasta el 30 de noviembre; a partir del 1 de diciembre, la única versión válida para emitir las facturas será la versión 3.3

[Documentación del SAT para emitir CFDI con la nueva versión 3.3](http://www.sat.gob.mx/informacion_fiscal/factura_electronica/Paginas/Anexo_20_version3.3.aspx)

Ejemplo de comó utilizar el servicio con [SOAP UI](https://www.soapui.org/downloads/soapui.html). Utilizamos la versión open source.

Se deberá hacer uso de la URL que hace referencia al WSDL, en cada petición realizada:

- [Timbox Pruebas](https://staging.ws.timbox.com.mx/timbrado_cfdi33/wsdl)

## Pasos a considerar antes de Timbrar CFDI
Para poder timbrar el CFDI, hay que considerado los siguientes pasos:
   
**<b>Paso 1</b>**
   
   Construir el XML en base al Anexo 20 de acuerdo al estándar definido por el SAT:
  
- [Esquema XSD](http://www.sat.gob.mx/sitio_internet/cfd/3/cfdv33.xsd) 
   
- [Estandar PDF](http://www.sat.gob.mx/informacion_fiscal/factura_electronica/Documents/cfdv33.pdf)
   
**<b>Paso 2</b>** 
   
   Obtener la cadena Original basandose en el estandar XSLT (Secuencia de cadena Original)
   
   [Estándar de transformación](http://www.sat.gob.mx/sitio_internet/cfd/3/cadenaoriginal_3_3/cadenaoriginal_3_3.xslt)
 
 Ejemplo de una Cadena Original
 
  ```
  ||3.3|2017-06-30T11:58:45|01|30001000000300023708|1510.00|MXN|1|1751.60|I|PUE|06300|AAA010101AAA1|SENTIENT SA DE CV|601|IAD121214B34|IT & SW Development Solutions de Mexico S de RL de CV|P01|10122100|5|M74|Kilo|Prueba Catalogos Nuevos|250.00|1250.00|24111500|1|KGM|kg|traslucida 90x90 cm. cal. 200|22.00|22.00|13101712|10|KGM|KG|POLIETILENO DE BAJA DENSIDAD|23.80|238.00|241.60||
  ```
**<b>Paso 3</b>**
   
Generar sello digital para los CFDIs
Tal como lo específica el anexo 20 en el inciso I Sección B "Generación de sellos digitales para comprobantes fiscales digitales a través de Internet"

1. Generación de la digestión o hash.
```
openssl dgst -sha256 -sign 'CSD01_AAA010101AAA.key.pem' -out 'digest.txt' 'cadena_original.txt'
```

2. Creación del archivo PEM de la llave privada.
```
openssl enc -in 'digest.txt' -out 'sello.txt' -base64 -A -K 'CSD01_AAA010101AAA.key.pem' 
```

## Creación del proyecto en SOAP UI
Para iniciar con el ejemplo del timbrado es necesario crear el proyecto con el URL Servicio.

1. El primer paso es crear el proyecto.

    ![](http://i.imgur.com/0ar7zY0.png)
    
2. Lo siguiente es introducir los datos para generar el servicio, en Initial WSDL colocamos el URL que utilizaremos en este caso staging, debemos de asegurarnos de que este seleccionado los siguientes puntos:
    * **Create sample requests for all operations?**
    * **Stores all file paths in project relatively to project file (requires save)**
    
    Después presionaremos el botón de OK

     ![](http://i.imgur.com/fn6qM7N.png)
     
3. El siguiente paso es aceptar el directorio donde se guardará el proyecto.

     ![](http://i.imgur.com/UCq1NwS.png)
     
4. A continuación, nos mostrara el proyecto creado con las peticiones para cada uno de los métodos del servicio.

     ![](http://i.imgur.com/250CyFV.png)
     

## Timbrar CFDI

Para hacer una petición de timbrado de un CFDI, deberá enviar las credenciales asignadas, así como el XML que desea timbrar convertido a una cadena base64, para ello recomendamos utilizar la página [https://www.base64encode.org/](https://www.base64encode.org/) en ella se puede pegar el XML deseado y se obtiene la cadena en base64:

Para hacer la petición solo necesitamos hacer doble click sobre **Request 1** debajo de **timbrar_cfdi**:

     ![](http://i.imgur.com/wxkGZ25.png)
     
Después de dar click nos aparecerá la siguiente ventana, debemos modificar la petición usando nuestros datos, como el siguiente código:
![](http://i.imgur.com/YeoGMB6.png)
```    
 <soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:WashOut">
   <soapenv:Header/>
   <soapenv:Body>
      <urn:timbrar_cfdi soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
         <username xsi:type="xsd:string">AAA010101000</username>
         <password xsi:type="xsd:string">h6584D56fVdBbSmmnB</password>
         <sxml xsi:type="xsd:string">PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz48Y2ZkaTpDb21wcm9iYW50ZSB4bWxuczp4c2k9Imh0dHA6Ly93d3cudzMub3JnLzIwMDEvWE1MU2NoZW1hLWluc3RhbmNlIiB4bWxuczpjZmRpPSJodHRwOi8vd3d3LnNhdC5nb2IubXgvY2ZkLzMiIFZlcnNpb249IjMuMyIgRmVjaGE9IjIwMTctMDYtMTRUMTE6NTg6NDUiIFNlbGxvPSJOZVh1and1QXBQVTljQnZCaTZkQVNXc1N3RkkwODVJdllINHZmTGlIaCtYeUF1bGgrWXpLQlNIZ2t6MWVKZ3hRSlplbVE2TmxkcEY1b3JZVkFPODBWd1ZDaHcyakh4eFZPV0ZJTUxYT3BTQlNYUkR1ZWtUbmd3anNoNnYzTDlja3RXV01URHhnUVhoN3U0eS9OV09RT1RDUXpuYjZEV3N0WG9laDFNeUhBMmVkemE3dlRucUlSbUdKd3lveFc5dllWK0NqV3FHemViSVpXVURHekc0M09MU1NrWTBVN2RTSG5qZHB6dGwvblBmV0VDdm12emdlaFB5TFN2ZEtSSkFMcUNGWkVIQ3lhRy9FQW83WUhTWjJIS3VMbTRsaXNjSjRmQzlqUjBUV3p6aTNteEpVdGZiempCRUtyS1NXaGxwanNhTG1vZXZGN0hyd1R2RUV3blFCYkE9PSIgRm9ybWFQYWdvPSIwMSIgTm9DZXJ0aWZpY2Fkbz0iMjAwMDEwMDAwMDAzMDAwMjI3NjciIENlcnRpZmljYWRvPSJNSUlGeGpDQ0E2NmdBd0lCQWdJVU1qQXdNREV3TURBd01EQXpNREF3TWpJM05qY3dEUVlKS29aSWh2Y05BUUVMQlFBd2dnRm1NU0F3SGdZRFZRUUREQmRCTGtNdUlESWdaR1VnY0hKMVpXSmhjeWcwTURrMktURXZNQzBHQTFVRUNnd21VMlZ5ZG1samFXOGdaR1VnUVdSdGFXNXBjM1J5WVdOcHc3TnVJRlJ5YVdKMWRHRnlhV0V4T0RBMkJnTlZCQXNNTDBGa2JXbHVhWE4wY21GamFjT3piaUJrWlNCVFpXZDFjbWxrWVdRZ1pHVWdiR0VnU1c1bWIzSnRZV05wdzdOdU1Ta3dKd1lKS29aSWh2Y05BUWtCRmhwaGMybHpibVYwUUhCeWRXVmlZWE11YzJGMExtZHZZaTV0ZURFbU1DUUdBMVVFQ1F3ZFFYWXVJRWhwWkdGc1oyOGdOemNzSUVOdmJDNGdSM1ZsY25KbGNtOHhEakFNQmdOVkJCRU1CVEEyTXpBd01Rc3dDUVlEVlFRR0V3Sk5XREVaTUJjR0ExVUVDQXdRUkdsemRISnBkRzhnUm1Wa1pYSmhiREVTTUJBR0ExVUVCd3dKUTI5NWIyRmp3NkZ1TVJVd0V3WURWUVF0RXd4VFFWUTVOekEzTURGT1RqTXhJVEFmQmdrcWhraUc5dzBCQ1FJTUVsSmxjM0J2Ym5OaFlteGxPaUJCUTBSTlFUQWVGdzB4TmpFd01qRXlNVEUyTXpSYUZ3MHlNREV3TWpFeU1URTJNelJhTUlHeU1Sb3dHQVlEVlFRREV4RlRSVTVVU1VWT1ZDQlRRU0JFUlNCRFZqRWFNQmdHQTFVRUtSTVJVMFZPVkVsRlRsUWdVMEVnUkVVZ1ExWXhHakFZQmdOVkJBb1RFVk5GVGxSSlJVNVVJRk5CSUVSRklFTldNU1V3SXdZRFZRUXRFeHhWVEVNd05URXhNamxIUXpBZ0x5QklSVWRVTnpZeE1EQXpORk15TVI0d0hBWURWUVFGRXhVZ0x5QklSVWRVTnpZeE1EQXpUVVJHVWs1T01Ea3hGVEFUQmdOVkJBc1VERkJ5ZFdWaVlYTmZRMFpFU1RDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTEt5UWxjZjFFb2lQalZBYm5JcW9Pb1c2QzRzendjakRubU4xWU0wNEFrN3NyUEljbjRCTCtpUHVOM3FGRktacnpHYnc0eVpWVWUyRUtUaUhnSy9nTlZiVXdPcGlVR25ZWTY4dERheVBvbExGc2RhdVh2ZExiWm0reVZObkh6bGJnWG4rLzdXRHZNYWU5cjNsNUlUWGRpQW1HdUJiREg2ZllwRTVQSGh4cnlhVzdsb2FWZjNTaTdrT0pjZ1hhRGRpeXVtdm5pUERUN3hWME1FcDB2bmNla3R3amdFalJXRWcvOVJBS3laazcreDBTTGt3ZUhQYzE4bWU4VzFhSGdZZ0tQSmZvZmZTYlFVM1ZUWXJvWnZoWUUwYS8vRkRIZUl6aVZFeXNOMHBqcm5WS0kxRlZ2eVl6bDQ3OFhJeXRTYnBRcUVtbHBTQXFxaE9pU29oUHhuVWFjQ0F3RUFBYU1kTUJzd0RBWURWUjBUQVFIL0JBSXdBREFMQmdOVkhROEVCQU1DQnNBd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dJQkFFOWtkcTdydDRoWDJNTDg1QmkrTXpDUkd6NWNmUFRxcUhDZ2hSdUtRS3lGeEpncUthWDVzT3dua25zVTRlYm1JK1ZuR1dHRDJncWJNVWRVenJzckhGeE5HTmU0K3VUbVhaQW9FOVVGa1JxTHM1TG0wblNpK1FmaGQ5MW01RHhRb1A4TWxpaTMycnFiQ1NVeHloVmFZOFQwS0xnd3d6RkxzYXF5OXZrN3U2TzQ3em1GeU4wQmxXTlFqZ1dQRkRFLzhPWWhiaFlUTERGa01jcmE5OGJVL2JnWFlIM1g3cmZYWWFic3ZmbUx2anJBWkdqS1pSNUovT1ZTQ0RMbGUwa0ZHY1FEMktLcTgrTnQzUkk0cUVoQU9ucExVL29iMlFhZ2JhM3M4aXYrd0NRaXU5aE9Lc0Z2WUpnNkJPZjhQbHFZeG5zUk9TODhpUnlyaWtKYTFvSW9zeTdjekFMZFBiNk9HWDg5Uk5ya1lNWkhWSjdmUEJFbzZ4M09xQ2xRditkTXAzZDVNM2QwVlVhTzVEU1dIY01nb3pob2xaMm5zemRibGViK29rdzAzUkJVM3h3YkhnU2hSODh2UWd3STBUcGxiVGpBUGNpMDYveEZHVzRQY0gxaXV3THNjOGE4K2JGTDBLK2k3OXVYWU9Ia0MwV29oNjVBcFY3MEVKeWN1TWo3SjNyNTkrSXk1Qm1GaCtJd2tTRS9ZdW00V0FvUUxjeHlrcGF2UTFZRUFvVEpieldIaDZlaWNtV0JsNW92ZDZGOFlsQ0tiakQrNlRvNnFkeVQ4MHpNOHRFMXVRRC9FaUdSNDlTeWtSWHFUem9Zb3Q4MjVubEZpb0w1dXd4cFJxYVcwUVMzSDdOd0E4YVZCY2VQUGRYK0dJbm11bXBIWUtvcVMwWS8yNDJ5R2d2TSIgU3ViVG90YWw9IjE1MTAuMDAiIE1vbmVkYT0iTVhOIiBUaXBvQ2FtYmlvPSIxIiBUb3RhbD0iMTc1MS42MCIgVGlwb0RlQ29tcHJvYmFudGU9IkkiIE1ldG9kb1BhZ289IlBVRSIgTHVnYXJFeHBlZGljaW9uPSIwNjMwMCIgeHNpOnNjaGVtYUxvY2F0aW9uPSJodHRwOi8vd3d3LnNhdC5nb2IubXgvY2ZkLzMgaHR0cDovL3d3dy5zYXQuZ29iLm14L3NpdGlvX2ludGVybmV0L2NmZC8zL2NmZHYzMy54c2QgIj48Y2ZkaTpFbWlzb3IgUmZjPSJVTEMwNTExMjlHQzAiIE5vbWJyZT0iU0VOVElFTlQgU0EgREUgQ1YiIFJlZ2ltZW5GaXNjYWw9IjYwMSIvPjxjZmRpOlJlY2VwdG9yIFJmYz0iSUFEMTIxMjE0QjM0IiBOb21icmU9IklUICZhbXA7IFNXIERldmVsb3BtZW50IFNvbHV0aW9ucyBkZSBNZXhpY28gUyBkZSBSTCBkZSBDViIgVXNvQ0ZEST0iUDAxIi8+PGNmZGk6Q29uY2VwdG9zPjxjZmRpOkNvbmNlcHRvIENsYXZlUHJvZFNlcnY9IjEwMTIyMTAwIiBDYW50aWRhZD0iNSIgQ2xhdmVVbmlkYWQ9Ik03NCIgRGVzY3JpcGNpb249IlBydWViYSBDYXRhbG9nb3MgTnVldm9zIiBWYWxvclVuaXRhcmlvPSIyNTAuMDAiIEltcG9ydGU9IjEyNTAuMDAiIFVuaWRhZD0iS2lsbyI+PGNmZGk6SW1wdWVzdG9zPjxjZmRpOlRyYXNsYWRvcz48Y2ZkaTpUcmFzbGFkbyBCYXNlPSIxMjUwLjAwIiBJbXB1ZXN0bz0iMDAyIiBUaXBvRmFjdG9yPSJUYXNhIiBUYXNhT0N1b3RhPSIwLjE2MDAwMCIgSW1wb3J0ZT0iMjAwLjAwIi8+PC9jZmRpOlRyYXNsYWRvcz48L2NmZGk6SW1wdWVzdG9zPjwvY2ZkaTpDb25jZXB0bz48Y2ZkaTpDb25jZXB0byBDbGF2ZVByb2RTZXJ2PSIyNDExMTUwMCIgQ2FudGlkYWQ9IjEiIENsYXZlVW5pZGFkPSJLR00iIERlc2NyaXBjaW9uPSJ0cmFzbHVjaWRhIDkweDkwIGNtLiBjYWwuIDIwMCIgVmFsb3JVbml0YXJpbz0iMjIuMDAiIEltcG9ydGU9IjIyLjAwIiBVbmlkYWQ9ImtnIj48Y2ZkaTpJbXB1ZXN0b3M+PGNmZGk6VHJhc2xhZG9zPjxjZmRpOlRyYXNsYWRvIEJhc2U9IjIyLjAwIiBJbXB1ZXN0bz0iMDAyIiBUaXBvRmFjdG9yPSJUYXNhIiBUYXNhT0N1b3RhPSIwLjE2MDAwMCIgSW1wb3J0ZT0iMy41MiIvPjwvY2ZkaTpUcmFzbGFkb3M+PC9jZmRpOkltcHVlc3Rvcz48L2NmZGk6Q29uY2VwdG8+PGNmZGk6Q29uY2VwdG8gQ2xhdmVQcm9kU2Vydj0iMTMxMDE3MTIiIENhbnRpZGFkPSIxMCIgQ2xhdmVVbmlkYWQ9IktHTSIgRGVzY3JpcGNpb249IlBPTElFVElMRU5PIERFIEJBSkEgREVOU0lEQUQiIFZhbG9yVW5pdGFyaW89IjIzLjgwIiBJbXBvcnRlPSIyMzguMDAiIFVuaWRhZD0iS0ciPjxjZmRpOkltcHVlc3Rvcz48Y2ZkaTpUcmFzbGFkb3M+PGNmZGk6VHJhc2xhZG8gQmFzZT0iMjM4LjAwIiBJbXB1ZXN0bz0iMDAyIiBUaXBvRmFjdG9yPSJUYXNhIiBUYXNhT0N1b3RhPSIwLjE2MDAwMCIgSW1wb3J0ZT0iMzguMDgiLz48L2NmZGk6VHJhc2xhZG9zPjwvY2ZkaTpJbXB1ZXN0b3M+PC9jZmRpOkNvbmNlcHRvPjwvY2ZkaTpDb25jZXB0b3M+PGNmZGk6SW1wdWVzdG9zIFRvdGFsSW1wdWVzdG9zVHJhc2xhZGFkb3M9IjI0MS42MCI+PGNmZGk6VHJhc2xhZG9zPjxjZmRpOlRyYXNsYWRvIEltcHVlc3RvPSIwMDIiIFRpcG9GYWN0b3I9IlRhc2EiIFRhc2FPQ3VvdGE9IjAuMTYwMDAwIiBJbXBvcnRlPSIyNDEuNjAiLz48L2NmZGk6VHJhc2xhZG9zPjwvY2ZkaTpJbXB1ZXN0b3M+PC9jZmRpOkNvbXByb2JhbnRlPg==</sxml>
      </urn:timbrar_cfdi>
   </soapenv:Body>
</soapenv:Envelope>
```
Después daremos click al botón ![](http://i.imgur.com/zp9cg7E.png) una vez hecho esto nos saldrá el resultado.

  ![](http://i.imgur.com/fwG4Rc2.png)
  
## Confirmación CFDI
La confirmación es única e irrepetible, Esto es para expedir un comprobante con importes o tipo de cambio fuera del rango establecido o en ambos casos. La confirmación deben registrar valores alfanuméricos de 5 posiciones.

Para la petición de confirmación de timbrado de un CFDI, deberá enviar las credenciales asignadas.

Para hacer la petición solo necesitamos hacer doble click sobre **Request 1** debajo de **confirmacion_cfdi**:
  
  ![](http://i.imgur.com/GubQcAw.png)
  
Despues de dar click nos saldrá la siguiente ventana, debemos modificar la petición usando nuestras credenciales, como el siguiente código:

![](http://i.imgur.com/rO23b9Q.png)

```
 <soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:WashOut">
   <soapenv:Header/>
   <soapenv:Body>
      <urn:confirmacion_cfdi soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
         <username xsi:type="xsd:string">AAA010101000</username>
         <password xsi:type="xsd:string">h6584D56fVdBbSmmnB</password>
      </urn:confirmacion_cfdi>
   </soapenv:Body>
</soapenv:Envelope> 
 ```
  
Después daremos click al botón ![](http://i.imgur.com/zp9cg7E.png) nos saldra el resultado.

![](http://i.imgur.com/7ZPYvy1.png)
 
## Cancelar CFDI
Para la cancelación son necesarias las credenciales asignadas, RFC del emisor, un arreglo de UUIDs, el archivo PFX convertido a cadena en base64 y el password del archivo PFX:

Crear un cliente para hacer la petición de cancelación al webservice:

Para hacer la petición solo necesitamos hacer doble click sobre **Request 1** debajo de **cancelar_cfdi**:

![](http://i.imgur.com/py8K2eL.png)
```
<soapenv:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" xmlns:xsd="http://www.w3.org/2001/XMLSchema" xmlns:soapenv="http://schemas.xmlsoap.org/soap/envelope/" xmlns:urn="urn:WashOut">
   <soapenv:Header/>
   <soapenv:Body>
      <urn:timbrar_cfdi soapenv:encodingStyle="http://schemas.xmlsoap.org/soap/encoding/">
         <username xsi:type="xsd:string">AAA010101000</username>
         <password xsi:type="xsd:string">h6584D56fVdBbSmmnB</password>
         <sxml xsi:type="xsd:string">PD94bWwgdmVyc2lvbj0iMS4wIiBlbmNvZGluZz0idXRmLTgiPz48Y2ZkaTpDb21wcm9iYW50ZSB4bWxuczp4c2k9Imh0dHA6Ly93d3cudzMub3JnLzIwMDEvWE1MU2NoZW1hLWluc3RhbmNlIiB4bWxuczpjZmRpPSJodHRwOi8vd3d3LnNhdC5nb2IubXgvY2ZkLzMiIFZlcnNpb249IjMuMyIgRmVjaGE9IjIwMTctMDYtMTRUMTE6NTg6NDUiIFNlbGxvPSJOZVh1and1QXBQVTljQnZCaTZkQVNXc1N3RkkwODVJdllINHZmTGlIaCtYeUF1bGgrWXpLQlNIZ2t6MWVKZ3hRSlplbVE2TmxkcEY1b3JZVkFPODBWd1ZDaHcyakh4eFZPV0ZJTUxYT3BTQlNYUkR1ZWtUbmd3anNoNnYzTDlja3RXV01URHhnUVhoN3U0eS9OV09RT1RDUXpuYjZEV3N0WG9laDFNeUhBMmVkemE3dlRucUlSbUdKd3lveFc5dllWK0NqV3FHemViSVpXVURHekc0M09MU1NrWTBVN2RTSG5qZHB6dGwvblBmV0VDdm12emdlaFB5TFN2ZEtSSkFMcUNGWkVIQ3lhRy9FQW83WUhTWjJIS3VMbTRsaXNjSjRmQzlqUjBUV3p6aTNteEpVdGZiempCRUtyS1NXaGxwanNhTG1vZXZGN0hyd1R2RUV3blFCYkE9PSIgRm9ybWFQYWdvPSIwMSIgTm9DZXJ0aWZpY2Fkbz0iMjAwMDEwMDAwMDAzMDAwMjI3NjciIENlcnRpZmljYWRvPSJNSUlGeGpDQ0E2NmdBd0lCQWdJVU1qQXdNREV3TURBd01EQXpNREF3TWpJM05qY3dEUVlKS29aSWh2Y05BUUVMQlFBd2dnRm1NU0F3SGdZRFZRUUREQmRCTGtNdUlESWdaR1VnY0hKMVpXSmhjeWcwTURrMktURXZNQzBHQTFVRUNnd21VMlZ5ZG1samFXOGdaR1VnUVdSdGFXNXBjM1J5WVdOcHc3TnVJRlJ5YVdKMWRHRnlhV0V4T0RBMkJnTlZCQXNNTDBGa2JXbHVhWE4wY21GamFjT3piaUJrWlNCVFpXZDFjbWxrWVdRZ1pHVWdiR0VnU1c1bWIzSnRZV05wdzdOdU1Ta3dKd1lKS29aSWh2Y05BUWtCRmhwaGMybHpibVYwUUhCeWRXVmlZWE11YzJGMExtZHZZaTV0ZURFbU1DUUdBMVVFQ1F3ZFFYWXVJRWhwWkdGc1oyOGdOemNzSUVOdmJDNGdSM1ZsY25KbGNtOHhEakFNQmdOVkJCRU1CVEEyTXpBd01Rc3dDUVlEVlFRR0V3Sk5XREVaTUJjR0ExVUVDQXdRUkdsemRISnBkRzhnUm1Wa1pYSmhiREVTTUJBR0ExVUVCd3dKUTI5NWIyRmp3NkZ1TVJVd0V3WURWUVF0RXd4VFFWUTVOekEzTURGT1RqTXhJVEFmQmdrcWhraUc5dzBCQ1FJTUVsSmxjM0J2Ym5OaFlteGxPaUJCUTBSTlFUQWVGdzB4TmpFd01qRXlNVEUyTXpSYUZ3MHlNREV3TWpFeU1URTJNelJhTUlHeU1Sb3dHQVlEVlFRREV4RlRSVTVVU1VWT1ZDQlRRU0JFUlNCRFZqRWFNQmdHQTFVRUtSTVJVMFZPVkVsRlRsUWdVMEVnUkVVZ1ExWXhHakFZQmdOVkJBb1RFVk5GVGxSSlJVNVVJRk5CSUVSRklFTldNU1V3SXdZRFZRUXRFeHhWVEVNd05URXhNamxIUXpBZ0x5QklSVWRVTnpZeE1EQXpORk15TVI0d0hBWURWUVFGRXhVZ0x5QklSVWRVTnpZeE1EQXpUVVJHVWs1T01Ea3hGVEFUQmdOVkJBc1VERkJ5ZFdWaVlYTmZRMFpFU1RDQ0FTSXdEUVlKS29aSWh2Y05BUUVCQlFBRGdnRVBBRENDQVFvQ2dnRUJBTEt5UWxjZjFFb2lQalZBYm5JcW9Pb1c2QzRzendjakRubU4xWU0wNEFrN3NyUEljbjRCTCtpUHVOM3FGRktacnpHYnc0eVpWVWUyRUtUaUhnSy9nTlZiVXdPcGlVR25ZWTY4dERheVBvbExGc2RhdVh2ZExiWm0reVZObkh6bGJnWG4rLzdXRHZNYWU5cjNsNUlUWGRpQW1HdUJiREg2ZllwRTVQSGh4cnlhVzdsb2FWZjNTaTdrT0pjZ1hhRGRpeXVtdm5pUERUN3hWME1FcDB2bmNla3R3amdFalJXRWcvOVJBS3laazcreDBTTGt3ZUhQYzE4bWU4VzFhSGdZZ0tQSmZvZmZTYlFVM1ZUWXJvWnZoWUUwYS8vRkRIZUl6aVZFeXNOMHBqcm5WS0kxRlZ2eVl6bDQ3OFhJeXRTYnBRcUVtbHBTQXFxaE9pU29oUHhuVWFjQ0F3RUFBYU1kTUJzd0RBWURWUjBUQVFIL0JBSXdBREFMQmdOVkhROEVCQU1DQnNBd0RRWUpLb1pJaHZjTkFRRUxCUUFEZ2dJQkFFOWtkcTdydDRoWDJNTDg1QmkrTXpDUkd6NWNmUFRxcUhDZ2hSdUtRS3lGeEpncUthWDVzT3dua25zVTRlYm1JK1ZuR1dHRDJncWJNVWRVenJzckhGeE5HTmU0K3VUbVhaQW9FOVVGa1JxTHM1TG0wblNpK1FmaGQ5MW01RHhRb1A4TWxpaTMycnFiQ1NVeHloVmFZOFQwS0xnd3d6RkxzYXF5OXZrN3U2TzQ3em1GeU4wQmxXTlFqZ1dQRkRFLzhPWWhiaFlUTERGa01jcmE5OGJVL2JnWFlIM1g3cmZYWWFic3ZmbUx2anJBWkdqS1pSNUovT1ZTQ0RMbGUwa0ZHY1FEMktLcTgrTnQzUkk0cUVoQU9ucExVL29iMlFhZ2JhM3M4aXYrd0NRaXU5aE9Lc0Z2WUpnNkJPZjhQbHFZeG5zUk9TODhpUnlyaWtKYTFvSW9zeTdjekFMZFBiNk9HWDg5Uk5ya1lNWkhWSjdmUEJFbzZ4M09xQ2xRditkTXAzZDVNM2QwVlVhTzVEU1dIY01nb3pob2xaMm5zemRibGViK29rdzAzUkJVM3h3YkhnU2hSODh2UWd3STBUcGxiVGpBUGNpMDYveEZHVzRQY0gxaXV3THNjOGE4K2JGTDBLK2k3OXVYWU9Ia0MwV29oNjVBcFY3MEVKeWN1TWo3SjNyNTkrSXk1Qm1GaCtJd2tTRS9ZdW00V0FvUUxjeHlrcGF2UTFZRUFvVEpieldIaDZlaWNtV0JsNW92ZDZGOFlsQ0tiakQrNlRvNnFkeVQ4MHpNOHRFMXVRRC9FaUdSNDlTeWtSWHFUem9Zb3Q4MjVubEZpb0w1dXd4cFJxYVcwUVMzSDdOd0E4YVZCY2VQUGRYK0dJbm11bXBIWUtvcVMwWS8yNDJ5R2d2TSIgU3ViVG90YWw9IjE1MTAuMDAiIE1vbmVkYT0iTVhOIiBUaXBvQ2FtYmlvPSIxIiBUb3RhbD0iMTc1MS42MCIgVGlwb0RlQ29tcHJvYmFudGU9IkkiIE1ldG9kb1BhZ289IlBVRSIgTHVnYXJFeHBlZGljaW9uPSIwNjMwMCIgeHNpOnNjaGVtYUxvY2F0aW9uPSJodHRwOi8vd3d3LnNhdC5nb2IubXgvY2ZkLzMgaHR0cDovL3d3dy5zYXQuZ29iLm14L3NpdGlvX2ludGVybmV0L2NmZC8zL2NmZHYzMy54c2QgIj48Y2ZkaTpFbWlzb3IgUmZjPSJVTEMwNTExMjlHQzAiIE5vbWJyZT0iU0VOVElFTlQgU0EgREUgQ1YiIFJlZ2ltZW5GaXNjYWw9IjYwMSIvPjxjZmRpOlJlY2VwdG9yIFJmYz0iSUFEMTIxMjE0QjM0IiBOb21icmU9IklUICZhbXA7IFNXIERldmVsb3BtZW50IFNvbHV0aW9ucyBkZSBNZXhpY28gUyBkZSBSTCBkZSBDViIgVXNvQ0ZEST0iUDAxIi8+PGNmZGk6Q29uY2VwdG9zPjxjZmRpOkNvbmNlcHRvIENsYXZlUHJvZFNlcnY9IjEwMTIyMTAwIiBDYW50aWRhZD0iNSIgQ2xhdmVVbmlkYWQ9Ik03NCIgRGVzY3JpcGNpb249IlBydWViYSBDYXRhbG9nb3MgTnVldm9zIiBWYWxvclVuaXRhcmlvPSIyNTAuMDAiIEltcG9ydGU9IjEyNTAuMDAiIFVuaWRhZD0iS2lsbyI+PGNmZGk6SW1wdWVzdG9zPjxjZmRpOlRyYXNsYWRvcz48Y2ZkaTpUcmFzbGFkbyBCYXNlPSIxMjUwLjAwIiBJbXB1ZXN0bz0iMDAyIiBUaXBvRmFjdG9yPSJUYXNhIiBUYXNhT0N1b3RhPSIwLjE2MDAwMCIgSW1wb3J0ZT0iMjAwLjAwIi8+PC9jZmRpOlRyYXNsYWRvcz48L2NmZGk6SW1wdWVzdG9zPjwvY2ZkaTpDb25jZXB0bz48Y2ZkaTpDb25jZXB0byBDbGF2ZVByb2RTZXJ2PSIyNDExMTUwMCIgQ2FudGlkYWQ9IjEiIENsYXZlVW5pZGFkPSJLR00iIERlc2NyaXBjaW9uPSJ0cmFzbHVjaWRhIDkweDkwIGNtLiBjYWwuIDIwMCIgVmFsb3JVbml0YXJpbz0iMjIuMDAiIEltcG9ydGU9IjIyLjAwIiBVbmlkYWQ9ImtnIj48Y2ZkaTpJbXB1ZXN0b3M+PGNmZGk6VHJhc2xhZG9zPjxjZmRpOlRyYXNsYWRvIEJhc2U9IjIyLjAwIiBJbXB1ZXN0bz0iMDAyIiBUaXBvRmFjdG9yPSJUYXNhIiBUYXNhT0N1b3RhPSIwLjE2MDAwMCIgSW1wb3J0ZT0iMy41MiIvPjwvY2ZkaTpUcmFzbGFkb3M+PC9jZmRpOkltcHVlc3Rvcz48L2NmZGk6Q29uY2VwdG8+PGNmZGk6Q29uY2VwdG8gQ2xhdmVQcm9kU2Vydj0iMTMxMDE3MTIiIENhbnRpZGFkPSIxMCIgQ2xhdmVVbmlkYWQ9IktHTSIgRGVzY3JpcGNpb249IlBPTElFVElMRU5PIERFIEJBSkEgREVOU0lEQUQiIFZhbG9yVW5pdGFyaW89IjIzLjgwIiBJbXBvcnRlPSIyMzguMDAiIFVuaWRhZD0iS0ciPjxjZmRpOkltcHVlc3Rvcz48Y2ZkaTpUcmFzbGFkb3M+PGNmZGk6VHJhc2xhZG8gQmFzZT0iMjM4LjAwIiBJbXB1ZXN0bz0iMDAyIiBUaXBvRmFjdG9yPSJUYXNhIiBUYXNhT0N1b3RhPSIwLjE2MDAwMCIgSW1wb3J0ZT0iMzguMDgiLz48L2NmZGk6VHJhc2xhZG9zPjwvY2ZkaTpJbXB1ZXN0b3M+PC9jZmRpOkNvbmNlcHRvPjwvY2ZkaTpDb25jZXB0b3M+PGNmZGk6SW1wdWVzdG9zIFRvdGFsSW1wdWVzdG9zVHJhc2xhZGFkb3M9IjI0MS42MCI+PGNmZGk6VHJhc2xhZG9zPjxjZmRpOlRyYXNsYWRvIEltcHVlc3RvPSIwMDIiIFRpcG9GYWN0b3I9IlRhc2EiIFRhc2FPQ3VvdGE9IjAuMTYwMDAwIiBJbXBvcnRlPSIyNDEuNjAiLz48L2NmZGk6VHJhc2xhZG9zPjwvY2ZkaTpJbXB1ZXN0b3M+PC9jZmRpOkNvbXByb2JhbnRlPg==</sxml>
      </urn:timbrar_cfdi>
   </soapenv:Body>
</soapenv:Envelope>
```
Después daremos click al botón ![](http://i.imgur.com/zp9cg7E.png) y una vez hecho esto nos saldrá el resultado.

   ![](http://i.imgur.com/oE7IB3H.png)

## Crear archivo PFX
Para poder crear el archivo PFX es necesario contar con el certificado y la llave privada.

Una vez teniendo esta informacion para la creación del PFX usamos el programa [OPENSSL](https://www.openssl.org/) en el cual utilizamos los siguientes tres comandos de consola que son los siguientes:

***Los certificados de prueba se encuentran en el proyecto en la carpeta certificados o puede descargarlos de [http://www.sat.gob.mx/informacion_fiscal/factura_electronica/Paginas/certificado_sello_digital.aspx](http://www.sat.gob.mx/informacion_fiscal/factura_electronica/Paginas/certificado_sello_digital.aspx)**

1. Creación del archivo PEM del certificado.
```
openssl x509 -in 'CSD01_AAA010101AAA.cer' -inform DER -out 'CSD01_AAA010101AAA.cer.pem' -outform PEM
```

2. Creación del archivo PEM de la llave privada.
```
openssl pkcs8 -inform DER -in 'CSD01_AAA010101AAA.key' -passin pass:12345678a -out 'CSD01_AAA010101AAA.key.pem'
```

3. Creación del archivo PFX.
```
openssl pkcs12 -export -out 'CSD01_AAA010101AAA.pfx' -in 'CSD01_AAA010101AAA.cer.pem' -inkey 'CSD01_AAA010101AAA.key.pem' -password pass:12345678a
```



