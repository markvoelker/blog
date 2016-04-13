+++ 
title = "DefCore: What OpenStack Devs Need to Know"
author = "markvoelker"
date = "2016-04-06"
url = "/defcore-for-devs"
socialsharing = true
description = "A quick guide for to DefCore for PTL's and other developers."
categories = ["DefCore","OpenStack"]
tags = ["defcore","openstack"]
+++

As DefCore has become more established over the past year or so, I've
started getting more questions from project developers and PTL's.  In most
cases, they're interested to know whether things they're thinking about
doing in their projects are "ok from a DefCore perspective" or
would be "ok from an interoperability perspective".  In some cases they're
basically just giving a quick (and appreciated!) heads-up and sometimes
they're generating terrific discussions.

So before we go on, let's take a quick timeout to give a some kudos to
developers and PTL's that are now more than ever thinking about
interoperability early on when making technical decisions--**that
is *fantastic*.**

In the interest of making DefCore more approachable, I thought
I'd try to write up some of the general themes that people are asking
about and that OpenStack devs seem to want to know about.
So, what does an OpenStack developer or PTL
need to know about DefCore with respect to things you want to do in your
project?

**Three terms you should know.**
--------------------------------

Consult the [Lexicon](https://github.com/openstack/defcore/blob/master/doc/source/process/Lexicon.rst)
for more lingo, but here a couple of key terms you should definitely know:

* [Capability](http://git.openstack.org/cgit/openstack/defcore/tree/doc/source/process/Lexicon.rst#n16):
  Generally, a feature or thing you can do with the software, usually exposed
  as an API.  For example, listing volumes in Cinder or starting an instance
  in Nova.  Capabilities are defined by tests (more on that later).
* [Criteria](http://git.openstack.org/cgit/openstack/defcore/tree/doc/source/process/CoreCriteria.rst):
  One of the factors DefCore considers when determining whether
  a given Capability should be required for OpenStack Powered(TM) products.
  There are twelve in all which you can find information about
  [here](https://github.com/openstack/defcore/blob/master/doc/source/process/CoreCriteria.rst).
* Guideline: A document published by DefCore and approved by the Board of
  Directors every six months which details the Capabilities that OpenStack
  Powered(TM) products must expose to end users and the sections of
  upstream code they must use to do so.  I've heard these referred to as
  "the DefCore standards" occasionally.  You can find the two most recently
  Board-approved Guidelines as of this writing
  [here](http://git.openstack.org/cgit/openstack/defcore/tree/2015.07.json)
  and [here](http://git.openstack.org/cgit/openstack/defcore/tree/2016.01.json).

**Any Capability DefCore considers *must* have tests.**
-----------------------------------------------------

When DefCore considers adding new Capabilities to a Guideline, there must
be some way for clouds to prove that they expose those Capabilities and do
so in an interoperable way.  The teams of developers in the OpenStack
community who are hacking on the code are in the best position to define
what a given feature does and how it should work, so DefCore uses tests
that are under the governance of the TC.  Today, those tests are mostly
the ones that live in Tempest, but we're also now able to use tests that live
in the individual project trees as long as they're runnable via the
[Tempest plugin interface](http://docs.openstack.org/developer/tempest/plugin.html) (by the recent request projects that have additional tests in-tree).

***Key takeaway: how DefCore defines a working Capability is dependant on
how you define it in your tests (which must be in Tempest or runnable
via the Tempest plugin interface).***

**Capabilities and their tests should not require admin credentials.**
----------------------------------------------------------------------

If I'm using one of the 
[many fine public clouds](http://www.openstack.org/marketplace/public-clouds/)
that are built on OpenStack, I'm not going to have admin privileges
(many users of private clouds won't either).
Therefore, things that are generally only exposed to admins aren't going
to appear in DefCore Guidelines: they can't be interoperable if end users
can't actually use them.  In many cases, what is exposed to admins
vs ordinary users is controlled by policy (e.g. the 
[policy.json file](http://git.openstack.org/cgit/openstack/nova/tree/etc/nova/policy.json)
that ships with most projects).
In many products the default settings that ship from upstream are used
by default, so that often becomes a good indicator: if it's admin-only
on git.openstack.org (such as the [ability to create provider networks
in Neutron](http://git.openstack.org/cgit/openstack/neutron/tree/etc/policy.json#n52))
it's probably not going to be exposed to end users in most clouds and
will therefore probably not appear in DefCore Guidelines.

The focus on end-user-accessible capabilities has an
important implication for tests.  If your tests (in Tempest or in-tree)
use admin credentials for features that are generally exposed to end users,
*we can't include those tests in DefCore Guidelines*.  If all tests for
a Capability use admin credentials, the Capability effectively has no tests
we can use, and therefore the whole Capability won't make it into a Guideline.

If users can't run the tests to verify that clouds support the Capabilities
in question, having them in the Guideline becomes fairly toothless.  It is
therefore very important that Capabilities you want to see included in
Guidelines both be exposed to end users by default and have tests that don't
require admin credentials.

***Key takeaway: DefCore considers only things that are exposed to end users.
Write your tests and policy files accordingly.***

**Adoption is key.**
--------------------

I've [previously written](../defcore-misconceptions-1/)
about how DefCore is a trailing indicator of market acceptance.  Basically
it boils down to this: if few users using a given Capability and few products
are supporting it, it's very unlikely to appear in DefCore's
Guidelines.  Market acceptance in practice often ends up affecting or
reflecting a number of the
[DefCore Criteria](https://github.com/openstack/defcore/blob/master/doc/source/process/CoreCriteria.rst).
For instance:

* Widely Deployed: Well, this one's sort of obvious.
* Used by Tools: If a feature or API isn't supported by many clouds, 
  it's less likely that external SDK's (jClouds, Fog, etc) and tools
  are going to add support for it since there's probably less demand
  for doing so.
* Used by Clients: if a feature isn't supported by OpenStack's own
  clients (OpenStackClient, the various python clients, perhaps Horizon) then
  that's probably a sign that the feature isn't fully matured yet and/or
  fewer people are using it (since there's no client support).
* Documented: End users tend to have a harder time using a given feature if
  it is poorly documented (or not documented at all).  
* Etc, etc, etc (you get the picture)

***Key takeaway: if you want to something end up as a required Capability
in the DefCore Guidelines, the best thing you can do is convince the
broader ecosystem to adopt it first.***

Note that you don't have to just write a feature and hope people like it.
Projects like libcloud or jClouds would probably be happy to review patches
you submit that add support for a new API.  The docs team would probably be
happy to review patches to the docs describing how to use your new feature.
[Planet OpenStack](http://planet.openstack.org/) would probably be happy to
aggregate your blog post about how great it is and why people should use it.

**More than one way of doing things *may* be ok...sort of.**
------------------------------------------------------------

Some folks have been surprised to learn that there's actually no rule
barring DefCore Guidelines from including multiple ways to do something.
There are a couple of scenarios in which this happens:

1. **API iteration.** Sometimes it's simply a matter of more than one version
   of an API being available: for example, in the Liberty release one
   could generate a token by talking to the Keystone v2 API (which was
   designated as SUPPORTED) or the Keystone v3 API (which was marked CURRENT).
   Two API's usually have separate tests, and all versions of the API might
   not be available in all the releases covered by a single Guideline.
   DefCore considers them separate Capabilities.  Clearly, this can be a
   bit of a conundrum: projects may want long support periods for older
   API's because they want to give end users ample time to migrate, or simply
   don't ever want to break users.  API microversioning has become a
   popular way of helping ease introduction of new API's in recent years.

2. **Same output, very different mechanics.**  In some cases, there are
   Capabilities in which the end result is the same but the mechanics
   and use cases are very, very different.  For example: I can create
   an image by snapshotting a running compute instance.  At the end of
   that operation, what I end up with is a bootable image.  I could also create
   a bootable image by uploading an image file to Glance.  The use cases here
   are obviously quite different though: in one case I'm creating an image from
   an instance I already have running in the cloud, and in the other I'm
   uploading one from outside the cloud.  So it's quite reasonable to think that
   end users might want to have both Capabilities exposed to them. 
   Additionally, here again we have separate API's with separate tests.
   This scenario generally isn't as much of a problem as long as it's well
   understood and well documented.

3. **Really pretty much the same.**  In a few cases, there really is
   more than one way to do a thing without substantial differences in
   the use cases.  For example, I can list images available to me via the
   Glance API or via the Nova image API (which is basically a proxy to
   Glance v1).  Both give me a listing of images, and assuming both are
   available I could probably use either one for most use cases.

4. **Some mixture of 1-3.**  Occasionally we run into cases where there's
   some combination of the above scenarios in play.  For example, there are
   plenty of clouds where two versions of the Glance API and the Nova image API
   are all exposed to users, so there are at least 3 ways to list images.

It's important to note that when there's more than one way to do something
it's sometimes harder for any of them to make their way into DefCore Guidelines.
That boils down to adoption and inertia: if there are several ways to do
the same thing then different SDK's, tools, and end users often pick different
ways of doing things, and no one (or more) way becomes clearly well adopted.
Once SDK's, tools, and end users have settled on a method of doing something,
there's a real cost to them to change to something else (and if it isn't broken,
why change?).  Therefore they tend to stick with the option they've
chosen unless they're given a really good reason to switch (like, for example,
a given API being officially deprecated).  

In some cases we may find that more than one method of doing something
does actually become very well adopted, supported by tools and clients, etc
and therefore score high enough to be included in a Guideline.  But in
practice, it's often a tougher hill to climb.

***Key takeaway: be careful when introducing multiple ways to do things.***
It's not strictly disallowed, but creating fragmentation
might make it harder for capabilities to become well adopted and therefore
become interoperable.

As an aside, there are a few things projects can do to help nudge acceptance
of one method or another when there are multiple ways to do something:

* If your project really doesn't intend to maintain an older API any more,
  it may be useful to start the [deprecation process](https://governance.openstack.org/reference/tags/assert_follows-standard-deprecation.html)
  sooner rather than later as a clear signal to users, SDK's, and tools
  that it's time to move (this also signals DefCore that the old API
  shouldn't score points for the Future Direction criteria, too).
* If you have a feature that you think is primarily oriented at internal
  use (e.g. one OpenStack service talking to another) and shouldn't be
  exposed to end users, you should clearly document it as such (and, if
  it makes sense to do so, restrict it's use via default settings in the
  policy files).

**Keep in mind there's a long tail.**
-------------------------------------

DefCore Guidelines cover three OpenStack releases.  For example, the most
recently Board-approved DefCore Guideline [covers Juno, Kilo, and Liberty](http://git.openstack.org/cgit/openstack/defcore/tree/2016.01.json#n9).
The previous Guideline covered [Icehouse, Juno, and Kilo](http://git.openstack.org/cgit/openstack/defcore/tree/2015.07.json#n9).
When a product asks for an [OpenStack Powered(TM)](http://www.openstack.org/brand/interop/)
logo and license agreement from the OpenStack Foundation, they can choose
to do so under either of the two most recently Board-approved Guidelines.
Which Guideline the product used is noted in the
[MarketPlace](http://www.openstack.org/marketplace).

Because a Guideline applies to three releases, a given Capability
must have been supported in at least all three of those upstream
releases.  If you introduce a new feature today, it's going to
take some time before it's realistically possible to get it into the
DefCore Guidelines.  And bear in mind that it's not just time: once the
Capability has been around for a while, it still
needs to meet all the other [DefCore Criteria](https://github.com/openstack/defcore/blob/master/doc/source/process/CoreCriteria.rst)
before it makes it's way into the Guidelines.

***Key takeaway: be patient and promote adoption.*** The Criteria we're using
[are documented](https://github.com/openstack/defcore/blob/master/doc/source/process/CoreCriteria.rst),
so if you're motivated to help a given capability achieve those criteria
(maybe by contributing support for it to popular tools/SDK's, or by
beefing up documentation, or by generally evalgelizing a feature or API),
go for it!

**There are some useful documents in the DefCore Git repo.**
-------------------------------------------------------------

Much of the documentation about how and when DefCore does things and
how it makes it's decisions is actually documented.  You're always free
to come ask questions (see below), but if you're the sort that would
like a little bedtime reading, you might start with things like:

* [The current process doc](https://github.com/openstack/defcore/blob/master/doc/source/process/2015B.rst)
  which explains timelines, key actors, etc.
* [The Criteria doc](https://github.com/openstack/defcore/blob/master/doc/source/process/CoreCriteria.rst)
  which explains factors that DefCore looks at when determining whether to
  include a Capability in it's Guideline documents.
* [The Foundation's Interop page](http://www.openstack.org/brand/interop/)
  which has information about the OpenStack Powered(TM) program and what's
  included in it today.  Among other things, you'll find links to the two
  most recent Board-approved Guidelines and contact information.

Oh, and if by chance you're a fan of Doctor Who you might also want to
peruse 
[this presentation](http://www.slideshare.net/markvoelker/defcore-the-interoperability-standard-for-openstack-53040869)
that I did a while back.  It's a little dated now, but you'll at least be
mildly entertained.

*Questions?  Comments?  Drop by #openstack-defcore on IRC or drop the
DefCore Committee a line at
[defcore-committee@lists.openstack.org](mailto:defcore-committee@lists.openstack.org).*
