Civitas
=======

This project is a [modification](https://github.com/azavea/GeoTrellis-features-demo) of a joint effort of the University of Tennessee at Chattanooga and Azavea, with funding from the Lyndhurst Foundation. The goal here is to provide step-by-step documentation to install and deploy a Geotrellis-based application in a distributed environment.

*Author's Note: The how is well documented, but not the why. I will update this guide to provide a more thorough review of the concepts employed in the following week*.

## Installation Guide

To begin, install some prequisite packages, their dependicies, and some workflow tools. 

```bash
apt-get install subversion git gcc g++ make
```

##### Install Oracle JDK 7

There are many flavors of the JVM, and after performance benchmarks the Geotrellis team recommends Oracle's JDK over OpenJDK. Download the appropriate JDK tarball to a directory of your choosing and uncompress it. 

```bash
tar -xvzf jdk-7u17-linux-x64.tar.gz
```

Move the JDK 7 directory to `/usr/lib`.

```bash
sudo mkdir -p /usr/lib/jvm
sudo mv ./jdk1.7.0_17/ /usr/lib/jvm/jdk1.7.0
```

Now run:

```bash
sudo update-alternatives --install "/usr/bin/java" "java" "/usr/lib/jvm/jdk1.7.0/bin/java" 1
sudo update-alternatives --install "/usr/bin/javac" "javac" "/usr/lib/jvm/jdk1.7.0/bin/javac" 1
sudo update-alternatives --install "/usr/bin/javaws" "javaws" "/usr/lib/jvm/jdk1.7.0/bin/javaws" 1
```

If you have issues, please refer to [this post](http://askubuntu.com/questions/55848/how-do-i-install-oracle-java-jdk-7?rq=1).

##### Scala

Scala is a JVM-based language that Geotrellis is built with. Please download the latest release [here](http://www.scala-lang.org/download/).

```bash
sudo mkdir /opt/scala
sudo tar -xvzf scala-2.10.1.tgz -C /opt/scala
export SCALA_HOME=/opt/scala/scala-2.10.1 >> ~/.profile
export PATH=$PATH:$SCALA_HOME/bin/ >> ~/.profile
```

##### Scala Build Tool (sbt)

sbt is a build tool for Scala projects. You can find the latest release [here](http://www.scala-sbt.org/release/docs/Getting-Started/Setup.html).

```bash
sudo mv sbt-launch.jar /usr/local/bin/
sudo touch sbt /usr/local/bin/ && sudo nano /usr/local/bin/sbt
```

Add the following to the newly created `sbt` file:

```bash
java -Xms512M -Xmx1536M -Xss1M -XX:+CMSClassUnloadingEnabled -XX:MaxPermSize=384M -jar `dirname $0`/sbt-launch.jar "$@"
```

Make the script executable.

```bash
sudo chmod +x /usr/local/bin/sbt
```

##### GDAL - Geospatial Data Abstraction Library

GDAL is used by to convert your raster files to the GeoTIFF format that will be subsequently converted to the [Azavea Raster Grid (ARG)](https://github.com/geotrellis/geotrellis/wiki/ARG-Specification) format. It is also used by PostgreSQL/PostGIS for raster support.

```bash
sudo svn checkout https://svn.osgeo.org/gdal/trunk/gdal /opt/gdal
cd /opt/gdal
sudo ./configure && sudo make && sudo make install
```

*Note: Building GDAL from source will take some time, so grab a cup of coffee.*

##### Geotrellis

Navigate to an appropriate directory and issue one of the following commands:

```bash
# Using SSH
git clone git@github.com:geotrellis/geotrellis.git

# Using HTTPS
git clone https://github.com/geotrellis/geotrellis.git
```

##### Civitas Demo Application

In a similar fashion issue the following:

```bash
# Using SSH
git clone git@github.com:baskinomics/civitas.git
```

## Deploying to a Distributed Cluster

Civitas is currently deployed to a distributed cluster composed of three machines, and this section will use this scenario as an example. The first prerequisite is to repeat the installation procedure on each machine in the cluster (see installation script).

### Cluster Dependecies and Configuration
Add the following dependency to one of your project's build file, e.g. `your-project/geotrellis/build.sbt`.:

```
"com.typesafe.akka" %% "akka-cluster" % "2.2.0"
```

Next there are a number of additions we must make to the `.../src/main/resources/application.conf` file. The first item to do is define variables that can also be provided via the sbt command line:

```
geotrellis.hostname = "localhost"
geotrellis.akka_port = 2551
geotrellis.cluster_seed   = "localhost"
geotrellis.cluster_seed = ${?GEOTRELLIS_CLUSTER_SEED_IP}
geotrellis.cluster_seed_port = "2551"
geotrellis.cluster_seed_port = ${?GEOTRELLIS_CLUSTER_SEED_PORT}
```

Following that we need to add the configuration for the Akka actor system, remoting, and the cluster. 

```
akka {
  actor {
    provider = "akka.cluster.ClusterActorRefProvider"
  }
  remote {
    log-sent-messages = on
    transport = "akka.remote.netty.NettyRemoteTransport"
    netty {
      hostname = "localhost"
      hostname = ${?geotrellis.hostname}
      port = ${geotrellis.akka_port}
      message-frame-size = 134217728
    }

    remote.server.message-frame-size = 134217728
   remote.client.message-frame-size = 134217728
   }
    cluster {
      seed_ip = "localhost"
      seed_port = "2551"
      seed-nodes = ["akka://GeoTrellis@"${geotrellis.cluster_seed}":"${geotrellis.cluster_seed_port}]
      metrics.enabled = off
      auto-join = on
      enabled = true
      auto-down = on
    }
  
}
```

Lastly we need to configre our Cluster Aware Router:

```
akka.actor.deployment {
  /clusterRouter = {
  router = round-robin

  metrics-selector = mix
  nr-of-instances = 1000
  max-nr-of-instances-per-node = 3
  cluster {
    enabled = on
    routees-path = "/user/civitas"
    allow-local-routees = on
  }
  }
}
```
### Distributing an Operation

### Initializing the Cluster

#### Start Seed Node

#### Joining to Seed Nodes

```bash
./sbt -Dgeotrellis.cluster_seed_ip=xxxx  
```

### Running the Application