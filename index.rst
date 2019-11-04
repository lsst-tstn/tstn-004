

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. note::

   This document describes the procedures for observing with the auxiliary telescope in the early
   stages of system integration and commission. The procedures are likely to change very rapidly in
   these stages so it is recommended that users keep a close eye on the document before doing any
   observations. Here we will also document some troubleshooting to commonly found issues. In case
   of questions contact the document authors.

Introduction
============

It is important to emphasize that this document focus on early operations of the Auxiliary
Telescope, with a significant part of the hardware and software still in need of considerable
advancements. For instance, we are still in the process of obtaining and validating the
telescope pointing model and optical alignment.

There is also some significant distinctions in the way we operate the telescope at this early
stages and the way we plan to operate during commissioning and, even more, during normal
operations. At this point we maintain a "low-level" kind of operations, both due to the need
to be in full control of all the components and simply because the lack of high level
operation software available. In fact, we are in the process of developing high level software
that will improve considerably the user experience as well as pave the way for the development of
"SAL Scripts" that ultimately will power the observatory
`Script Queue <https://ts-scriptqueue.lsst.io>`_.

Furthermore, some issues
pointed in this document may have been corrected in the meantime so it is very likely
that the document will contain some outdated information. We will make an effort to document
corrections as they occur but be aware that changes may happen faster then we are able to update
this document. Users may want to check for more updated versions in the edition navigation page or
check the `github repository <https://github.com/lsst-tstn/tstn-004>`_ for any pending pull
requests.

.. _net-arch:

Network architecture and connectivity
=====================================

From the user perspective, the summit network can broken down into two main systems; the campus
and the control network. Regardless if you are at the summit, the base or in Tucson if you are
connected to the LSST network (e.g. LSST-WAP) you will have access to the summit network. The
control network on the other side is only accessible from bastion computers on the summit. These
bastions are connected to both the campus network and the control network, thus giving users
access to the control network through :ref:`ssh tunneling <ssh-rules>`.

  .. attention::
     An important aspect of the control network is that it **does not** have access to the internet.
     This does create some issues, for instance, to update software on computers connected solely to
     the control network on the fly.

A list of the host computers IP address can be found
`here <https://confluence.lsstcorp.org/x/qw6SBg>`_.

.. _ssh-rules:

Useful ssh Tunneling rules
--------------------------

Paste the following rules to :file:`~/.ssh/config` on the computer you plan on using for the
observations.

This rule will enable access to a :ref:`jupyter notebook server <jupyter>`. Currently each user is
given a jupyter server running on a separate docker container. Each container has it's own IP
address on the control network and each user receive it's own token to access the server. In the
future we will use DM LSP system. Instructions will be updated accordingly.

.. code-block:: basic
   :name: chile-jupyter

     Host chile-jupyter
          Hostname 139.229.162.118
          User <username>
          LocalForward 8885 192.168.1.2??:8885

This rule will enable connection to the GenericCamera :ref:`live view server <liveview>`.

.. code-block:: basic
   :name: chile-liveview

     Host chile-liveview
          Hostname 139.229.162.118
          User <username>
          LocalForward 8881 192.168.1.218:8888

This rule is to enabled ``wget`` on GenericCamera to download :ref:`fits images <take-image>`.

  .. warning::
     This will be deprecated once proper LFA handling is implemented.

.. code-block:: basic
   :name: chile-wget

     Host chile-wget
          Hostname 139.229.162.118
          User <username>
          LocalForward 8001 192.168.1.216:8000

This rule is to connect to the machine that hosts the liveview server (see :ref:`issue-liveview`).

.. code-block:: basic
   :name: liveview-host

     Host liveview-host
          Hostname 139.229.162.114
          User <username>


Once the rule is appended to :file:`~/.ssh/config` it should be possible to just entry
``ssh <Host>``. The user will enter the bastion machine specified in the ``Hostname`` entry
on the rule and the tunnel specified on ``LocalForward`` will be set and the service will
be available on the user ``localhost:<local-port>``. The format of the ``LocalForward`` parameter
is ``<local-port> <remote-host>:<remote-port>``. Feel free to change ``<local-port>`` to any
suitable range on the machine used for the observations.

It is also possible to send the ssh command to
the background while tunneling by adding the options ``-N -f``.

To log in to the notebook server;

.. prompt:: bash

   ssh -N -f chile-jupyter

