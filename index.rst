Introduction
============

This document describes the operations concept and deployment architecture for the Portal Aspect of the LSST Science Platform (LSP).

The Portal Aspect is constrained fairly loosely by high-level LSST requirements from the LSR (LSE-29) and OSS (LSE-30), and by a few DM-level requirements from the DMSR (LSE-61).
The primary constraints on its design come from the LSP Requirements, LDM-554.
The basic design concept is described in the LSP Vision document, LSE-319, and elaborated in the LSP Design document, LDM-542.

Construction-era Task
---------------------

The development of the Portal Aspect was undertaken by the Caltech/IPAC group in LSST Data Management.
It evolved from the original "Science User Interface and Tools" (SUIT) concept in the LSST construction proposal into a component of the "Science Platform" concept which was introduced after the start of LSST construction.

Limitations
-----------

Because of the action taken under DM descope option DM-10, the original design for the Portal Aspect was not able to be carried out in full during LSST construction.
This note largely describes the Portal software as it stands following the end of the ramp-down of LSST funding for development by Caltech/IPAC.
In some places it refers to future directions which could be taken up again to provide improved capabilities in line with the original design, but this document does not attempt a comprehensive description of the originally envisioned, complete, Portal Aspect.


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
  - Overplotting of spatially organized tabular data on images; and

- The ability to transmit data to, and control, a Portal Aspect session from a Python client, at two levels:

  - A low-level, Portal-specific client library which provides access to almost all of the Portal's visualization functionality; and
  - An implementation of the LSST :py:mod:`lsst.afw.display` abstraction providing a high-level, device-independent interface, but with functionality limited to that available in the abstraction layer.

In addition, primarily as transitional capabilities, the present Portal Aspect implementation includes:

- The ability to query all the IRSA image and catalog holdings; and
- A legacy capability to access the original "PDAC" LSST prototype data services.

The Portal Aspect is presented primarily as a Web application, but is also usable embedded in a JupyterLab session as a JupyterLab extension.
The latter capability is used in LSST to enable image and catalog visualization driven from Python code in the Notebook Aspect (Nublado).
This is possible both through the display-independent :py:mod:`lsst.afw.display` Python interface and the Portal-specific :py:mod:`firefly_client` library, mentioned above.

Per LSST Project specification the Portal is intended to be accessible only to users who have authenticated and are authorized to use it.
In the 2019 state of LSP A&A, this is achieved by making the Portal entirely unavailable without an appropriate LSST authorization token; i.e., there is nothing like a limited-capability "anonymous user" version of the Portal.
Accordingly, only the LSP top-level landing page is visible to unauthenticated or unauthorized users.


Data Access
-----------

The interaction of the Portal Aspect with LSST Data is primarily mediated by IVOA services, with the possible addition of non-standard services identified to the Portal Aspect software by IVOA DataLink records.
(This capability is not presently enabled in the LSST data access services.)

The principal means of data discovery is through one or more LSST-provided TAP services.
The TAP services provide access to object- and source-oriented catalogs, as well as to catalogs of image metadata.

The primary image metadata catalog will be available through the ObsTAP standard of presenting a single table at the IVOA-defined name "ivoa.ObsCore".
At the time of writing this table is available only in prototype form, through an experimental ObsTAP service, and serves only metadata for the AllWISE atlas images at IPAC.
There is not yet a production LSST ObsTAP service providing access to metadata for LSST-hosted precursor image data.
When that service becomes available, the Portal in its current form should permit querying image metadata and visualizing the images obtained.

Catalog Queries
^^^^^^^^^^^^^^^

The Portal Aspect user interface provides for catalog queries simply as TAP queries against the corresponding tables in the LSST databases.

The graphical query builder provides an interface for selecting one of the available schemas in the service, and from there one of the tables in that schema.
Once a table is selected, the query builder identifies spatial coordinate columns, if present, and provides for cone and polygon searches against those columns.
Multi-object spatial searches are not yet implemented, post-DM-10, but the underlying LSST TAP service and databases also do not yet support the TAP ``UPLOAD`` mechanism that would be required to implement them efficiently.

The UI also provides for temporal-range searches against tables for which it can determine that a time column is present - or for a user-specified time column.
In the current version, the time column must be expressed in MJD, but times can be entered in human-readable forms as well as in MJD, and the UI will display the appropriate conversion.

The UI is designed to be extensible to provide similar specialized query assistants for other types of values - wavelength in particular was planned - but no additional ones have been implemented at this time.

