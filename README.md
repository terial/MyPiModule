###########################################################################################
#TITLE# DRONE MyPiDrone Project Kev&Phil : Copter QUAD Project 1 and Project 2
- Project 1 : TAROT 650 Copter QUAD with Raspberry PI2 & Navio+ controler (status finished)
- Project 2 : Racer 30cm Copter QUAD with Raspberry PI3 & Navio2 controler (under construction)
- raspian Kernel 4.4.y 
- www.MyPiDrone.com MyPiDrone kev&phil Project
- https://github.com/MyPiDrone/MyPiModule 
- Version 2.3 : September 2016 
- https://github.com/MyPiDrone/MyPiModule/blob/master/mavproxy_MyPiModule.py
- README here: https://github.com/MyPiDrone/MyPiModule/blob/master/README.md
###########################################################################################

 Changelog :

      version 2.4 : minor changes
          
       - Added : Python Replay Video Thread in MyPiModule
       - Added : python psutil Loadavg CPU% IOWAIT%

#########################################################################################

      version 2.3 : Overlaying telemetry text on video before Wifibroadcast

       - Added picamera python module inside mavproxy_MyPiModule.py to broadcast video and telemetry text 
         overlayed before transmit with tx Wifibroadcasting (instead of raspivid) (1)
         - See chapter : "Overlaying text on the output" here : https://picamera.readthedocs.io/en/release-1.12/recipes1.html#overlaying-text-on-the-output
         - Now mavproxy_MyPiModule.py is controling video and photos snapshot with telemetry text overlayed (255 chars max) :
            - Used picamera python module instead of raspivid (1)
       - Video Wifibroadcast ON/OFF (default ON) : RC6 LOW (OFF) or RC6 HIGH (ON)
       - Video recording in h264 format on SD card : automatic start/stop with Drone Armed/Disarmed
       - Photos JPEG on SD card (one snapshot per minute) : automatic start/stop with Drone Armed/Disarmed
       - Videos and photos are stored in /root/fpv/videos directory
       - Viewing last Video with Wifibroadcast (redo video) : STANDBY + DISARMED, LOW YAW (RC4) and PITCH HIGH (RC2)
         (Warning : during redo video the MyPiModule is paused)
        
          - (1) Here a python sample with a named pipe MyPiCamera_sample.py and command execution with  tx :
              - mkfifo /tmp/MyPiCamera.pipein
              - MyPiCamera_sample.py | tee $VIDEO | $WifiBroadcast_TX -p $PORT -b $BLOCK_SIZE -r $FECS -f $PACKET_LENGTH $WLAN 1>/dev/null 2>&1 &
              - sleep 3
              - echo 'My telemetry text' > /tmp/MyPiCamera.pipein

          - (2) manage_video.sh and myvideo.service not used anymore

###########################################################################################

      version 2.2 :

        - add myrtl function to set RTL mode (Return To Launch)
        - add mystabilize function to set STABILIZE mode
        - set RTL mode when RC8 range HIGH_MARK to HIGH_MARK+100
        - set STABILIZE mode when RC8 range LOW_MARK-100 to LOW_MARK
        - support raspian Kernel 4.4.y emlid img for RIP2/RPI3 NAVIO+/NAVIO2

###########################################################################################

        - MyPiDrone_drone_install.sh                                     DRONE : to install MyPiModule
        - MyPiDrone_gcs_install.sh                                       GCS   : to install start_rx and wifiap
        - MyPiModule_build_and_git_update.sh                             DRONE : build MyPiModule and update github
        - mavproxy_MyPiModule.py                                         DRONE : module MAVProxy
        - rc.local                                                       DRONE : exec StartArduCopter-quad.sh
        - ArduCopter-quad.service                                        DRONE : systemd call /usr/local/bin/start_ArduCopter-quad.sh
        - mavproxy.service                                               DRONE : systemd call /usr/local/bin/start_MAVProxy_MyPiModule.sh
        - wifiap.service                                                 GCS   : systemd call /usr/local/bin/start_MAVProxy_MyPiModule.sh
        - maanage_network.sh                                             DRONE : ifup/ifdown/status wlan0 called by MyPiModule 
        - start_tx_and_recording_with_picamera_video_input.sh            DRONE : start Video Wifibroadcast
        - start_tx_and_recording_with_raspivid_video_input_on_wifiap.sh  DRONE : start Video Broadcast over Wifi AP : Beta test
        - start_ap.sh                                                    GCS   : start Wifi AP on GCS
        - start_rx.sh                                                    GCS   : rx 2.3ghz and display Video on GCS
        - start_rx_and_broadcast_over_ap.sh                              GCS   : rx 2.3Ghz and streaming video over AP to myMediaCodecPlayer-for-FPV Android application (socat UDP port 5000 with CSL-300 5Ghz)
        - start_rx_and_broadcast_over_ap_with_diversity.sh               GCS   : start_rx_and_broadcast_over_ap.sh + diversity (2 X TPLINK)
        - hostapd.conf                                                   GCS   : Wifi Access Point configuration
        - dnsmasq.conf                                                   GCS   : dnsmasq configuration
        - show_modules.sh                                                DRONE : tools show params modules

###########################################################################################


The Raspberry PI2 consumes between 1A and 2A according to the WiFi is enabled or not and MAVproxy module (MyPiModule) 
will therefore help preserve the battery and control various functions of RPI2 (AP Wifi, Video Wifibroadcast, shutdown, reboot)
 from the radio (Taranis x9d opentx in our project)

