.. _minio-mc-batch-generate:

=====================
``mc batch generate``
=====================

.. default-domain:: minio

.. contents:: Table of Contents
   :local:
   :depth: 2

.. mc:: mc batch generate

.. versionchanged:: MinIO RELEASE.2022-10-08T20-11-00Z or later

Syntax
------

.. start-mc-batch-generate-desc

The :mc:`mc batch generate` command creates a basic YAML-formatted template file for the specified job type.

.. end-mc-batch-generate-desc

After MinIO creates the file, open it in your preferred text editor tool to further customize.
You can define one job task definition per batch file.

See :ref:`job types <minio-batch-job-types>` for the supported jobs you can generate.

.. tab-set::

   .. tab-item:: EXAMPLE

      The following command creates a basic YAML file for a replicate job on the ``mybucket`` bucket of the ``myminio`` alias.

      .. code-block:: shell
         :class: copyable

         mc batch generate myminio/mybucket replicate

   .. tab-item:: SYNTAX

      The command has the following syntax:

      .. code-block:: shell
         :class: copyable

         mc [GLOBALFLAGS] batch generate \
                                TARGET   \
                                JOBTYPE

      .. include:: /includes/common-minio-mc.rst
         :start-after: start-minio-syntax
         :end-before: end-minio-syntax

Parameters
~~~~~~~~~~

.. mc-cmd:: TARGET
   :required:
   
   The :ref:`alias <alias>` used to generate the YAML template file.
   The specified ``alias`` does not restrict the deployment(s) where you can use the generated file.
   
   For example:

   .. code-block:: none

      mc batch generate myminio replicate

.. mc-cmd:: JOBTYPE
   :required:
   
   The type of job to generate a YAML document for.
   
   Currently, :mc:`mc batch` supports the ``replicate`` and ``keyrotate`` job types.


Global Flags
~~~~~~~~~~~~

.. include:: /includes/common-minio-mc.rst
   :start-after: start-minio-mc-globals
   :end-before: end-minio-mc-globals

Examples
--------

Generate a ``yaml`` File for a Replicate Job Type
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The following command generates a YAML blueprint for a replicate type batch job and names the file ``replicate`` with the ``.yaml`` extension:

.. code-block:: shell
   :class: copyable

   mc batch generate alias replicate > replicate.yaml

- Replace ``alias`` with the :mc:`alias <mc alias>` to use to generate the yaml file.

- Replace ``replicate`` with the type of job to generate a yaml file for.
 
  :mc:``mc batch`` supports the ``replicate`` and ``keyrotate`` job types. 


S3 Compatibility
~~~~~~~~~~~~~~~~

.. include:: /includes/common-minio-mc.rst
   :start-after: start-minio-mc-s3-compatibility
   :end-before: end-minio-mc-s3-compatibility

.. _minio-batch-job-types:

Job Types
---------

:mc:`mc batch` currently supports the following job task types:

- ``replicate``
  
  Replicate objects between two MinIO deployments.
  Provides similar functionality to :ref:`bucket replication <minio-bucket-replication>` as a batch job rather than continual scanning function.

- ``keyrotate``

  .. versionadded:: MinIO RELEASE.2023-04-07T05-28-58Z 
  
  Rotate the sse-s3 or sse-kms keys for objects at rest on a MinIO deployment.

``replicate``
~~~~~~~~~~~~~

Use the ``replicate`` job type to create a batch job that replicates objects from the local MinIO deployment to another MinIO location.

The YAML **must** define the source and target deployments.
If the _source_ deployment is remote, then the _target_ deployment **must** be ``local``.
Optionally, the YAML can also define flags to filter which objects replicate, send notifications for the job, or define retry attempts for the job.

.. versionchanged:: MinIO RELEASE.2023-04-07T05-28-58Z

   You can replicate from a remote MinIO deployment to the local deployment that runs the batch job.

For the **source deployment**

- Required information

  .. list-table::
     :widths: 25 75
     :width: 100%

     * - ``type:``
       - Must be ``minio``.
     * - ``bucket:`` 
       - The bucket on the deployment.

- Optional information

  .. list-table::
     :widths: 25 75
     :width: 100%

     * - ``prefix:`` 
       - The prefix on the object(s) that should replicate.

     * - ``endpoint:`` 
       - | Location of the deployment to use for either the source or the target of a replication batch job. 
         | For example, ``https://minio.example.net``. 
         |
         | If the deployment is the :ref:`alias` specified to the command, omit this field to direct MinIO to use that alias for the endpoint and credentials values. 
         | Either the source deployment *or* the remote deployment *must* be the :ref:`"local" <minio-batch-local>` alias.
         | The non-"local" deployment must specify the ``endpoint`` and ``credentials``.

     * - ``path:``
       - | Directs MinIO to use Path or Virtual Style (DNS) lookup of the bucket.
         | 
         | - Specify ``on`` for Path style
         | - Specify ``off`` for Virtual style
         | - Specify ``auto`` to let MinIO determine the correct lookup style.
         |
         | Defaults to ``auto``.

     * - ``credentials:`` 
       - | The ``accesskey:`` and ``secretKey:`` or the ``sessionToken:`` that grants access to the object(s).
         | Only specify for the deployment that is not the :ref:`local <minio-batch-local>` deployment. 