The UI also displays the schema of the selected table.
This display includes checkboxes for selecting the columns to be retrieved by the query; they are initialized based on the ``principal`` Boolean attributes of the columns in `TAP_SCHEMA``, if available, to provide a basic "most commonly used" subset.
The display also allows for direct entry of constraints on both numeric and string columns' values in the query, in a simple syntax like "< 8.5".

As an alternative to the graphical query builder, the user can input ADQL queries directly.
The ADQL query screen contains a schema browser to assist the user with the input of table and column names; this is driven by the ``TAP_SCHEMA`` data available on the selected TAP service.

When the graphical query builder has been used to construct a query, the UI provides the user the option of executing it directly or of switching to the ADQL-entry screen with the constructed query already displayed.

Image Queries
^^^^^^^^^^^^^

The interface provides for image queries in the form of TAP queries against image metadata tables.
It recognizes tabular results with ObsCore-style schemas, and automatically displays them with a basic image-browsing interface added to the usual tabular-data visualization.
The original intent was to recognize ObsTAP services, and later on also to recognize CAOM2-compatible tablesets in TAP services, and provide an enhanced query UI customized to image metadata queries, beyond the basic catalog-query UI already provided.
This plan was not able to be carried out before the DM-10 action, largely because of the lack of IVOA-compliant image services in LSST at that time.
While a custom UI for image metadata ObsTAP queries is therefore not yet available, ObsTAP queries can still be performed through the standard TAP interface.
The principal limitation is that spatial queries in the graphical UI can only be carried out against the central coordinates of the image, not against its borders.
(The full range of ObsTAP queries can still be carried out in the Portal by typing explicit ADQL.)

Image cutout services were intended to be provided via IVOA SODA service(s), identified via DataLink records returnd from the image metadata queries.
Again, because of the lack of an IVOA compliant SODA service in LSST during the period of active Portal development, this plan had not been carried out before DM-10.
The "PDAC"-era Portal provided for image cutouts, and this capability is still present in the currently deployed Portal, but this functionality was based on the LSST prototype-era non-IVOA DAX "imgserv" image service, and this capability will be lost once the legacy instance of that service is shut down.

As of June 2020, LSST does not yet provide a coordinated combination of an ObsTAP service from which image IDs can be obtained, and a SODA service with which those IDs can be used to create cutouts, let alone DataLink records which could be used to identify the SODA service from an image metadata query result.

Service Selection
^^^^^^^^^^^^^^^^^

In the 2019 state of the LSP, each "LSP instance" provides its own TAP endpoint, accessible with a URL relative to the instance's base URL.
This facilitates setting up multiple instances of the LSP with access to significantly different datasets (e.g., ones accessible only to project staff vs. ones usable by outside scientists for system evaluation) or different authorization requirements.

Other TAP services, either within the LSP or external, may also be accessed via the Portal UI.
A limited list of commonly useful services is pre-configured into the application, and others may be specified by explicit URL entry.
No IVOA registry-query capability for service discovery is currently exposed, nor are the prototype LSST data services registered in any IVOA registry at this time.
It is not yet determined whether LSST will run its own Registry service.

Custonization
^^^^^^^^^^^^^

The post-DM-10 Portal is entirely metadata-driven and contains little knowledge of LSST-specific details other than the URL conventions above, and the need to forward credentials to the service.
The original Portal design envisioned augmenting these "generic" data access interfaces with more user-friendly curated screens that guided LSST users to the most important datasets and the most commonly useful starting points.
A basic version of this work was done for the original non-IVOA data services used in the "PDAC" prototype, but had not been replicated for the updated data service model by the time of the DM-10 ramp-down, partly because of the lack of IVOA image services.
In any event this curation and customization would be unsustainable without continued Portal development effort to follow the LSST data model as it evolves.

The ``TAP_SCHEMA`` mechanism defined in the TAP 1.1 standard is used to permit the Portal to self-configure to the available data.
As of 2019 no LSST-specific additions to the TAP 1.1 schema for ``TAP_SCHEMA`` are required, though in the original Portal design some optional additional metadata -- compatible with the ``TAP_SCHEMA`` standard's documented points of extensibility -- would have facilitated providing a richer user interface.
For example, this was envisioned to support the provision of links to detailed documentation on tables and even table columns.

The Firefly TAP query capability that is used by the Portal implementation relies on synchronous TAP queries to obtain ``TAP_SCHEMA`` metadata, but then uses the asynchronous TAP protocol to execute user queries.
This permits the application to return a query-job URL through the UI, which can then be passed to other applications to obtain the results of the same query.


Deployment
----------

The Portal Aspect is a Web-based client-server application.
A containerized server process, implemented in Java, runs within the LSP deployment, providing both application services and the source of the HTML and JavaScript code required for the execution of the client side in the user's Web browser.

Kubernetes Deployments
^^^^^^^^^^^^^^^^^^^^^^

As with the rest of the Science Platform components, the Portal Aspect server-side application is intended to be deployed in Kubernetes.
To facilitate this, the server side code, as part of its release process, is built into a Docker image that contains both the compiled Java code to execute on the server and the HTML and JavaScript code to be served to the client's browser.

The actual server process is constructed using Apache Tomcat, and both the Tomcat code and a full Java run-time (JRE) are included in the Docker image.

Actual deployment in Kubernetes is controlled by a "Helm chart" created by the SQuaRE team with guidance from IPAC.
Each LSP instance can be configured to run a different named release of the Portal image.

The deployment is expected to follow the LSP convention that the Portal Web application is available at the URL *(LSP-instance-root-URL)*``/portal/app`` (the base ``/portal`` endpoint itself is reserved for an originally envisioned "start page" for the Portal, but this was not implemented post-DM-10).
The Portal code expects that, following the LSP convention, the Notebook Aspect will be deployed at *(LSP-instance-root-URL)*``/nb``, and the API Aspect at *(LSP-instance-root-URL)*``/api``, with the TAP service specifically at *(LSP-instance-root-URL)*``/api/tap``.
The URL conventions involved are defined in Document XXX.

As of early 2020, standing instances are maintained at NCSA at `https://lsst-lsp-stable.ncsa.illinois.edu/portal/app` and `https://lsst-lsp-int.ncsa.illinois.edu/portal/app`, with the latter used preferentially for testing of new releases.

