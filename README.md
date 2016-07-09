# Introduction
TC-in-a-box is intended to be an environment to perform functional testing of code for TC. TA3 services are provided, and stubs for TA1 and TA2 services will eventually be available as test producers or consumers.  We provide the steps for a manual installation, and we also provide instructions for creating a local VM using vagrant.

# Suggested Hardware (or VM specs)
1. At least 2 CPUs
2. At least 4 GB of memory

# Tested environments
The easy installation has been tested on both Ubuntu 14.04, Mac OS X and Windows 7.

# Easy Installation into VM (Linux or Mac)
1. Install VirtualBox
2. Install Vagrant (version 1.5 or higher is recommended, see https://www.vagrantup.com/downloads.html)
3. Clone this repo to get the TC-in-a-box Vagrantfile, and navigate into the repo top level directory.  You must be in this directory for the below steps to work.
4. Run ```./pre-vagrant.sh``` to run preparation steps.  This should attempt to use the ssh for authentication based on the public key that was provided for use with the TA3 Gitlab.
5. Run ```vagrant up``` to bring up the VM.
6. Run ```vagrant ssh``` to log into the VM.

   For windows, "vagrant ssh" will not work directly, so it is recommended you use another ssh client, either openssh with cygwin or a native windows client likt PuTTY.  SSH to 127.0.0.1, port 2222, user: vagrant, password: vagrant

## Detecting if provisioning succeeded
Provisioning is successful unless you see the following error message:
```
The SSH command responded with a non-zero exit status. Vagrant
assumes that this means the command failed. The output for this command
should be in the log above. Please read the output to determine what
went wrong.
```

We have configured our provisioning scripts to immediately fail if any individual command within our scripts fails.  If you do see the above message, then right above that there should be a hint as to what caused the issue.  Please provide this information in a bug report if you are seeing a reproducible error, and we will work with you to try and resolve it.

## Using Vagrant
More information on how to use vagrant can be found at https://docs.vagrantup.com/v2/getting-started/index.html.  The basics are as follows:
 * Provision a machine, or turn it on if it is already provisioned: `vagrant up`
 * Log into the machine: `vagrant ssh` (except on Windows)
 * Turn off the machine: `vagrant halt`
 * Delete the machine: `vagrant destroy`

These machines are intended to be ephemeral and reproducible.  It is common to destroy and recreate the machines to ensure a clean environment.

## Getting data into the environment
The top level directory of tc-in-a-box is mounted onto the tc-in-a-box machine at /vagrant.  You can easily import files or work within repos by using that directory.

## Working around provisioning errors
If you encounter a provisioning error, it should be immediately apparent.  When an error occurs, please file an issue in this project's issue tracker.  If you think the error does not affect you, then you can try to force provision, which will ignore errors.  When doing this, uncomment the line in the `Vagrantfile` that says:

```ruby
    #s.args = "-f"
```

You may also want to run the following when doing your provisioning:

```sh
vagrant up --no-color | tee vagrant.log
```

This should give you a legible log, and you can grep through it for the word `force-provision` to see which errors were ignored.

# Manual Installation (Linux-based)
These instructions are provided for two reasons:
 * They are a rough guideline for installing things on any system.
 * Some people may prefer not to run in a VM.

However, be aware that these instructions will likely not work on any arbitrary system.  They have been successfully tested on Ubuntu 14.04, but they do not work as is on OS X.  If you would like to install on a system running something other than Linux, please get in touch with TA3 team members for assistance.

## 1. Install Kafka binaries
```sh
cd /opt
KAFKA_VER=0.9.0.0
SCALA_VER=2.11
sudo wget http://apache.mirrors.pair.com/kafka/${KAFKA_VER}/kafka_${SCALA_VER}-${KAFKA_VER}.tgz
sudo tar xvzf kafka_${SCALA_VER}-${KAFKA_VER}.tgz
```

## 2. Install git and configure for TA3 gitlab
If it is not installed already, install git in your environment.  Also, make sure you have provided TA3 with the necessary credentials to access their git repos.

## 3. Install TA3 APIs
Navigate to the location where you want to clone all of your repos.  These instructions assume you are in that location.  Currently, integration tests assume that these repos are installed in one's home directory.

### Java
1. Install requirements
   * Java JRE and JDK
   * Maven

2. Clone schema project
   ```sh
   git clone https://git.tc.bbn.com/bbn/ta3-serialization-schema.git
   ```

3. Generate and install schema Java bindings
   ```sh
   cd ta3-serialization-schema
   mvn clean exec:java
   mvn install
   cd ..
   ```

4. Clone Java API bindings project
   ```sh
   git clone https://git.tc.bbn.com/bbn/ta3-api-bindings-java.git
   ```

5. Install Avro API Java bindings
   ```sh
   cd ta3-api-bindings-java/tc-bbn-avro
   mvn clean install
   mvn assembly:assembly
   cd ../..
   ```

6. Install Kafka API Java bindings
   ```sh
   cd ta3-api-bindings-java/tc-bbn-kafka
   mvn clean install
   mvn assembly:assembly
   cd ../..
   ```

7. (Optiponal) Install [TA1 Ingestor](#4-ta1-ingestors-optional)

### Python2 or Python3
1. Install requirements
   * Python (2 and/or 3)
   * Python development headers (2 and/or 3)
   * Python setuptools (2 and/or 3)

1. Clone Python API bindings project
   ```sh
   git clone https://git.tc.bbn.com/bbn/ta3-api-bindings-python.git
   ```

2. Install API bindings
   ```sh
   cd ta3-api-bindings-python
   # Be sure to invoke the python version you want to use for development.
   sudo python setup.py install
   cd ..
   ```

   Note that you may see an error like this:

   ```sh
   #include <librdkafka/rdkafka.h>
                              ^

   compilation terminated.
   ---------------
   INFO: Failed to build rdkafka extension:
   command 'x86_64-linux-gnu-gcc' failed with exit status 1
   INFO: will now attempt setup without extension.
   ---------------
   ```

   This error can be safely ignored, as we do not currently install the librdkafka extension as part of our setup instructions.

### C
This section can be skipped for now, as the API is not totally ready.  For those who are curious on trying out the limited pieces that are there, see the [top level project README](https://git.tc.bbn.com/bbn/ta3-api-bindings-c).

1. Install requirements
   * Avro 1.8.0 C bindings
   * libjansson-dev
   * librdkafka
   * log4c
   * pkg-config

2. Clone C API bindings project
   ```sh
   git clone https://git.tc.bbn.com/bbn/ta3-api-bindings-c.git
   ```

3. Install API bindings  
   Coming soon!

## 4. TA1 Ingestors - Optional

### MIT/ClearScope: Clone ClearScope Data Ingestor project
   ```sh
   git clone https://git.tc.bbn.com/bbn/ta1-integration-clearscope.git
   ```
   Install ClearScope Data Ingestor
   ```sh
   cd ta1-integration-clearscope/
   mvn clean install
   mvn assembly:assembly
   cd ../..
   ```

## 5. TA3 Management Tools
### 5.1. Install the latest saltstack
See https://docs.saltstack.com/en/latest/topics/installation/
N.B. Installing from stock Ubuntu repos is known to **not** work.

For Ubuntu installations, we have been installing the following packages:
 * salt-master
 * salt-minion
 * salt-ssh
 * salt-syndic

### 5.2. Clone the tc-salt-services project
```sh
git clone https://git.tc.bbn.com/bbn/tc-salt-services.git
```

### 5.3. Edit config files for tc-in-a-box base experiment
* cd tc-salt-services
* Set `vmfile` environment variable with ```export vmfile=tc.txt```.  You may also want to add this command to `~/.profile` for future logins.
* Create tc.txt if it does not exist already.  More information on the target format of this file can be found at https://git.tc.bbn.com/bbn/tc-salt-services/wikis/home#experiment-assignment-file.  Populate the file with the following, updating <host_ip> and <hostname> according to your system:

```
# @tc@ VM IP    VM Hostname     Host    ZK ID   Kafka   KafkaManager    TA
<host_ip>       <hostname>      tc      1       1       -               3
```

### 5.4. Set up Salt master and minion
You will need to modify `config.env` for your master setup.
  * Update masterip to your host's IP address (not the loopback IP)
  * Update vmuser to your username
  * Update privkey to the private key you wish to use for ssh connections
  * Update publickey to the public key you wish to use for ssh connections --  note that this key must be in your user's `~/.ssh/authorized_keys` file
  * Update zookeeper clientport to 2181

You also need to create the directory where the kafka logs are:

```sh
sudo mkdir -p /opt/kafka/kafka-logs
```

See https://git.tc.bbn.com/bbn/tc-salt-services/wikis/NewSaltMaster for the remaining instructions for the salt master.  In step 3, make sure you chown to your own user and group instead of the bbn user and group.  If you are not sure which group to use for your system, just specifying the user should be OK.  Also, make sure to run the sed commands listed in step 6.

You will also have the master link to the latest ```test_producer_consumer.py``` file from ```/srv/salt/_modules```.  Note that an older version of the file is likely already there, and it can be overwritten.

For the salt minion, you need to have a few files in specific locations in order for tc-salt-services to be able to drive the integration tests.  You will need to create symlinks to the following files from `/opt/starc` on your machine:
 * The latest `working kafkaclients-1.0-SNAPSHOT-jar-with-dependencies.jar` jar file
 * The latest working `ta3-api-bindings-python` directory
 * The latest working `ta3-serialization-schema/avro` directory

## 6. Integration tests
Simply navigate to the directory where you would like to install the integration test package and clone it:
```sh
git clone https://git.tc.bbn.com/bbn/integration-tests.git
```

# Testing tc-in-a-box
After you have your environment set up, you might want to try validating it.  A very simple way to do so is to run the producer/consumer integration test suite, which will exercise the TA3 services, the salt services management scripts, and all language bindings.  If you have selectively installed language bindings then you will need to run individual tests from the integration test suite.

To run the full test suite:

```sh
cd integration-tests
python starc_test.py
```

If some tests fail, then there is likely something wrong with the installation.

To test only specific language bindings, you will need to pass a test method qualified by its encapsulating test class.  For example, to test a java producer and consumer:

```sh
cd integration-tests
python starc_test.py IntegrationKafkaLangTest.test_java_prod_java_cons
```

## Manually testing the services 
If you want to test the services manually (instead of using salt), here are the steps
The same steps below may be performed using our automated salt scripts (see next section)

  1. Start zookeeper in the VM
 
    ```
   vagrant ssh
   cd /opt/kafka_2.11-0.9.0.0/bin
   sudo ./zookeeper-server-start.sh ../config/zookeeper.properties
   # this starts zookeeper at port 2181
   ```
  2. Start a kafka broker in the VM
   ```
   vagrant ssh
   cd /opt/kafka_2.11-0.9.0.0/bin
   sudo ./kafka-server-start.sh ../config/server.properties 
   # this starts zookeeper at port 9092
   ```
  3. Start a consumer
   ```
   vagrant ssh
   cd ta3-api-bindings-java/tc-bbn-kafka
   java -jar target/kafkaclients-1.0-SNAPSHOT-jar-with-dependencies.jar test-1-1 -np -ks 10.0.2.15:9092 -csf ../../ta3-serialization-schema/avro/TCCDMDatum.avsc -v
   # this starts a consumer on topic test-1-1 
   #  -v option is for verbose
   #  -np means no producer (just the consumer thread)
   #  replace the IP address of the VM with your IP address if different than 10.0.2.15
   # to see all options, just run the jar without any parameters
   # see https://git.tc.bbn.com/bbn/ta3-api-bindings-java/tree/master/tc-bbn-kafka Readme for more information on starting consumers and producers
   ```
  4. Start a producer
   ```
   vagrant ssh
   cd ta3-api-bindings-java/tc-bbn-kafka
   java -jar target/kafkaclients-1.0-SNAPSHOT-jar-with-dependencies.jar test-1-1 -nc -ks 10.0.2.15:9092 -d 10 -psf ../../ta3-serialization-schema/avro/TCCDMDatum.avsc -n 100 -v
   # this starts a producer on topic test-1-1 
   #  -v option is for verbose
   #  -nc means just the producer thread (no consumer)
   #  -d 10 will publish for 10 seconds and shutdown
   #  replace the IP address of the VM with your IP address if different than 10.0.2.15
   # to see all options, just run the jar without any parameters
   # see https://git.tc.bbn.com/bbn/ta3-api-bindings-java/tree/master/tc-bbn-kafka Readme for more information on starting consumers and producers
   ```
   You should see the end to end flow of records (serialized according to the schema) from producer through kafka to consumer
  5. To stop the services first shutdown the kafka broker then zookeeper
  6. To clean the logs including the kafka queue and topics
   ```
   sudo rm -rf /opt/zookeeper
   sudo rm -rf /opt/kafka/kafka-logs
   ```

## Automated testing: starting the services above using salt
  1. Start the STARC services (currently zookeeper and kafka)
  
    ```
   vagrant ssh
   cd tc-salt-services
   ./salt.sh starc.start
   # this starts both zookeeper and kafka
   ./salt.sh starc.isup
   # this checks the status of the services the output should say {'zookeeper': "'imok'", 'kafka': 'Listening'}
   ```
  2. Start the consumer using kafka.run_consumer. This starts a java consumer using default parameter values that will consume records on the "test" topic for 15 seconds.
   ```
   cd tc-salt-services
   ./salt.sh kafka.run_consumer
   # The output is the pid of the consumer process
   # Check the status of the consumer with kafka.consumer_status
   ./salt.sh kafka.consumer_status
   # The output is the last 10 lines of the log file
   # The log file will indicate if anything was consumed, and provide a summary at the end
   ```
  3. Start the producer using kafka.run_producer.  This starts a java producer using default parameters that produces random LabeledEdge records on the test topic for 10 seconds
   ```
   cd tc-salt-services
   ./salt.sh kafka.run_producer
   # The output is the pid of the producer process
   # Check the status of the producer using kafka.producer_status
   ./salt.sh kafka.producer_status
   # The output is the last 10 lines of the producer log file
   ```
  4. Start another consumer using consume_latest. This consumer gets the last record published on each partition of the test topic and outputs the record data
   ```
   cd tc-salt-services
   ./salt.sh kafka.consume_latest
   # The output should be the deserialized record data for the last item published
   ```
  5. To stop the services 
   ```
   cd tc-salt-services
   ./salt.sh starc.stop
   # this stops both zookeeper and kafka
   ./salt.sh starc.isup
   # this checks the status of the services the output should say {'zookeeper': "None", 'kafka': 'None'}
   ```
  6. To clean the logs including the kafka queue and topics
   ```
   cd tc-salt-services
   ./salt.sh starc.deleteLogs
   # this cleans both kafka and zookeeper
   ```
The kafka.run_producer, run_consumer, and consume_latest methods have additional optional parameters that can control the simple client behavior (https://git.tc.bbn.com/bbn/tc-salt-services/wikis/home#run_producer)

## External Testing

You can run producers and consumer clients on your host machine and connect to the TA3 services running inside the tc-in-a-box VM.  
To do this properly, the tc-in-a-box VM needs to have an IP that is accessible from the client machine.  There are multiple ways to accomplish this, and a later update to tc-in-a-box will implement a more seamless and automatic method.
For now, the default setup is to use NAT and port forwarding.  This means that the tc-in-a-box VM will bind to localhost and port forwarding in Virtual Box will be used to forward any traffic on two ports to tc-in-a-box.

The instructions below are for setting this up manually in a few quick steps.  These instructions assume that you have tc-in-a-box up and running.

  1. Stop the starc services (if they're up)
   
    ```
    cd tc-salt-services
    ./salt.sh starc.stop
    ```
  2. Delete the logs
    
    ```
    cd tc-salt-services
    ./salt.sh starc.deleteLogs
    ```
  3. Modify the advertised.host.name property for kafka in the kafka server template properties file
   ```
   cd /srv/salt/kafka
   edit kafka.server.properties.template
   ```
   Uncomment the advertised.host.name property (remove the first #) and edit the property value to be 127.0.0.1
   ```
   advertised.host.name=127.0.0.1
   ```
   This is the IP that kafka will advertise to clients. (If you manually set up the tc-in-a-box VM networking a different way, you would want to put whatever IP is externally routable here.)
  4. Reconfigure the TA3 services with state.highstate
   ```
   cd tc-salt-services
   ./salt.sh state.highstate
   # Verify this step worked by checking that advertised.host.name is set properly in /opt/kafka_2.11-0.9.0.0/conf/server.properties
   ```
  5. Start the TA3 services
   ```
   cd tc-salt-services
   ./salt.sh starc.start
   ```
  6. Set up VirtualBox Port Forwarding
    Using the VirtualBox Manager UI
      1. Select the tc-in-a-box VM, right click, Settings, Network
      2. Click "Port Forwarding"
      3. Click the Add button (top right)
        * Name: Kafka
        * Protocol: TCP
        * Host IP: 127.0.0.1
        * Host Port: 9092
        * Guest Port: 9092
      4. Click the Add button again
        * Name: Zookeeper
        * Protocol: TCP
        * Host IP: 127.0.0.1
        * Host Port: 2181
        * Guest Port: 2181
      5. Press Ok
    Note: You can choose to use different Host ports if you wish, you'll just need to pass the correct host ports to your clients in the kafka and/or zookeeper connection string parameters.
  7. Run your clients on your host machine
     If you changed the Host port for kafka, you will need to update the kafka connection string, which is the ip:port for the kafka broker with a default of "127.0.0.1:9092"
  
# test
