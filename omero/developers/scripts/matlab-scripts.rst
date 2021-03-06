MATLAB and Python
=================

MATLAB functionality can be mixed into Python scripts via the
Python package `Mlabwrap <http://mlabwrap.sourceforge.net>`_.

Installing Mlabwrap
-------------------

To install MlabWrap follow the
`installation guide <http://mlabwrap.sourceforge.net/#installation>`_ and
make sure that the paths are set for the environment variables:

::

    LD_LIBRARY_PATH=$MATLABROOT/bin/Platform
    MLABRAW_CMD_STR=$MATLABROOT/bin/matlab

Example MATLAB scripts
----------------------

Below are some sample scripts showing MATLAB being launched from
OMERO.scripts. MATLAB functions can also call the |OmeroJava| interface to 
access the server from the MATLAB functions.

Calling a simple MATLAB function
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

::

    import omero, omero.scripts as script
    # import mlabwrap to launch matlab.
    from mlabwrap import matlab;  
    client = script.client("rand.py", "Get matrix of random numbers drawn from a uniform distribution",  
                            script.Long("x").inout(), script.Long("y").inout())
    client.createSession()

    x = client.getInput("x").val
    y  = client.getInput("y").val

    # call the matlab rand function via mlabwrap will automatically launch matlab 
    # if it is not already running on the system and call the rand method.
    val = matlab.rand(x,y);
    print val

Using the OMERO interface inside MATLAB
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

This example shows the MATLAB script being called, passed to the client
object and accessing the same client instance as the script.

::

    import omero, omero.scripts as script
    # import mlabwrap to launch matlab.
    from mlabwrap import matlab;  
    client = script.client("projection.py", "Call the matlab projection code",  
                            script.String("iceConfig").in(), script.String("user").in(),
                            script.String("password"),
                            script.Long("pixelsId").inout(), script.String("method").inout()
                            script.Long("stack").inout())
    client.createSession()

    iceConfig = client.getInput("pixelsId").val
    user = client.getInput("pixelsId").val
    password = client.getInput("pixelsId").val
    method  = client.getInput("method").val
    stack = client.getInput("stack").val;

    image = matlab.performProjection(iceConfig, username, password, pixelsId, stack, method);

The MATLAB projection method
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
::

    function performProjection(iceConfig, username, password, pixelsId, zSection, method)

    omerojavaService = createOmeroJavaService(iceConfig, username, password);
    pixels = getPixels(omerojavaService, pixelsId);
    stack = getPlaneStack(omerojavaService, pixelsId, zSection);
    projectedImage = ProjectionOnStack(stack, method);

::

    function [resultImage] = ProjectionOnStack(imageStack,type)

    [zSections, X, Y] = size(imageStack);

    if(strcmp(type,'mean') || strcmp(type, 'sum'))
        resultImage = squeeze(sum(imageStack));
        if(strcmp(type,'mean'))
            resultImage = resultImage./zSections;
        end
    end
    if(strcmp(type,'max'))
        resultImage = squeeze(max(imageStack,[],1));
    end
