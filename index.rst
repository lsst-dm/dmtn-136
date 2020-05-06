Introduction
============

This document describes the operations concept and deployment architecture for the Portal Aspect of the LSST Science Platform (LSP).

The Portal Aspect is constrained fairly loosely by high-level LSST requirements from the LSR and OSS, and by a few DM-level requirements from the DMSR
The primary constraints on its design come from the LSP Requirements, LDM-554.
The basic design concept is described in the LSP Vision document, LSE-319, and in the LSP Design document, LDM-542.

Limitations
-----------

Because of the action taken under DM descope option DM-10, the original design for the Portal Aspect was not able to be carried out in full during LSST construction.
This note largely describes the Portal software as it stands following the end of the ramp-down of LSST funding for development by Caltech/IPAC.
In some places it refers to future directions which could be taken up again to provide improved capabilities, but this document does not otherwise attempt to describe the originally envisioned, complete, Portal Aspect.


Operations Concept
==================

User Interaction
----------------

The Portal Aspect provides the following capabilities:

- TAP queries for access to available tables of catalogs, image metadata, and ingested EFD data, constructed either by UI or by explicit ADQL entry, and applicable both to LSST and external TAP services;
- Image and image metadata visualization, for both FITS and HiPS images, obtained in the following ways:

  - TAP queries to ObsCore-formatted image metadata, either in an ObsTAP service or in other tables with the ObsCore data model;
  - Upload of ObsCore-compatible tabular image metadata, either via the UI or the Python interface;
  - Upload of FITS or HiPS image data from a specified URL; or
  - Direct upload of FITS image data;

- Tabular data visualization and exploratory data analysis, including:

  - Display of data tables, with sorting, filtering, and row-selection capabilities;
  - Plotting of tabular data, including 1D and 2D histograms, line plots, and scatterplots, with the abilities of plotting mathematical transformations on the table variables and graphically selecting data; and
  - Overplotting of spatially organized tabular data on images;

- The ability to query all the IRSA image and catalog holdings; and
- A legacy capability to access the original "PDAC" LSST prototype data services.

The Portal Aspect is presented primarily as a Web application, but is also usable embedded in a JupyterLab session as a JupyterLab extension.
The latter capability is used in LSST to enable image and catalog visualization driven from Python code in the Notebook Aspect (Nublado).
This is possible both through the display-independent ``afw.display`` Python interface and through the Portal-specific ``firefly_client`` library.

Per LSST Project specification the Portal is intended to be accessible only to users who have authenticated and are authorized to use it.
In the 2019 state of LSP A&A, this is achieved by making the Portal entirely unavailable without an appropriate LSST authorization token; i.e., there is nothing like a limited-capability "anonymous user" version of the Portal.
Accordingly, only the LSP top-level landing page is visible to unauthenticated or unauthorized users.


Data Access
-----------

The interaction of the Portal Aspect with LSST Data is primarily mediated by IVOA services, with the possible addition of non-standard services identified to the Portal Aspect software by IVOA DataLink records.

The principal means of data discovery is through one or more LSST-provided TAP services.
The TAP services provide access to object- and source-oriented catalogs, as well as to catalogs of image metadata.

The primary image metadata catalog will be available through the ObsTAP standard of presenting a single table at the IVOA-defined name "ivoa.ObsCore".
However, at the time of writing this table is not yet available -- i.e., there is no ObsTAP service.
When that service becomes available, the Portal in its current form should permit querying image metadata and visualizing the images obtained.

The Portal Aspect provides for catalog queries simply as TAP queries against the corresponding tables in the LSST databases.

It provides for image queries in the form of TAP queries against image metadata tables, and recognizes tables with ObsCore-style schemas to provide a basic image browsing capability.
Image cutout services are intended to be provided via IVOA SODA service(s), ideally identified via DataLink records in the image metadata queries.
However, this plan was not able to be carried out before the DM-10 action, largely because of the lack of IVOA-compliant image services at that time.

The "PDAC"-era Portal provided for image cutouts, but these were obtained from the prototype non-IVOA DAX "imgserv" image service, and this capability will be lost once the legacy instance of that service is shut down.

