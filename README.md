# Distro plugin for [Tutor](https://docs.tutor.overhang.io)

## What is distro

Distro is an opinioned openedx distribution with some custom stuff to have an easy-to-use
and a ready to deploy in local or in development openedx distribution.
This can be watch like a tutor-plugin but is taken a little bit far away.

## Installation

```bash
pip install git+https://github.com/eduNEXT/tutor-contrib-edunext-distro@v15.2.0
```

## Usage

```bash
tutor plugins enable distro

# Validator commands for config file
tutor distro syntax_validator
tutor distro repository-validator

# Enabler commands
tutor distro enable-themes
tutor distro enable_private_packages
```

### Documentation

Distro plugin manages a set of settings that you can configure, to know how to do that check:

- [How to custumize distro](./docs/how_to_customize_distro.rst)
- [How to add a new package](./docs/how_to_add_new_packages.rst)

# Required tutor settings

This plugin works with some docker images. These are defined by default
if you have different images that aren't based on these, you can have some problems.

```yaml
DOCKER_IMAGE_OPENEDX: "docker.io/ednxops/distro-edunext-edxapp:palma"
DOCKER_IMAGE_OPENEDX_DEV: "docker.io/ednxops/distro-edunext-edxapp-dev:palma"
```

Also, you need an edx-platform version distro compatible.

| openedx | distro   | tutor |
| ------- | -------- | ----- |
| lilac   | limonero | v12   |
| maple   | mango    | v13   |
| nutmeg  | nuez     | v14   |
| olive   | olmo     | v15   |
| palm    | palma    | v16   |

:warning: **NOTE**: From Olmo version Distro has not defaulted packages. Now it is necessary to add the packages you want in ``config.yml`` file. See [How to add a new package](./docs/how_to_add_new_packages.rst)

You can find distro releases on https://github.com/edunext/edunext-platform.

```yaml
EDX_PLATFORM_REPOSITORY: "https://github.com/eduNEXT/edunext-platform.git"
EDX_PLATFORM_VERSION: "ednx-release/palma.master"
```

# Packages

## How to add a new package

In your config.yml you can set any package following this structure:

```yaml
DISTRO_MY_PACKAGE_NAME_DPKG:
  index: git
  name: eox-package # directory name
  # ---- git package variables
  repo: eox-package # git repository name
  domain: github.com
  path: eduNEXT
  protocol: ssh
  # ---- end git package variables
  version: master
  private: true
  variables:
    development: {}
    production: {}
# If you want to install a package from pip
# you must set the index to pip and remove repository but this
# won't be installed as editable.
```

Package's variables will be used on cms and lms settings.

