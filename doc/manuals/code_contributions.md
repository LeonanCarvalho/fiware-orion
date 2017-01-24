# Sample code

This section contains code snippets (most of them contributed by
external developers) that can be used as examples for programming with
Orion Context Broker in different technologies. Note that Orion Context
Broker evolves with time so it could happen that the code examples get
obsolete in some moment (the publication date is provides as reference).

## Java

Thanks to Kurento (http://www.kurento.org) a Java library that shows how
to access to Orion Context Broker:
<https://github.com/KurentoLegacy/kmf-orion-connector/tree/develop>
(published June 2014).

Thanks to Alejandro Villamarin (published around October 2013):
```java
       import com.sun.jersey.api.client.Client;
       import com.sun.jersey.api.client.ClientResponse;
       import com.sun.jersey.api.client.WebResource;

       //Test the Orion Broker, query for Room
       String urlString = "http://orion.lab.fiware.org:1026/v1/queryContext";
       String XML = "<?xml version=\"1.0\" encoding=\"UTF-8\"?>" + 
            "<queryContextRequest>" + 
            "<entityIdList>" + 
            "<entityId type=\"Room\" isPattern=\"true\">" + 
            "<id>Room.*</id>" + 
            "</entityId>" +
            "</entityIdList>" + 
            "<attributeList>" + 
            "<attribute>temperature</attribute>" + 
            "</attributeList>" + 
            "</queryContextRequest>";      
       try {     
          Client client = Client.create();
          WebResource webResource = client.resource(urlString);
          ClientResponse response = webResource.type("application/xml").header("X-Auth-Token", "your_auth_token").post(ClientResponse.class, XML);
       
          if (response.getStatus() != 200) {
             System.err.println(response.getEntity(String.class));
        throw new RuntimeException("Failed : HTTP error code : " + response.getStatus());
          }
       
          System.out.println("Output from Server .... \n");
          String output = response.getEntity(String.class);
          System.out.println(output);
       
       } 
       catch (Exception e) {
          System.err.println("Failed. Reason is " + e.getMessage());
       }
```
## JavaScript

Using JQuery AJAX, thanks to Marco Vereda Manchego ([original
post](http://marcovereda.blogspot.com.es/2013/10/code-snippet-for-custom-http-post.html))
(published around October 2013):
```javascript
     function capture_sensor_data(){  
     var contentTypeRequest = $.ajax({  
          url: 'http://orion.lab.fiware.org:1026/v1/queryContext',  
          data: '<?xml version="1.0" encoding="UTF-8"?>  
                    <queryContextRequest>  
                         <entityIdList>  
                              <entityId type = "Sensor" isPattern="true">  
                                   <id>urn:smartsantander:testbed:.*</id>   
                              </entityId>   
                         </entityIdList>   
                         <attributeList>   
                              <attribute>Latitud</attribute>  
                              <attribute>Longitud</attribute>  
                         </attributeList>       
                    </queryContextRequest>  ',  
          type: 'POST',  
          dataType: 'xml',  
          contentType: 'application/xml',  
          headers: { 'X-Auth-Token' :   
               'you_auth_token'}  
          });  
          contentTypeRequest.done(function(xml){   
               var latitud = new Array();  
               var longitud = new Array();  
               var i = 0;       
               var len = $(xml).find("contextAttribute").children().size();  
               $(xml).find('contextAttribute').each(function(){  
                         if (  $(this).find('type').text() == "urn:x-ogc:def:phenomenon:IDAS:1.0:latitude"  
                               && $(this).find('contextValue').text() != "" )  
                         {   
                              latitud[i] = $(this).find('contextValue').text(); i=i+1;  
                         }  
                         if ($(this).find('type').text() == "urn:x-ogc:def:phenomenon:IDAS:1.0:longitude"  
                                && $(this).find('contextValue').text() != ""       )  
                         {   
                              longitud[i] = $(this).find('contextValue').text();   
                         }  
                                                                      });  
               for (var j=0; j<i; j++){                 
                    console.log("DEBUG :" + latitud[j] + "," + longitud[j]);                 
               }                      
          });                 
          contentTypeRequest.fail(function(jqXHR, textStatus){     
               console.log( "DEBUG :  Ajax request failed... (" + textStatus + ' - ' + jqXHR.responseText + ")." );  
          });  
          contentTypeRequest.always(function(jqXHR, textStatus){       });  
     }
```
## Arduino

Thanks to Enrique Hernandez Zurita, using Orion 0.11.0 and 0.12.0
(published around June 2014):
```c
    #include <SPI.h>
    #include <WiFi.h>
    #include <WString.h>
    char ssid[] = "SSID"; 
    char pass[] = "CONTRASEÑA";    
              
    int status = WL_IDLE_STATUS;
    IPAddress server(130,206,82,115);
    WiFiClient client;
    String line="";
    int value=0;
    int led=9;

    void setup() {
      
        pinMode(led,OUTPUT);
        Serial.begin(9600); 
        while (!Serial) {  ;
        }
        if (WiFi.status() == WL_NO_SHIELD) {

        Serial.println("WiFi shield not present"); 
        while(true);
        } 
        while (status != WL_CONNECTED) { 
        Serial.print("Attempting to connect to SSID: ");
        Serial.println(ssid); 
        status = WiFi.begin(ssid, pass);
        delay(5000); 
        } 
        Serial.println("Connected to wifi");
        Serial.println("\nStarting connection to server..."); 
        
        if (client.connect(server, 1026)) {
        Serial.println("connected to server");
        client.println("POST /NGSI10/queryContext HTTP/1.1");
        client.println("Host: 130.206.82.115:1026");
        client.println("User-Agent: Arduino/1.1");
        client.println("Connection: close");
        client.println("Content-Type: application/xml");
        client.print("Content-Length: ");
        client.println("227");
        client.println();
        client.println("<?xml version=\"1.0\" encoding=\"UFT-8\"?>");
        client.println("<queryContextRequest>");
        client.println("<entityIdList>");
        client.println("<entityId type=\"UPCT:ACTUATOR\" isPattern=\"false\">");
        client.println("<id>UPCT:ACTUATOR:1</id>");
        client.println("</entityId>");
        client.println("</entityIdList>");
        client.println("<attributeList/>");
        client.println("</queryContextRequest>");
        client.println();
        
        Serial.println("XML Sent");
        
        }else{Serial.println("Impossible to connect");}
    }

    void loop() {
        if(value=1){digitalWrite(led, HIGH);}
      
        while (client.available()) {
        char c = client.read();
        Serial.write(c);
        line=line + c;
            if(c==10){
            value=searchValue(line,value);
            line="";
            }
         }
      
         if (!client.connected()) {
         Serial.println();
         Serial.println("disconnecting from server.");
         Serial.println(value);
         client.stop();
         while(true);
         }
    }


    int searchValue(String s,int i) {
      int beginning,ending;
      String val;
      beginning=s.indexOf('>');
      ending=s.indexOf('<', beginning+1);
        if(s.startsWith("contextValue",s.indexOf('<')+1)){
        val =s.substring(beginning+1,ending);
        return val.toInt();
        }else{return i;}
    }
```

## PHP 

Open source PHP library created by @leonancarvalho support v1 and v2 API calls.
More examples can also be viewed at [Library](https://github.com/VM9/orion-explorer-php-frame-work) repository or an [Implementation](https://github.com/VM9/fiware-orion-explorer) repository

```php
<?php
include './autoloader.php'; //Using composer
$ip = "130.206.82.115";
$orion = new Orion\NGSIAPIv1($ip);
try{
    /**
     * Entity Creation
     * Ref: https://fiware-orion.readthedocs.io/en/develop/user/walkthrough_apiv1/index.html#entity-creation
     */
    echo "<h3>Request: </h3>", PHP_EOL;
    echo "<pre>";
    $Create = new \Orion\Operations\updateContext();
    $contextResponses = $Create->addElement("Room1", "Room", false)
            ->addAttrinbute("temperature", "float", "23")
            ->addAttrinbute("pressure", "integer", "720")
            ->setAction("APPEND")
            ->send($orion); //To send it you must give the orion connection as parameter
//            $SECONDUPDATE->send($orion2);//You also can work with 2 connections using this way, sending same entity to 2 different instances
    
    $Create->getRequest()->prettyPrint();//The contextElements sent to orion context broker in json format
    echo "</pre>";
    echo "<h3>Response: </h3>", PHP_EOL;
    echo "<pre>";
    $contextResponses->prettyPrint();
    echo "</pre>";
    
/**
     * Query Context operation
     * Ref: https://fiware-orion.readthedocs.io/en/develop/user/walkthrough_apiv1/index.html#query-context-operation
     */
echo "<h2>Basic</h2>";
    echo "<h3>Request : </h3>", PHP_EOL;
    echo "<pre>";
    $queryContext = new Orion\Operations\queryContext();
    $queryResponse = $queryContext->addElement("Room1", "Room")
            ->send($orion);
    $responseData = $queryResponse->get();


    $queryContext->getRequest()->prettyPrint();
    echo "</pre>";
    echo "<h3>Response: </h3>", PHP_EOL;
    echo "<pre>";
    $queryResponse->prettyPrint();
    echo "</pre>";
}catch (\Exception $e) {
    echo "<h1>", get_class($e), "</h1><h3>", $e->getMessage(), "</h3>";
    echo $e->getFile(), " [", $e->getLine(), "]<br>";
    echo "<pre>", $e->getTraceAsString(), "</pre>";
    if(method_exists($e, "getResponse")){
       echo "<pre>Orion Response:",PHP_EOL , $e->getResponse(),"</pre>";
    }
}
```
