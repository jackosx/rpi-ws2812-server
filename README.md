#Rpi-ws2812-server
This is a small program for driving the WS281x (a.k.a. [NeoPixel](https://www.sparkfun.com/products/12999)) LEDs from a webserver, command line, text file, Python,... using the Raspberry Pi. It uses the rpi_ws281x PWM driver code from jgarff ([https://github.com/jgarff/rpi_ws281x](https://github.com/jgarff/rpi_ws281x)). The LEDs can be controlled by sending text commands to a tcp socket. These commands can be generated by a webserver script, android app,... It's also possible to control the leds directly from the command line or by loading text file containing some predefined color patterns.

#Installation
On the raspberry you open a terminal window and type following commands:
* sudo apt-get update
* sudo apt-get install gcc make git
* git clone https://github.com/tom-2015/rpi-ws2812-server.git
* cd rpi-ws2812-server
* make
* sudo chmod +x ws2812svr

#Testing
Hook up your LEDs to the PWM output of the Raspberry Pi and start the program:

* sudo ./ws2812svr

Now first initialize the driver code from jgarff by typing 'setup'.
On the following line you must **replace 10** by the number of leds you have attached!.

* setup channel_1_count=10

Now you can type commands to change the color of the leds.
For example make them all red:

* fill 1,FF0000
* render

#Supported commands
Here is a list of commands you can type or send to the program. All commands have optional comma seperated parameters. The parameters must be in the correct order except for the 'setup' command.

* Setup command must be called everytime the program is started:
```
setup  
   channel_1_count=1,        #number of leds on channel 1  
   freq=800000,               #frequency used to talk to the leds  
   dma=5,                     #dma channel  
   channels=1,                #number of led strings / PWM outputs / channels used  
   channel_1_gpio=18,         #GPIO number of channel 1  
   channel_1_invert=0,        #Invert the output of channnel 1 (in case you are using an 3.3V->5V level shifter)  
   channel_1_brightness=255,  #default brightness of the leds (can be changed with brightness command)  
   channel_2_count=0,         #number of leds in channel 2  
   channel_2_gpio=0,          #GPIO number for channel 2  
   channel_2_invert=0,        #invert the output of channel 2  
   channel_2_brightness=255   #brightness for channel 2  

Example:  
   setup channel_1_count=10
```

* Render command sends the internal buffer to all leds
```
render   
    <channel>,          #send the internal color buffer to all the LEDS of <channel> default is 1  
    <start>,            #before render change the color of led(s) beginning at <start> (0=led 1)  
    <RRGGBBRRGGBB...>   #color to change the led at start Red+green+blue (no default)  
```

* rotate command moves all color values of 1 channel
```
rotate  
    <channel>,         #channel to rotate (default 1)  
    <places>,          #number of places to move each color value (default 1)  
    <direction>,       #direction (0 or 1) for forward and backwards rotating (default 0)  
    <RRGGBB>           #first led(s) get this color instead of the color of the last led  
```

* rainbow command creates rainbows or gradient fills
```
rainbow  
    <channel>,         #channel to fill with a gradient/rainbow (default 1)  
    <count>,           #number of times to repeat the rainbow in the channel (default 1)  
    <start_color>,     #color to start with value from 0-255 where 0 is red and 255 pink (default is 0)  
    <end_color>        #color to end with value from 0-255 where 0 is red and 255 pink (default 255)  
```

* fill command fills number of leds with a color value
```
fill  
    <channel>,          #channel to fill leds with color (default 1)  
    <RRGGBB>,           #color to fill (default FF0000)  
    <start>,            #at which led should we start (default is 0)  
    <len>               #number of leds to fill with the given color after start (default all leds)  
```

* brightness command changes the brightness of the leds without affecting the color value
```
brightness  
    <channel>,          #channel to change brightness (default 1)  
    <brightness>        #brightness 0-255 (default 255)  
```

* delay command waits for number of milliseconds
```
delay  
    <milliseconds>      #enter number of milliseconds to wait	  
```
	
#Special keywords
You can add `do ... loop` to repeat commands when using a file or TCP connection.

For example the commands between `do` and `loop` will be executed 10 times:
```
do   
   <enter commands here to repeat>    
loop 10
```
(Endless loops can be made by removing the '10')

For `do ... loop` to work from a TCP connection we must start a new thread. 
This thread will continue to execute the commands when the client disconnects from the TCP/IP connection. 
The thread will automatically stop executing the next time the client reconnects (ideal for webservers).

For example:
```
thread_start   
   do  
      rotate 1,1,2  
     render  
     delay 200  
  loop  
thread_stop  
<client must close connection now>   
```

#PHP example
First start the server program:

* sudo ./ws2812svr -tcp

Then run the php code from the webserver:

```PHP
//create a rainbow for 10 leds on channel 1:  
send_to_leds("setup channel_1_count=10;rainbow;brightness 1,32;");  
function send_to_leds ($data){  
   $sock = fsockopen("127.0.0.1", 9999);  
   fwrite($sock, $data);  
   fclose($sock);  
}
```

#Command line parameters
* sudo ./ws2812svr -tcp 9999  
  Listens for clients to connect to port 9999 (default).
* sudo ./ws2812svr -f text_file.txt  
  Loads commands from text_file.txt.
* sudo ./ws2812svr -p /dev/ws281x  
  Creates a file called `/dev/ws281x` where you can write you commands to with any other programming language (do-loop not supported here).