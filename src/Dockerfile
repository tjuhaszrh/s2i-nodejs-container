FROM {{ spec.s2i_base }}{% if not ":" in spec.s2i_base %}:latest{% endif %}

# This image provides a Node.JS environment you can use to run your Node.JS
# applications.

EXPOSE 8080

# Add $HOME/node_modules/.bin to the $PATH, allowing user to make npm scripts
# available on the CLI without using npm's --global installation mode
# This image will be initialized with "npm run $NPM_RUN"
# See https://docs.npmjs.com/misc/scripts, and your repo's package.json
# file for possible values of NPM_RUN

ENV NODEJS_VERSION={{ spec.version }} \
    NPM_RUN=start \
    NAME=nodejs \
    NPM_CONFIG_PREFIX=$HOME/.npm-global \
    PATH=$HOME/node_modules/.bin/:$HOME/.npm-global/bin/:$PATH \
    CNB_STACK_ID=com.redhat.stacks.{{ spec.org }}-nodejs-{{ spec.version }} \
    CNB_USER_ID=1001 \
    CNB_GROUP_ID=0

ENV SUMMARY="Platform for building and running Node.js $NODEJS_VERSION applications" \
    DESCRIPTION="Node.js $NODEJS_VERSION available as container is a base platform for \
building and running various Node.js $NODEJS_VERSION applications and frameworks. \
Node.js is a platform built on Chrome's JavaScript runtime for easily building \
fast, scalable network applications. Node.js uses an event-driven, non-blocking I/O model \
that makes it lightweight and efficient, perfect for data-intensive real-time applications \
that run across distributed devices."

LABEL summary="$SUMMARY" \
      description="$DESCRIPTION" \
      io.k8s.description="$DESCRIPTION" \
      io.k8s.display-name="Node.js $NODEJS_VERSION" \
      io.openshift.expose-services="8080:http" \
      io.openshift.tags="builder,$NAME,${NAME}${NODEJS_VERSION}" \
      io.openshift.s2i.scripts-url="image:///usr/libexec/s2i" \
      io.s2i.scripts-url="image:///usr/libexec/s2i" \
      io.buildpacks.stack.id="com.redhat.stacks.{{ spec.org }}-nodejs-{{ spec.version }}" \
      com.redhat.dev-mode="DEV_MODE:false" \
      com.redhat.deployments-dir="${APP_ROOT}/src" \
      com.redhat.dev-mode.port="DEBUG_PORT:5858" \
      com.redhat.component="{{ spec.redhat_component }}" \
      name="{{ spec.img_name }}" \
      com.redhat.license_terms="https://www.redhat.com/en/about/red-hat-end-user-license-agreements#UBI" \
      maintainer="SoftwareCollections.org <sclorg@redhat.com>" \
      help="For more information visit https://github.com/sclorg/s2i-nodejs-container" \
      usage="s2i build <SOURCE-REPOSITORY> {{ spec.full_image_name | default(spec.img_name) }} <APP-NAME>"

RUN {% if spec.uses_dnf_modules %}{{ spec.environment_setup }}
    MODULE_DEPS="{{ spec.build_deps }}" && \
    INSTALL_PKGS="$MODULE_DEPS {{ spec.pkgs }}" && \
{% else %}INSTALL_PKGS="{{ spec.build_deps }} {{ spec.pkgs }}" && \
{% endif %}
{% if not spec.needs_versioned_links %}\
    ln -s /usr/lib/node_modules/nodemon/bin/nodemon.js /usr/bin/nodemon && \
{% if spec.needs_python3_link and spec.version == "20" %}\
    ln -s /usr/libexec/platform-python /usr/bin/python3 && \
{% endif %}\
{% endif %}\
    {% if spec.uses_yum_for_v20 and spec.version == "20" %}yum{% else %}{{ spec.pkg_manager | default("dnf") }}{% endif %} install -y --setopt=tsflags=nodocs $INSTALL_PKGS && \
    rpm -V $INSTALL_PKGS && \
{% if spec.needs_versioned_links %}\
    rm -f /usr/bin/node && ln -s /usr/bin/node-$NODEJS_VERSION /usr/bin/node && \
    rm -f /usr/bin/npm && ln -s /usr/bin/npm-$NODEJS_VERSION /usr/bin/npm && \
    rm -f /usr/bin/npx && ln -s /usr/bin/npx-$NODEJS_VERSION /usr/bin/npx && \
{% endif %}\
    node -v | grep -qe "^v$NODEJS_VERSION\." && echo "Found VERSION $NODEJS_VERSION" && \
{% if spec.post_install %}    {{ spec.post_install }}
{% endif %}    {% if spec.uses_yum_for_v20 and spec.version == "20" %}yum{% else %}{{ spec.pkg_manager | default("dnf") }}{% endif %} -y clean all --enablerepo='*'

# Copy the S2I scripts from the specific language image to $STI_SCRIPTS_PATH
COPY ./s2i/bin/ $STI_SCRIPTS_PATH
RUN chmod +x $STI_SCRIPTS_PATH/init-wrapper

# Copy extra files to the image.
COPY ./root/ /

# Drop the root user and make the content of /opt/app-root owned by user 1001
RUN chown -R 1001:0 ${APP_ROOT} && chmod -R ug+rwx ${APP_ROOT} && \
    rpm-file-permissions

USER 1001

{% if spec.prod == "rhel8" and spec.version != "20" %}
# Enable the SCL for all bash scripts.
ENV BASH_ENV=/opt/rh/gcc-toolset-13/enable \
    ENV=/opt/rh/gcc-toolset-13/enable \
    PROMPT_COMMAND=". /opt/rh/gcc-toolset-13/enable"
{% endif %}

# Set the default CMD to print the usage of the language image
CMD $STI_SCRIPTS_PATH/usage