LSST in any event does not yet provide a coordinated combination of an ObsTAP service from which image IDs can be obtained, and a SODA service with which those IDs can be used to create cutouts, let alone DataLink records which could be used to identify the SODA service from an image metadata query result.

In the 2019 state of the LSP, each "LSP instance" provides its own TAP endpoint, relative to the instance's base URL.
This facilitates setting up multiple instances with access to significantly different datasets (e.g., ones accessible only to project staff vs. ones usable by outside scientists for system evaluation) or different authorization requirements.

Other TAP services may be accessed via the Portal UI.
A limited list of commonly useful services is pre-configured into the application, and others may be specified by explicit URL entry.
No IVOA registry-query capability for service discovery is currently exposed, nor are the prototype LSST data services registered in any IVOA registry at this time.
It is not yet determined whether LSST will run its own Registry service.

The post-DM-10 Portal is entirely metadata-driven and contains little or no coded-in knowledge of LSST-specific details other than the URL conventions above.
The original Portal design envisioned augmenting these "generic" data access interfaces with more user-friendly curated screens that guided LSST users to the most important datasets and the most commonly useful starting points.
A basic version of this work was done for the original non-IVOA data services used in the "PDAC" prototype, but had not been replicated for the updated data service model by the time of the DM-10 ramp-down, partly because of the lack of IVOA image services.
In any event this curation and customization would be unsustainable without continued Portal development effort to follow the LSST data model as it evolves.

The TAP_SCHEMA mechanism defined in the TAP 1.1 standard is used to permit the Portal to self-configure to the available data.
As of 2019 no LSST-specific additions to this are required, though in the original Portal design some optional additional metadata -- compatible with the TAP_SCHEMA standard's documented points of extensibility -- would have facilitated providing a richer user interface.
For example, this might have permitted the provision of links to detailed documentation on tables and even table columns.

The Firefly TAP query capability that is used by the Portal implementation relies on synchronous TAP queries to obtain TAP_SCHEMA metadata, but then uses the asynchronous TAP protocol to execute user queries.
This permits the application to return a query-job URL through the UI, which can then be passed to other applications to obtain the results of the same query.


Deployment
----------

The Portal Aspect is a Web-based client-server application.
A containerized server process, implemented in Java, runs within the LSP deployment, providing both application services and the source of the HTML and JavaScript code required for the execution of the client side in the user's Web browser.

As with the rest of the Science Platform components, the Portal Aspect server-side application is intended to be deployed in Kubernetes.
To facilitate this, the server side code, as part of its release process, is built into a Docker image that contains both the compiled Java code to execute on the server and the HTML and JavaScript code to be served to the client's browser.

The actual server process is constructed using Apache Tomcat, and both the Tomcat code and a full Java run-time (JRE) are included in the Docker image.

Actual deployment in Kubernetes is controlled by a "Helm chart" created by the SQuaRE team with guidance from IPAC.
Each LSP instance can be configured to run a different named release of the Portal image.

The deployment is expected to follow the LSP convention that the Portal Web application is available at the URL *(LSP-instance-root-URL)*``/portal/app`` (the base ``/portal`` endpoint itself is reserved for an originally envisioned "start page" for the Portal, but this was not implemented post-DM-10).
The Portal code expects that, following the LSP convention, the Notebook Aspect will be deployed at *(LSP-instance-root-URL)*``/nb``, and the API Aspect at *(LSP-instance-root-URL)*``/api``, with the TAP service specifically at *(LSP-instance-root-URL)*``/api/tap``.
The URL conventions involved are defined in Document XXX.

As of early 2020, standing instances are maintained at NCSA at `https://lsst-lsp-stable.ncsa.illinois.edu/portal/app` and `https://lsst-lsp-int.ncsa.illinois.edu/portal/app`, with the latter used preferentially for testing of new releases.

