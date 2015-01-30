=====================
Kagency Build Commons
=====================

This project defines a common build pipeline for ant based projects as well as
a common set of modules hooking into this build pipeline.

The tasks you can call are:

* clean

* initialize

* prepare

* test

  * test-dynamic

    * test-unit

    * test-feature

  * test-static

* package

The modules, you can activate in your custom build.xml hook into those steps as
apropriate. The modules themselves are build with the following constraints in
mind:

* Be multi-platform as far as possible

* Try to fail with errors which are easy to fix

* Seperate concerns as far as possible – do one job well.

The custom build.xml can be build in different variants.

Simple variant
==============

The simple variant expects the build-commons to already be intialized somehow.
It will just fail hard, if the main.xml etc. are not available::

    <?xml version="1.0" encoding="UTF-8"?>
    <project name="My Project" basedir="./" default="test">
        <!--
            Include local project properties.
        -->
        <property file="${basedir}/build.properties.local" />
        <property file="${basedir}/build.properties" />

        <!--
            Import main target defintions (extension points)
        -->
        <import file="${basedir}/ant/main.xml" />

        <!--
            Enable used modules
        -->
        <import file="${basedir}/ant/modules/composer.xml" />
        <import file="${basedir}/ant/modules/phpunit.xml" />
        <import file="${basedir}/ant/modules/behat.xml" />
        <import file="${basedir}/ant/modules/checkstyle.xml" />
        <import file="${basedir}/ant/modules/pdepend.xml" />
        <import file="${basedir}/ant/modules/phpcpd.xml" />
        <import file="${basedir}/ant/modules/phpmd.xml" />
    </project>

In the ``<project>``-Tag you define the prject name and the default target. The
``<property>``-Tags include custom local properties. You can define any
settings in those two lines.

The first ``<import>`` loads the mian build-pipeline definition. You always
want to include this one.

All following import statements include optional modules. Just include those
you want to use. This entirely depends on you project. You might also want to
include project specific custom "modules".

Automatic Initialization (GIT)
==============================

You can create a build.xml which ensures the submodule is initialized on the
first run. It looks fairly similar to the one above, but all imports are
optional (so that they do not fail, of not available) and it executes a git
submodule intialization, if the subdirectory is not available::

    <?xml version="1.0" encoding="UTF-8"?>
    <project name="My Project" basedir="./" default="install-and-run">

        <!--
            Include local project properties.
        -->
        <property file="${basedir}/build.properties.local" />
        <property file="${basedir}/build.properties" />

        <!--
            Import main target defintions (extension points)
        -->
        <import optional="true" file="${basedir}/ant/main.xml" />

        <!--
            Enable used modules
        -->
        <import optional="true" file="${basedir}/ant/modules/composer.xml" />
        <import optional="true" file="${basedir}/ant/modules/phpunit.xml" />
        <import optional="true" file="${basedir}/ant/modules/behat.xml" />
        <import optional="true" file="${basedir}/ant/modules/checkstyle.xml" />
        <import optional="true" file="${basedir}/ant/modules/pdepend.xml" />
        <import optional="true" file="${basedir}/ant/modules/phpcpd.xml" />
        <import optional="true" file="${basedir}/ant/modules/phpmd.xml" />

        <!--
            Task group, which installs the build-commons, if they do not exist yet.
        -->
        <target name="-install:check">
            <condition property="-install:dir-exists">
                <available file="${basedir}/ant" type="dir"/>
            </condition>
        </target>

        <target name="install" depends="-install:check" unless="-install:dir-exists">
            <exec executable="git" failonerror="true" dir="${basedir}">
                <arg value="submodule" />
                <arg value="update" />
                <arg value="--init" />
            </exec>

            <echo>Build-Commons submodule intialized. Please re-run the build.</echo>
            <fail />
        </target>

        <target name="install-and-run" depends="install">
            <antcall target="test" />
        </target>
    </project>

