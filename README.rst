.. image:: https://img.shields.io/badge/dmtn--136-lsst.io-brightgreen.svg
   :target: https://dmtn-136.lsst.io
.. image:: https://github.com/lsst-dm/dmtn-136/workflows/CI/badge.svg
   :target: https://github.com/lsst-dm/dmtn-136/actions/

..
  Uncomment this section and modify the DOI strings to include a Zenodo DOI badge in the README
  .. image:: https://zenodo.org/badge/doi/10.5281/zenodo.#####.svg
     :target: http://dx.doi.org/10.5281/zenodo.#####

#################################################################
LSST Science Platform Portal Aspect Design and Maintenance Manual
#################################################################

DMTN-136
========

This note will provide concise information about the operational concept for the LSP Portal Aspect, including the core Firefly software around which it is structured, as well as information about the deployment and maintenance of the Portal Aspect software.

**Links:**

- Publication URL: https://dmtn-136.lsst.io
- Alternative editions: https://dmtn-136.lsst.io/v
- GitHub repository: https://github.com/lsst-dm/dmtn-136
- Build system: https://github.com/lsst-dm/dmtn-136/actions/


Build this technical note
=========================

You can clone this repository and build the technote locally with `Sphinx`_:

.. code-block:: bash

   git clone https://github.com/lsst-dm/dmtn-136
   cd dmtn-136
   pip install -r requirements.txt
   make html

.. note::

   In a Conda_ environment, ``pip install -r requirements.txt`` doesn't work as expected.
   Instead, ``pip`` install the packages listed in ``requirements.txt`` individually.

The built technote is located at ``_build/html/index.html``.

Editing this technical note
===========================

You can edit the ``index.rst`` file, which is a reStructuredText document.
The `DM reStructuredText Style Guide`_ is a good resource for how we write reStructuredText.

Remember that images and other types of assets should be stored in the ``_static/`` directory of this repository.
See ``_static/README.rst`` for more information.

The published technote at https://dmtn-136.lsst.io will be automatically rebuilt whenever you push your changes to the ``master`` branch on `GitHub <https://github.com/lsst-dm/dmtn-136>`_.

Updating metadata
=================

This technote's metadata is maintained in ``metadata.yaml``.
In this metadata you can edit the technote's title, authors, publication date, etc..
``metadata.yaml`` is self-documenting with inline comments.

Using the bibliographies
========================

The bibliography files in ``lsstbib/`` are copies from `lsst-texmf`_.
You can update them to the current `lsst-texmf`_ versions with::

   make refresh-bib

Add new bibliography items to the ``local.bib`` file in the root directory (and later add them to `lsst-texmf`_).

.. _Sphinx: http://sphinx-doc.org
.. _DM reStructuredText Style Guide: https://developer.lsst.io/restructuredtext/style.html
.. _this repo: ./index.rst
.. _Conda: http://conda.pydata.org/docs/
.. _lsst-texmf: https://lsst-texmf.lsst.io
