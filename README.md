# Headless Ubuntu/Xfce container with VNC/noVNC and configurable Firefox

## accetto/ubuntu-vnc-xfce-firefox-plus

[Docker Hub][this-docker] - [Git Hub][this-github] - [Changelog][this-changelog] - [Wiki][this-wiki]

***

![badge-docker-pulls][badge-docker-pulls]
![badge-docker-stars][badge-docker-stars]
![badge-github-release][badge-github-release]
![badge-github-release-date][badge-github-release-date]

**TIP** Unless you need [nss_wrapper][nsswrapper], you can also use my newer image [accetto/xubuntu-vnc-novnc-firefox][accetto-docker-xubuntu-vnc-novnc-firefox], which is a streamlined version of this image ([image hierarchy][accetto-xubuntu-vnc-novnc-wiki-image-hierarchy]). If you also don't need [noVNC][novnc], you can use even a slimmer image [accetto/xubuntu-vnc-firefox][accetto-docker-xubuntu-vnc-firefox], which is a member of another growing family of application images ([image hierarchy][accetto-xubuntu-vnc-wiki-image-hierarchy]). The newer images include also **sudo** command.

***

**Attention:** Resources for building images with default Firefox installation without configuration support can be found in its own GitHub repository [ubuntu-vnc-xfce-firefox][accetto-github-ubuntu-vnc-xfce-firefox]. Resources for building base images are in the GitHub repository [accetto/ubuntu-vnc-xfce][accetto-github-ubuntu-vnc-xfce].

**This repository** contains resources for building a Docker image based on [Ubuntu][docker-ubuntu] with [Xfce][xfce] desktop environment, **VNC**/[noVNC][novnc] servers for headless use and the current [Firefox][firefox] web browser with pre-configuration support.

The image can be successfully built and used on Linux, Windows, Mac and NAS devices. It has been tested with [Docker Desktop][docker-desktop] on [Ubuntu flavours][ubuntu-flavours], [Windows 10][docker-for-windows] and [Container Station][container-station] from [QNAP][qnap].

Containers created from this image make perfect light-weight web browsers. They can be thrown away easily and replaced quickly, improving browsing privacy. They run under a non-root user by default, improving browsing security.

They make also excellent long-term browsers, because the preferences and profiles can be pre-configured and stored on volumes that survive container destruction.

There are two ways of customization. Firstly, it's possible to force Firefox preferences by modifying the provided **user.js** file. Secondly, it's possible to use a complete Firefox profile, previously created on a volume. The [HOWTO][this-wiki-howto] page in [Wiki][this-wiki] describes it in more details.

