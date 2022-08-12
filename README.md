# emonPiCM
emonPi Continuous Monitoring Firmware from Robert Wall with fixes to allow RF TX in byte wise fashion to support emonGLCD

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


