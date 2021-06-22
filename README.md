<h3 align="center">Option Pricing Demo with Chronicle Queue</h3>


<!-- TABLE OF CONTENTS -->
<details open="open">
  <summary>Table of Contents</summary>
  <ol>
    <li>
      <a href="#about-the-project">About The Project</a>
      <ul>
        <li><a href="#library-choice">Library Choice</a></li>
        <li><a href="#built-with">Built With</a></li>
      </ul>
    </li>
    <li>
      <a href="#getting-started">Getting Started</a>
      <ul>
        <li><a href="#prerequisites">Prerequisites</a></li>
        <li><a href="#installation">Installation</a></li>
      </ul>
    </li>
    <li><a href="#usage">Usage</a></li>
    <li><a href="#performance-test-note">Performance Test Note</a>
      <ul>
        <li><a href="#cpu-core-binding">CPU Core binding</a></li>
        <li><a href="#test-machine">Test machine</a></li>
        <li><a href="#statistics">statistics</a></li>
      </ul>
   </li>
    <li><a href="#chronicle-queue---some-learning-notes">Chronicle Queue - some learning notes</a></li>
    <li><a href="#the-math-parts---some-learning-notes">The Math parts - some learning notes</a></li>
    <li><a href="#contact">Contact</a></li>
  </ol>
</details>



<!-- ABOUT THE PROJECT -->
## About The Project


Problem Statement
Traders want a system to show the real-time value of her portfolio which consist of three types of products:
1.	Common stocks.
2.	European Call options on common stocks.
3.	European Put options on common stocks.

### Library Choice
There is also a suggestion to use Chronicle Queue. I didn't know this library before, thus thinking it would be a good 
learning opportunity and fun to explore how far we can push by using the Chronicle lib.
The intention is to try stressing the code and lib, thus there isn't any "vertical strategy" at all,
eg time throttle or implicit throttle, if some readers may wonder

### Built With

