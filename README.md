NiFi Parcel
===========

This repository provides a [parcel](https://github.com/cloudera/cm_ext) to install Apache NiFi as a service usable by Cloudera Manager on Linux distributions.

# Install Steps
0. Install Prerequisites: `cloudera/cm_ext`
```sh
cd /tmp
git clone https://github.com/cloudera/cm_ext
cd cm_ext/validator
mvn install
```

1. Create parcel & CSD for RHEL/CentOS 7.
To build a parcel for other distros change the `DISTRO_SUFFIX` environment variable: the list of supported distro suffixes is available here https://github.com/cloudera/cm_ext/wiki/Parcel-distro-suffixes
```sh
$ cd /tmp
# First, create NiFi package
$ git clone http://github.com/apache/nifi
$ cd nifi
$ mvn clean install
$ cd /tmp
$ git clone http://github.com/prateek/nifi-parcel
$ cd nifi-parcel
$ POINT_VERSION=5 DISTRO_SUFFIX=el7 VALIDATOR_DIR=/tmp/cm_ext ./build-parcel.sh /tmp/nifi/nifi-assembly/target/nifi-*-SNAPSHOT-bin.tar.gz
$ VALIDATOR_DIR=/tmp/cm_ext ./build-csd.sh
```

2. Serve Parcel using Python
```sh
$ cd build-parcel
$ python -m SimpleHTTPServer 14641
# navigate to Cloudera Manager -> Parcels -> Edit Settings
# Add fqdn:14641 to list of urls
# Download, distribute and activate the NIFI parcel
```

4. Move CSD to Cloudera Manager's CSD Repo
```sh
# transfer build-csd/NIFI-1.0.jar to CM's host
$ cp NIFI-1.0.jar /opt/cloudera/csd
$ mkdir /opt/cloudera/csd/NIFI-1.0
$ cp NIFI-1.0.jar /opt/cloudera/csd/NIFI-1.0
$ cd /opt/cloudera/csd/NIFI-1.0
$ jar xvf NIFI-1.0.jar
$ rm -f NIFI-1.0.jar
$ sudo service cloudera-scm-server restart
# Wait a min, go to Cloudera Manager -> Add a Service -> NiFi
```

## Important note

NiFi integration with Cloudera is not as good as a native service. Cloudera manager does not configure it as a cluster automatically
and it cannot control NiFi configuration either. Unfortunately, you will still have to edit nifi.propertoes file on each node and
configure it to work in cluster and to use Cloudera’s Zookeeper. More information [here](http://www.idata.co.il/2017/10/integrating-apache-nifi-with-cloudera-manager/).

## Pending items
- Currently `NiFi` runs under the `root` user
- Expose config options under Cloudera Manager
  - Conf folder from parcels is used, this needs to be migrated to ConfigWriter
- Expose metrics from NiFi
