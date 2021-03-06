Site
====

The ``site`` consists of the files ``site.conf`` and ``site.mk``.
In the first community based values are defined, which both are processed
during the build process and runtime.
The last is directly included in the make process of Gluon.

Configuration
-------------

The ``site.conf`` is a lua dictionary with the following defined keys.

hostname_prefix
    A string which shall prefix the default hostname of a device.

site_name
    The name of your community.

site_code
    The code of your community. It is good practice to use the TLD of
    your community here.

prefix4
    The IPv4 Subnet of your community mesh network in CIDR notation, e.g.
    ::

       prefix4 = '10.111.111.0/18'

prefix6
    The IPv6 subnet of your community mesh network, e.g.
    ::

       prefix6 = 'fdca::ffee:babe:1::/64'

timezone
    The timezone of your community live in, e.g.
    ::

      -- Europe/Berlin
      timezone = 'CET-1CEST,M3.5.0,M10.5.0/3'

ntp_server
    List of NTP servers available in your community or used by your community, e.g.:
    ::

       ntp_servers = {'1.ntp.services.ffeh','2.tnp.services.ffeh'}

opkg : optional
    ``opkg`` package manager configuration.

    There are two optional fields in the ``opkg`` section:

    - ``openwrt`` overrides the default OpenWrt repository URL
    - ``extra`` specifies a table of additional repositories (with arbitrary keys)

    ::

      opkg = {
        openwrt = 'http://opkg.services.ffeh/openwrt/%n/%v/%S/packages',
        extra = {
          modules = 'http://opkg.services.ffeh/modules/gluon-%GS-%GR/%S',
        },
      }

    There are various patterns which can be used in the URLs:

    - ``%n`` is replaced by the OpenWrt version codename (e.g. "chaos_calmer")
    - ``%v`` is replaced by the OpenWrt version number (e.g. "15.05")
    - ``%S`` is replaced by the target architecture (e.g. "ar71xx/generic")
    - ``%GS`` is replaced by the Gluon site code (as specified in ``site.conf``)
    - ``%GV`` is replaced by the Gluon version
    - ``%GR`` is replaced by the Gluon release (as specified in ``site.mk``)

regdom : optional
    The wireless regulatory domain responsible for your area, e.g.:
    ::

      regdom = 'DE'

    Setting ``regdom`` in mandatory if ``wifi24`` or ``wifi5`` is defined.

wifi24 : optional
    WLAN configuration for 2.4 GHz devices.
    ``channel`` must be set to a valid wireless channel for your radio.

    There are currently three interface types available. You many choose to
    configure any subset of them:

    - ``ap`` creates a master interface where clients may connect
    - ``mesh`` creates an 802.11s mesh interface with forwarding disabled
    - ``ibss`` creates an ad-hoc interface

    Each interface may be disabled by setting ``disabled`` to ``true``.
    This will only affect new installations.
    Upgrades will not changed the disabled state.

    ``ap`` requires a single parameter, a string, named ``ssid`` which sets the interface's ESSID.

    ``mesh`` requires a single parameter, a string, named ``id`` which sets the mesh id.

    ``ibss`` requires two parametersr: ``ssid`` (a string) and ``bssid`` (a MAC).
    An optional parameter ``vlan`` (integer) is supported.

    Both ``mesh`` and ``ibss`` accept an optional ``mcast_rate`` (kbit/s) parameter for setting the default multicast datarate.
    ::

       wifi24 = {
         channel = 11,
         ap = {
           ssid = 'entenhausen.freifunk.net',
         },
         mesh = {
           id = 'entenhausen-mesh',
           mcast_rate = 12000,
         },
         ibss = {
           ssid = 'ff:ff:ff:ee:ba:be',
           bssid = 'ff:ff:ff:ee:ba:be',
           mcast_rate = 12000,
         },
       },

wifi5 : optional
    Same as `wifi24` but for the 5Ghz radio.

next_node : package
    Configuration of the local node feature of Gluon
    ::

      next_node = {
        ip4 = '10.23.42.1',
        ip6 = 'fdca:ffee:babe:1::1',
        mac = 'ca:ff:ee:ba:be:00'
      }

