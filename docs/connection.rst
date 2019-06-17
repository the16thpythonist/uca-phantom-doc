########################
Connecting to the Camera
########################

The phantom camera will be connected to the operating computer by an ethernet cable using one of the cameras ethernet
ports: Ether the "normal" 1G interface or the 10G interface. The most important thing to make sure of is to correctly
set the IP address and netmask for the ethernet interface *of the operating computer*, which is connected to the
machine.

======================
The "connect" property
======================

To connect the camera from within the program, in such a way, that subsequent calls to the ``grab`` command will
succeed to deliver image frames, essentially two steps are required.

1) Establish control connection: To create a socket connection to send control commands over, the ``connect`` porperty
of the camera object has to be set to TRUE. This will implicitly trigger the internal connect function to be executed

2) Starting the readout threads: To properly receive image data from the camera, additional threads, which accept the
incoming data connections from the camera have to be started. This is done by calling the ``start_recording``
method

C example:

.. code-block:: c

    // complete program shortened ...
    manager = uca_plugin_manager_new();
    camera = uca_plugin_manager_get_camera(manager, "phantom", &error, c);

    // Connecting to the camera and starting the readout threads
    g_object_set(G_OBJECT(camera), "connect", TRUE, NULL);
    uca_camera_start_recording(camera, &error);

Python example:

.. code-block:: python

    # complete program shortened ...
    plugin_manager = Uca.PluginManager()
    camera = plugin_manager.get_camerav('phantom', [])

    # Connecting the camera and starting the readout threads
    camera.props.connect = True
    camera.start_recording()


======================
The discovery protocol
======================

To establish a connection to the camera, it offers a UDP discovery protocol, where the phantom plugin will send a UDP
broadcast to the IP range, on which the phantom cameras operate and then waits for a response from a camera. This
response will then expose the IP address to the phantom plugin, without the need to manually specify the IP address of
the specific camera model used.

*Although it is adviced to specify the IP address explicitly, as the discovery protocol is not yet reliably implemented
and may cause issues from time to time.*


Using the discovery protocol
============================

When using the discovery protocol no additional steps are required.

Explicitly providing the IP address
===================================

To explicitly provide the camera with an IP address, just set the ``network-address`` property of the camera object to
the string of the IP address

**NOTE**: When using the python bindings for libuca, properties that contain a dash "-" in their name for C will have
an underscore "_" instead in python!

C example:

.. code-block:: c

    // complete program shortened ...
    // Setting the IP address before(!) connecting
    g_object_set(G_OBJECT(camera), "network-address", "100.100.189.94", NULL);

Python example:

.. code-block:: python

    # complete program shortened ...
    # Setting the IP address before(!) connecting
    camera.props.network_address = "100.100.189.94"

===========================================
Specifying the interface for 10G connection
===========================================

Transmitting data using the 10G interface is partially as fast as it is, because the image data is not transmitted
using TCP packets (a protocol with a lot of overhead), but by raw ethernet frames. This type of transmission has
minimal overhead, because the data is not being transmitted in the likes of a conversation, it is rather all dumped
into the ethernet at the same time.

To receive this type of data, the phantom plugin needs to know at which ethernet interface the camera is connected
to the operating computer, so it knows "where to listen for the data dump".

Thus, when using the 10G connection, the name of the used interface will have to be supplied as well, by setting the
``network-interface`` property of the camera object to the string name of the interface.

C example:

.. code-block:: c

    // complete program shortened ...
    // This flag will tell the camera to use the 10G interface
    g_object_set(G_OBJECT(camera), "enable-10ge", TRUE, NULL);
    // Supplying the interface name
    g_object_set(G_OBJECT(camera), "network-interface", "eth0", NULL);

Python example:

.. code-block:: python

    # complete program shortened ...
    # This flag will tell the camera to use the 10G interface
    camera.props.enable_10ge = True
    # Supplying the interface name
    camera.props.network_interface = "eth0"

=======================
Putting it all together
=======================

To show a complete example to connect the camera using the 10G interface and explicitly providing the IP address of
the camera:

C example:

.. code-block:: c

    #include <glib-object.h>
    #include <uca/uca-plugin-manager.h>
    #include <uca/uca-camera.h>

    int main(int argc, char *argv[]) {
        GError *error = NULL;

        manager = uca_plugin_manager_new();
        camera = uca_plugin_manager_get_camera(manager, "phantom", &error, "");

        // Setting IP address manually &
        // enable 10G network
        g_object_set(G_OBJECT(camera), "network-address", "172.16.31.157", NULL);
        g_object_set(G_OBJECT(camera), "network-interface", "eth0", NULL);
        g_object_set(G_OBJECT(camera), "enable-10ge", TRUE, NULL);

        // Connection the camera
        g_object_set(G_OBJECT(camera), "connect", TRUE, NULL);

        // Starting the readout threads
        uca_camera_start_recording(camera, &error);

        // Grabbing images...
    }