and open the address ``localhost:8885`` on a browser.

To open the liveview;

.. prompt:: bash

   ssh -N -f chile-liveview

and open the address ``localhost:8881`` on a browser.

To download fits images taken with the Generic Camera;

.. prompt:: bash

   ssh -N -f chile-wget
   wget http://localhost:8001/<image_name>


.. _tools:

Monitoring and Interactive tools
================================

Here is a list of currently available tools to monitor and interact with the system and a quick
overview of how to use them. More details on how to perform specific tasks with the telescope
are described :ref:`furthermore <ops>`.

.. _efd:

Engineering and Facility Database (EFD)
---------------------------------------

The EFD is responsible for listening to and storing all data (Telemetry, Event, Commands and
Acknowledgements) sent by components and users interacting with the components. The most recent
incarnation of the EFD uses an influx database to store the data in a time-series database.
See `sqr-034 <https://sqr-034.lsst.io>`_ for details about the EFD implementation.

Data from the summit is available on chronograf and can be accessed at
`<http://summit-chronograf-efd.lsst.codes/>`_.

On the left hand side of the web page there is a tab with links to the different actions one can
perform with chronograf. Probably the two most useful tabs are ``Dashboards`` and
``Explore``; the first will take you to a list of available dashboards created by users that
gathers important information about the subsystems. The "Summary state monitor" is a good
example of general information one would be interested in during an observing night.

.. important::
    The chronograf dashboards are shared between all users. If you feel like you need
    to make any change to the already existing dashboards, make sure to create a copy
    of the one you plan on editing and change that one instead.

The ``Explore`` tab let users perform free hand queries to the database using a sql-like
language.

.. _jupyter:

Jupyter Lab Servers
-------------------

Jupyter notebooks are very popular amongst astronomers, specially in the LSST collaboration.
They provide an easy and simple way of running Python code interactively through a
web browser and give the additional benefit of combining documentation (using markdown language)
and code. Users can run Jupyter notebook server locally on their own machine or on servers,
enabling a cloud-like environment with access to powerful computing or, in the case of
the LSST control system, to specialized functionality.

The most recent incarnation of Jupyter notebooks is Jupyter Labs. It provides access to a similar
environment as that of a notebooks but with additional functionality.

We envision that Jupyter Lab servers will be a fundamental part of LSST control system,
enabling users to perform low and high level operations in a well-known and
interactive environment.

The current system deployment uses individual Jupyter Lab servers for each user running out of
individual docker containers. In the nearby future, the plan is to start using DM LSP
(:cite:`LDM-542`) environment to manage user servers and environment.

The first step to access a Jupyter Lab server is to tunnel using the IP and token information
provided for each user by the Telescope & Site team point of contact. Once the ssh tunnel is
up it should be possible to open a web page on ``localhost:<local-port>``. After entering
the provided token, you should see the :ref:`Jupyter Lab interface <fig-jupyter-interface>`.

.. figure:: /_static/jupyter_interface.jpg
   :name: fig-jupyter-interface
   :target: ../_images/jupyter_interface.jpg
   :alt: Jupyter interface

In the left hand there is a file browser navigation screen which, by default, have two directories;
:file:`develop` and :file:`repos`. The :file:`develop` directory is a bind mount on the server that
runs the Jupyter Lab containers. Inside there is a repository for notebooks
(:file:`develop/ts_notebooks`) with examples and work notebooks from other users (separated by
username). Feel free to browse and edit any notebook within this repo. Be sure to commit and push
any work you may have done and eventually make Pull Request to the original repo so other users can
see and use work that was done.

The :file:`repos` directory, on the other hand, contains some basic repos that ships with the
container with the T&S software used to power the control system. Any data in this directory,
or in the home folder, will be lost if the container is restarted. It is advisable to only keep
important data inside the user designated folder (e.g. :file:`develop`).

.. _love:

LSST Operations and Visualization Environment (LOVE)
----------------------------------------------------

.. note::
    TBD

.. _queue:

Script Queue
------------

.. note::
    TBD


.. _csc:

Auxiliary Telescope Commandable SAL Components (CSCs)
=====================================================


:ref:`This diagram <fig-attcs>` shows all the CSCs (light blue boxes) that are currently being
used at the summit, their connections, the users jupyter servers and the salkafka producer that is
responsible for capturing all SAL traffic, serialize it in avro an send it over to kafka to be
inserted on the influx database (see `sqr-034 <https://sqr-034.lsst.io>`_ for more information
about the EFD).

