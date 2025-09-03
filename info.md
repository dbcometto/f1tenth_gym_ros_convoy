# Info
This file contains some useful information

## Running the simulation with a GPU

First, run
```bash
$ docker compose up
```

In a separate terminal, run
```bash
$ docker exec -it f1tenth_gym_ros /bin/bash
```
to gain access to the simulation container's terminal.

Then, running
```bash
$ ros2 launch f1tenth_gym_ros gym_bridge_launch.py
```
brings up the simulation with RViz on the host.

## Running the simulation without a GPU

First, run
```bash
$ docker compose -f docker-compose-nogpu.yml up
```
from `[ws]/src/f1tenth_gym_ros`

That builds and runs the simulation container and the novnc viewer container.

Then, you can navigate to [http://localhost:8080/vnc.html](http://localhost:8080/vnc.html) in a web browser to view the vnc viewer container.

To interact with the simulation container, run
```bash
$ docker exec -it f1tenth_gym_ros-sim-1 /bin/bash
```




## Troubleshooting

I was having trouble getting the container to output correctly.  This was a successful test.

First, run a test container with

```bash
prime-run docker run -it --rm   --gpus all   --privileged   -e DISPLAY=$DISPLAY   -e NVIDIA_DRIVER_CAPABILITIES=all --runtime=nvidia  -e NVIDIA_VISIBLE_DEVICES=all   -v /tmp/.X11-unix:/tmp/.X11-unix   nvidia/cudagl:11.4.2-runtime-ubuntu20.04    bash
```
Next, inside the container run:
```bash
apt update && apt install mesa-utils mesa-utils-extra -y
```
This installs some tools.

For info, run
```bash
eglinfo | grep -i -E "vendor|device|version"
```
or 
```bash
glxinfo | grep "OpenGL"
```
which verifies which GPU either GLX or EGL is using.

Finally, running
```bash
LIBGL_ALWAYS_INDIRECT=0 es2gears
```
successfully outputted an image of three spinning gears on the host.


### Prime Run
Note that in order to run things with the nvidia gpu, a new script in `home/bin` was created that sets the environment variables necessary to use the nvidia gpu.  The OS is set as using the nvidia gpu `on-demand` (can be found using `$ prime-select query`), and the `prime-run` script requests the nvidia gpu.

### Applying this to the simulation


First, running the container using 
```bash
prime-run docker run -it --rm   --gpus all   --privileged   -e DISPLAY=$DISPLAY   -e NVIDIA_DRIVER_CAPABILITIES=all --runtime=nvidia  -e NVIDIA_VISIBLE_DEVICES=all   -v /tmp/.X11-unix:/tmp/.X11-unix   --volume .:/sim_ws/src/f1tenth_gym_ros -- f1tenth_gym_ros
```
opens the container properly.  Next, we can see the output from `glxinfo` and `eglinfo` below.
```bash
root@9c70282f1612:/sim_ws# glxinfo | grep "OpenGL"
OpenGL vendor string: Intel
OpenGL renderer string: Mesa Intel(R) Xe Graphics (TGL GT2)
OpenGL core profile version string: 4.6 (Core Profile) Mesa 21.2.6
OpenGL core profile shading language version string: 4.60
OpenGL core profile context flags: (none)
OpenGL core profile profile mask: core profile
OpenGL core profile extensions:
OpenGL version string: 4.6 (Compatibility Profile) Mesa 21.2.6
OpenGL shading language version string: 4.60
OpenGL context flags: (none)
OpenGL profile mask: compatibility profile
OpenGL extensions:
OpenGL ES profile version string: OpenGL ES 3.2 Mesa 21.2.6
OpenGL ES profile shading language version string: OpenGL ES GLSL ES 3.20
OpenGL ES profile extensions:
root@9c70282f1612:/sim_ws# eglinfo | grep -i -E "vendor|device|version"
    EGL_EXT_platform_base EGL_EXT_device_base EGL_EXT_device_enumeration
    EGL_EXT_device_query EGL_KHR_client_get_all_proc_addresses
    EGL_EXT_platform_x11 EGL_EXT_platform_device
    EGL_MESA_platform_surfaceless EGL_EXT_explicit_device
EGL API version: 1.5
EGL vendor string: NVIDIA
EGL version string: 1.5
error: XDG_RUNTIME_DIR not set in the environment.
error: XDG_RUNTIME_DIR not set in the environment.
EGL API version: 1.5
EGL vendor string: Mesa Project
EGL version string: 1.5
Device platform:
EGL API version: 1.5
EGL vendor string: NVIDIA
EGL version string: 1.5
```

Now, `LIBGL_ALWAYS_INDIRECT=0 es2gears` also draws a spinning wheel in our host display!  

Okay, tested out the new docker compose file, which handles the command automatically.  Once inside,
```bash
$ es2gears
```
automatically outputted the gears to the host!  And likewise with rviz!