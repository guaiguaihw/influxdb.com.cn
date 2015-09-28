+++
title = "downloads"
layout = "sidebar"
+++
# InfluxDB Downloads

## Version 0.9.4.1 (Stable)

#### OS X

- Via [Homebrew](http://brew.sh/)

		brew update
		brew install influxdb

#### Ubuntu & Debian

- 64-bit system install instructions

		wget https://s3.amazonaws.com/influxdb/influxdb_0.9.4.1_amd64.deb
		sudo dpkg -i influxdb_0.9.4.1_amd64.deb

#### RedHat & CentOS

- 64-bit system install instructions

		wget https://s3.amazonaws.com/influxdb/influxdb-0.9.4.1-1.x86_64.rpm
		sudo yum localinstall influxdb-0.9.4.1-1.x86_64.rpm

## Version 0.9.5 (Nightly)
Nightly builds are created once-a-day, at midnight San Francisco, CA time, using the top-of-tree of [master](https://github.com/influxdb/influxdb/tree/master) source code. These builds will include all the latest fixes, but also undergo only basic testing.

#### Ubuntu & Debian

- 64-bit system install instructions

        wget https://s3.amazonaws.com/influxdb/influxdb_nightly_amd64.deb
        sudo dpkg -i influxdb_nightly_amd64.deb

#### RedHat & CentOS

- 64-bit system install instructions

        wget https://s3.amazonaws.com/influxdb/influxdb-nightly-1.x86_64.rpm
        sudo yum localinstall influxdb-nightly-1.x86_64.rpm


### 32-Bit Packages
The industry is gradually [moving away from support for 32-bit x86 architectures](https://golang.org/doc/go1.5) so we do not provide packaged 32-bit binaries. However, we do endeavour to ensure the source can be compiled for a 32-bit x86 architecture at all times. To that end our [CI system](https://circleci.com/gh/influxdb/influxdb/tree/master) currently compiles 32-bit binaries and runs the unit test suite against the 32-bit build, in addition to the main 64-bit build. If compilation or unit testing for 32-bit architecture fails, we fix it.

However, we do reserve the right to break 32-bit compatibilty in the future, should design and implementation require it, but there are no plans to do so at this time.

### Deprecated Releases

Deprecated versions are no longer actively developed.

- [version 0.8](/docs/v0.8/introduction/installation.html)


# Telegraf Downloads

## Version 0.1.8

#### OS X

- Via [Homebrew](http://brew.sh/)

		brew update
		brew install telegraf

#### Ubuntu & Debian

- 64-bit system install instructions

		wget http://get.influxdb.org/telegraf/telegraf_0.1.8_amd64.deb
		sudo dpkg -i telegraf_0.1.8_amd64.deb

#### RedHat & CentOS

- 64-bit system install instructions

		wget http://get.influxdb.org/telegraf/telegraf-0.1.8-1.x86_64.rpm
		sudo yum localinstall telegraf-0.1.8-1.x86_64.rpm

