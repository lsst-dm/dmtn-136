Limitations
===========

Because of the action taken under DM descope option DM-10, the original design for the Portal was not able to be carried out in full.
This note largely describes the Portal software as it stands following the end of the ramp-down of LSST funding for development by Caltech/IPAC.
In some places it refers to future directions which could be taken up again to provide improved capabilities.


Operations Concept
==================

The interaction of the Portal Aspect with LSST Data is primarily mediated by IVOA services, with the possible addition of non-standard services identified to the Portal Aspect software by IVOA DataLink records.

The principal means of data discovery is through one or more LSST-provided TAP services.
The TAP services provide access to object- and source-oriented catalogs, as well as to catalogs of image metadata.
The primary image metadata catalog will be available through the ObsTAP standard of presenting a single table at the IVOA-defined name "ivoa.ObsCore".

The Portal provides catalog queries simply as TAP queries against the corresponding tables in the LSST databases.
It provides image queries in the form of TAP queries against image metadata tables, and recognizes tables with ObsCore-style schemas to provide a basic image browsing capability.
Image cutout services are intended to be provided via IVOA SODA service; however, the work to connect to those services was not completed by the end of the ramp-down period.

In the 2019 state of the LSP, each "LSP instance" provides its own TAP endpoint, relative to the instance's base URL.
This facilitates setting up multiple instances with access to significantly different datasets, or, potentially, views of the same underlying datasets.
The URL conventions involved are defined in Document XXX.

The post-DM-10 Portal is entirely metadata-driven and contains little or no knowledge of LSST-specific details other than the URL conventions above.
The original Portal design envisioned augmenting these "generic" data access interfaces with more user-friendly curated screens that guided LSST users to the most important datasets and the most commonly useful starting points.
This work was unsustainable without continued IPAC work to follow the LSST data model as it evolved.

The TAP_SCHEMA mechanism defined in the TAP 1.1 standard is used to permit the Portal to self-configure to the available data.
As of 2019 no LSST-specific additions to this are required, though in the original Portal design some optional additional metadata would have facilitated providing a richer user interface.


Implementation
==============

Firefly Core
------------

The Portal is based on the Caltech/IPAC-written open-source "Firefly" client-server application.

Firefly provides components for FITS and HiPS image visualization, tabular data query, tabular data display, and a scientific visualization environment for tabular data, based on the Plotly.js library, with extensive brushing, filtering, and linking capabilities, and supporting the overlay of tabular data in sky coordinates on images.

The core Firefly library provides support for UI-based and native ADQL queries against TAP services.
It also comes with support for legacy non-IVOA interfaces to all the major IRSA data holdings.
During LSST commissioning these will be left in place in the expectation that they will provide useful adjunct data for users attempting to place the LSST data into the context of other all-sky surveys such as 2MASS and WISE/NEOWISE, but will probably be phased out in favor of a TAP-only interface to external datasets in the LSST Operations era.

The Firefly package, and a small set of related packages detailed below, are all maintained on an ongoing basis by IPAC.
The code is on Github and is maintained as open-source.
The IPAC Firefly team creates tags and releases of Firefly in source form as well as in the form of a "default application" that provides a large subset of the capabilities of the post-DM-10 Portal Aspect.
This "Firefly application" is released as a Docker image on Dockerhub in the ipac/firefly repository.


The "suit" Application
----------------------

The Portal application is constructed via the LSST package lsst/suit, which provides a thin layer of overall application organization and options-setting over the core Firefly components and their default behaviors.

The name "suit" derives from the legacy name of the IPAC-based team in the distributed DM organization: Science User Interface and Tools (SUIT).
Following the later decision on the LSP nomenclature, however, the "suit" name has been de-emphasized in user-facing application presentations and documentation, in favor of "Portal Aspect" and related nomenclature.

The Portal application is built by combining the lsst/suit and Caltech-IPAC/firefly code at source-code level, with build scripts from the Firefly core repository, and the release version of Firefly against which to build controlled by a file in the lsst/suit repository.
In this way, a tag of the lsst/suit repository also explicitly determines the version of Firefly to be used.

As of October 2019 the builds of the Portal application are still performed by IPAC, but it is planned to transfer this responsibility to LSST-DM in Tucson during this fiscal year, in order to conform to evolving DM release management procedures.
The present build system produces a Docker image on Dockerhub in the ipac/suit repository.


Languages
---------

The Firefly and Portal application code are written in a combination of Java (compatible with JDK 11 and JRE 11, the most recent long-term-support releases as of 2019), and JavaScript ES6.
The build process is based on Gradle.


Build and Release Process Details
=================================


Deployment Procedures
=====================

Debugging Deployments
---------------------


..
  Technote content.

  See https://developer.lsst.io/restructuredtext/style.html
  for a guide to reStructuredText writing.

  Do not put the title, authors or other metadata in this document;
  those are automatically added.

  Use the following syntax for sections:

  Sections
  ========

  and

  Subsections
  -----------

  and

  Subsubsections
  ^^^^^^^^^^^^^^

  To add images, add the image file (png, svg or jpeg preferred) to the
  _static/ directory. The reST syntax for adding the image is

  .. figure:: /_static/filename.ext
     :name: fig-label

     Caption text.

   Run: ``make html`` and ``open _build/html/index.html`` to preview your work.
   See the README at https://github.com/lsst-sqre/lsst-technote-bootstrap or
   this repo's README for more info.

   Feel free to delete this instructional comment.

:tocdepth: 1

.. Please do not modify tocdepth; will be fixed when a new Sphinx theme is shipped.

.. sectnum::

.. TODO: Delete the note below before merging new content to the master branch.

.. note::

   **This technote is not yet published.**

   This note will provide concise information about the operational concept for the LSP Portal Aspect, including the core Firefly software around which it is structured, as well as information about the deployment and maintenance of the Portal Aspect software.

.. Add content here.
.. Do not include the document title (it's automatically added from metadata.yaml).

.. .. rubric:: References

.. Make in-text citations with: :cite:`bibkey`.

.. .. bibliography:: local.bib lsstbib/books.bib lsstbib/lsst.bib lsstbib/lsst-dm.bib lsstbib/refs.bib lsstbib/refs_ads.bib
..    :style: lsst_aa