.. figure:: /_static/ATTCS.jpg
   :name: fig-attcs
   :target: ../_images/ATTCS.jpg
   :alt: AuxTel components


.. _ops:

Basic Operations Procedures
===========================

This section explains how one can perform the basic operations with the telescope using the
Jupyter Lab server. Here we assume you was able to login to the server assigned to you and
either open an existing notebook or create an empty one to work with.

.. important::
    You will noticed that most of the tasks shown here will have two ways of being performed,
    using the high level software and low level software. At the time of this writing the
    low level controls, where the user sends commands to individual CSCs and have little
    feed back, are the only ones tested on sky. The high level operations, as one can see,
    provides a much easier way to execute these operations, but have not been sanctioned yet.

.. note::
    Notebooks with the procedures can be found on the :file:`develop/ts_notebooks/examples`
    folder.

.. _startup:

Startup procedure
-----------------

At the end of the day, before observations starts, most CSCs will be unconfigured and
in ``STANDBY`` state. The first step in starting up the system is to enable all CSCs.
Putting a CSC in the ``ENABLED`` state required the transition from ``STANDBY`` to
``DISABLED`` and then from ``DISABLED`` to ``ENABLED``. When transitioning from
``STANDBY`` to ``DISABLED`` it is possible to provide a ``settingsToApply`` that selects
a configuration for the CSC. Some CSCs won't need any settings while others will.
It is possible to check what are the available settings by looking at the ``settingVersions``
event.

After all CSCs are in the ``ENABLED`` state, we proceed to open the dome slit, setup the
ATPneumatics and startup the ATAOS. After the dome has finished opening, the telescope
covers are opened and the procedure is complete.

.. note::
    In some cases, if the dome controller is restarted, the dome will need to be homed. At the time
    of this writing there is no fixture that allow the procedure to be executed without human
    intervention. The process is documented in :ref:`issues`.

The startup procedure is encapsulated in the task ``startup()`` from the ``ATTCS`` class provided
by ``ts_standardscript``, which is available in the Jupyter server. The high level operation can be
run by doing the following:

::

    from lsst.ts.standardscripts.auxtel.attcs import ATTCS

    attcs = ATTCS()
    await attcs.start_task
    settings = {"atdome": "test.yaml":, "ataos" : "measured_20190908.yaml", "athexapod": "Default1"}
    await attcs.startup(settings)

Although this procedure implements all the basic steps and checks, it has not been tested at the
telescope yet. For now the sanctioned procedure is to execute this series of commands on a jupyter
notebooks. This is ta

::

    from lsst.ts import salobj

Setup remotes for all the AT components

::

    d = salobj.Domain()

::

    atmcs = salobj.Remote(d, "ATMCS")
    atptg = salobj.Remote(d, "ATPtg")
    ataos = salobj.Remote(d, "ATAOS")
    atpne = salobj.Remote(d, "ATPneumatics")
    athex = salobj.Remote(d, "ATHexapod")
    atdome = salobj.Remote(d, "ATDome", index=1)
    atdomtraj = salobj.Remote(d, "ATDomeTrajectory")

::

    await asyncio.gather(atmcs.start_task,
                         atptg.start_task,
                         ataos.start_task,
                         atpne.start_task,
                         athex.start_task,
                         atdome.start_task,
                         atdomtraj.start_task)

Enable all components.

::

    await asyncio.gather(salobj.set_summary_state(atmcs, salobj.State.ENABLED, timeout=120),
                         salobj.set_summary_state(atptg, salobj.State.ENABLED),
                         salobj.set_summary_state(ataos, salobj.State.ENABLED, settingsToApply="measured_20190908.yaml"),
                         salobj.set_summary_state(atpne, salobj.State.ENABLED),
                         salobj.set_summary_state(athex, salobj.State.ENABLED, settingsToApply="Default1"),
                         salobj.set_summary_state(atdome, salobj.State.ENABLED, settingsToApply="test.yaml"),
                         salobj.set_summary_state(atdomtraj, salobj.State.ENABLED))

Open dome shutter

::

    await atdome.cmd_moveShutterMainDoor.set_start(open=True)

Wait until the dome in fully open, then execute the next step to open the telescope
cover

::

    await atpne.cmd_openM1Cover.start()

Finally, enable the ATAOS correction for the M1 pressure.

