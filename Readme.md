!!!WARNING!!!

This set of scripts configures 2 VM's with the subnet 192.168.192.0/18, so if you have any parts of this subnet in your network you should alter the values in the Vagrantfile !

The certificates included are self-signed and have no value in case you want to steal these ;)

To run this test you will need Vagrant and VirtualBox installed:
https://www.vagrantup.com/downloads.html
https://www.virtualbox.org/wiki/Downloads

Running the test requires just 1 command::
    $ vagrant up

This will bring up both machines:

server: Has docker installed in swarm mode with 2 services
    1. traefik - a highperformant HTTP load balancer, to balance requests among 2 backends
    2. whoami - a simple HTTP service which prints the hostname

loader: Runs a bunch of ab benchmarks on a "broken" network to simulate bad clients


The test takes approximately 30 minutes (on a i5 Macbook Pro), and as a result there will be a number of ESTABLISHED connections to the load balancer on server machine, even though the tests are done and all "clients" are dead. Can be checked by::
    $ vagrant ssh server
    $ sudo -s
    # nsenter -t `pidof traefik` -n lsof -nPitcp:443

Also traefik web interface can be accessed at: http://127.0.0.1:8083

And the web service itself via:
    http    -> http://127.0.0.1:8080
    https   -> https://127.0.0.1:8443