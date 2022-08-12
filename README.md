# emonPiCM
emonPi Continuous Monitoring Firmware from Robert Wall with fixes to allow RFM69 transmit with RF69 native packets in byte-wise fashion to support emonGLCD.

To test, install both .py file into /opt/openenergymonitor/emonhub/src/interfacers as the originals have not had their send functions fixed after the python3 move, nor are compatible with the new API to send in emonpiCM.

To actually send data, you should send to a MQTT topic thus :

emonhub/tx/<dest_id>/values/msg/byte1,byte2,..byteN

Example :
emonhub/tx/20/values/msg/10,13,46,53,12,08,22,-2100,5175,800,2702

Will send to a emonGLCD with id 20 and results in it dispaying the time as 13:46.53, -2100W utility reading, 5175 pv reading, 8Kw utility used, 27Kw solar generated. The fist byte (dec 10, hex 0xA) is used as an identifer byte inside the emonGLCD code so it knows this packet is for it.

This will route it's way through emonhub MQTT interfacer and out to OEM interfacer, which will send the message via the atmel/pi serial bridge from transmission using the RFM69

This is confirmed working with a RFM based emonGLCD, using the code found at 
https://github.com/alandpearson/EmonGLCD/tree/datetime/firmware/SolarPV_rfm69

More work is required to get a emonGLCD with RF12 inside to work with RF69M packets.

You will need to ensure that your emonhub.conf uses EmonHubOEMInterfacer and NOT EmonHubJeenodeInterface as follows :
```
[[RFM2Pi]]
    Type = EmonHubOEMInterfacer
    [[[init_settings]]]
        com_port = /dev/ttyAMA0
        com_baud = 38400                        # 9600 for old RFM12Pi
    [[[runtimesettings]]]
        pubchannels = ToEmonCMS,
        subchannels = ToRFM12, emonCMS

        group = 210
        frequency = 433
        baseid = 5                             # emonPi / emonBase nodeID
        quiet = true                            # Report incomplete RF packets (no implemented on emonPi)
        calibration = 230V                      # (UK/EU: 230V, US: 110V)
#        interval =  20                       # Interval to transmit time to emonGLCD (seconds)
```

Additionally, here is the emonhub.conf definition required for emonGLCD :

```
[[20]]
    nodename = emonglcd
    firmware =V1_1
    hardware = emonglcd
    [[[rx]]]
     names = temperature,
     datacodes = h
     scales = 0.01,1
     units = c
  [[[tx]]]
     names = type,hour,minute,second,day,month,year,utilityW,solarW,utilityKwh,solarKwh
     datacodes =b,b,b,b,b,b,b,h,h,H,H
     units = nodeId,type,h,min,sec,day,mon,year,W,W,kwh,kwh
     
```     