Authentication and Authorization Considerations
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

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

Standalone Firefly Server
^^^^^^^^^^^^^^^^^^^^^^^^^

In addition to the Portal Aspect application deployed in the LSP instances in Kubernetes, at this time LSST also maintains a standalone Firefly server, for image visualization purposes, on an NCSA virtual machine at `http://lsst-demo.ncsa.illinois.edu/firefly/`.
This deployment is "vanilla Firefly" - that is, it does not use code from the ``lsst/suit`` repository and does not have access to the LSST-specific TAP services - and is based on an image from the ``ipac/firefly`` DockerHub repository.
However, it is fully functional for use with :py:mod:`lsst.afw.display` image visualization, and has been used in place of the Portal Aspect server(s) inside the LSP instances in order to improve performance, in particular, working around some issues with the Kubernetes networking stack.

This server is deployed as a bare Docker container, without Kubernetes management, on the ``lsst-demo`` VM.
The container image includes the Firefly application, a Tomcat server, and a JRE.

External access to the server, and implementation of HTTPS communication with the outside world, is mediated by an Apache proxy server, running an image from ``ipac/proxy`` on DockerHub.
Communication between the proxy server and the Firefly server is via unencrypted HTTP.
The proxy server holds the necessary certificate information for the HTTPS security.


Implementation
==============

Firefly Core
------------

The Portal is based on the Caltech/IPAC-written open-source "Firefly" client-server application (see `https://github.com/Caltech-IPAC/firefly`).

Firefly provides components for FITS and HiPS image visualization, tabular data query, and tabular data display, as well as a scientific visualization environment for tabular data, based on the Plotly.js library, with extensive brushing, filtering, and linking capabilities, and supporting the overlay of tabular data in sky coordinates on images.

The core Firefly library provides support for UI-based and native ADQL queries against TAP services.
It also comes with support for legacy non-IVOA interfaces to all the major IRSA data holdings.
During LSST commissioning these will be left in place in the expectation that they will provide useful adjunct data for users attempting to place the LSST data into the context of other all-sky surveys such as 2MASS and WISE/NEOWISE, but these may be phased out in favor of a TAP-only interface to external datasets in the LSST Operations era.
(It would clearly be surprising to provide preferential access to IRSA data from the operations-era Portal Aspect.)

The Firefly package, and a small set of related packages detailed below, are all maintained on Github by IPAC.
The IPAC Firefly team creates tags and releases of Firefly in source form as well as in the form of a "default application" that provides a large subset of the capabilities of the post-DM-10 Portal Aspect, but without any LSST-specific knowledge or branding.
This "Firefly application" is released as a Docker image on Dockerhub in the ``ipac/firefly`` repository.

It is this application which is used to implement the ``lsst-demo`` Firefly server mentioned above.

The "suit" Application
----------------------

The Portal application is constructed from code in the LSST package ``lsst/suit``, which provides a thin layer of overall application organization and options-setting over the core Firefly components and their default behaviors.

The name "suit" derives from the legacy name of the IPAC-based team in the distributed DM organization: Science User Interface and Tools (SUIT).
Following the later decision on the LSP nomenclature, however, the "suit" name has been de-emphasized in user-facing application presentations and documentation, in favor of "Portal Aspect" and related nomenclature.

The Portal application is built by combining the ``lsst/suit`` and ``Caltech-IPAC/firefly`` code at source-code level, controlled by build scripts from the Firefly core repository and a thin layer of "suit"-specific build scripts and configuration in the "suit" repository.
The release version of Firefly against which to build is controlled by a file in the ``lsst/suit`` repository, :file:`suit/config/firefly_build.tag`.
In this way, a tag of the ``lsst/suit`` repository also explicitly determines the version of Firefly to be used.

As of October 2019 the builds of the Portal application are still performed by IPAC, but it is planned to transfer this responsibility to LSST-DM in Tucson before the end of LSST construction, in order to conform to evolving DM release management procedures.
The present build system produces a Docker image, with the "suit" release tag, on Dockerhub in the ``ipac/suit`` repository.

If LSST moves toward the use of a private registry for operational container images, the Portal Aspect deployment will follow suit.


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