For the **target deployment**

- Required information

  .. list-table::
     :widths: 25 75
     :width: 100%

     * - ``type:`` 
       - Must be ``minio``.
     * - ``bucket:`` 
       - The bucket on the deployment.

- Optional information

  .. list-table::
     :widths: 25 75
     :width: 100%
  
     * - ``prefix:`` 
       - The prefix on the object(s) to replicate.

     * - ``endpoint:`` 
       - | The location of the target deployment.
         |
         | If the target is the :ref:`alias <alias>` specified to the command, you can omit this and the ``credentials`` fields.
         | If the target is "local", the source *must* specify the remote deployment with ``endpoint`` and ``credentials``.


     * - ``credentials:`` 
       - The ``accesskey`` and ``secretKey`` or the ``sessionToken`` that grants access to the object(s).
    
For **filters**

.. list-table::
   :widths: 25 75
   :width: 100%

   * - ``newerThan:`` 
     - A string representing a length of time in ``#d#h#s`` format.
       
       Only objects newer than the specified length of time replicate.
       For example, ``7d``, ``24h``, ``5d12h30s`` are valid strings.
   * - ``olderThan:`` 
     - A string representing a length of time in ``#d#h#s`` format.
       
       Only objects older than the specified length of time replicate.
   * - ``createdAfter:`` 
     - A date in ``YYYY-MM-DD`` format.
  
       Only objects created after the date replicate.
   * - ``createdBefore:`` 
     - A date in ``YYYY-MM-DD`` format.
       
       Only objects created prior to the date replicate.

For **notifications**

.. list-table::
   :widths: 25 75
   :width: 100%

   * - ``endpoint:`` 
     - The predefined endpoint to send events for notifications.
   * - ``token:`` 
     - An optional :abbr:`JWT <JSON Web Token>` to access the ``endpoint``.

For **retry attempts**

If something interrupts the job, you can define how many attempts to retry the job batch.
For each retry, you can also define how long to wait between attempts.

.. list-table::
   :widths: 25 75
   :width: 100%

   * - ``attempts:`` 
     - Number of tries to complete the batch job before giving up.
   * - ``delay:`` 
     - The least amount of time to wait between each attempt.


Sample YAML
+++++++++++

.. literalinclude:: /includes/code/replicate.yaml
   :language: yaml

``keyrotate``
~~~~~~~~~~~~~

.. versionadded:: MinIO RELEASE.2023-04-07T05-28-58Z 

Use the ``keyrotate`` job type to create a batch job that cycles the :ref:`sse-s3 or sse-kms keys <minio-sse-data-encryption>` for encrypted objects.

Required information
++++++++++++++++++++

  .. list-table::
     :widths: 25 75
     :width: 100%

     * - ``type:`` 
       - Either ``sse-s3`` or ``sse-kms``.
     * - ``key:`` 
       - Only for use with the ``sse-kms`` type. 
         The key to use to unseal the key vault.
     * - ``context:``
       - Only for use with the ``sse-kms`` type.
         The context within which to perform actions. 

   
Optional information
++++++++++++++++++++

For **flag based filters**

.. list-table::
   :widths: 25 75
   :width: 100%

   * - ``newerThan:`` 
     - A string representing a length of time in ``#d#h#s`` format.
       
       Keys rotate only for objects newer than the specified length of time.
       For example, ``7d``, ``24h``, ``5d12h30s`` are valid strings.
   * - ``olderThan:`` 
     - A string representing a length of time in ``#d#h#s`` format.
       
       Keys rotate only for objects older than the specified length of time.
   * - ``createdAfter:`` 
     - A date in ``YYYY-MM-DD`` format.
  
       Keys rotate only for objects created after the date.
   * - ``createdBefore:`` 
     - A date in ``YYYY-MM-DD`` format.
       
       Keys rotate only for objects created prior to the date.
   * - ``tags:``
     - Rotate keys only for objects with tags that match the specified ``key:`` and ``value:``.  
   * - ``metadata:``
     - Rotate keys only for objects with metadata that match the specified ``key:`` and ``value:``.  
   * - ``kmskey:``
     - Rotate keys only for objects with a KMS key-id that match the specified value.  
       This is only applicable for the ``sse-kms`` type. 

For **notifications**

.. list-table::
   :widths: 25 75
   :width: 100%

   * - ``endpoint:`` 
     - The predefined endpoint to send events for notifications.
   * - ``token:`` 
     - An optional :abbr:`JWT <JSON Web Token>` to access the ``endpoint``.

For **retry attempts**

If something interrupts the job, you can define a maximum number of retry attempts.
For each retry, you can also define how long to wait between attempts.

.. list-table::
   :widths: 25 75
   :width: 100%

   * - ``attempts:`` 
     - Number of tries to complete the batch job before giving up.
   * - ``delay:`` 
     - The amount of time to wait between each attempt.


Sample YAML
+++++++++++

.. literalinclude:: /includes/code/keyrotate.yaml
   :language: yaml