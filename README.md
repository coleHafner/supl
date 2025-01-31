# What this is about

This is an implementation of OMA SUPL and 3GPP RRLP protocols used in
Assisted GPS (AGPS). Only client (mobile) initiated case is implemented.

The package provides 2 user level executables

1) supl-client and
2) supl-proxy

and supporting SUPL/RRLP library libsupl.


## supl-client

supl-client connect to a SUPL server over the Internet and gets GPS
assistance (A-GPS) data, like satellite ephemeris and almanac data
from it. The received data can then be fed to a GPS receiver (assuming
the receiver accepts such data) to speed up satellite aquistion.

Usage:
```
supl-client options [supl-server]
Options:
  # set current gsm/wcdma cell id
  --cell                    gsm:mcc,mns:lac,ci|wcdma:mcc,msn,uc	

  # set known gsm cell id with position
  --cell                    gsm:mcc,mns:lac,ci:lat,lon,uncert	

  --format human            machine parseable output
  --debug n                 1 == RRLP, 2 == SUPL, 4 == DEBUG
  --debug-file file         write debug to file
  --help                    show this help
```
Example:
`supl-client --cell=gsm:244,5:0x59e2,0x31b0:60.169995,24.939995,127 --cell=gsm:244,5:0x59e2,0x31b0`

The default SUPL server is supl.nokia.com.

Supl server may not return all assistance information if --gsm-cell
is not given. You can give some cell id to supl server but position
information returned by the server (based on the cell id) can be
overridden by --set-pos in the output. Format for this is 

--set-pos=lat,lon,uncertainty

where both lat and lon are decimal degrees (N and E positive) and
uncertainty in meters.

Also if enviroment variable SUPL_FAKE_POS it is used as the
position esitmate coming from the supl server. Format is the same
as in --set-pos.

With test options '-t 0', '-t 1' or '-t 2' you  get a quick feeling
how and if things will work - if they work ;-)


## supl-proxy

Usage:
supl-proxy supl-server

Sets up a proxy and displays SUPL / RRLP traffic between the client
and server. Convinient for debugging and figuring out the protocol.

To use it, you must direct your mobile to use your proxy server as its
SUPL server. In Nokia N95 this is in Tool -> Settings -> General ->
Positioning -> Postioning server. In Nokia N900 such setting is also
availabe in the settings application.

You must also install a SSL root certificate to the phone. The root
certificate must be set as trusted certificate in the phone. In N95
just set all uses as trusted (in the certificate manager). In N900 you
must use command line tool cmcli (available in maemosec-certman-tools
package) and install the root certificate into common-ca domain.

The proxy server must be given a server sertificate (signed by the
root certificate) and private key. The proxy reads them from
cert/srv-{cert,priv}.pem files.

### How to generate keys and certificates

** You can skip this section if you do not use supl-proxy **

All SSL keys and certificates can be generated with supl-cert tool if
you do not have those around already.

Run supl-cert with your proxy server domain name as the only argument:

~/src/supl-1.0 $ supl-cert proxy.dot.com

It will create CA certificate and private key files ca-cert.pem and
ca-priv.pem, and proxy server certificate (signed by the CA) and
server private key files srv-cert.pem and srv-priv.pem.

File ca-cert.pem should be sent (e.g. OBEX push for N95 or simple copy
for N900) to your mobile and then installed into certificate
store. Make sure to give it SSL trust.

supl-proxy expects to find srv-cert.pem and src-priv.pem from the
current directory. CA private key ca-priv.pem is not needed anymore.

# Compiling

## Compile without ASN.1 compiler

If you do not have asn1c (http://lionet.info/asn1c) available do:
```
~/src/supl $ ./configure --precompiled-asn1
~/src/supl $ make
~/src/supl $ sudo make install
```

That's it.


## Compile with ASN.1 compiler 

You should use asn1c version 0.9.23, there are some issues with
earlier versions.

Get the asn1c source from http://lionet.info/asn1c/download.html or
clone the git repository: `~src/ $ git clone git://github.com/vlm/asn1c.git`

and then configure, compile and install it first:
```
~/src/ $ cd asn1c
~/src/asn1c $ make
~/src/asn1c $ sudo make install
```
You may want get rid of your distribution provided asn1c compiler
first, or at least make sure the following compilation uses this newer
version.

Then proceed with compiling this stuff
```
~/src/supl $ ./configure
~/src/supl $ make
~/src/supl $ sudo make install
```


## Fixing ASN specs

There is a confict with NavigationModel which is defined by both supl
and rrlp. Rename defination and every reference to NavigationModel
with XNavigationModel in src/supl-posinit.asn.


## ASN.1 compilation from sources

Generation of compiled C-files from ASN.1 sources should happen
automatically with make. But if you add new ASN.1 files or PDUs you
must check and modify the Makefile in src/asn-rrlp or src/asn-supl
directory to include any new structures you introduced into ASN files.

You can give path to asn1c skeleton files to configure script as

~/src/supl $ ./configure --asn1c-skeletons /path/to/asn1c-git-HEAD


## Getting MCC, MNC, LAC and CI

You need to provide your position estimate to supl server as
cellular transmitter tower ids. One way to get them is to
go to the AT-command interpreter in your phone and ask for them:

at+cops?
+COPS: 0,2,"24405",2
OK
at+creg=2
OK
at+creg?
+CREG: 2,1,"59E2","31B0"
OK
at+creg=0
OK

==> you get needed values

MCC = 244
MCN = 05
LAC = 0x59e2
CI = 0x31b0


Alternatively you may setup supl-proxy on your machine and see
what your device talks to the supl server.
