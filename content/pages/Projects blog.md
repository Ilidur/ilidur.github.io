---
title: Projects blog
tags:
categories:
date: 2023-10-03
lastMod: 2023-10-06
---
# Focus stacking rig #projects

  + Brief: Reuse 3D printer components to learn how to make a basic mechatronic system

  + MVP: A one axis slide for photography

  + Requires:

    + Control board

    + alu frame

  + ## Progress

    + ## 2023-10-05

      + On a suggestion from a hackspace member I tried scanning the camera's connection with wireshark. While I couldn't connect both the laptop and the phone with the application at the same time so I can sniff a proven app, it did volunteer SSDP packages which I wasn't able to request myself. From there I got the url for the device descriptor. Next I need to get the right api calls.
```
HTTP/1.1 200 OK
Accept-Range: none
Content-Length: 2357
Content-Type: text/xml; charset="utf-8"
Connection: close
Date: Thu, 01 Jan 1970 00:18:41 GMT
Server: UPnP/1.0 MINT-X/1.8.1

<?xml version="1.0"?><root xmlns="urn:schemas-upnp-org:device-1-0">
  <specVersion>
    <major>1</major>
    <minor>0</minor>
  </specVersion>
  <device>
    <deviceType>urn:schemas-upnp-org:device:Basic:1</deviceType>
    <friendlyName>ILCE-6000_2</friendlyName>
    <manufacturer>Sony Corporation</manufacturer>
    <manufacturerURL>http://www.sony.net/</manufacturerURL>
    <modelDescription>SonyDigitalMediaServer</modelDescription>
    <modelName>SonyImagingDevice</modelName>
    <UDN>uuid:000000001000-1010-8000-9AF1702626E7</UDN>
    <serviceList>
      <service>
        <serviceType>urn:schemas-sony-com:service:ScalarWebAPI:1</serviceType>
        <serviceId>urn:schemas-sony-com:serviceId:ScalarWebAPI</serviceId>
        <SCPDURL/>
        <controlURL/>
        <eventSubURL/>
      </service>
      <service>
        <serviceType>urn:schemas-sony-com:service:DigitalImaging:1</serviceType>
        <serviceId>urn:schemas-sony-com:serviceId:DigitalImaging</serviceId>
        <SCPDURL>/DigitalImagingDesc.xml</SCPDURL>
        <controlURL>/upnp/control/DigitalImaging</controlURL>
        <eventSubURL></eventSubURL>
      </service>
    </serviceList>
    <av:X_ScalarWebAPI_DeviceInfo xmlns:av="urn:schemas-sony-com:av">
      <av:X_ScalarWebAPI_Version>1.0</av:X_ScalarWebAPI_Version>
      <av:X_ScalarWebAPI_ServiceList>
        <av:X_ScalarWebAPI_Service>
          <av:X_ScalarWebAPI_ServiceType>guide</av:X_ScalarWebAPI_ServiceType>
          <av:X_ScalarWebAPI_ActionList_URL>http://192.168.122.1:8080/sony</av:X_ScalarWebAPI_ActionList_URL>
          <av:X_ScalarWebAPI_AccessType/>
        </av:X_ScalarWebAPI_Service>
        <av:X_ScalarWebAPI_Service>
          <av:X_ScalarWebAPI_ServiceType>accessControl</av:X_ScalarWebAPI_ServiceType>
          <av:X_ScalarWebAPI_ActionList_URL>http://192.168.122.1:8080/sony</av:X_ScalarWebAPI_ActionList_URL>
          <av:X_ScalarWebAPI_AccessType/>
        </av:X_ScalarWebAPI_Service>
        <av:X_ScalarWebAPI_Service>
          <av:X_ScalarWebAPI_ServiceType>camera</av:X_ScalarWebAPI_ServiceType>
          <av:X_ScalarWebAPI_ActionList_URL>http://192.168.122.1:8080/sony</av:X_ScalarWebAPI_ActionList_URL>
          <av:X_ScalarWebAPI_AccessType/>
        </av:X_ScalarWebAPI_Service>

      </av:X_ScalarWebAPI_ServiceList>
    </av:X_ScalarWebAPI_DeviceInfo>
  </device>
</root>
```

      + This isn't very useful other than confirming what was already known from the timelapse app.

      + So I suspect my POST is malformed. I switched to reading the API and the doc recommends I call `getApplicationInfo` which didn't work.

      + Trying to call using cURL resulted in a successful api call! 