mesh : optional
    Options specific to routing protocols.

    At the moment, only the ``batman_adv`` routing protocol has such options:

    The optional value ``gw_sel_class`` sets the gateway selection class. The default
    class 20 is based on the link quality (TQ) only, class 1 is calculated from
    both the TQ and the announced bandwidth.
    ::

       mesh = {
         batman_adv = {
           gw_sel_class = 1,
	 },
       }


fastd_mesh_vpn
    Remote server setup for the fastd-based mesh VPN.

    The `enabled` option can be set to true to enable the VPN by default.

    If `configurable` is `false` or unset, the method list will be replaced on updates
    with the list in the site configuration. Setting `configurable` to `true` will allow the user to
    add the method ``null`` to the front of the method list or remove ``null`` from it,
    and make this change survive updates. Settings configurable is necessary for the
    package `gluon-luci-mesh-vpn-fastd`, which adds a UI for this configuration.

    In any case, the ``null`` method should always be the first method in the list
    if it is supported at all. You should only set `configurable` to `true` if the
    configured peers support both the ``null`` method and methods with encryption.
    ::

      fastd_mesh_vpn = {
        methods = {'salsa2012+umac'},
	-- enabled = true,
	-- configurable = true,
        mtu = 1280,
        groups = {
          backbone = {
            limit = 1,
            peers = {
              peer1 = {
                key = 'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX',
                remotes = {'ipv4 "vpn1.entenhausen.freifunk.net" port 10000'},
              },
            }
          }
        },

        bandwidth_limit = {
          -- The bandwidth limit can be enabled by default here.
          enabled = false,

          -- Default upload limit (kbit/s).
          egress = 200,

          -- Default download limit (kbit/s).
          ingress = 3000,
        },
      }

mesh_on_wan : optional
    Enables the mesh on the WAN port (``true`` or ``false``).

mesh_on_lan : optional
    Enables the mesh on the LAN port (``true`` or ``false``).

autoupdater : package
    Configuration for the autoupdater feature of Gluon.
    ::

      autoupdater = {
        branch = 'experimental',
        branches = {
          stable = {
            name = 'stable',
            mirrors = {
              'http://[fdca:ffee:babe:1::fec1]/firmware/stable/sysupgrade/',
              'http://[fdca:ffee:babe:1::fec2]/firmware/stable/sysupgrade/',
            },
            good_signatures = 2,
            pubkeys = {
              'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX', -- someguy
              'XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX', -- someother
            }
          }
        }
      }

roles : optional
    Optional role definitions. With this nodes will announce their role inside the mesh.
    In the backend this adds the facility to distinguish between normal, backbone and
    service nodes or even gateways (if they advertise the role, also). It is up to
    the community which roles to define. See the section below as an example.
    ``default`` takes the default role which is set initially. This value should be
    part of ``list``. If you want node owners to change the role via config mode add
    the package ``gluon-luci-node-role`` to ``site.mk``.

    The strings to display in the LuCI interface can be configured per language in the
    ``i18n/en.po``, ``i18n/de.po``, etc. files of the site repository using message IDs like
    ``gluon-luci-node-role:role:node`` and ``gluon-luci-node-role:role:backbone``.
    ::

      roles = {
        default = 'node',
        list = {
          'node',
          'test',
          'backbone',
          'service',
        },
      },

setup_mode : package
    Allows skipping setup mode (config mode) at first boot when attribute
    ``skip`` is set to ``true``. This is optional and may be left out.
    ::

      setup_mode = {
        skip = true,
      },

legacy : package
    Configuration for the legacy upgrade path.
    This is only required in communities upgrading from Lübeck's LFF-0.3.x.
    ::

      legacy = {
             version_files = {'/etc/.freifunk_version_keep', '/etc/.eff_version_keep'},
             old_files = {'/etc/config/config_mode', '/etc/config/ffeh', '/etc/config/freifunk'},
             config_mode_configs = {'config_mode', 'ffeh', 'freifunk'},
             fastd_configs = {'ffeh_mesh_vpn', 'mesh_vpn'},
             mesh_ifname = 'freifunk',
             tc_configs = {'ffki', 'freifunk'},
             wifi_names = {'wifi_freifunk', 'wifi_freifunk5', 'wifi_mesh', 'wifi_mesh5'},
      }

Packages
--------

