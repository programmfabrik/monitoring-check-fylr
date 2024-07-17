# check_fylr
This is a Nagios plug-in which checks whether a fylr instance is up and responding in a given time frame.

## mandatory input

Needs a URL, as first command line argument.

Recommended: 
* Use the basic fylr front-end URL in your host configuration and
* append the API call `/api/v1/settings` to the URL in the Nagios server configuration. See below for a tested example.

## optional input

Warning timeout in seconds:

~~~~
-w 4
~~~~

Critical timeout in seconds:

~~~~
-c 8
~~~~

String to search for in the answer of fylr.
Example command line option which sets the default:

~~~~
-s index_names
~~~~

## Nagios server configuration example

On your Nagios server add: (in e.g. /etc/nagios-plugins/config/local/fylr.cfg)

~~~~
define command {
    command_name    check_by_ssh_fylr
    command_line    /usr/lib/nagios/plugins/check_by_ssh -t 10 -H $HOSTADDRESS$ -C "/usr/local/nagios/libexec/monitoring-check-fylr/check_fylr '$ARG1$$_HOSTFYLR_APIURL$' -w '$_HOSTFYLR_WARN$' -c '$_HOSTFYLR_CRIT$' -s '$_HOSTFYLR_APISTRING$'"
}
~~~~

Set the default values, in this example in /etc/icinga/objects/generic-host_icinga.cfg (Icinga is a Nagios fork):

~~~~
        _FYLR_WARN                   4
        _FYLR_CRIT                   8
        _FYLR_APIURL                 /api/v1/settings
        _FYLR_APISTRING              index_names
~~~~

Also add a service definition:

~~~~
define service {
        use                             generic-service
        host_name                       foo
        service_description             fylr instance
        check_command                   check_by_ssh_fylr!https://uni-atlantis.de
        }
~~~~

Note: This is where the basic fylr front-end URL has to go (in this example and configuration style), separated with a ! in the check_command line, above.

There are other ways to set up the connection between Nagios client and server. But if you followed the style shown here, then also make sure that the nagios user of the Nagios server is able to log into nagios@client without interactive barriers (SSH-key instead of password and the host key has been accepted by confirming with "yes" during a prior connection).

## installation example on a client
On the server where docker for fylr is running, but not inside a docker container, do:

~~~~
mkdir -p /usr/local/nagios/libexec
cd /usr/local/nagios/libexec
git clone https://github.com/programmfabrik/monitoring-check-fylr
~~~~

It bears repetition to say that Nagios has to be able to connect automatically, either by SSH as in the example above, or by other means.

## check_mk host configuration example

On your monitored Host, install check_mk_agent and do:

~~~~
mkdir -p /usr/local/nagios/libexec /usr/lib/nagios/plugins
cd /usr/local/nagios/libexec
git clone https://github.com/programmfabrik/monitoring-check-fylr
cd /usr/lib/nagios/plugins
ln -s /usr/local/nagios/libexec/monitoring-check-fylr/check_fylr .
~~~~
Replace the following URL with your fylr URL.
~~~~
echo "fylr /usr/lib/nagios/plugins/check_fylr 'https://url.fylr.de/api/v1/settings' -w '2' -c '8' -s 'index_names'" >> /etc/check_mk/mrpe.cfg
chmod 600 /etc/check_mk/mrpe.cfg
~~~~
Test:
~~~~
check_mk_agent
~~~~
check_mk should find the fylr plugin at the next service discovery check.