In the dev environment your package will be cloned on **/openedx/extra_deps/MY-PLUGIN-NAME**
if you want to edit it you can
[mount a volume](https://docs.tutor.overhang.io/dev.html?highlight=bind#manual-bind-mount-to-any-directory) to that path.

### Private package

In your new package you can set the setting **private** on true, It's mean that this won't be cloned
from a public repository, for it works you should run the command to clone private packages:

```bash
tutor distro enable-private-packages
```

- **local**: It will be necessary to build a new image and run the command tutor local do init && tutor local start again.
- **dev**: you must run the command tutor dev do init && tutor dev start again.

# Themes

Declare the path of your themes using `tutor config save --set DISTRO_THEMES_ROOT="your_path"`,
by default the themes path goes here **/openedx/themes**

:warning: **NOTE**: From Olmo version Distro has not defaulted themes path. Now it is necessary to add the themes path in ``config.yml`` file or running command above.

## How to add a theme

You can override the default themes on the config.yml but
this will remove them if you don't define them again.

Set the themes to clone:

```yaml
DISTRO_THEMES:
  - domain: github.com
    name: ednx-saas-themes
    path: eduNEXT
    protocol: ssh
    repo: ednx-saas-themes
    version: edunext/mango.master
```

Set themes dir:

```yaml
DISTRO_THEME_DIRS:
  - /openedx/themes/ednx-saas-themes/edx-platform
  - /openedx/themes/ednx-saas-themes/edx-platform/bragi-children
  - /openedx/themes/ednx-saas-themes/edx-platform/bragi-generator
```

Set themes name:

```yaml
DISTRO_THEMES_NAME:
  - bragi
```

Run the command to clone the themes:

```bash
tutor distro enable-themes
```

- **local**: you must build a new image to add the new themes and
  compile statics and run the command `tutor local do init && tutor local start` again.
- **dev**: you must run the command `tutor dev do init && tutor dev start` again.
  - **since tutor 13.0.0** you should recompile statics in the container, you could run the next command to do it:
  ```bash
  openedx-assets themes --theme-dirs THEME_DIRS --themes THEME_NAMES
  ```

:warning: **NOTE**: From Olmo version Distro has not defaulted themes. Now it is necessary to add the themes in ``config.yml`` file.

# Build a new image

**requirements:** you should have enabled the distro plugin, also you had have run the commands `tutor distro enable-themes` and `tutor distro enable-private-packages`.

1. You should change 2 variables in your config.yml to define the new DOCKER_IMAGE_OPENEDX and DOCKER_IMAGE_OPENEDX_DEV to use.

2. You should run the next command:

```bash
export DOCKER_BUILDKIT=1
tutor images build -a BUILDKIT_INLINE_CACHE=1 --docker-arg="--cache-from" --docker-arg="ednxops/distro-edunext-edxapp:mango" -a EDX_PLATFORM_REPOSITORY=https://github.com/eduNEXT/edunext-platform.git -a EDX_PLATFORM_VERSION=ednx-release/mango.master openedx
```

If you are using another edx-platform you should change it in the commando.

3. That command will create a new image with the tag defined in your DOCKER_IMAGE_OPENEDX, now, you should run the next command:

```bash
tutor images push openedx
```

# Validator Commands

## Check git repository URL

If you want to make sure that the git repository urls in the config.yml file are valid, run the following command:

```bash
tutor distro repository-validator
```

The command will check the git URLs of the OPENEDX_EXTRA_PIP_REQUIREMENTS element, for example: git+https://github.com/openedx/DoneXBlock@2.0.1#egg=done-xblock

It will also check all elements that end in DPKG and have the parameter private: false, for example:

```bash
DISTRO_EOX_HOOKS_DPKG:
  index: git
  name: eox-hooks
  repo: eox-hooks
  domain: github.com
  path: eduNEXT
  protocol: https
  private: false
  variables:
    development: {}
    production: {}
  version: master
```

## Check syntax in configuration file

If you want to validate the syntax of the config.yml file, run the following command:

```bash
tutor distro syntax-validator
```

The command will check the configuration for:

- Packages, ending whit _DPKG
- OPENEDX_EXTRA_PIP_REQUIREMENTS
- DISTRO_THEMES
- Theme settings like DISTRO_THEMES_NAME, DISTRO_THEME_DIRS and DISTRO_THEMES_ROOT
- INSTALL_EXTRA_FILE_REQUIREMENTS
- OPENEDX_EXTRA_SETTINGS

# Other Options

## How to add custom middlewares

You should set the variable **DISTRO_EXTRA_MIDDLEWARES** in your config.yml to add a new
middleware to **settings.MIDDLEWARE**

```yaml
DISTRO_EXTRA_MIDDLEWARES:
  - middleware.test.1
  - middleware.test.2
```

## How to add extra files requirements

You should set the variable **INSTALL_EXTRA_FILE_REQUIREMENTS** in your config.yml file if you need to install extra files with. The structure should be like:

```yaml
INSTALL_EXTRA_FILE_REQUIREMENTS:
  path: ./requirements/extra_file/
  files: [
    /edunext/base.txt,
    /test/test.txt
  ]
```

It's important that ``.txt`` files are added in requirements directory, similar to EXTRA PIP REQUIREMENTS from [Tutor](https://docs.tutor.overhang.io/configuration.html#installing-extra-xblocks-and-requirements).

## How to enable openedx extra settings

You should set the variable **OPENEDX_EXTRA_SETTINGS** in your config.yml file if you need to enable ``cms_env``, ``lms_env`` or ``pre_init_lms_task`` settings to plugins works as expected. For now the principals settings should be like this:

```yaml
OPENEDX_EXTRA_SETTINGS:
  cms_env: [
    USE_EOX_TENANT: true
  ]
  lms_env: [
    USE_EOX_TENANT: true,
    ENABLE_EOX_THEMING_DERIVE_WORKAROUND: true
  ]
  pre_init_lms_tasks: [
    ./manage.py lms migrate contenttypes,
    ./manage.py lms migrate eox_core,
    ./manage.py lms migrate eox_tenant,
    ./manage.py lms migrate eox_tagging,
    ./manage.py lms migrate eox_audit_model
  ]
```

The list could grow according to the needs that arise at the time of configuring the plugins.

:warning: **Note**: Other Options as ``INSTALL_EXTRA_FILE_REQUIREMENTS`` and ``OPENEDX_EXTRA_SETTINGS`` are included from Olmo version, you can use it from this release.

# License

This software is licensed under the terms of the AGPLv3.
