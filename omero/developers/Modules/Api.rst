OMERO Application Programming Interface
=======================================

All interaction with the OMERO server takes place via several API services
available from a ServiceFactory. A service factory is obtained from the client
connection e.g. Python:

::

    import omero.clients

    client = omero.client("localhost")
    session = client.createSession("username", "password")   # this is the service factory
    adminService = session.getAdminService()                 # now we can get/create services

-  The :javadoc:`Service factory API <slice2html/omero/api/ServiceFactory.html>`
   has methods for creating Stateless and Stateful services, see below.

   -  Stateless services are obtained using "getXXX" methods e.g.
      ``getQueryService()``
   -  Stateful services are obtained using "createXXX" methods e.g.
      ``createRenderingEngine()``

-  Services will provide access to omero.model.objects. You will then
   need the :javadoc:`API for these objects <slice2html/omero/model.html>`,
   e.g. Dataset, Image, Pixels, etc.

Services list
-------------

The :sourcedir:`ome.api <components/common/src/ome/api>` package in the common
component defines the central "verbs" of the OMERO system. All external
interactions with the system should happen with these verbs, or services. Each
OMERO service belongs to a particular service level with each level calling
only on services from lower levels.

Service Level 1 (direct database and Hibernate connections)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

-  AdminService:
   :source:`src <components/common/src/ome/api/IAdmin.java>`,
   :javadoc:`API <slice2html/omero/api/IAdmin.html>`
   for working with Experimenters, Groups and the current Context
   (switching groups etc.).
-  ConfigService:
   :source:`src <components/common/src/ome/api/IConfig.java>`,
   :javadoc:`API <slice2html/omero/api/IConfig.html>`
   for getting and setting config parameters.
-  ContainerService:
   :javadoc:`API <slice2html/omero/api/IContainer.html>`
   for loading Project, Dataset, Image, Screen, Plate hierarchies.
-  LdapService:
   :source:`src <components/common/src/ome/api/ILdap.java>`,
   :javadoc:`API <slice2html/omero/api/ILdap.html>`
   for communicating with LDAP servers.
-  MetadataService:
   :javadoc:`API <slice2html/omero/api/IMetadata.html>`
   for working with Annotation and retrieving acquisition metadata e.g. instrument.
-  PixelsService:
   :javadoc:`API <slice2html/omero/api/IPixels.html>`
   for pixels stats and creating Images with existing or new Pixels.
-  ProjectionService
   :javadoc:`API <slice2html/omero/api/IProjection.html>`
   for performing projections of Pixels sets.
-  QueryService:
   :source:`src <components/common/src/ome/api/IQuery.java>`,
   :javadoc:`API <ome/api/IQuery.html>`
   for custom SQL-like queries.
-  RenderingSettingsService
   :javadoc:`API <slice2html/omero/api/IRenderingSettings.html>`
   for copying, pasting & resetting rendering settings.
-  RepositoryInfo
   :javadoc:`API <slice2html/omero/api/IRepositoryInfo.html>`
   disk space stats.
-  RoiService
   :javadoc:`API <slice2html/omero/api/IRoi.html>`
   working with ROIs (now deprecated).
-  ScriptService
   :javadoc:`API <slice2html/omero/api/IScript.html>`
   for uploading and launching Python scripts.
-  SessionService
   :javadoc:`API <slice2html/omero/api/ISession.html>`
   for creating and working with OMERO sessions.
-  ShareService
   :javadoc:`API <slice2html/omero/api/IShare.html>` (now deprecated).
-  TimelineService
   :javadoc:`API <slice2html/omero/api/ITimeline.html>`
   for queries based on time.
-  TypesService
   :javadoc:`API <slice2html/omero/api/ITypes.html>`
   for Enumerations.
-  UpdateService:
   :source:`src <components/common/src/ome/api/IUpdate.java>`,
   :javadoc:`API <ome/api/IUpdate.html>`
   for saving and editing omero.model objects.

Service Level 2
^^^^^^^^^^^^^^^

-  :source:`IContainer <components/common/src/ome/api/IContainer.java>`
-  :source:`ITypes <components/common/src/ome/api/ITypes.java>`

Stateful/Binary Services
^^^^^^^^^^^^^^^^^^^^^^^^

-  RawFileStore:
   :source:`src <components/common/src/ome/api/RawFileStore.java>`,
   :javadoc:`API <ome/api/RawFileStore.html>` for reading and writing files
-  RawPixelsStore:
   :source:`src <components/common/src/ome/api/RawPixelsStore.java>`,
   :javadoc:`API <ome/api/RawPixelsStore.html>` for reading and writing pixels data
