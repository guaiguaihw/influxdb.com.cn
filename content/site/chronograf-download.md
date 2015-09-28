+++
title = "Chronograf"
layout = "chronograf-download"
+++
#  Installing Chronograf

Installing Chronograf on either a Debian/Ubuntu or RedHat/CentOS distribution of Linux is easy. Just choose the correct package for your platform:

## Ubuntu & Debian

- 64-bit system instructions

		wget https://s3.amazonaws.com/get.influxdb.org/chronograf/chronograf_0.1.0_amd64.deb
		sudo dpkg -i chronograf_0.1.0_amd64.deb

## RedHat & CentOS

- 64-bit system instructions

		wget https://s3.amazonaws.com/get.influxdb.org/chronograf/chronograf-0.1.0-1.x86_64.rpm
		sudo yum localinstall chronograf-0.1.0-1.x86_64.rpm

# Usage

If you installed Chronograf via a .deb or .rpm package, you should be able to simply run `sudo service chronograf start`.
The Chronograf startup script needs root permission to ensure that it can write to /var/log, but the actual executable runs as a normal user.

If you did not install Chronograf via a package, you can just directly run the executable, e.g. `/opt/chronograf/chronograf`.

## Command-line options

To see the most up-to-date command-line options, run `chronograf --help`.

By default, the Chronograf server binds to localhost on port 10000.
You can override this by specifying e.g. `-bind=0.0.0.0:80` to bind to all interfaces on port 80 (this will make your Chronograf instance publicly accessible, which may not be what you want).

You don't have to specify the location of your InfluxDB server when you start Chronograf, but if unspecified, you will be prompted inside the web application.
To configure the location of your InfluxDB cluster at startup, use the command-line option `-influx`, e.g. `-influx=http://localhost:8086` to connect to an InfluxDB server running on the same machine as Chronograf on the default InfluxDB port.

If you installed Chronograf via a package, you can set the default command-line arguments by specifying e.g. `args="-bind=chronograf.example.com:80 -influx=http://influx.example.com:8086"` in `/etc/default/chronograf`.

# Contact Us

For questions about Chronograf, contact us at [support@influxdb.com](mailto:support@influxdb.com?subject=Chronograf Support).
