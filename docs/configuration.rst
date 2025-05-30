.. _configuration:

=============
Configuration
=============


Command Line Options
====================

Locust is configured mainly through command line arguments.

.. code-block:: console

    $ locust --help

.. literalinclude:: cli-help-output.txt
    :language: console



.. _environment-variables:

Environment Variables
=====================

Options can also be set through environment variables. They are typically the same as the command line argument
but capitalized and prefixed with ``LOCUST_``:

On Linux/macOS:

.. code-block::

    $ LOCUST_LOCUSTFILE=custom_locustfile.py locust

On Windows:

.. code-block::

    > set LOCUST_LOCUSTFILE=custom_locustfile.py
    > locust

.. _configuration-file:

Configuration File
==================

Options can also be set in a configuration file in the
`config or TOML file format <https://github.com/bw2/ConfigArgParse#config-file-syntax>`_.

Locust will look for ``~/.locust.conf``, ``./locust.conf`` and ``./pyproject.toml`` by default.
You can specify an additional file using the ``--config`` flag.

.. code-block:: console

    $ locust --config custom_config.conf

Here's an example:

locust.conf
--------------

.. code-block:: ini

    locustfile = locust_files/my_locust_file.py
    headless = true
    master = true
    expect-workers = 5
    host = https://target-system
    users = 100
    spawn-rate = 10
    run-time = 10m
    tags = [Critical, Normal]

Have a look later in this page for `All available configuration options`_

pyproject.toml
--------------

When using a TOML file, configuration options should be defined within the ``[tool.locust]`` section.

.. code-block:: toml

    [tool.locust]
    locustfile = "locust_files/my_locust_file.py"
    headless = true
    master = true
    expect-workers = 5
    host = "https://target-system"
    users = 100
    spawn-rate = 10
    run-time = "10m"
    tags = ["Critical", "Normal"]

.. note::

    Configuration values are read (and overridden) in the following order:

    .. code-block:: console

       ~/.locust.conf -> ./locust.conf -> ./pyproject.toml -> (file specified using --conf) -> env vars -> cmd args


All available configuration options
===================================

Here's a table of all the available configuration options, and their corresponding Environment and config file keys:

.. include:: config-options.rst

Running without the web UI
==========================

See :ref:`running-without-web-ui`

Using multiple Locustfiles at once
==================================

``-f/--locustfile`` accepts multiple, comma-separated locustfiles.

Example:

With the following file structure:

.. code-block::

    ├── locustfiles/
    │   ├── locustfile1.py
    │   ├── locustfile2.py
    │   └── more_files/
    │       ├── locustfile3.py
    │       ├── _ignoreme.py

.. code-block:: console

    $ locust -f locustfiles/locustfile1.py,locustfiles/locustfile2.py,locustfiles/more_files/locustfile3.py

Locust will use ``locustfile1.py``, ``locustfile2.py`` & ``more_files/locustfile3.py``

Additionally, ``-f/--locustfile`` accepts directories as an option. Locust will recursively
search specified directories for ``*.py`` files, ignoring files that start with "_".

Example:

.. code-block:: console

    $ locust -f locustfiles

Locust will use ``locustfile1.py``, ``locustfile2.py`` & ``more_files/locustfile3.py``



You can also use ``-f/--locustfile`` for web urls. This will download the file and use it as any normal locustfile.

Example:

.. code-block:: console

    $ locust -f https://raw.githubusercontent.com/locustio/locust/master/examples/basic.py


.. _class-picker:

Pick User classes, Shapes and tasks from the UI
===============================================

You can select which Shape class and which User classes to run in the WebUI when running locust with the ``--class-picker`` flag.
No selection uses all the available User classes.

For example, with a file structure like this:

.. code-block::

    ├── src/
    │   ├── some_file.py
    ├── locustfiles/
    │   ├── locustfile1.py
    │   ├── locustfile2.py
    │   └── more_files/
    │       ├── locustfile3.py
    │       ├── _ignoreme.py
    │   └── shape_classes/
    │       ├── DoubleWaveShape.py
    │       ├── StagesShape.py


.. code-block:: console

    $ locust -f locustfiles --class-picker

The Web UI will display:

.. image:: images/userclass_picker_example.png

The class picker additionally allows for disabling individual User tasks, changing the weight or fixed count, and configuring the host.

It is even possible to add custom attributes that you wish to be configurable for each User. Simply add a ``json`` classmethod
to your user:

.. code-block:: python

    class Example(HttpUser):
        @task
        def example_task(self):
            self.client.get(f"/example/{self.some_custom_arg}")

        @classmethod
        def json(self):
            return {
                "host": self.host,
                "some_custom_arg": "example"
            }

Configure Users from command line
=================================

You can update User class attributes from the command line too, using the ``--config-users`` argument:

.. code-block:: console

    $ locust --config-users '{"user_class_name": "Example", "fixed_count": 1, "some_custom_attribute": false}'

