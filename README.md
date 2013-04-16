Civitas v.0.1
=======

## Installation Guide (Development)

This installation guide is geared towards developing with Geotrellis on a local machine for development purposes. Additional documentation will be contributed for use in a production setting. 

Install prequisite packages and their dependicies. 

```bash
sudo apt-get install subversion git gcc g++ make
```

#### Install Oracle JDK 7

Download the appropriate JDK tarball to a directory of your choosing and uncompress it. 

```bash
cd ~
sudo wget http://download.oracle.com/otn-pub/java/jdk/7u17-b02/jdk-7u17-linux-x64.tar.gz
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

```bash
sudo wget http://www.scala-lang.org/downloads/distrib/files/scala-2.10.1.tgz
sudo mkdir /opt/scala
sudo tar -xvzf scala-2.10.1.tgz -C /opt/scala
export SCALA_HOME=/opt/scala/scala-2.10.1 >> ~/.profile
export PATH=$PATH:$SCALA_HOME/bin/ >> ~/.profile
```

#### Scala Build Tool (sbt)

```bash
sudo wget http://repo.typesafe.com/typesafe/ivy-releases/org.scala-sbt/sbt-launch//0.12.3/sbt-launch.jar
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

#### GEOS - Geometry Engine, Open Source

GEOS is used by PostgreSQL/PostGIS to process vector operations on your spatial data.

```bash
cd ~
wget http://download.osgeo.org/geos/geos-3.3.8.tar.bz2
tar xvf geos-3.3.8.tar.bz2 -C /opt/
cd /opt/geos-3.3.8
sudo ./configure && sudo make && sudo make install
```

#### PostgreSQL/PostGIS

Install PostgreSQL and its dependicies.

```bash
sudo apt-get install build-essential postgresql-9.1 postgresql-server-dev-9.1 libxml2-dev libproj-dev libjson0-dev xsltproc docbook-xsl docbook-mathml
```

Add support for PostGIS Raster.

```bash
sudo apt-get install libgdal1-dev
```

Download and build PostGIS

```bash
cd ~
wget http://download.osgeo.org/postgis/source/postgis-2.0.3.tar.gz
sudo tar xfvz postgis-2.0.3.tar.gz -C /opt/
cd /opt/postgis-2.0.3
sudo ./configure && sudo make && sudo make install
```
#### Geotrellis