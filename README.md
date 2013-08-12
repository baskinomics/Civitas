Civitas
=======

This project is a [modification](https://github.com/azavea/GeoTrellis-features-demo) of a joint effort of the University of Tennessee at Chattanooga and Azavea, with funding from the Lyndhurst Foundation. The goal here is to provide step-by-step documentation to install and deploy a Geotrellis-based application in a distributed environment.

## Installation Guide

To begin, install some prequisite packages, their dependicies, and some workflow tools. 

```bash
apt-get install subversion git gcc g++ make
```

#### Install Oracle JDK 7

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

#### Scala

Scala is a JVM-based language that Geotrellis is built with. Please download the latest release [here](http://www.scala-lang.org/download/).

```bash
sudo mkdir /opt/scala
sudo tar -xvzf scala-2.10.1.tgz -C /opt/scala
export SCALA_HOME=/opt/scala/scala-2.10.1 >> ~/.profile
export PATH=$PATH:$SCALA_HOME/bin/ >> ~/.profile
```

#### Scala Build Tool (sbt)

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

#### GDAL - Geospatial Data Abstraction Library

GDAL is used by to convert your raster files to the GeoTIFF format that will be subsequently converted to the [Azavea Raster Grid (ARG)](https://github.com/geotrellis/geotrellis/wiki/ARG-Specification) format. It is also used by PostgreSQL/PostGIS for raster support.

```bash
sudo svn checkout https://svn.osgeo.org/gdal/trunk/gdal /opt/gdal
cd /opt/gdal
sudo ./configure && sudo make && sudo make install
```

*Note: Building GDAL from source will take some time, so grab a cup of coffee.*

#### Geotrellis

Navigate to an appropriate directory and issue one of the following commands:

```bash
# Using SSH
git clone git@github.com:geotrellis/geotrellis.git

# Using HTTPS
git clone https://github.com/geotrellis/geotrellis.git
```