Creating your first plugin
==========================

So you've decided to take the plunge and create your first Velocity plugin?
That's awesome! This page will help you get you going.

Set up your environment
-----------------------

You're going to need the `JDK <http://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html>`_
and an IDE (we like `IntelliJ IDEA <https://www.jetbrains.com/idea/>`_, but any
IDE will work).

I know how to do this. Give me what I need!
-------------------------------------------

Maven repository
^^^^^^^^^^^^^^^^

+----------+-------------------------------------------------+
| **Name** | ``velocity``                                    |
+----------+-------------------------------------------------+
| **URL**  | ``https://repo.velocitypowered.com/snapshots/`` |
+----------+-------------------------------------------------+

Dependency
^^^^^^^^^^

+-----------------+-------------------------+
| **Group ID**    | ``com.velocitypowered`` |
+-----------------+-------------------------+
| **Artifact ID** | ``velocity-api``        |
+-----------------+-------------------------+
| **Version**     | ``1.0.9``               |
+-----------------+-------------------------+

Javadocs
^^^^^^^^

Javadocs are available at `jd.velocitypowered.com <https://jd.velocitypowered.com/>`_.

Setting up your first project
-----------------------------

If you need help setting up your project, don't worry!

Set up your build system
^^^^^^^^^^^^^^^^^^^^^^^^

You will need to set up a build system before you continue. Discussing how to
set up a build system for your project is out of scope for this page, but you
can look at the `Gradle <https://docs.gradle.org/current/userguide/userguide.html>`_
or `Maven <https://maven.apache.org/guides/getting-started/index.html>`_ documentation
for assistance.

Setting up the dependency with Gradle
"""""""""""""""""""""""""""""""""""""

Add the following to your ``build.gradle``:

.. code-block:: groovy

    repositories {
        maven {
            name 'velocity'
            url 'https://repo.velocitypowered.com/snapshots/'
        }
    }

    dependencies {
        compile 'com.velocitypowered:velocity-api:1.0.9'
    }

.. note::
    As of Gradle 5, you must also specify the API dependency as an annotation
    processor, otherwise plugin annotations won't be processed into the
    ``velocity-plugin.json`` file.

    .. code-block:: groovy

        dependencies {
            compile 'com.velocitypowered:velocity-api:1.0.9'
            annotationProcessor 'com.velocitypowered:velocity-api:1.0.9'
        }

Setting up the dependency with Maven
""""""""""""""""""""""""""""""""""""

Add the following to your ``pom.xml``:

.. code-block:: xml

    <repositories>
        <repository>
            <id>velocity</id>
            <url>https://repo.velocitypowered.com/snapshots/</url>
        </repository>
    </repositories>

    <dependencies>
        <dependency>
            <groupId>com.velocitypowered</groupId>
            <artifactId>velocity-api</artifactId>
            <version>1.0.0-SNAPSHOT</version>
            <scope>provided</scope>
        </dependency>
    </dependencies>

Create the plugin class
^^^^^^^^^^^^^^^^^^^^^^^

