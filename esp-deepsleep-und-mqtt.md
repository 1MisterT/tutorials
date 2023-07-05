 # ESP DeepSleep und MQTT   
DeepSleep mit Arduino nach MQTT Upload   
 --- 
 ##    
 ## Dokumentation:   
[GitHub - plapointe6/EspMQTTClient at ecb7151b1719798034019f7fd12c2277c4707470](https://github.com/plapointe6/EspMQTTClient/tree/ecb7151b1719798034019f7fd12c2277c4707470)    
[NodeMCU - Deep Sleep des ESP8266](https://draeger-it.blog/nodemcu-deep-sleep-des-esp8266/)    
 ## Voraussetzung:   
- ESP8266 (bis jetzt nur damit getestet)   
- Funktionierender MQTT Broker   
   
   
 ## Vorüberlegung:   
Ein Arduino/ESP misst Sensordaten, sendet diese per MQTT und geht für eine bestimmte Zeit in den DeepSleep um Strom zu sparen und nicht unötig viele Sensorwerte zu senden.   
   
 ## Code & Beschreibung   
Zuvor natürlich MQTT Setup und evtl. Sensor Daten erfassen   
MQTT upload und DeepSleep   
```C
bool upload = false;
void loop() {
	char sensorValue = "Hello";

	if (client.isConnected() && !upload) {
		//Bereit für MQTT Push
      upload = true;
      client.publish(...); //MQTT Nachricht senden -> keine Details
    }
  
  if (!upload) {
    //auch jetzt noch MQTT Loop, um mit dem Broker zu verbinden etc.
    client.loop();
  }

  if (client.isConnected() && upload) {
	//nach erfolgreichen Upload, geht der ESP in den DeepSleep
    delay(100);
    Serial.println("DeepSleep beginnt jetzt!");
    ESP.deepSleep(15 * 60 * 1000000); //DeepSleep für 15 Minuten 15min * 60sec * 1000000usec
	//nach dem DeepSleep beginnt der Arduino wie nach einem Reset von vorn
  }
}
```
Bei jedem Loop ohne Upload, wird einfach der Client Loop aufgerurfen, welcher MQTT Daten senden und empfangen lässt. Ohne diesen loop, kann der ESP keine Verbindung mit dem MQTT Broker herstellen.   
```C
if (!upload) {    
    client.loop();
}
```
Wenn eine Verbindung besteht und noch kein Upload statt gefunden hat, werden die Daten per MQTT hochgeladen. Nach dem Upload wird **upload** auf true gesetzt um einen weiteren Upload an dieser Stelle zu verhindern   
```C
if (client.isConnected() && !upload) {
      client.publish(…); 
      upload = true;
}
```
Nach MQTT Verbindung und Erfolgreichen Upload, wird der ESP in den DeepSleep versetzt. Interessanterweise wird kein weitere **client.loop()** benötigt um die Daten per MQTT hochzuladen. Sollte der upload per MQTT nicht funktionieren, könnte man hier eventuell noch einen `client.loop()` einsetzten.    
```C
if (client.isConnected() && upload) {
    Serial.println("DeepSleep beginnt jetzt!");
    ESP.deepSleep(time_in_us);
  }
```
Zu beachten der DeepSleep wird in MicroSekunden gezählt, d.h. 5s = 5 \* 10⁶ us.   
   