The Kubernetes ingress rules for the Portal Aspect endpoints include authorization-based redirects as documented in DMTN-094.
The ingress is configured to require the user's authorization to include the ``exec-portal`` capability.
The Portal Aspect application is therefore not required to enforce any authorization test itself.
The application does retrieve the user's personal name from the ``Authorization: Bearer`` token in the post-redirect URL, and displays it in the application's header bar.
Note that the Portal Aspect implementation is required to be aware of the authorization protocols adopted for LSST in order to be able to construct TAP queries against the LSST services with the proper headers: it must pass on the ``Authorization: Bearer`` token as part of any LSST TAP HTTP requests.
Presently the query results obtained from the LSST asynchronous TAP service are at redirected URLs on external servers and do not require authorization headers.
Currently there is no adequate IVOA standard for having the application - or not - of these headers be service-metadata-driven, so, in short, there is contingent, LSST-specific knowledge in the Portal application in this area.
The present scheme is temporary; at least by the time of operations, it will be required that these result URLs are not universally accessible.
Portal modifications will be necessary to follow that change.
This presents a management complication and schedule risk for the LSP, because of the very limited IPAC resources available for maintenance; it will be important for there to be substantial advance notice to IPAC of when any change to this protocol is rolled out.

Note that because the ``exec-portal`` capability is not sufficient on its own to allow for TAP queries (where, at least, ``read-tap`` is required), the technical possibility exists for a user with an unusual set of capabilities to be able to access the Portal Aspect but not perform any TAP (including ObsTAP) queries and therefore not be able to see any LSST data.
This is not likely to be a situation that arises in practice.


Implementation
==============

Firefly Core
------------

The Portal is based on the Caltech/IPAC-written open-source "Firefly" client-server application (`see https://github.com/Caltech-IPAC/firefly`).

Firefly provides components for FITS and HiPS image visualization, tabular data query, and tabular data display, as well as a scientific visualization environment for tabular data, based on the Plotly.js library, with extensive brushing, filtering, and linking capabilities, and supporting the overlay of tabular data in sky coordinates on images.

The core Firefly library provides support for UI-based and native ADQL queries against TAP services.
It also comes with support for legacy non-IVOA interfaces to all the major IRSA data holdings.
During LSST commissioning these will be left in place in the expectation that they will provide useful adjunct data for users attempting to place the LSST data into the context of other all-sky surveys such as 2MASS and WISE/NEOWISE, but these may be phased out in favor of a TAP-only interface to external datasets in the LSST Operations era.

The Firefly package, and a small set of related packages detailed below, are all maintained on Github by IPAC.
The IPAC Firefly team creates tags and releases of Firefly in source form as well as in the form of a "default application" that provides a large subset of the capabilities of the post-DM-10 Portal Aspect, but without any LSST-specific knowledge or branding.
This "Firefly application" is released as a Docker image on Dockerhub in the ipac/firefly repository.


The "suit" Application
----------------------

The Portal application is constructed from code in the LSST package lsst/suit, which provides a thin layer of overall application organization and options-setting over the core Firefly components and their default behaviors.

The name "suit" derives from the legacy name of the IPAC-based team in the distributed DM organization: Science User Interface and Tools (SUIT).
Following the later decision on the LSP nomenclature, however, the "suit" name has been de-emphasized in user-facing application presentations and documentation, in favor of "Portal Aspect" and related nomenclature.

The Portal application is built by combining the lsst/suit and Caltech-IPAC/firefly code at source-code level, controlled by build scripts from the Firefly core repository and a thin layer of "suit"-specific build scripts and configuration in the "suit" repository, with the release version of Firefly against which to build controlled by a file in the lsst/suit repository.
In this way, a tag of the lsst/suit repository also explicitly determines the version of Firefly to be used.

As of October 2019 the builds of the Portal application are still performed by IPAC, but it is planned to transfer this responsibility to LSST-DM in Tucson during the 2020 fiscal year, in order to conform to evolving DM release management procedures.
The present build system produces a Docker image, with the "suit" release tag, on Dockerhub in the ipac/suit repository.


Languages
---------

The Firefly and Portal application code are written in a combination of Java (compatible with JDK 11 and JRE 11, the most recent long-term-support releases as of 2019), and JavaScript ES6.
The build process is based on Gradle.
For compatibility with other IPAC applications, as of early 2020 the production release builds are still performed with JDK 8, though the code is also compatible with JDK 11.
Production builds will move to JDK 11 over time; the schedule for this work at IPAC is not yet determined as of the time of writing.

The run-time environment in the Docker images is based on (OS version), (JRE version), and (Tomcat version) as of December 2019.


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
