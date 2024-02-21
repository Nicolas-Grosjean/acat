[> SCBUS](https://confluence-sms.group.jenoptik.corp/x/zIIc)

In order to gain access to scbus data from DORA, the DORA structure ScBusRequest was introduced. This is served by the scbusgateway.

- [Workflow and Usage](https://confluence-sms.group.jenoptik.corp/display/NSP/scbusgateway#scbusgateway-WorkflowandUsage)
- [DORA Model](https://confluence-sms.group.jenoptik.corp/display/NSP/scbusgateway#scbusgateway-DORAModel)
    - [STRUCT ScBusRequest](https://confluence-sms.group.jenoptik.corp/display/NSP/scbusgateway#scbusgateway-STRUCTScBusRequest)
    - [ENUM RequestState](https://confluence-sms.group.jenoptik.corp/display/NSP/scbusgateway#scbusgateway-ENUMRequestState)
    - [ENUM ScBusType](https://confluence-sms.group.jenoptik.corp/display/NSP/scbusgateway#scbusgateway-ENUMScBusType)
- [How to fill ScBusRequest data to retrieve SCBUS data](https://confluence-sms.group.jenoptik.corp/display/NSP/scbusgateway#scbusgateway-HowtofillScBusRequestdatatoretrieveSCBUSdata)
- [State diagram](https://confluence-sms.group.jenoptik.corp/display/NSP/scbusgateway#scbusgateway-Statediagram)
- [Even / Reaction Matrix](https://confluence-sms.group.jenoptik.corp/display/NSP/scbusgateway#scbusgateway-Even/ReactionMatrix)

# Workflow and Usage

The scbus differentiates between commands and events. Parameters are sent from A to B without an acknowledgment via an event, as is the case with commands, whereby a reaction occurs which, if necessary, provides a result value. In a good case, the reaction is a two-stage STA:ACK and a STA:OK, in the case of an error, a STA:ERR or there is no reaction, which should lead to a timeout in the transmitter.

In order to receive a SCBUS value via DORA, an instance of ScBusRequest must be created with the desired data.

For the ScBus type "CMD", the scbusgateway then sends a corresponding command on the SCBUS, waits for the response and provides this as an update to the ScBusRequest instance.

As long as this instance is available, it will be maintained by the scbusgateway when new data is received on this topic. So if another bus participant changes this parameter, this can be tracked via DORA.

  

**An example:** To read the temperature of the backplane, we have to generate a ScBusRequest with the data type = <CMD>, name = BACKP.SENSOR.READ, arg = TEMP, requestState = <request>.

The scbus gateway will react with an Update, setting the values requestState = <done>, result = 42.6, and as well lastUpdate and origin.

If the value is later reported in the system with a different value, the scbusgateway would send an update of ScBusRequest accordingly. If the value is to be actively queried again, the requestState must be set back to <request>.

# DORA Model

### STRUCT ScBusRequest

File: [model/ScBusRequest.dora](http://as-git-01.jenoptik.corp/LS/sw-components/scbusgateway/-/blame/main/model//ScBusRequest.dora)

|Tag|Kind|Type|Name|Comment|
|---|---|---|---|---|
|1 [KEY]|ENUM|ScBusType|type||
|2 [KEY]||string|name||
|3 [KEY]||string|arg|must not be missing !|
|4||string|value||
|5||string|result||
|6|ENUM|RequestState|requestState||
|7||string|origin||
|8||TimePoint|lastUpdate||

### ENUM RequestState

File: [model/RequestState.dora](file:///home/pgraw/src/robot/swrepo/dev_sr390_sr395/application/general/scbusgateway/git-import/git@as-git-01.jenoptik.corp:LS/sw-components/scbusgateway/-/blame/main/model/RequestState.dora)

|Tag|Value|Name|Comment|
|---|---|---|---|
|1|0|request||
|2|1|done||
|3|2|error||
|4|3|timeout||

### ENUM ScBusType

File: [model/ScBusType.dora](file:///home/pgraw/src/robot/swrepo/dev_sr390_sr395/application/general/scbusgateway/git-import/git@as-git-01.jenoptik.corp:LS/sw-components/scbusgateway/-/blame/main/model/ScBusType.dora)

|Tag|Value|Name|Comment|
|---|---|---|---|
|1|0|CMD||
|2|1|EVT||

# How to fill ScBusRequest data to retrieve SCBUS data

|GQL Topic|type|name|arg *)|value||
|---|---|---|---|---|---|
||CMD|BACKP.SENSOR.READ|TEMP||the temperature from the backplane|
||CMD|BACKP.VERSION.READ|||the version code of the backplane|
||CMD|SCCAPTURE.MODE.GET|||the system state of the camera|
||CMD|SCCAPTURE.EEPROM.USE|EEPROM  <br>CACHE|||
||CMD|SCSTORAGE.CONFIG.GET|SYSTEM.TIME.SYNC||the current reference clock: clock, ntp, gps, pntp, ptp|
||CMD|SCSTORAGE.EEPROM.CHECK|||result values: NOT_AVAIL, EMPTY, INVALID, CHANGED, OK|
||CMD|||||
||CMD|SCIPCAM.INFO|<ID>||the info record from ipcam with index <ID>|
|||t.b.c.||||
||EVT|HW.GPS.TIME|||liefert die von sccapture per SCBUS bereitgestellte Zeit aus dem GPS Empfänger (erfährt sekündlich ein Update)|

*) the element **arg** of the structure is part of the key and must not be missing, it has to be present but empty if no value is necessary.

# State diagram
![[scbus_states.PNG]]
[

](https://confluence-sms.group.jenoptik.corp/plugins/gliffy/editor.action?inline=false&pageId=142009494&name=scbusgateway%20event%20processing&displayName=scbusgateway%20event%20processing&ceoid=142009494&key=NSP&lastPage=%2Fpages%2Fviewpage.action%3FpageId%3D142009494&macroId=)[

](https://confluence-sms.group.jenoptik.corp/plugins/gliffy/viewer.action?inline=false&pageId=142009494&name=scbusgateway%20event%20processing&version=2&ceoid=142009494&key=NSP&lastPage=%2Fpages%2Fviewpage.action%3FpageId%3D142009494&imageUrl=%2Fdownload%2Fattachments%2F142009494%2Fscbusgateway%2520event%2520processing.png%3Fversion%3D2%26modificationDate%3D1707908834823%26api%3Dv2&gonUrl=%2Fdownload%2Fattachments%2F142009494%2Fscbusgateway%2520event%2520processing%3Fapi%3Dv2%26version%3D2)

[

](https://confluence-sms.group.jenoptik.corp/display/NSP/scbusgateway#)

 

# Even / Reaction Matrix

  

|Event (incomming paket)|Actions|
|---|---|
|DORA:ScBusRequest|react if (state == request) and (type == CMD):<br><br>- send a corresponding scbus CMD message<br>- create a BusCmd object to track reaction<br>- start a timer to monitor progress (assigned to BusCmd)<br><br>react if (state == request) and (type == EVT):<br><br>- TODO (send corresponding scbus EVT)<br><br>else<br><br>- no operation|
|SCBUS:CMD|- create a BusCmd object to track reaction|
|SCBUS:STA:ACK|- no operation|
|SCBUS:STA:OK|- find correlating BusCmd, build "request => result" pair<br>- check if corresponding ScBusRequest exists<br>- add result to ScBusRequest<br>- stop timer<br>- drop BusCmd object|
|SCBUS:STA:ERR|- find correlating BusCmd, build "request => result" pair<br>- check if corresponding ScBusRequest exists<br>- push "error" to ScBusRequest<br>- stop timer<br>- drop BusCmd object|
|Timeout|- find related BusCmd<br>- find related ScBusequest<br>- push "timeout" to ScBusRequest<br>- drop BusCmd object|
|SCBUS:EVT|- check if corresponding ScBusRequest exists<br>- push data update to ScBusRequest|

  

  

[Gefällt mir](https://confluence-sms.group.jenoptik.corp/display/NSP/scbusgateway)Sei der Erste, dem dies gefällt.

- Keine Stichwörter
- [Stichwörter bearbeiten](https://confluence-sms.group.jenoptik.corp/display/NSP/scbusgateway# "Stichwörter bearbeiten ("l")")

## Kommentar

1. [![Benutzersymbol: m.steding](https://confluence-sms.group.jenoptik.corp/images/icons/profilepics/Avatar-16.png)](https://confluence-sms.group.jenoptik.corp/display/~m.steding)
    
    #### [Marius Steding](https://confluence-sms.group.jenoptik.corp/display/~m.steding) sagt:
    
    Kommunikationsverhalten von der API:
    
    **Quelle: [Graw, Peter](https://confluence-sms.group.jenoptik.corp/display/~peter.graw)**   
    _deshalb skizziere ich hier mal den Ablauf wie ich in sehe (primär den Gutfall):_
    
      
    
    - - _jenapi sendet einen ApiCfgReq, als Parameter ist dort ein ScBusRequest eingebettet._
            - _Der ApiCfgReq hat eine RequestId aber kein configResult._
            - _Der ScBusRequest hat den requestState <request>._
        - _Confman legt für diesen ApiCfgReq jetzt eine interne Management Struktur (IMS) an und sendet den ScBusRequest nach vorheriger Prüfung weiter an das scbusgateway._
        - _Das Update für den ScBusRequest wird vom confman bewertet und der IMS zugeordnet._
        - _Schließlich wird dazu eine Response an die jenAPI generiert (Update auf den ApiCfgReq)._
            - _Für die Zuordnung wird keine RequestId benötigt._
            - _Je nach Status des ScBusRequest fällt das configResult im ApiCfgReq aus._
                - _Done => success_
                - _Timeout => timeout_
                - _Error => failed_
    
      
    
    _Für ein direktes erzeugen eines ScBusRequest am confman vorbei, könnte es für die jenAPI hilfreich sein, eine RequestId in die Struktur aufzunehmen. Wäre diese aber Teil des Keys, könnte es mehrere Requests zu einem Parameter geben, was aus Sicht der SC unschön wäre. Der Status am ScBusRequest hat als State Machine einen anderen Charakter als der result Wert des ApiCfgReq, man könnte beide aber zusammenlegen. Blöd ist, dass durch unsere Zerfaserung der Module durch jede gemeinsame Nutzung einer Substruktur eine (neue) Abhängigkeit entstehen könnte._