The ``site.mk`` is a Makefile which should define constants
involved in the build process of Gluon.

GLUON_SITE_PACKAGES
    Defines a list of packages which should installed additional
    to the ``gluon_core`` package.

GLUON_RELEASE
    The current release version Gluon should use.

GLUON_PRIORITY
    The default priority for the generated manifests (see the autoupdater documentation
    for more information).

GLUON_LANGS
    List of languages (as two-letter-codes) to include for the web interface. Should always contain
    ``en``.

.. _site-config-mode-texts:

Config mode texts
-----------------

The community-defined texts in the config mode are configured in PO files in the ``i18n`` subdirectory
of the site configuration. The message IDs currently defined are:

gluon-config-mode:welcome
    Welcome text on the top of the config wizard page.

gluon-config-mode:pubkey
    Information about the public VPN key on the reboot page.

gluon-config-mode:reboot
    General information about the reboot page.

There is a POT file in the site example directory which can be used to create templates
for the language files. The command ``msginit -l en -i ../../docs/site-example/i18n/gluon-site.pot``
can be used from the ``i18n`` directory to create an initial PO file called ``en.po`` if the ``gettext``
utilities are installed.

.. note::

   An empty ``msgstr``, as is the default after running ``msginit``, leads to
   the ``msgid`` being printed as-is. It does *not* hide the whole text, as
   might be expected.

   Depending on the context, you might be able to use comments like
   ``<!-- empty -->`` as translations to effectively hide the text.

Examples
--------

site.mk
^^^^^^^

.. literalinclude:: ../site-example/site.mk
  :language: makefile

site.conf
^^^^^^^^^

.. literalinclude:: ../site-example/site.conf
  :language: lua

i18n/en.po
^^^^^^^^^^

.. literalinclude:: ../site-example/i18n/en.po
  :language: po

i18n/de.po
^^^^^^^^^^

.. literalinclude:: ../site-example/i18n/de.po
  :language: po

modules
^^^^^^^

.. literalinclude:: ../site-example/modules
  :language: makefile

site-repos in the wild
^^^^^^^^^^^^^^^^^^^^^^

This is a non-exhaustive list of site-repos from various communities:

* `site-ffbs <https://github.com/ffbs/site-ffbs>`_ (Braunschweig)
* `site-ffhb <https://github.com/FreifunkBremen/gluon-site-ffhb>`_ (Bremen)
* `site-ffda <https://github.com/freifunk-darmstadt/site-ffda>`_ (Darmstadt)
* `site-ffgoe <https://github.com/freifunk-goettingen/site-ffgoe>`_ (Göttingen)
* `site-ffhh <https://github.com/freifunkhamburg/site-ffhh>`_ (Hamburg)
* `site-ffhgw <https://github.com/lorenzo-greifswald/site-ffhgw>`_ (Greifswald)
* `site-ffhl <https://github.com/freifunk-luebeck/site-ffhl>`_ (Lübeck)
* `site-ffmd <https://github.com/FreifunkMD/site-ffmd>`_ (Magdeburg)
* `site-ffmwu <https://github.com/freifunk-mwu/site-ffmwu>`_ (Mainz, Wiesbaden & Umgebung)
* `site-ffmyk <https://github.com/FreifunkMYK/site-ffmyk>`_ (Mayen-Koblenz)
* `site-ffm <https://github.com/freifunkMUC/site-ffm>`_ (München)
* `site-ffms <https://github.com/FreiFunkMuenster/site-ffms>`_ (Münster)
* `site-ffnw <https://git.freifunk-ol.de/root/siteconf.git>`_ (Nordwest)
* `site-ffpb <https://git.c3pb.de/freifunk-pb/site-ffpb>`_ (Paderborn)
* `site-ffka <https://github.com/ffka/site-ffka>`_ (Karlsruhe)
* `site-ffrl <https://github.com/ffrl/sites-ffrl>`_ (Rheinland)
* `site-ffrg <https://github.com/ffruhr/site-ffruhr>`_ (Ruhrgebiet)
* `site-ffs <https://github.com/freifunk-stuttgart/site-ffs>`_ (Stuttgart)
* `site-fftr <https://github.com/freifunktrier/site-fftr>`_ (Trier)
