Varnish on OpenShift
====================

Note: This is a work in progress community sponsored cartridge.  It is very
likely to change over time as we continue to integrate with OpenShift.

This cartridge allows users to run varnish applications on OpenShift.  To get
started you'll need a website you want to accelerate.  For most people this
will be some other application on OpenShift (for example, a drupal or wordpress
site).

Once you have a site you'd like to accelerate, create a new varnish
application.  At this time the varnish cartridge can't be added to other
applications.  So this means you'll have two applications running, one varnish
and one application.

    rhc app create -a varnish -t https://raw.github.com/mmcgrath-openshift/openshift-cartridge-varnish/master/metadata/manifest.yml

Next you will need to edit the "default.vcl" file inside the newly cloned git
repo.  Remember, this is a new git repo created just for varnish (a whole new
application).  You'll need to edit this config to point to the running app.


Once changes are ready, commit and push them then browse to your newly accelerated website:


Notes about OpenShift and Scaled Apps
=====================================
Because of the default settings in our scaled application containers, our
load balancer will create a cookie to help support sticky sessions.  Because
of this, those pages can't be balanced.  Remember in the current architecture
for the Varnish cartridge it goes:

browser -> Node Proxy -> Varnish Gear -> Node Proxy -> HAProxy -> App Gear

This is inefficient and sometimes painful (we're working on it) but you still
see a major speed up due to caching even in this configuration (sometimes by
two orders of magnitude or more).

If you need to use the scaled app server, you'll need to log in to that gear
and edit the haproxy.cfg file.  This is a bad idea and will likely break things
in the future but it's your only option right now.  Basically you need
to edit:

    haproxy/conf/haproxy.cfg

Find a line that looks something like:

    server local-gear 127.9.27.1:8080 check fall 2 rise 3 inter 2000 cookie local-52706032e0b8cd41fe0001ae

And remove the last two fields (the cookie local-52706032e0b8cd41fe0001ae).  It
should end up looking like:

    server local-gear 127.9.27.1:8080 check fall 2 rise 3 inter 2000 cookie

But with your own IP in it.  It is highly likely our systems will re-add this
information from time to time.  If performance suddenly tanks, take a look and
see if you need to re-remove that info.


Notes about varnish
===================

These binaries came straight from the varnish website (see below).  Questions
about how to use varnish should be directed at Google or the varnish website:
https://www.varnish-cache.org/

It's important to know how to read headers with varnish.  Looking for things
like:

    Cache-Control: private

Is not good and will likely prevent you from caching those pages.  Don't let
it discourage you though, we've all been there :)


Where'd it come from?
=====================

Varnish binaries and source can be downloaded from
http://repo.varnish-cache.org/redhat/varnish-3.0/el6/