-  RenderingEngine:
   :source:`src <components/common/src/omeis/providers/re/RenderingEngine.java>`,
   :javadoc:`API <slice2html/omero/api/RenderingEngine.html>` for viewing images,
   see :doc:`/developers/Server/RenderingEngine` for more details
-  ThumbnailStore:
   :source:`src <components/common/src/ome/api/ThumbnailStore.java>`,
   :javadoc:`API <ome/api/ThumbnailStore.html>` for retrieving thumbnails
-  :source:`IScale <components/common/src/ome/api/IScale.java>` for scaling rendered images

A complete list of service APIs can be found
:javadoc:`here <slice2html/omero/api.html>` and some examples of API usage in
Python are provided. Java or C++ code can use the same API in a very similar
manner.

Discussion
----------

Reads and writes
^^^^^^^^^^^^^^^^

IQuery and IUpdate are the basic building blocks for the rest of the
(non-binary) API. IQuery is based on QuerySources and QueryParemeters
which are explained under :doc:`/developers/Server/Queries`. The goal of this
design is to make wildly separate definitions of queries (templates,
db-stored, Java code, C# code, …) runnable on the server.

IUpdate takes any graph composed of
:source:`IObject <components/model/src/ome/model/IObject.java>`
objects and checks them for dirtiness. All changes to the graph are
stored in the database if the user calling IUpdate has the proper
permissions, otherwise an exception is thrown.

Dirty checks follow the Three Commandments:

#. Any IObject-valued field with unloaded set to true is treated as a proxy
   and is reloaded from the database.
#. Any collection-valued field with a null value is re-loaded from the
   database.
#. Any collection-valued field with the FILTERED flag is assumed to be
   dirty and is loaded from the database, with the future option of
   examining the filtered collection for any new and updated values and
   applying them to the real collection. Deletions cannot happen
   this way since it would be unclear if the object was filtered or 
   deleted.

Administration
^^^^^^^^^^^^^^

The :source:`IAdmin <components/common/src/ome/api/IAdmin.java>` interface
defines all the actions necessary to administer the
:doc:`/sysadmins/server-security`. It is explained further on the
:doc:`/developers/Modules/Api/AdminInterface` page.

Model Object Java
^^^^^^^^^^^^^^^^^

Certain operations, like those dealing with data management and viewing,
happen more frequently than others e.g. defining microscopes. Those have
been collected in the 
:source:`IContainer <components/common/src/ome/api/IContainer.java>`
interface. IContainer simplifies a few very common queries, and there is a
related package ``omero.gateway.model.\*`` for working with the returned graphs.
OMERO.insight works almost exclusively with the IContainer interface mostly 
indirectly via the :ref:`javagateway`.

Examples
--------

::

    // Saving a simple change
    Dataset d = iQuery.get(Dataset.class, 1L);
    d.setName("test");
    iUpdate.saveObject(d);

    // Creating a new object
    Dataset d = new Dataset();
    d.setName("test"); // not-null fields must be filled in
    iUpdate.saveObject(d);

    // Retrieving a graph
    Set<Dataset> ds = iQuery.findAllByQuery("from Dataset d left outer join d.images where d.name = 'test'", null);

Stateless versus stateful services
----------------------------------

A stateless service has no client-noticeable lifecycle and all instances can
be treated equally. A new stateful service, on the other hand, will be created
for each client-side proxy, see the ``ServiceFactory.createXXX`` methods. Once
obtained, a stateful service proxy can only be used by a single user. After
task completion, the service should be closed i.e. ``proxy.close()`` to free up
server resources.

How to write a service
----------------------

A tutorial is available at :doc:`/developers/Server/HowToCreateAService`.
See :doc:`/developers/build-system` for more information
on how the annotated service will be deployed.
In the case of :doc:`/developers/server-blitz`, the
service must be properly defined under 
:sourcedir:`components/blitz/resources`.

OMERO annotations for validation
--------------------------------

The server-side implementation of these interfaces makes use of `Java annotations <https://docs.oracle.com/javase/tutorial/java/annotations/basics.html>`_
and an :doc:`AOP </developers/Server/Aop>` interceptor to validate all method
parameters. Calls to ``pojos.findContainerHierarchies`` are first caught by a
method interceptor, which checks for annotations on the parameters and, if
available, performs the necessary checks. The interceptor also makes proactive
checks. For a range of parameter types such as Java Collections it requires
that annotations exist and will refuse to proceed if not implemented.

An API call of the form:

::

        pojos.findContainerHierarchies(Class, Set, Map)

is implemented as

::

         pojos.findContainerHierarchies(@NotNull Class, @NotNull @Validate(Integer.class) Set, Map)

.. seealso:: :doc:`/developers/Server/Queries`, :doc:`/developers/Server/RenderingEngine`, :doc:`/developers/Modules/ExceptionHandling`