The main functions of MyPiModule (MAVProxy module):

* my_battery_check function:

    - Preserve the minimum battery voltage if the drone is in STANDBY mode and if the engines are not armed.
      Under these conditions with a weak battery module executes the stop Raspberry PI 2 (init 0)

* my_rc_check function:

    - Run a shutdown of RPI2 from the radio:
        - Conditions: STANDBY + DISARMED
        - On the radio: LOW YAW (RC4) and ROLL HIGH (RC1)
    - Perform a reboot of RPI2 from the radio:
        - Conditions: STANDBY + DISARMED
        - On the radio: LOW YAW (RC4) and ROLL LOW (RC1)
    - enable / disable the wireless network from the radio wlan0:
        - RC8 LOW : wlan0 DOWN and SINGLE MODE OFF
        - RC8 range LOW_MARK-100 to LOW_MARK : set mode STABILIZE *NEW*
        - RC8 MIDDLE : SINGLE MODE ON
        - RC8 range HIGH_MARK to HIGH_MARK+100 : set mode RTL
        - RC8 HIGH : ifup wlan0 
    - enable / disable the video on wifibroadcast wlan1 from the radio:
        - RC6 LOW (also used to tilt the camera left): Video wifibroadcast ON
        - RC6 HIGH (also used to tilt the camera to right): Video wifibroadcast OFF
    - Run a Redo Video WBC with the radio:
        - Conditions: STANDBY + DISARMED
        - On the radio: LOW YAW (RC4) and PITCH HIGH (RC2)

==> Logs: /var/log/mavproxy_MyPiModule.log


* Parameters for functions and telemetry:
  (Change in /.mavinit.scr or /root/.mavinit.scr if necessary).

    - mydelayinit: : 30 seconds delay to reboot or shutdown to allow cancel.
    - myminremain : 10% (low battery remaining mark).
    - myminvolt : 10V (battery low voltage mark).
    - mytimebat : 5sec interval data mesurement of the battery voltage.
    - mytimerc : 4sec interval data mesurement of the radio channels.
    - mytimeTText : 0.5sec telemetry text refresh Interval Time
    - myrcvideo : channel to control video WBC on / off, default 6.
    - myrcyaw and myrcroll and myrcpitch : channels to control the shutdown, reboot, redo video, default 4, 1, 2.
    - myrcnet : channel to control wlan0 on/off, default 8
    - mychanneltx : Wifibroadcast 2.4Ghz (CH 11) or 2.3Ghz (CH -19)
    - mydebug : True or False
    - myinterfaceadmin : wlan0
    - myinterfacetx : wlan1
    - mylog : /var/log/mavproxy_MyPiModule.log
    - mylogverbose : True or False
    - mypipeout : /tmp/MyPiCamera.pipeout : see start_tx_and_recording_with_raspivid_video_input.sh
    - myseqinit : 15sec before init var and start polling
    - myseqpoll : 10sec polling interval to control network status, video recording, flightmode.
    - myvideopath : /root/fpv/videos

* Video functions: ** NEW in 2.3 **

    - Video Wifibroadcast ON/OFF (default ON) : RC6 LOW (OFF) or RC6 HIGH (ON)
    - Video recording in h264 format on SD card : automatic start/stop with Drone Armed/Disarmed
    - Photos JPEG on SD card (one snapshot per minute) : automatic start/stop with Drone Armed/Disarmed
    - Videos and photos are stored in /root/fpv/videos directory
    - Viewing last Video with Wifibroadcast (redo video) : STANDBY + DISARMED, LOW YAW (RC4) and PITCH HIGH (RC2) 
      (Warning : during redo video the MyPiModule is paused)

* Console mode functions:

    - mybat       : battery status
    - myshut      : execute a shutdown (to cancel shutdown execute a new request in time delay of 30 secondes)
    - myreboot    : execute a reboot (to cancel reboot execute a new request in time delay of 30 secondes)
    - myrtl       : set RTL mode
    - mystabilize : set STABILIZE mode

    Shutdown and reboot may be canceled : execute a new command before delay (30sec) to do that .

    The STATUSTEXT progress message is displayed on the screen 2 on the telemetry radio.

    A YAW MAX 3 seconds (ARMED) also cancels all requests for shutdown or reboot in progress.

* Here the module test procedure:

    -1- Install MAVproxy with git clone https://github.com/Dronecode/MAVProxy.git
    
    -2- Create your MAVProxy Module/modules/mavproxy_MyPiModule.py module available here: git clone https://github.com/MyPiDrone/MyPiModule
    
    -3- Execute python setup.py build install
    
    -4- Execute ArduPilot:
      /usr/bin/ArduCopter-quad -A /dev/ttyAMA0 -C udp:127.0.0.1:14550
      
    -5- Execute MAVProxy (in console mode remove --deamon) /usr/local/bin/mavproxy.py --master=udp:127.0.0.1:14550 --quadcopter --out=/dev/ttyUSB0,57600  --default-modules='MyPiModule,mode' --show-errors --daemon 

     You can also load the module when MAVProxy is already started with the command module load MyPiModule or module reload MyPiModule
     You can add this self-loading in file /.mavinit.scr or /root/.mavinit.scr by adding the line module load MyPiModule
     MAVproxy load ten modules and --default-modules='MyPiModule' option allows load only the list of desired modules (comma separator) to consume less CPU on RPI2 and therefore less battery.

    ==> Observe the behavior in the log file: tail -f /var/log/mavproxy_MyPiModule.log


*** the end ***