```
curl -X POST  \
    -d '{"method": "getApplicationInfo",
    "params": [],                        
    "id": 1,  
    "version": "1.0"}' \
    http://192.168.122.1:8080/sony/camera
{"result":["Smart Remote Control SR\/4.10 __SAK__","2.1.4"],"id":1}%      
 ```

      + The docs say most cameras don't need this but mine does:
```
curl -v -H "Content-Type: application/json" -X POST  \
    -d '{"method": "startRecMode",
    "params": [],
    "id": 1,
    "version": "1.0"}' \
    http://192.168.122.1:8080/sony/camera

Note: Unnecessary use of -X or --request, POST is already inferred.
*   Trying 192.168.122.1:8080...
* Connected to 192.168.122.1 (192.168.122.1) port 8080 (#0)
> POST /sony/camera HTTP/1.1
> Host: 192.168.122.1:8080
> User-Agent: curl/8.1.2
> Accept: */*
> Content-Type: application/json
> Content-Length: 105
> 
< HTTP/1.1 200 OK
< Connection: close
< Content-Length: 21
< Content-Type: application/json
< 
* Closing connection 0
{"result":[0],"id":1}%                                                              
```

      + I'm now in live view!

      + I also wanted to test which header values are mandatory. So I reformulated the request to contain all the data in one line, then had to find a way to send raw TCP requests.
```
curl -v -H "Content-Type: application/json" -X POST  \          
    -d '{"method": "startRecMode", "params": [], "id": 1, "version": "1.0"}' \
    http://192.168.122.1:8080/sony/camera

Note: Unnecessary use of -X or --request, POST is already inferred.
*   Trying 192.168.122.1:8080...
* Connected to 192.168.122.1 (192.168.122.1) port 8080 (#0)
> POST /sony/camera HTTP/1.1
> Host: 192.168.122.1:8080
> User-Agent: curl/8.1.2
> Accept: */*
> Content-Type: application/json
> Content-Length: 67
> 
< HTTP/1.1 200 OK
< Connection: close
< Content-Length: 21
< Content-Type: application/json
< 
* Closing connection 0
{"result":[0],"id":1}%
```

      + I found a post saying I can do it by using telnet:
```
curl telnet://192.168.122.1:8080 <<< 'POST /sony/camera HTTP/1.1
Host: 192.168.122.1:8080
User-Agent: curl/8.1.2
Content-Length: 67

{"method": "startRecMode", "params": [], "id": 1, "version": "1.0"}'
HTTP/1.1 200 OK
Connection: close
Content-Length: 21
Content-Type: application/json

{"result":[0],"id":1}%             
```

      + After a bit of fiddling, success!
```
I (10270) camera rig: got ip:192.168.122.131
I (10270) esp_netif_handlers: sta ip: 192.168.122.131, mask: 255.255.255.0, gw: 192.168.122.1
I (12690) camera rig: ... allocated socket
I (12690) camera rig: ... connected
I (12700) camera rig: ... socket send success
I (12700) camera rig: ... set socket receiving timeout success
HTTP/1.1 200 OK
Connection: close
Content-Length: 21
Content-Type: application/json

{"result":[0],"id":1}I (12940) camera rig: ... done reading from socket. Last read return=0 errno=128.
```
The trick was a `\r\n` at the end

      + I refactored the code to call two commands, one to set up the camera and the other to take the picture with `actTakePicture`

      + Still waiting on motor controllers so its a good time to pause the project for a few days and rest.

    + ## 2023-10-04

      + ### Building the combined app

        + I took 3 projects (i2c_oled, fast_scan and http_request) and combined them

        + I had to configure the platform, then build. This failed as my app was too big. I changed that in the `menuconfig` in Partition table -> Single Factory App Large

        + OK! so i have the following functionality:

          + Display information

        + Connected to wifi and got ip!
