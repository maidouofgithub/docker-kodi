# ehough/docker-kodi

Dockerized Kodi with audio and video.

![Kodi screenshot](https://kodi.tv/sites/default/files/page/field_image/about--devices.jpg "Kodi screenshot")

## Features

* fully-functional [Kodi](https://kodi.tv/) installation in a [Docker](https://www.docker.com/) container
* **audio** ([ALSA or PulseAudio](https://kodi.wiki/view/Linux_audio)) and **video** (with optional OpenGL hardware 
  video acceleration) via [x11docker](https://github.com/mviereck/x11docker/)
* simple, Ubuntu-based image that adheres to the [official Kodi installation instructions](https://kodi.wiki/view/HOW-TO:Install_Kodi_for_Linux#Installing_Kodi_on_Ubuntu-based_distributions)
* clean shutdown of Kodi when its container is terminated

## Host Prerequisites

The host system will need the following:

1. **Linux** and [**Docker**](https://www.docker.com)

   Though not yet extensively tested, this image should work on any Linux distribution with a functional
   Docker installation.
   
1. **A connected display and speaker(s)**

   If you're looking for a headless Kodi installation, look elsewhere!

1. **[X](https://www.x.org/) or [Wayland](https://wayland.freedesktop.org/)**

   Ensure that the packages for an X or Wayland server are present on the Docker host. Please consult your OS's 
   documentation if you're not sure what to install. A display server does *not* need to be running ahead of time.

1. **[x11docker](https://github.com/mviereck/x11docker/)**

   `x11docker` allows Docker-based applications to utilize X and/or Wayland on the host. Please follow the `x11docker` 
   [installation instructions](https://github.com/mviereck/x11docker#installation) and ensure that you have a 
   [working setup](https://github.com/mviereck/x11docker#examples) on the Docker host.
       
## Usage

### Starting Kodi

Use `x11docker` to start a [variant of `erichough/kodi`](#image-variants). Detailing the myriad of `x11docker` options is beyond 
the scope of this document; please consult the [`x11docker` documentation](https://github.com/mviereck/x11docker/) to 
find the set of options that work for your system.

Below is an example command (split into multiple lines for clarity) that starts Kodi with a fresh X.Org X server on 
virtual terminal 7 with ALSA sound, no window manager, hardware video acceleration, a persistent Kodi home directory, 
and a shared read-only Docker volume for media files:

    $ x11docker --xorg                                \
                --vt 7                                \
                --alsa                                \
                --wm none                             \                
                --gpu                                 \
                --homedir /host/path/to/kodi/home     \
                -- "-v /host/path/to/media:/media:ro" \
                erichough/kodi:alsa
           
Note that the optional argument passed after `--`, which defines additional arguments to be passed to `docker run`, 
needs to be enclosed in quotes.
           
By default, the container will invoke `kodi-standalone` upon startup. This will boot Kodi and should work 
well for most installations. If you would like to customize this behavior, you can utilize the environment variable 
`KODI_COMMAND` to call additional scripts or processes before starting Kodi. For example, to reduce the priority of the 
Kodi process:

    $ x11docker ... -- '--cap-add SYS_NICE --env KODI_COMMAND="nice kodi-standalone"' erichough/kodi

### Stopping Kodi

There are two ways to stop the running Kodi container:

1. **(Preferred) Send `SIGHUP` or `SIGTERM` to the `x11docker` process.** 

       $ kill -SIGTERM <pid>

   **WARNING**: If you run `x11docker` from a terminal, **do not use `Ctrl-C`**  to end the process as this will cause 
   Kodi to crash spectacularly. Instead, open another shell and use `kill`.
   
1. **Use `docker stop`**
   
       $ docker stop <containerid>
       
When the container receives a signal to terminate, from either of the two means above, it will ask Kodi to shut down 
gracefully and wait for up to 10 seconds before timing out and allowing Docker to forcefully terminate the container. 
Usually Kodi only takes a few seconds to shut down, so 10 seconds should be plenty of time. However if you would like to
extend this timeout for any reason, you can utilize the environment variable `KODI_QUIT_TIMEOUT`. For example, to wait 
120 seconds before timing out:

    $ x11docker ... -- '--env KODI_QUIT_TIMEOUT=120' erichough/kodi
    
Note that if you increase this timeout, you should *only* stop Kodi with `docker stop` *and* use its `--time` option to 
match your desired timeout. e.g.

    $ docker stop --time=120 <containerid>
    
Unless you really need more than 10 seconds, and for simplicity's sake, it's better to signal `x11docker` instead of 
using `docker stop`.

### Example systemd Service Unit

    [Unit]
    Description=Dockerized Kodi
    Requires=docker.service
    After=network.target docker.service
    
    [Service]
    ExecStart=/usr/bin/x11docker ... erichough/kodi:alsa 
    Restart=always
    KillMode=process
    
    [Install]
    WantedBy=multi-user.target

## Image Variants

Choose an image based on your preferred audio system: [ALSA or PulseAudio](https://kodi.wiki/view/Linux_audio).
Very broadly speaking, ALSA is better for systems that are dedicated to Kodi while PulseAudio is better for 
desktop-like systems.

|                  | ALSA | PulseAudio |
|------------------|------|------------|
| **Kodi Krypton** | <ul><li>`erichough/kodi:latest`</li><li>`erichough/kodi:alsa`</li><li>`erichough/kodi:alsa-latest`</li><li>`erichough/kodi:alsa-krypton`</li></ul>  | <ul><li>`erichough/kodi:pulseaudio`</li><li>`erichough/kodi:pulseaudio-latest`</li><li>`erichough/kodi:pulseaudio-krypton`</li></ul>        |
|                  |      |            |

## Contributing

Constructive criticism and contributions are welcome! Please 
[submit an issue](https://github.com/ehough/docker-kodi/issues/new) or 
[pull request](https://github.com/ehough/docker-kodi/compare).

## Future Work

* Kodi PVR support