Create a new class (let's say ``com.example.velocityplugin.VelocityTest`` and paste
this in:

.. code-block:: java

    package com.example.velocityplugin;

    import com.google.inject.Inject;
    import com.velocitypowered.api.plugin.Plugin;
    import com.velocitypowered.api.proxy.ProxyServer;
    import org.slf4j.Logger;

    @Plugin(id = "myfirstplugin", name = "My First Plugin", version = "1.0-SNAPSHOT",
            description = "I did it!", authors = {"Me"})
    public class VelocityTest {
        private final ProxyServer server;
        private final Logger logger;
        
        @Inject
        public VelocityTest(ProxyServer server, Logger logger) {
            this.server = server;
            this.logger = logger;

            logger.info("Hello there, it's a test plugin I made!");
        }
    }

What did you just do there? There's quite a bit to unpack, so let's focus on the
Velocity-specific bits:

.. code-block:: java

    @Plugin(id = "myfirstplugin", name = "My First Plugin", version = "1.0-SNAPSHOT",
            description = "I did it!", authors = {"Me"})
    public class VelocityTest {

This tells Velocity that this class contains your plugin (``myfirstplugin``) so that
it can be loaded once the proxy starts up. Velocity will detect where the plugin
will reside when you compile your plugin.

.. code-block:: java

        @Inject
        public VelocityTest(ProxyServer server, Logger logger) {
            this.server = server;
            this.logger = logger;

            logger.info("Hello there, it's a test plugin I made!");
        }

This looks like magic! How is Velocity doing this? The answer lies in the ``@Inject``,
which indicates that Velocity should inject a ``ProxyServer`` and the ``Logger``
when constructing your plugin. These two interfaces will help you out as you begin
working with Velocity. We won't talk too much about dependency injection: all you
need to know is that Velocity will do this.

All you need to do is build your plugin, put it in your ``plugins/`` directory, and
try it! Isn't that nice? In the next section you'll learn about how to use the API.

A word of caution
^^^^^^^^^^^^^^^^^

In Velocity, plugin loading is split into two steps: construction and initialization.
The code in your plugin's constructor is part of the construction phase. There is
very little you can do safely during construction, especially as the API does not
specify which operations are safe to run during construction. Notably, you can't
register an event listener in your constructor, because you need to have a valid
plugin registration, but Velocity can't register the plugin until the plugin has
been constructed, causing a "chicken or the egg" problem.

To break this vicious cycle, you should always wait for initialization, which is
indicated when Velocity fires the ``ProxyInitializeEvent``. We can do things on
initialization by adding a listener for this event, as shown below. Note that
Velocity automatically registers your plugin main class as a listener.

.. code-block:: java

        @Subscribe
        public void onProxyInitialization(ProxyInitializeEvent event) {
            // Do some operation demanding access to the Velocity API here.
            // For instance, we could register an event:
            server.getEventManager().register(this, new PluginListener());
        }

Alternatively: Kotlin
^^^^^^^^^^^^^^^^^^^^^

Kotlin is the second-most popular language capable of running on the JVM (behind Java).
As such, it's a viable option for developing Velocity plugins; additionally, its more
modern syntax makes it a very attractive choice.

Here are some brief notes to assist in starting a Velocity plugin using Kotlin:

Your build script will look a little different than the above, because of the additional
dependency on the Kotlin stdlib and a specialized annotation processor that must be used
in place of Gradle's default. Also, it will probably be written in Kotlin itself, rather
than Groovy.

Your ``build.gradle.kts`` should include the following:

.. code-block:: kotlin

    plugins {
        kotlin("jvm") version "1.4.10"
        kotlin("kapt") version "1.4.10"
        id("com.github.johnrengelman.shadow") version "6.0.0"
    }

    repositories {
        mavenCentral()
        maven {
            url = uri("https://repo.velocitypowered.com/snapshots/")
        }
    }

    var velocityApi = "com.velocitypowered:velocity-api:1.0.9"

    dependencies {
        implementation(kotlin("stdlib"))
        implementation(velocityApi)
        kapt(velocityApi)
    }
    
    tasks["build"].dependsOn("shadowJar")

``kapt`` is the aforementioned Kotlin-specific annotation processor,
and ``shadow`` enables building "fat jars", which Velocity requires.

Then, your plugin class can look something like this:

.. code-block:: kotlin

    package com.example.velocityplugin

    import com.google.inject.Inject
    import com.velocitypowered.api.event.Subscribe
    import com.velocitypowered.api.event.proxy.ProxyInitializeEvent
    import com.velocitypowered.api.plugin.Plugin
    import com.velocitypowered.api.proxy.ProxyServer
    import org.slf4j.Logger

    @Plugin(
        id = "myfirstplugin",
        name = "My First Kotlin Plugin",
        version = "1.0-SNAPSHOT",
        description = "I did it!",
        authors = ["Me"]
    )
    class VelocityTest @Inject constructor(var server: ProxyServer, var logger: Logger) {

        @Subscribe
        fun onProxyInitialize(event: ProxyInitializeEvent) {
            logger.info("On proxy initialize")
            server.eventManager.register(this, PluginListener())
        }

    }

If you're familiar with Java and interested in learning Kotlin,
you can hit the ground running using the `Kotlin Koans <https://play.kotlinlang.org/koans/overview>`_ site.
