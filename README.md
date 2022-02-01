# WPE Webkit for the RaspberryPi

This project uses 3 different balena blocks:

* [balena WPE](https://github.com/Igalia/balena-browser-wpe)
* [balena Weston](https://github.com/Igalia/balena-weston)
* [balena Audio](https://github.com/balenablocks/audio)

![balena WPE design](https://github.com/igalia/balena-wpe/blob/master/static/resources/design.svg?raw=true)

It provides a Web based screen display running on [WPE
WebKit](https://www.igalia.com/wpe/). Contrary to other solutions, this
project runs the browser on the top of a
[Wayland compositor](https://wayland.freedesktop.org/) (Weston).

Weston aims to be a lean, fast and predictable Wayland compositor. It
provides a full-featured environment for non-desktop applications what
eventually can run a Web browser such as automotive, embedded, in-flight,
industrial, kiosks, set-top boxes and TVs.

WPE WebKit allows embedders to create simple and performant systems
based on Web platform technologies. It is a WebKit port designed
with flexibility and hardware acceleration in mind, leveraging
common 3D graphics APIs for best performance.

Pulse Audio provides all the audio support required for the containerized
browser.

[![](https://balena.io/deploy.svg)](https://dashboard.balena-cloud.com/deploy?repoUrl=https://github.com/igalia/balena-wpe)
---

## Table of contents

- [Table of contents](#table-of-contents)
- [Hardware requirements](#hardware-requirements)
- [Getting started](#getting-started)
  * [Fork the balena-wpe Project and create your own Fleet](#fork-the-balena-wpe-project-and-create-your-own-fleet)
- [Content Load](#content-load)
  * [Changing content at runtime](#changing-content-at-runtime)
  * [Offline content](#offline-content)
- [Settings](#settings)
  * [RaspberryPi `config-txt` settings](#raspberrypi--config-txt--settings)
  * [Browser settings](#browser-settings)
  * [Sound settings](#sound-settings)
  * [Useful environment variables for debugging](#useful-environment-variables-for-debugging)
- [Showcases](#showcases)
- [Development](#development)
  * [Semantic commits](#semantic-commits)

## Hardware requirements

* Raspberry Pi 3/4
* SD card
* HDMI TV or Monitor
* Keyboard, mouse or a Touchscreen

## Getting started

### Fork the balena-wpe project and create your own fleet

* Sign up on [balena](https://dashboard.balena-cloud.com/signup)
* Go through the [getting started guide](http://balena.io/docs/raspberrypi/nodejs/getting-started/)
  and create a new application
* Clone this repository to your local workspace
* Add the _balena remote_ to your local workspace using the useful shortcut in
  the dashboard UI `remoteadd`
* `git push balena master`
* Your device will be updated Over-The-Air

## Content Load

The URL displayed by the browser launcher can be set using the `WPE_URL`
environment variable. The default value is [wpewebkit.org](http://www.webkit.org)

| Environment variable   | Default               | Example
|------------------------|-----------------------|---------------------
| **`WPE_URL`**          | https://wpewebkit.org | https://igalia.com

### Changing content at runtime

balena-browser-wpe ships with [tohora](https://github.com/mozz100/tohora/), which
provides a web interface for changing target URLs at runtime on port 8080.

### Offline content

If you want your device to display content even without internet, you can add
your content in the docker image and point the browser to them. Append a
similar Dockerfile fragment to your project:

```Dockerfile
COPY public_html /var/lib/public_html

ENV WPE_URL="file:///var/lib/public_html/index.html"
```

## Settings

### RaspberryPi `config-txt` settings

A lot of the configuration of this project is about setting up `config.txt`.
The way you do this on balena is by setting some special fleet configuration
variables.

How to set the `config.txt` is pretty well documented in the RaspberryPi
official [documentation](https://www.raspberrypi.org/documentation/computers/config_txt.html).
You can overwrite these settings in the balena dashboard setting
variables in your environment using the prefix `RESIN_HOST_CONFIG_`.
[Read more](https://balena.io/docs/configuration/advanced/#modifying-config-txt-remotely).

Probably you will be interested in setting the GPU memory to a more suitable
value for hardware accelerated graphics.

| Key                                 | Value
|-------------------------------------|----------
|**`RESIN_HOST_CONFIG_gpu_mem_256`**  | `128`
|**`RESIN_HOST_CONFIG_gpu_mem_512`**  | `196`
|**`RESIN_HOST_CONFIG_gpu_mem_1024`** | `396`

### Browser settings

| Environment variable                       | Options   | Default | Description
|--------------------------------------------|-----------|---------|---------------------------------------------------
| **`WPE_COG_PLATFORM_FDO_VIEW_FULLSCREEN`** | `0`, `1`  | `1`     | Enables the fullscreen mode in the browser screen
| **`WPE_COG_PLATFORM_FDO_VIEW_HEIGHT`**     | `integer` | `720`   | Vertical resolution
| **`WPE_COG_PLATFORM_FDO_VIEW_WIDTH`**	     | `integer` | `1280`  | Horizontal resolution
| **`WPE_COG_RELAUNCH`**                     | `0`, `1`  | `unset` | Enables forcing relaunch of the browser
| **`WPE_COG_RELAUNCH_DELAY`**               | `integer` | `5`     | Add delay during forced relaunch

By setting `WPE_ENABLE_INSPECTOR_SERVER` to `on` you can connect to the
Web browser inspector server in the board by loading the following URL `inspector://<raspberrypi IP>:12321`
in an Epiphany Browser.

| Environment variable              | Options          |  Default       | Description
|-----------------------------------|------------------|----------------|---------------------------
| **`WPE_ENABLE_INSPECTOR_SERVER`**	| `0`,`1`          | `0`            | Enables the Web Inspector

Alternatively you can use
[docker-libwebkit2gtk](https://gitlab.com/saavedra.pablo/docker-libwebkit2gtk)
to use the remote Web Inspector.

``` sh
export URL="inspector://<your raspberrypi IP>:12321"
export XAUTHORITY="/run/user/1000/gdm/Xauthority"  # or "xhost +"

cd docker-libwebkit2gtk
sudo -E ./run.sh
URL: inspector://192.168.1.170:12321
Sending build context to Docker daemon  81.92kB
...
Opening URL: inspector://192.168.1.170:12321
```

### Sound settings

balena-browser-wpe relies on balena-audio for the audio processing. Check the specific
[settings](https://github.com/balenablocks/audio#sendreceive-audio) of this
block for audio settings.

Alternatively the block can be also configured to route the audio through another
Pulser Server by setting the `PULSE_SERVER` environment variable.

| Environment variable       | Options                                               |  Default
|----------------------------|-------------------------------------------------------|-----------------
| **`WPE_PULSE_SERVER`**     | `unix:/run/pulse/pulseaudio.socket`, `tcp:audio:4317` | `unix:/run/pulse/pulseaudio.socket`

### Useful environment variables for debugging

| Environment variable         | Example        | Description
|------------------------------|----------------|----------------
| **`EGL_LOG_LEVEL`**          | `debug`        | Change the log level of the main library and the drivers in Mesa
| **`GALLIUM_HUD`**            | `load+cpu+fps` | Activate and set the Gallium3D Heads-Up Display
| **`GST_DEBUG`**              | `*:DEBUG`      | Set the log level output in GStreamer
| **`G_MESSAGES_DEBUG`**       | `all`          | Set the log level output in GLib
| **`LIBGL_DEBUG`**            | `verbose`      | If defined Mesa LibGL debug information will be printed to stderr. If set to `verbose` additional information will be printed
| **`MESA_DEBUG`**             | `1`            | If set to `1` Mesa error messages are printed to stderr
| **`WAYLAND_DEBUG`**          | `1`            | If set to `1` the Wayland debug output is printed to stderr

| Environment variable        | Options          |  Default       | Description
|-----------------------------|------------------|----------------|--------------
| **`CPU_SCALING_GOVERNOR`**  | [options](https://www.kernel.org/doc/html/latest/admin-guide/pm/cpufreq.html#generic-scaling-governors) | performance | Scaling governors

## Showcases

* CSS transformation and animations in [WebKit poster circle example](https://webkit.org/blog-files/3d-transforms/poster-circle.html)
    * <img src="https://github.com/igalia/balena-wpe/blob/master/static/resources/postercircle.gif?raw=true" />
* Playing an embedded [Youtube](https://wpewebkit.org) video
    * <img src="https://github.com/igalia/balena-wpe/blob/master/static/resources/wpewebkitorg.gif?raw=true" />
* Playing [Bouncyballs](https://bouncyballs.org/)
    * <img src="https://github.com/igalia/balena-wpe/blob/master/static/resources/bouncyballs.gif?raw=true" />
* Rendering WebGL [Aquarium](https://webglsamples.org/aquarium/aquarium.html) and [Abstract shapes](https://mrdoob.neocities.org/023/)
    * <img src="https://github.com/igalia/balena-wpe/blob/master/static/resources/webgl_aquarium.gif?raw=true" />
	* <img src="https://github.com/igalia/balena-wpe/blob/master/static/resources/webgl_asbtractshapes.gif?raw=true" />


## Development

## Semantic commits

* Install:

  ``` sh
  sudo apt install npm
  sudo -E npm install -g versionist
  sudo -E npm install -g balena-versionist
  ```

* Add commits with semantic format:

  ```
  Semantic description of the change

  ...Details ...

  Change-Type: <type>  # major, minor or patch
  ```

* Update the changelog:

  ```
  balena-versionist
  ```

* Commit the changes in `balena.yml`, `VERSION` and `CHANGELOG.md` to
  generate a release using the `vX.X.X` format in the commit message.