```
I (4268) camera rig: got ip:192.168.0.57
I (4268) esp_netif_handlers: sta ip: 192.168.0.57, mask: 255.255.255.0, gw: 192.168.0.1
```

        + got http request!
```
I (16312) camera rig: ... connected
I (16322) camera rig: ... socket send success
I (16322) camera rig: ... set socket receiving timeout success
HTTP/1.0 404 Not Found
Cache-Control: max-age=604800
Content-Type: text/html; charset=UTF-8
Date: Wed, 04 Oct 2023 14:28:18 GMT
Expires: Wed, 11 Oct 2023 14:28:18 GMT
Server: EOS (vny/044F)
Vary: Accept-Encoding
Content-Length: 1256
Connection: close

<!doctype html>
<html>
<head>
    <title>Example Domain</title>

    <meta charset="utf-8" />
    <meta http-equiv="Content-type" content="text/html; charset=utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <style type="text/css">
    body {
        background-color: #f0f0f2;
        margin: 0;
        padding: 0;
        font-family: -apple-system, system-ui, BlinkMacSystemFont, "Segoe UI", "Open Sans", "Helvetica Neue", Helvetica, Arial, sans-serif;
        
    }
    div {
        width: 600px;
        margin: 5em auto;
        padding: 2em;
        background-color: #fdfdff;
        border-radius: 0.5em;
        box-shadow: 2px 3px 7px 2px rgba(0,0,0,0.02);
    }
    a:link, a:visited {
        color: #38488f;
        text-decoration: none;
    }
    @media (max-width: 700px) {
        div {
            margin: 0 auto;
            width: auto;
        }
    }
    </style>    
</head>

<body>
<div>
    <h1>Example Domain</h1>
    <p>This domain is for use in illustrative examples in documents. You may use this
    domain in literature without prior coordination or asking for permission.</p>
    <p><a href="https://www.iana.org/domains/example">More information...</a></p>
</div>
</body>
</html>
I (16652) camera rig: ... done reading from socket. Last read return=0 errno=128.

```

        + The camera uses the SSDP service discovery process. This requires a UDP send/recv set up on top of the HTTP requests. The send is on a multicast address. After looking at some examples and stressing out, I remembered an old android app I used to work on https://github.com/ThibaudM/timelapse-sony that had to go through the same process. Looking at their implementation they offload the SSDP service to another app and store the addresses in a list. So I retrieved my camera's IP from there.

        + 

    + ## 2023-10-03

      + Goal for today was to set up the dev environment for the controller

        + The controller I'm using is the Seeed Xiao ESP32C3

        + The dev environment is a linux VM inside UTM VM environment. This posed a challenge in connecting the ESP32C3 because I had to pipe the USB. This was a setting in the Input tab of the VM config. Then I have to manually connect the USB after each load from the USB dropdown of the VM.

        + Getting examples to work was not straightforward. I was using the SSD1306 OLED which had an IIC address on the back selected with resistor pins, but the OLED had a completely different address of 3C.

        + Success!

        + ![IMG_20231003_233005496.jpg](/assets/img_20231003_233005496_1696372838944_0.jpg)

      + Next up tomorrow is to set up a wifi connection example

    + ### 2023-10-02

      + Disassembled TronXY S5 3d printer and scavenged parts

        + ![IMG_20231002_221639851.jpg](/assets/img_20231002_221639851_1696332184660_0.jpg)

      + Ordered some motor controllers

      + Plan for software/control

        + Use ESP32C3 to control both motor and camera via wifi.

        + Use Sony QX1 that I have to keep it light and trigger commands via wifi api from the rig controller.