::

    await ataos.cmd_enableCorrection.set_start(m1=True)

If the dome needs to be homed then run the following command:

::

    await atdome.cmd_homeAzimuth.start()

.. _pointing:

Pointing
--------

The action of pointing and start tracking involves sending a command to the pointing component
(``ATPtg``) and then waiting for the telescope and dome to be in position while making sure
:ref:`all components <fig-attcs>` remain in ``ENABLED`` state.


When using the ``ATTCS`` class it is possible to perform the task with the following set of
commands:

::

    import asyncio

    import astropy.units as u
    from astropy.time import Time
    from astropy.coordinates import ICRS, Angle

    from lsst.ts.standardscripts.auxtel.attcs import ATTCS


Initializing ``ATTCS`` class.

::

    attcs = ATTCS()
    await attcs.start_task

Run the slew task. This task will only finish when the telescope and the dome are
positioned. Also, this will set the sky position angle (angle between y-axis and North) to
be zero (or 180. if zero is not achievable). It is posssible to user RA/Dec and rotator
as hexagesimal strings or floats (and mix and match them). For instance,

::

    await attcs.slew_icrs(ra="20:25:38.85705", dec="-56:44:06.3230", sky_pos=0., target_name="Alf Pav")

or 

::

    await attcs.slew_icrs(ra=20.42746, dec=-56.73508, sky_pos=0., target_name="Alf Pav")


It is also possible to slew to an RA/Dec target and request a rotator position. To do that use the
``rot_pos`` argument instead of ``sky_pos``. Note that this will request ``rot_pos`` at the
requested time, which will change as the telescope track the object.

::

    await attcs.slew_icrs(ra="20:25:38.85705", dec="-56:44:06.3230", rot_pos=0., target_name="Alf Pav")

As with the :ref:`startup` procedure, this task has not been tested at the telescope yet.
For now the sanctioned procedure is to execute the slew and track by commanding the pointing
component individually. This also means the user have to handle the rotator angle
computations. In this mode we only support setting the rotator position to a certain angle.
Due to some binding issues we have been trying to keep the rotator as close to zero as possible.

::

    import logging
    import yaml

    import numpy as np
    from matplotlib import pyplot as plt
    import astropy.units as u
    from astropy.time import Time
    from astropy.coordinates import AltAz, ICRS, EarthLocation, Angle, FK5
    import asyncio

    from lsst.ts import salobj

    from lsst.ts.idl.enums import ATPtg

::

    from astropy.utils import iers
    iers.conf.auto_download = False

::

    d = salobj.Domain()

::

    atmcs = salobj.Remote(d, "ATMCS")
    atptg = salobj.Remote(d, "ATPtg")
    ataos = salobj.Remote(d, "ATAOS")
    atpne = salobj.Remote(d, "ATPneumatics")
    athex = salobj.Remote(d, "ATHexapod")
    atdome = salobj.Remote(d, "ATDome", index=1)
    atdomtraj = salobj.Remote(d, "ATDomeTrajectory")

::

    await asyncio.gather(atmcs.start_task,
                         atptg.start_task,
                         ataos.start_task,
                         atpne.start_task,
                         athex.start_task,
                         atdome.start_task,
                         atdomtraj.start_task)


The next cell sets the observatory location. This is needed to compute
the Az/El of the target to set the camera rotation angle. We are trying
to keep the angle close to zero.

::

    location = EarthLocation.from_geodetic(lon=-70.747698*u.deg,
                                           lat=-30.244728*u.deg,
                                           height=2663.0*u.m)

This next cell defines a target.

::

    ra = Angle("20:25:38.85705", unit=u.hour)
    dec = Angle("-56:44:06.3230", unit=u.deg)
    target_name="Alf PAv"
    radec = ICRS(ra, dec)

This next cell will slew to the target and set the camera rotation angle
to zero. Not that, unlike ``attcs.slew_icrs`` this call returns right away and does not
provide any feedback of when the telescope and dome arrives at the requested position.

