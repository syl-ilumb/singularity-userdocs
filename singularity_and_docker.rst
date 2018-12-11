=====================================
Singularity and Docker/OCI containers
=====================================

.. TODO Singularity Hub ? <-- I would not worry about either Singularity Hub or
    .. the contaienr library on this page.  Both of those host containers that are 
    .. in native Singularity formats.  I think this page is more for interacting 
    .. with things in Docker or OCI format.  (feel free to delete comment.)

--------
Overview
--------

.. TODO Overview content ... 

.. TODO relocate below ??? 

.. Review the overview of the Sy interface ... 


.. ------------------------------------------
.. Making use of pre-built Singularity images
.. ------------------------------------------

.. SHUB and NVIDIA ... 


----------------------------
Interoperability with Docker
----------------------------

Running action commands on Docker Hub images
============================================

.. info about shell, run, and exec on Docker Hub images
.. explanation that layers are downloaded and then "spatted out to disk" to 
    .. create an ephemeral Singularity container in which commands are run


Making use of pre-built images from the Docker Hub
==================================================

Singularity can make use of pre-built images available from the `Docker Hub <https://hub.docker.com/>`_. By specifying the ``docker://`` URI for an image you have already located, Singularity can ``pull``  it - e.g.: 

.. code-block:: none

    $ singularity pull docker://godlovedc/lolcow
    WARNING: Authentication token file not found : Only pulls of public images will succeed
    INFO:    Starting build...
    Getting image source signatures
    Copying blob sha256:9fb6c798fa41e509b58bccc5c29654c3ff4648b608f5daa67c1aab6a7d02c118
     45.33 MiB / 45.33 MiB [====================================================] 2s
    Copying blob sha256:3b61febd4aefe982e0cb9c696d415137384d1a01052b50a85aae46439e15e49a
     848 B / 848 B [============================================================] 0s
    Copying blob sha256:9d99b9777eb02b8943c0e72d7a7baec5c782f8fd976825c9d3fb48b3101aacc2
     621 B / 621 B [============================================================] 0s
    Copying blob sha256:d010c8cf75d7eb5d2504d5ffa0d19696e8d745a457dd8d28ec6dd41d3763617e
     853 B / 853 B [============================================================] 0s
    Copying blob sha256:7fac07fb303e0589b9c23e6f49d5dc1ff9d6f3c8c88cabe768b430bdb47f03a9
     169 B / 169 B [============================================================] 0s
    Copying blob sha256:8e860504ff1ee5dc7953672d128ce1e4aa4d8e3716eb39fe710b849c64b20945
     53.75 MiB / 53.75 MiB [====================================================] 3s
    Copying config sha256:73d5b1025fbfa138f2cacf45bbf3f61f7de891559fa25b28ab365c7d9c3cbd82
     3.33 KiB / 3.33 KiB [======================================================] 0s
    Writing manifest to image destination
    Storing signatures
    INFO:    Creating SIF file...
    INFO:    Build complete: lolcow_latest.sif

This ``pull`` results in a *local* copy of the Docker image in SIF, the Singularity Image Format:

.. code-block:: none

    $ file lolcow_latest.sif 
    lolcow_latest.sif: a /usr/bin/env run-singularity script executable (binary data)

In translating to SIF, individual layers of the Docker image have been *combined* into a single, native file for use via Singularity; there is no need to subsequently ``build`` the image for Singularity. For example, you can now ``exec``, ``run`` or ``shell`` into the SIF version via Singularity. See :ref:`Interact with images<quick-start>`_. 

.. TODO improve ref above to quick start ... interact 
    .. Should explain here or in previous section that docker to Singularity is 
    .. a one-way operation because info is lost.
    .. Also some workds on how this is considered less reproducible than pulling
    .. from the container library.  
    .. Also should talk about how a build and a pull are really the same thing
    .. under the hood.  Both are really "builds" in the sense that the layers 
    .. are built into a Singularity image.

.. note:: 

    The above authentication warning originates from a check for the existence of ``${HOME}/.singularity/sylabs-token``. It can be ignored when making use of the Docker Hub. 

.. note:: 

    ``singularity search [search options...] <search query>`` does *not* support Docker registries like `Docker Hub <https://hub.docker.com/>`_. Use the search box at Docker Hub to locate Docker images. Docker ``pull`` commands, e.g., ``docker pull godlovedc/lolcow``, can be easily translated into the corresponding command for Singularity. The Docker ``pull`` command is available under "DETAILS" for a given image on Docker Hub. 

``inspect`` reveals metadata for the container encapsulated via SIF:

.. code-block::

        $ singularity inspect lolcow_latest.sif 

    {
        "org.label-schema.build-date": "Thursday_6_December_2018_17:29:48_UTC",
        "org.label-schema.schema-version": "1.0",
        "org.label-schema.usage.singularity.deffile.bootstrap": "docker",
        "org.label-schema.usage.singularity.deffile.from": "godlovedc/lolcow",
        "org.label-schema.usage.singularity.version": "3.0.1-40.g84083b4f"
    }

SIF files built from Docker images are *not* crytographically signed:

.. code-block::

    $ singularity verify lolcow_latest.sif 
    Verifying image: lolcow_latest.sif
    ERROR:   verification failed: error while searching for signature blocks: no signatures found for system partition

.. TODO Need to fix ref below ... 

The ``sign`` command allows a cryptographic signature to be added. Refer to 
:ref:`Signing and Verifying Containers <signNverify>` for details. But caution
should be exercised in signing images from Docker Hub because, unless you build
an image from scratch (OS mirrors) you are probably not really sure about the
complete contents of that image. 

.. note::

    ``pull`` actually builds a SIF file that corresponds to the image you retrieved from the Docker Hub. Updates to the image on the Docker Hub will *not* be reflected in your *local* copy. 

.. the line below should probaby be added to a larger discussion in which the 
.. entire URI is explained.  I think the existing explanation is pretty good,
.. but probably needs style edits. 
In our example ``docker://godlovedc/lolcow``, ``godlovedc`` specifies a Docker Hub user, whereas ``lolcow`` is the name of the repository. Adding the option to specifiy an image tag, the generic version of the URI is ``docker://<hub-user>/<repo-name>[:<tag>]``. `Repositories on Docker Hub <https://docs.docker.com/docker-hub/repos/>`_ provides additional details.


.. What about a private Docker repo ??? 
.. TODO What about private Docker registries? How does signing/verification work in that case? 



.. TODO Account for locally cached Docker images - further research required ...  

.. I suggest the following additional topics to round the page out.  Maybe we can 
.. carve off topics and work on the page together.
.. 
.. Using docker bootstrap agent in a def file (link to appendix)
..     Must figure out all of the CMD and ENTRYPOINT stuff.  afaik it has changed?
.. The breakdown of the URI is useful and should be retained (but edited)
..     https://www.sylabs.io/guides/2.6/user-guide/singularity_and_docker.html#how-do-i-specify-my-docker-image
.. Using custom authentication for a private Docker Hub repo (may need to set one
..     up for testing)
.. Using a different registry.  quay.io would provide a good example
.. How to use nvidia's cloud
.. build a singularity container from local docker images (ask Ian and/or Michael)
..     running in daemon
..     sitting on host 
.. build from an OCI bundle (ask Ian and/or Michael.)
.. The best practices section is also useful and should likely be retained


