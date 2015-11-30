openshift-diy-mojolicious
====================

Steps to support Perl 5.16 or newer version on OpenShift.com


Steps:
--------

Create new app using diy-0.1 cartridge:

	rhc app create diy diy-0.1
	rhc ssh -a diy


Download & Build new version of Perl from source:

	cd $OPENSHIFT_DATA_DIR
	mkdir download
	cd download
	wget -c -nd http://www.cpan.org/src/5.0/perl-5.16.3.tar.gz
	tar -xf perl-5.16.3.tar.gz
	cd perl-5.16.3
	./Configure -des -Dprefix="$OPENSHIFT_DATA_DIR/perl"
	make 
	make install
	cd "$OPENSHIFT_DATA_DIR/perl/bin"
	./perl -v

Then install additional Perl modules:

	cd "$OPENSHIFT_DATA_DIR/perl/bin"
	HOME=$OPENSHIFT_DATA_DIR ./perl cpan
	> cpan[1]    notest install Mojolicious and_other_modules
	> quit


Roll your own webserver
----------------------

To start your own web server run on host $OPENSHIFT_DIY_IP and on port $OPENSHIFT_DIY_PORT (usually port 8080)  :

	
	"$OPENSHIFT_DATA_DIR/perl/bin/morbo" --listen 'http://$OPENSHIFT_DIY_IP:$OPENSHIFT_DIY_PORT' "$OPENSHIFT_REPO_DIR/diy/script/diy"


However, Please edit this file:

	.openshift/action_hooks/start

to 

	#!/bin/bash
	MOJO_HOME="$OPENSHIFT_REPO_DIR/diy" "$OPENSHIFT_DATA_DIR/perl/bin/morbo" \
              --listen="http://$OPENSHIFT_DIY_IP:$OPENSHIFT_DIY_PORT" \
              "$OPENSHIFT_REPO_DIR/diy/script/diy" |& /usr/bin/logshifter -tag diy &

to autostart your webserver.


and also:

	.openshift/action_hooks/stop

to

    #!/bin/bash
    source $OPENSHIFT_CARTRIDGE_SDK_BASH
    
    # The logic to stop your application should be put in this script.
    if [ -z "$(ps -ef | grep diy | grep -v grep)" ]
    then
        client_result "Application is already stopped"
    else
        kill `ps -ef | grep diy | grep -v grep | awk '{ print $2 }'` > /dev/null 2>&1
    fi


to stop your webserver.

Also replace in "diy/script/diy"
    
    this:
        "use lib 'lib';"
    
    to:
        use lib $ENV{OPENSHIFT_REPO_DIR} . '/diy/lib';



Test on your browser:
----------------------

	http://[diy]-[yourdomain].rhcloud.com



Maintenance
------------

To stop/start/restart your app:

	rhc ssh -a diy
	
	rhc app stop -a diy
	rhc app start -a diy
	rhc app restart -a diy