::

    # Figure out what is the rotPA that sets nasmith rotator close to zero.
    time_data = await atptg.tel_timeAndDate.next(flush=True, timeout=2)
    curr_time_atptg = Time(time_data.tai, format="mjd", scale="tai")
    print(curr_time_atptg)
    coord_frame_altaz = AltAz(location=location, obstime=curr_time_atptg)
    alt_az = radec.transform_to(coord_frame_altaz)

    await atptg.cmd_raDecTarget.set_start(
        targetName=target_name,
        targetInstance=ATPtg.TargetInstances.CURRENT,
        frame=ATPtg.CoordFrame.ICRS,
        epoch=2000,  # should be ignored: no parallax or proper motion
        equinox=2000,  # should be ignored for ICRS
        ra=radec.ra.hour,
        declination=radec.dec.deg,
        parallax=0,
        pmRA=0,
        pmDec=0,
        rv=0,
        dRA=0,
        dDec=0,
        rotPA=180.-alt_az.alt.deg,
        rotFrame=ATPtg.RotFrame.FIXED,
        rotMode=ATPtg.RotMode.FIELD,
        timeout=10
    )

In case you need to stop tracking, use the next cell!

::

    await atptg.cmd_stopTracking.start(timeout=10)

Use the next cell in case you need to offset to center the target on the
FoV.

This will set total offsets. So, if you say ``el=0`` and ``az=-30`` and
then later you do ``el=30`` and ``az=0.``, it will reset the offset in
azimuth to zero and make an offset of 30arcs in elevation.

::

    await atptg.cmd_offsetAzEl.set_start(el=0.,
                                         az=-100.,
                                             num=0)

If you want to make persistent offsets you can use the following method.

::

    await atptg.cmd_offsetAzEl.set_start(el=0.,
                                         az=-100.,
                                         num=1)

If you want to add your offset to a pointing model file, do the
following.

::

    await atptg.cmd_pointNewFile.start()
    await asyncio.sleep(1.)
    await atptg.cmd_pointAddData.start()
    await asyncio.sleep(1.)
    await atptg.cmd_pointCloseFile.start()



.. _liveview:

Using GenericCamera Liveview
----------------------------

The GenericCamera liveview mode can be used for quick look of telescope pointing, to check
that a target is centered on the field after a slew was performed or to quickly evaluate the
optics. When liveview mode is activated, the GenericCamera CSC will start a web server and
start streaming the images taken with the selected exposure time. To visualize the images streamed
by the CSC we created a separate web server that connects to the CSC stream and display the images.
This is illustrated in the following :ref:`diagram <fig-liveview>`.

.. figure:: /_static/LiveView.jpg
   :name: fig-liveview
   :target: ../_images/LiveView.jpg
   :alt: AuxTel components


This is how to start live view in the GenericCamera;

::

    from lsst.ts import salobj
    import asyncio

::

    d = salobj.Domain()

::

    r = salobj.Remote(d, "GenericCamera", 1)

::

    await r.start_task

Before starting live view, make sure to enable the CSC with the 4x4 binning settings.

::

    await salobj.set_summary_state(r, salobj.State.ENABLED, settingsToApply="zwo_4x4.yaml")


When starting live view mode the user must specify the exposure time, which also sets the frame
rate of the stream. So far, we have tested this with up to 0.25s exposure times.

::

    await r.cmd_startLiveView.set_start(expTime=0.5)

Once live view has started, make sure you have the :ref:`live view ssh rule running <chile-wget>`,
then you should be able to access the live view server by opening ``localhost:8881`` on a
browser.

.. attention::
    The web server that streams the live view data is not in a stable state. If the browser is not
    loading the page you may have to check the process running the live view server and restart
    it. Check the :ref:`issues` session for more information about how to restart it.

To stop live view, you just need to run the following command.

::

    await r.cmd_stopLiveView.start(timeout=10)


.. _take-image:

Using GenericCamera to take (fits) images
-----------------------------------------

The GenericCamera CSC was designed to emulate the same behaviour as that of the
ATCamera and MTCamera CSCs. That means the commands and events have the same name and,
as much as possible, the same payload and the events marking the different stages of
image acquisition are also published at approximately the same stages.

To take an image with the GenericCamera first make sure that live view is not running. If
live view is running the take image command will be rejected. Then, to take an image:

::

    r.evt_endReadout.flush()
    await r.cmd_takeImages.set_start(numImages=1,
                                     expTime=10.,
                                     shutter=True,
                                     imageSequenceName='alf_pav'
                                    )

    end_readout = await r.evt_endReadout.next(flush=False, timeout=5.)

    print(end_readout.imageName)

You can download the image on your notebook server using the following command;

::

    import wget
    filename = wget.download(f"http://192.168.1.216:8000/{end_readout.imageName}.fits")