To configure multiple users you pass multiple arguments to ``--config-users``, or use a JSON Array. You can also pass a path to a JSON file.

.. code-block:: console

    $ locust --config-users '{"user_class_name": "Example", "fixed_count": 1}' '{"user_class_name": "ExampleTwo", "fixed_count": 2}'
    $ locust --config-users '[{"user_class_name": "Example", "fixed_count": 1}, {"user_class_name": "ExampleTwo", "fixed_count": 2}]'
    $ locust --config-users my_user_config.json

When using this way to configure your users, you can set any attribute.

.. note::

    ``--config-users`` is a somewhat experimental feature and the json format may change even between minor Locust revisions.

Configuring Profiles
====================

Profiles are an advanced feature that allow for grouping and filtering testruns. A profile may be set using the ``--profile`` argument
or by inputting a value in the advanced options from the web ui.

It may be useful to see a list of existing profiles as options in the form. If you have such a list, you may set it in your locustfile:

.. code-block:: python

    from locust import events

    @events.init.add_listener
    def on_locust_init(environment, **kwargs):
        environment.web_ui.template_args["all_profiles"] = ["one", "two", "three"]


Custom arguments
================

See :ref:`custom-arguments`

Customization of statistics settings
====================================

Default configuration for Locust statistics is set in constants of stats.py file.
It can be tuned to specific requirements by overriding these values.
To do this, import locust.stats module and override required settings:

.. code-block:: python

    import locust.stats
    locust.stats.CONSOLE_STATS_INTERVAL_SEC = 15

It can be done directly in Locust file or extracted to separate file for common usage by all Locust files.

The list of statistics parameters that can be modified is:

+-----------------------------------------+----------------------------------------------------------------------------------+----------------------------------------------------------------------+
| Parameter name                          | Purpose                                                                          | Default value                                                        |
+-----------------------------------------+----------------------------------------------------------------------------------+----------------------------------------------------------------------+
| STATS_NAME_WIDTH                        | Width of column for request name in console output                               | terminal size or 80                                                  |
+-----------------------------------------+----------------------------------------------------------------------------------+----------------------------------------------------------------------+
| STATS_TYPE_WIDTH                        | Width of column for request type in console output                               | 8                                                                    |
+-----------------------------------------+----------------------------------------------------------------------------------+----------------------------------------------------------------------+
| CSV_STATS_INTERVAL_SEC                  | Interval for how frequently the CSV file is written if this option is configured | 1                                                                    |
+-----------------------------------------+----------------------------------------------------------------------------------+----------------------------------------------------------------------+
| CONSOLE_STATS_INTERVAL_SEC              | Interval for how frequently results are written to console / chart UI            | 2                                                                    |
+-----------------------------------------+----------------------------------------------------------------------------------+----------------------------------------------------------------------+
| HISTORY_STATS_INTERVAL_SEC              | Interval for how frequently results are written to history                       | 5                                                                    |
+-----------------------------------------+----------------------------------------------------------------------------------+----------------------------------------------------------------------+
| CURRENT_RESPONSE_TIME_PERCENTILE_WINDOW | Window size/resolution - in seconds - when calculating the current response      | 10                                                                   |
|                                         | time percentile                                                                  |                                                                      |
+-----------------------------------------+----------------------------------------------------------------------------------+----------------------------------------------------------------------+
| PERCENTILES_TO_REPORT                   | List of response time percentiles to be calculated & reported                    | [0.50, 0.66, 0.75, 0.80, 0.90, 0.95, 0.98, 0.99, 0.999, 0.9999, 1.0] |
+-----------------------------------------+----------------------------------------------------------------------------------+----------------------------------------------------------------------+
| PERCENTILES_TO_STATISTICS               | List of response time percentiles in the screen of statistics for UI             | [0.95, 0.99]                                                         |
+-----------------------------------------+----------------------------------------------------------------------------------+----------------------------------------------------------------------+
| PERCENTILES_TO_CHART                    | List of response time percentiles in the screen of chart for UI                  | [0.5, 0.95]                                                          |
+-----------------------------------------+----------------------------------------------------------------------------------+----------------------------------------------------------------------+

Customization of additional static variables
============================================

This table lists the constants that are set within locust and may be overridden.

+-------------------------------------------+--------------------------------------------------------------------------------------+
| Parameter name                            | Purpose                                                                              |
+-------------------------------------------+--------------------------------------------------------------------------------------+
| locust.runners.WORKER_LOG_REPORT_INTERVAL | Interval for how frequently worker logs are reported to master. Can be disabled      |
|                                           | by setting to a negative number                                                      |
+-------------------------------------------+--------------------------------------------------------------------------------------+
| locust.web.HOST_IS_REQUIRED               | Makes the host field for the webui required                                          |
+-------------------------------------------+--------------------------------------------------------------------------------------+
