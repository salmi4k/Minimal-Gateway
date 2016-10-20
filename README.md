# 0: Introduction  
This guide covers how to create a minimal gateway that sends mock data to Appiot.
In this example the code is being run on an Intel Edison. It should work equally well
on any computer/device that has Java, Maven and an internet connection.

# 1: Prerequisites
### 1.1: Edison setup
Follow the instructions on this page: (there are links to other OS'
s if you're not on Windows)
https://software.intel.com/en-us/get-started-edison-windows

### 1.2: Java Cacerts
if the file /usr/lib/jvm/java-8-openjdk/jre/lib/security/cacerts is empty (or 
non-existing) you can copy the cacerts from the computer you're working on.
This is a problem Intel Edisons when doing a fresh install.

In my case they were located under
/c/program\ Files/Java/jdk1.8.0_101/jre/lib/security/cacerts

### 1.3: Maven
```sh
$ wget http://apache.mirrors.spacedump.net/maven/maven-3/3.3.9/binaries/apache-maven-3.3.9-bin.tar.gz
$ tar xf apache-maven-3.3.9-bin.tar.gz
$ export PATH=${PWD}/apache-maven-3.3.9/bin/:$PATH
```

# 2: Build the Gateway
### 2.1 Set up $SENSATION_HOME
Create sensation home folder and assign the variable $SENSATION_HOME to it's 
path.
```sh
$ mkdir ~/sensation_home
$ export SENSATION_HOME=~/sensation_home
```
Set up sensation-client.properties file
```sh
$ cd $SENSATION_HOME
$ echo "log_search_string=samplegateway" >> sensation-client.properties
$ echo "log_level=INFO" >> sensation-client.properties
$ echo "report_interval=1000" >> sensation-client.properties
$ echo "sendInterval=1000" >> sensation-client.properties
$ echo "queue_size=1000" >> sensation-client.properties
```

### 2.2 Build Gateway
###### 2.2.1 Configure gateway

```sh
$ git clone https://github.com/ApplicationPlatformForIoT/Minimal-Gateway.git
```

```sh
$ cd Minimal-Gateway/
$ nano src/main/java/com/appiot/examples/gateway/samplegateway/EdisonGatewayMock.java
```
 -  Set the variable 'deviceId' on line 32 to a unique id.

Remember the deviceId value, it will be needed in step 5.2.

###### 2.2.2 build
Compile a runnable jar file
```sh
$ mvn compile assembly:single
```

# 3 Register gateway in Appiot

### 3.1 Create location
  - Go to the Appiot page.
  - Press 'Settings' -> 'Locations'
  - Now the name of your Device Network should show up with a little cloud next to it.
Click on it and then press 'Add child'.
  - Give the location a name. E.g. 'Sweden' and then save.

Your newly created Location should now be visible in the border on the left side
of the screen.

### 3.2: Create gateway type
  - Go to the Appiot page.
  - Press 'Settings' -> 'Hardware Types'.
  - Press 'Create'.
  - Name it 'Edison Gateway' and give it a unique id (a large number preferably).
  - Press 'Save'

### 3.3: Register gateway
  - Go back to your Location Appiot page.
  - Press 'Register Gateway'.
  - Give it a unique serial number and choose the Gateway Type that you just 
  created.
  - Name you gateway something creative, like "My Edison Gateway".
  - Press 'Register'.

### 3.4: Export device

  - Go to the your newly created gateway and press 'Actions' -> 'Download Tickets'.
  (if you came from the previous section you should be on the right page).
  - Write the content of the downloaded file to $SENSATION_HOME/registrationticket.json
  on the gateway Edison.

# 4: Start the Gateway
There should now be a runnable jar file under
SampeGateway/target/EdisonGateway-0.0.1-jar-with-dependencies.jar
Run it with
```sh
$ java -jar Edison-Gateway/target/EdisonGateway-0.0.1-jar-with-dependencies.jar
```

You should now be able to see the readings from the other Edison
written in the terminal.

# 5: Register device in Appiot
### 5.1 Create device type
  - Go to 'Settings' -> 'Hardware Types'
  - Then press Device Types and then Create.
  - Name the device type "Edison Temperature" and give it a unique id 
  in the range 20000-30000.
  - Press 'Add Sensor' and a choose the type 'Temperature'.
  - Press 'Save'.

### 5.2 Register Device
For this step to work the gateway Edison device needs to be running (it should 
be printing measurements in the console)
  - Go back to the front page and press 'Register Device'.
  - Set the Serial Number to the deviceId you chose in step 2.2.1.
  - Choose your newly created Device Type "Edison Temperature".
  - Give your device a fitting name, e.g. "Edison Temperature Device".
  - Choose the gateway that you created in step 3.2.
  - Press 'Continue' and then 'Register'.

Now you should be able to go to your newly registered device on the Appiot page and see a nice 
graph of temperature measurements.

# 6: Explanation

Once we have all the tools we need, we set up a sensation home directory. This is a directory where we 
store files that are related to the communication between the backend and our gateway. It also holds 
some settings which is what we are supplying in section 2.1.

Then we download the gateway code and specify the id of our device. In this example, we are pretending
that we have a device that is doing measurements. And our pretend device needs to have an id for the backend
to make sense of it.

Then in step 3 we set up some infrastructure in Appiot to help us keep track of which gateway and device
we are working with. The last thing we do in step 3 is to download a registration ticket. This ticket is 
used by the gateway to verify that it is indeed the gateway that we created in Appiot. We place it in the
sensation home folder and name it registrationticket.json so that the gateway can find it.

Once we have the have the ticket we can start the gateway on our device. At this point however, there isn't
any device registered in Appiot with the device id that we are sending our measurements with. The data
won't get registered. So we register a device with the same id as the one that our gateway is pushing data
from.

When the device is created it will communicate to the gateway that the device has been instantiated in Appiot.
Once that is done we should start seeing the data in the Appiot web page.