* [Chronicle-Queue](https://github.com/OpenHFT/Chronicle-Queue)


<!-- GETTING STARTED -->
## Getting Started

Build instructions

### Prerequisites

* jdk1.8
  ```sh
  java -version
  java version "1.8.0_261"
  Java(TM) SE Runtime Environment (build 1.8.0_261-b12)
  Java HotSpot(TM) 64-Bit Server VM (build 25.261-b12, mixed mode)
  ```

### Installation
Unix

If there is dos2unix command, you can use that instead of the sed command below
1. Run gradlew build
   ```sh
   chmod 755 gradlew
   sed -i 's/\x0D$//' gradlew
   ./gradlew build
   ```
2. Setup SqlLite and populate static data
   ```sh
   cd scripts
   chmod 755 *.sh
   sed -i 's/\x0D$//' *.sh
   ./setupDbAndStaticData.sh
   ```

Window
1. Run gradlew build
   ```sh
   gradlew.bat build
   ```
2. Setup SqlLite and populate static data
   ```sh
   cd scripts
   .\setupDbAndStaticData.bat
   ```

Firewall
1. In case if  you see connection error from the gradle build, please create the gradle properties file and
add proxy settings
   ```shell
   gradle.properties
   systemProp.http.proxyHost=hostname
   systemProp.http.proxyPort=8080
   systemProp.http.proxyUser=username
   systemProp.http.proxyPassword=xxx
   systemProp.https.proxyHost=hostname
   systemProp.https.proxyPort=8080
   systemProp.https.proxyUser=username
   systemProp.https.proxyPassword=xxx
   ```

<!-- USAGE EXAMPLES -->
## Usage

(Please ensure you have run the setupDbAndStaticData step above before you start)

Unix
1. Start Market Data and Pricing Service Simulation
   ```sh
    ./startSimulation.sh
   ```
2. Start Portfolio result subscriber
   ```sh
   ./startResultPrinting.sh
   ```

Window
1. Start Market Data and Pricing Service Simulation
   ```sh
    .\startSimulation
   ```
2. Start Portfolio result subscriber
   ```sh
   .\startResultPrinting
   ```

Settings
1. We can run the command directly inside the .sh / .bat and adjust with the following parameters to play around

   ```shell
   # -Dsst 
   # the simulation sleep time in between, in ms.
   Example: no sleeping time
   java -cp ../build/libs/OptionPricingPoC-1.0-SNAPSHOT.jar -Dsst=0  cph.optionpricing.demo.simulation.SimulationMain
   ```

   ```shell
   # -Dpprt
   # -Dpprt is the portfolio refresh time, in ms
   Example: refresh portfolio every 1 second
   java -cp ../build/libs/OptionPricingPoC-1.0-SNAPSHOT.jar -Dpprt=1000 cph.optionpricing.demo.simulation.PortfolioResultPrintMain
   ```



<!-- LEARNING NOTES -->
## Performance Test Note

### CPU Core binding

#### MarketDataSimulation and PricingServiceSimulation
1. CPU core binding is enabled by using Chronicle Thread Affinity, when running on linux for above 
   However I didn't manage to get a test machine with CPU isolation enabled, to prevent kernel interrupt. 
   Thus the thread affinity lock may not produce the best result
   
   
2. Virtual Machine
   
   Please note the test is only running on VM rather than a bare metal box, so result can be far from limit still

#### PortfolioResultsSubscriber   
1. No CPU binding enabled
2. A busy reading thread which continuously read from CQ

   
### Test machine
VM (Partition info unknown) Running on actual hardware with spec below:

Attribute|Value
--------|------
|Modelname| Intel(R)Xeon(R)CPU E5-2650 v3 @ 2.30GHz|
|CPU family| 6|
|L1d cache| 32k|
|L1i cache| 32k|
|L2 cache| 256k|
|L3 cache|25600K|
|NUMAnode0CPU(s)|0-3|

### Statistics 
Below gives a rough idea and to give a feel of the magnitude. However, it may be far from accurate as 
we are counting the end result on printing thread only. It already includes market data simulation,pricing of underlying options for each symbol, Chronicle write/read and result
aggregation.

Simulation ran for 4 mins, with 0 sleep time in between. 

(Around 26 million market data, 52 million pricing call, Chronicle Read/Write)

On average for every second:
   ```shell
   ####Portfolio update since last 1.0 seconds
  AAPL                #updates 177701
  TELSA               #updates 177754
   ```

In every 1 second, all above actions happened.
In above example, 177701+177754=355433 of above actions happened per second.
=> 2.81 us

It more of less matches the indication from the Chronicle docs. The extra us is probably from the pricing call. However, above test setup is rough it may 
not run long enough so that the data can be more accurate

## Chronicle Queue - some learning notes 

1. Queue Rolling setting - I didn't enable rolling update,therefore the queue file grows continuously. At the point of hitting
   disk space limit, there is scary error about the pointer states.
   
   
2. When I run the simulation, after several millions messages,
   I noticed sometime there is an info message from Chronicle, saying it's taking
   more us than expected for the internal Queue extension. Looks like it's the same case of the usual array length 
   extention, there may be setting to pre intialize and occupy the space but I haven't explored on that yet
   

## The Math parts - some learning notes
1. Cumulative Density Function - At first I didn't pay too much attention as I thought it's simply plug and 
   play into a simple function call. However, only at implementation stage I realized the offer form other libs
   like apache creates many intermediate objects along the way. A bit deviate of the intension of stressing the code
   and try to minimize object allocation whenever we could. End up I found a paper talking about anti-derivative and
   there is a clean approximation form.
   

2. Geometric Brownian Motion - The implicit number of trading days assumptions per year from the constant
   7257600 in appendix
   
   

## To Dos.....

### JMH for the picing call performance
Microbenmark on each componet sperately so that I can get a true idea of latency in each part

### Object allocation monitoring
A note on this point, in the CQ part, I tried to adopt the usual tricks to avoid object allocation. However, 
one piece I can't avoid in this POC implemention is the big data map for the static data. In between when temp value
is getting from it, temporary object is there and I want to understand how big is the implication there.

For real implementation, probably we may also try to leverage the Chronicle Zero Allocation hashing Lib or Chronicle map


### Rolling setting for Chronicle Queue


### Horizonatal Scaling
see "hsconcept" package


<!-- CONTACT -->
## Contact

Pak Hong Chan - [@pak](https://www.linkedin.com/in/pak-chan-91aa5738/) - pakhong.chan@gmail.com