Note that this only works from the Jupyter notebook server as it is connected to the control
network.

You can download the image produced by the command above on your local computer
by running the following ``wget`` on the command line (make sure the
:ref:`chile-wget ssh rule is running <chile-wget>`).

.. prompt:: bash

   wget http://localhost:8001/<image_name>


.. _issues:

Troubleshooting
===============

Here we describe some of the currently known issues and how to resolve them.

.. _issue-atmcs:

ATMCS won't get out of FAULT State
----------------------------------

In some situations the ATMCS will go to ``FAULT`` state and it will reject the ``standby`` command,
preventing to recover the system. We have been working on tracking this issue down but,
should you encounter this issue it is possible to recover by pressing the e-stop button on
the main cabinet (close to the telescope pier) and on the dome cabinet (east building wall on lower
level) and then executing the recover procedure.

To recover from e-stop, release both e-stop buttons, press and hold the "start" button on the
dome cabinet for five seconds, then press and hold the "start" button on the main cabinet for
5s. This should clear the ``FAULT`` state and leave the ATMCS in ``STANDBY``.


.. _issue-liveview:

Live view server is not responding
----------------------------------

The :ref:`live view server <fig-liveview>` that is responsible for receiving images from the
GenericCamera and streaming it to a user we browser is still in a very rough shape. The server
connect to the GenericCamera over a TCP/IP socket and provides an image streaming server using a
simple tornado web server. The connector that is responsible for receiving images from the
CSC is still not capable of handling a dropped connection. That means, if there is a connection
issue it is not capable of regenerating and continuing operations. Moreover, if the liveview
mode is switched off on the CSC, the connection is also dropped and the live view server is also
not capable of reconnecting.

If any of this happens the easiest solution is to restart the live view server. For that, you
will need to connect to the container running the liveview server, kill the running procedure and
restarting the process. This can be summarized as follows;

.. prompt:: bash

    ssh liveview-host
    docker attach gencam_lv_server
    python liveview_server.py

Once the live view server is running you can detach from the container by doing ``Crtl+p Crtl+q``.

.. _build-idl:

Building CSC interfaces
-----------------------

To communicate with a CSC, we use a class provided by ``salobj`` called ``Remote``.
As you can see on previous sessions, the ``Remote`` receives the name of the CSC as
an argument, which ultimately, specifies the interface to load.

In order for the ``Remote`` to load this interface it needs to have the set of
idl libraries available. In some cases, the interface for the CSC that you plan
on communicating may no be readily available on the Jupyter notebook server. If
this is the case you will see an exception like the following when trying to
create the ``Remote``.

::

    ---------------------------------------------------------------------------
    RuntimeError                              Traceback (most recent call last)
    <ipython-input-2-470a83f93eee> in <module>
    ----> 1 r = salobj.Remote(salobj.Domain(), "Component")

    ~/repos/ts_salobj/python/lsst/ts/salobj/remote.py in __init__(self, domain, name, index, readonly, include, exclude, evt_max_history, tel_max_history, start)
        137             raise TypeError(f"domain {domain!r} must be an lsst.ts.salobj.Domain")
        138
    --> 139         salinfo = SalInfo(domain=domain, name=name, index=index)
        140         self.salinfo = salinfo
        141

    ~/repos/ts_salobj/python/lsst/ts/salobj/sal_info.py in __init__(self, domain, name, index)
        152         self.idl_loc = domain.idl_dir / f"sal_revCoded_{self.name}.idl"
        153         if not self.idl_loc.is_file():
    --> 154             raise RuntimeError(f"Cannot find IDL file {self.idl_loc} for name={self.name!r}")
        155         self.parse_idl()
        156         self.ackcmd_type = ddsutil.get_dds_classes_from_idl(self.idl_loc, f"{self.name}::ackcmd")

    RuntimeError: Cannot find IDL file /home/saluser/repos/ts_idl/idl/sal_revCoded_Component.idl for name='Component'

But, instead of ``Component`` it will be the name of the CSC you tried to connect to.
To resolve this issue, you will need to build the libraries. You can do that by putting the
following commands on a notebook cell:

::

    %%script bash
    make_idl_files.py <Component>

Again, you will need to replace ``<Component>`` by the name of the CSC.


.. Add content here.
.. Do not include the document title (it's automatically added from metadata.yaml).

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
    :style: lsst_aa