Frequently used preferences and profiles can be also embedded into new customized images, built by the user. The ready-to-use Dockerfiles are already provided (see [below](#user-content-image-set)). The [HOWTO][this-wiki-howto] page in [Wiki][this-wiki] describes how to build such images.

Running in background is the primary scenario for the containers, but using them interactively in foreground is also possible. For examples see the description below or the [HOWTO][this-wiki-howto] section in [Wiki][this-wiki].

The image is based on the [accetto/ubuntu-vnc-xfce][accetto-docker-ubuntu-vnc-xfce] image, just adding the [Firefox][firefox] browser and the resources for its customization.

The image inherits the following components from its [base image][accetto-docker-ubuntu-vnc-xfce]:

- light-weight [Xfce][xfce] desktop environment
- high-performance VNC server [TigerVNC][tigervnc] (TCP port **5901**)
- [noVNC][novnc] HTML5 clients (full and lite) (TCP port **6901**)
- popular text editor [vim][vim]
- lite but advanced graphical editor [mousepad][mousepad]
- **ping** utility
- container start-up options

**Please note** that **Firefox 67** has changed default profile handling and therefore the pre-created folder `profile0.default`, which contains the file `user.js` with user preferences, will not be used automatically, until you explicitly choose the folder before the first Firefox start. The other option is to copy the user preferences afterwards using the provided script. You can find more information about it in the [issue #3][this-issue-3].

The following desktop launchers have been added for your convenience:

- **FF Profile Manager** starts the Firefox with the argument **-P** so you can choose Firefox profiles or create new ones conveniently. It is recommended to choose the pre-created profile folder `profile0.default` before the first Firefox start. Note that it shows up as the **default** profile in the Profile Manager's list and that the actual profile data will be created by the Firefox itself.
- **Copy FF Preferences** starts the script for copying the file **user.js**, containing your own Firefox preferences, to one or more Firefox profiles interactively. You can use it if you haven't chosen the profile before the first Firefox start or you have created new ones later.

Running containers in background is the primary scenario this image has been developed for. However, running in foreground can be useful in many cases. See the description below for examples of using the containers both ways.

The image is regularly maintained and rebuilt. The history of notable changes is documented in [CHANGELOG][this-changelog].

![screenshot-container][screenshot-container]

### Image set

- [accetto/ubuntu-vnc-xfce-firefox-plus][this-docker]

  - `latest` based on `accetto/ubuntu-vnc-xfce:latest`

    ![badge-VERSION_STICKER_LATEST][badge-VERSION_STICKER_LATEST]
    ![badge-github-commit-latest][badge-github-commit-latest]

- **accetto/ubuntu-vnc-xfce-firefox-plus-preferences**

    This image is not actually contained in the [Docker repository][accetto-docker], because it is intended to keep pre-configured user-specific **Firefox preferences**. Users can put their favorite preferences into the configuration files and build the image using the provided [Dockerfile-plus-preferences][this-dockerfile-plus-preferences]. The [HOWTO][this-wiki-howto] page in [Wiki][this-wiki] describes it in more details.

- **accetto/ubuntu-vnc-xfce-firefox-plus-profile**

    This image is also not actually contained in the [Docker repository][accetto-docker], because it is intended to keep pre-configured user-specific **Firefox profiles**. Users can prepare a full Firefox profile and build the image using the provided [Dockerfile-plus-profile][this-dockerfile-plus-profile]. The [HOWTO][this-wiki-howto] page in [Wiki][this-wiki] describes it in more details.

### Ports

Following **TCP** ports are exposed:

- **5901** used for access over **VNC**
- **6901** used for access over [noVNC][novnc]

The default **VNC user** password is **headless**.

### Volumes

The containers do not create or use any external volumes by default. However, the following folders make good mounting points:

- /home/headless/Documents/
- /home/headless/Downloads/
- /home/headless/Music/
- /home/headless/Pictures/
- /home/headless/Public/
- /home/headless/Templates/
- /home/headless/Videos/

The following mounting point is specific to Firefox:

- /home/headless/.mozilla

Both *named volumes* and *bind mounts* can be used. More about volumes can be found in [Docker documentation][docker-doc] (e.g. [Manage data in Docker][docker-doc-managing-data]).

## Firefox multi-process

Firefox multi-process (also known as **Electrolysis** or just **E10S**) causes in Docker container heavy crashing (**Gah. Your tab just crashed.**) and therefore it needs to be disabled.

In Firefox versions till **67.0.4** it could be done by setting the preferences **browser.tabs.remote.autostart** and **browser.tabs.remote.autostart.2** to **false**. However, Mozilla has removed this possibility since the Firefox version **68.0**.

Since than it can be done only by setting the following environment variable:

```bash
MOZ_FORCE_DISABLE_E10S
```

Therefore the image tagged `latest` sets this variable to **1** by using the build argument **ARG_MOZ_FORCE_DISABLE_E10S**.

Note that any value will actually disable the multi-process feature, so the both following settings would have the same effect:

```bash
MOZ_FORCE_DISABLE_E10S=1
MOZ_FORCE_DISABLE_E10S=0
```

Building an image without the build argument **ARG_MOZ_FORCE_DISABLE_E10S** enables the Firefox multi-process feature.

To check whether the Firefox multi-process is enabled or disabled, navigate the web browser to the following URL:

```bash
about:support
```


## Running containers in background (detached)

Created containers will run under the non-root user **headless:headless** by default.

The following container will listen on automatically selected **TCP** ports of the host computer:

```docker
docker run -d -P accetto/ubuntu-vnc-xfce-firefox-plus
```

The following container will listen on the host's **TCP** ports **25901** (VNC) and **26901** (noVNC):

```docker
docker run -d -p 25901:5901 -p 26901:6901 accetto/ubuntu-vnc-xfce-firefox-plus
```

The following container wil create or re-use the local named volume **my\_Downloads** mounted as `/home/headless/Downloads`. The container will be accessible through the same **TCP** ports as the one above:

```docker
docker run -d -P -v my_Downloads:/home/headless/Downloads accetto/ubuntu-vnc-xfce-firefox-plus
```

or using the newer syntax with **--mount** flag:

```docker
docker run -d -P --mount source=my_Downloads,target=/home/headless/Downloads accetto/ubuntu-vnc-xfce-firefox-plus
```

More usage examples can be found in [Wiki][this-wiki] (section [HOWTO][this-wiki-howto]).

## Running containers in foreground (interactively)

The image supports the following container start-up options: `--wait` (default), `--skip`, `--debug` (also `--tail-log`) and `--help`. This functionality is inherited from the [base image][accetto-docker-ubuntu-vnc-xfce].

The following container will print out the help and then it'll remove itself:

```docker
docker run --rm accetto/ubuntu-vnc-xfce-firefox-plus --help
```

Excerpt from the output, which describes the other options:

```docker
OPTIONS:
-w, --wait      (default) Keeps the UI and the vnc server up until SIGINT or SIGTERM are received.
                An optional command can be executed after the vnc starts up.
                example: docker run -d -P accetto/ubuntu-vnc-xfce
                example: docker run -it -P accetto/ubuntu-vnc-xfce /bin/bash

-s, --skip      Skips the vnc startup and just executes the provided command.
                example: docker run -it -P accetto/ubuntu-vnc-xfce --skip /bin/bash

-d, --debug     Executes the vnc startup and tails the vnc/noVNC logs.
                Any parameters after '--debug' are ignored. CTRL-C stops the container.
                example: docker run -it -P accetto/ubuntu-vnc-xfce --debug

-t, --tail-log  same as '--debug'

-h, --help      Prints out this help.
                example: docker run --rm accetto/ubuntu-vnc-xfce
```

It should be noticed, that the `--debug` start-up option does not show the command prompt even if the `-it` run arguments are provided. This is because the container is watching the incoming vnc/noVNC connections and prints out their logs in real time. However, it is easy to attach to the running container like in the following example.

In the first terminal window on the host computer, create a new container named **foo**:

```docker
docker run --name foo accetto/ubuntu-vnc-xfce-firefox-plus --debug
```

In the second terminal window on the host computer, execute the shell inside the **foo** container:

```docker
docker exec -it foo /bin/bash
```

## Using headless containers

There are two ways, how to use the created headless containers. Note that the default **VNC user** password is **headless**.

### Over VNC

To be able to use the containers over **VNC**, a **VNC Viewer** is needed (e.g. [TigerVNC][tigervnc] or [TightVNC][tightvnc]).

The VNC Viewer should connect to the host running the container, pointing to the host's TCP port mapped to the container's TCP port **5901**.

For example, if the container has been created on the host called `mynas` using the parameters described above, the VNC Viewer should connect to `mynas:25901`.

### Over noVNC

To be able to use the containers over [noVNC][novnc], an HTML5 capable web browser is needed. It actually means, that any current web browser can be used.

The browser should navigate to the host running the container, pointing to the host's TCP port mapped to the container's TCP port **6901**.

However, the containers offer two [noVNC][novnc] clients - **lite** and **full**. The connection URL differs slightly in both cases. To make it easier, a simple startup page is implemented.

If the container have been created on the host called `mynas` using the parameters described above, then the web browser should navigate to `http://mynas:26901`.

The startup page will show two hyperlinks pointing to the both noVNC clients:

- `http://mynas:26901/vnc_lite.html`
- `http://mynas:26901/vnc.html`

It's also possible to provide the password through the links:

- `http://mynas:26901/vnc_lite.html?password=headless`
- `http://mynas:26901/vnc.html?password=headless`

## Issues

If you have found a problem or you just have a question, please check the [Issues][this-issues] and the [Troubleshooting][this-wiki-troubleshooting], [FAQ][this-wiki-faq] and [HOWTO][this-wiki-howto] sections in [Wiki][this-wiki] first. Please do not overlook the closed issues.

If you do not find a solution, you can file a new issue. The better you describe the problem, the bigger the chance it'll be solved soon.

## Credits

Credit goes to all the countless people and companies who contribute to open source community and make so many dreamy things real.

***

[this-docker]: https://hub.docker.com/r/accetto/ubuntu-vnc-xfce-firefox-plus/
[this-github]: https://github.com/accetto/ubuntu-vnc-xfce-firefox-plus

[this-changelog]: https://github.com/accetto/ubuntu-vnc-xfce-firefox-plus/blob/master/CHANGELOG.md

[this-issues]: https://github.com/accetto/ubuntu-vnc-xfce-firefox-plus/issues
[this-issue-3]: https://github.com/accetto/ubuntu-vnc-xfce-firefox-plus/issues/3

[this-wiki]: https://github.com/accetto/ubuntu-vnc-xfce-firefox-plus/wiki
[this-wiki-howto]: https://github.com/accetto/ubuntu-vnc-xfce-firefox-plus/wiki/How-to
[this-wiki-troubleshooting]: https://github.com/accetto/ubuntu-vnc-xfce-firefox-plus/wiki/Troubleshooting
[this-wiki-faq]: https://github.com/accetto/ubuntu-vnc-xfce-firefox-plus/wiki/Frequently-asked-questions

[this-dockerfile-plus-preferences]: https://github.com/accetto/ubuntu-vnc-xfce-firefox-plus/blob/master/Dockerfile-plus-preferences
[this-dockerfile-plus-profile]: https://github.com/accetto/ubuntu-vnc-xfce-firefox-plus/blob/master/Dockerfile-plus-profile

[accetto-github]: https://github.com/accetto/
[accetto-docker]: https://hub.docker.com/u/accetto/

[accetto-github-ubuntu-vnc-xfce]: https://github.com/accetto/ubuntu-vnc-xfce
[accetto-docker-ubuntu-vnc-xfce]: https://hub.docker.com/r/accetto/ubuntu-vnc-xfce/

[accetto-github-ubuntu-vnc-xfce-firefox]: https://github.com/accetto/ubuntu-vnc-xfce-firefox/

[accetto-docker-xubuntu-vnc-firefox]:https://hub.docker.com/r/accetto/xubuntu-vnc-firefox
[accetto-xubuntu-vnc-wiki-image-hierarchy]: https://github.com/accetto/xubuntu-vnc/wiki/Image-hierarchy

[accetto-docker-xubuntu-vnc-novnc-firefox]: https://hub.docker.com/r/accetto/xubuntu-vnc-novnc-firefox
[accetto-xubuntu-vnc-novnc-wiki-image-hierarchy]: https://github.com/accetto/xubuntu-vnc-novnc/wiki/Image-hierarchy

[docker-ubuntu]: https://hub.docker.com/_/ubuntu/
[docker-doc]: https://docs.docker.com/
[docker-doc-managing-data]: https://docs.docker.com/storage/
[docker-for-windows]: https://hub.docker.com/editions/community/docker-ce-desktop-windows
[docker-desktop]: https://www.docker.com/products/docker-desktop

[qnap]: https://www.qnap.com/en/
[container-station]: https://www.qnap.com/solution/container_station/en/

[ubuntu-flavours]: https://www.ubuntu.com/download/flavours

[firefox]: https://www.mozilla.org
[mousepad]: https://github.com/codebrainz/mousepad
[novnc]: https://github.com/kanaka/noVNC
[nsswrapper]: https://cwrap.org/nss_wrapper.html
[tigervnc]: http://tigervnc.org
[tightvnc]: http://www.tightvnc.com
[vim]: https://www.vim.org/
[xfce]: http://www.xfce.org

[screenshot-container]: https://raw.githubusercontent.com/accetto/ubuntu-vnc-xfce-firefox-plus/master/ubuntu-vnc-xfce-firefox-plus.jpg

<!-- docker badges -->

[badge-docker-pulls]: https://badgen.net/docker/pulls/accetto/ubuntu-vnc-xfce-firefox-plus?icon=docker&label=pulls

[badge-docker-stars]: https://badgen.net/docker/stars/accetto/ubuntu-vnc-xfce-firefox-plus?icon=docker&label=stars

<!-- github badges -->

[badge-github-release]: https://badgen.net/github/release/accetto/ubuntu-vnc-xfce-firefox-plus?icon=github&label=release

[badge-github-release-date]: https://img.shields.io/github/release-date/accetto/ubuntu-vnc-xfce-firefox-plus?logo=github

<!-- latest tag badges -->

[badge-VERSION_STICKER_LATEST]: https://badgen.net/badge/version%20sticker/ubuntu18.04.3-firefox69.0.2/blue

[badge-github-commit-latest]: https://images.microbadger.com/badges/commit/accetto/ubuntu-vnc-xfce-firefox-plus.svg
