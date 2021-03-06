.. _changelog:

==============================
 Change history for Faust 1.4
==============================

This document contain change notes for bugfix releases in
the Faust 1.4 series. If you're looking for previous releases,
please visit the :ref:`history` section.

.. _version-1.4.2:

1.4.2
=====
:release-date: 2018-12-14 P.M PDT
:release-by: Ask Solem (:github_user:`ask`)

- **Agent**: Allow ``yield`` in agents that use ``Stream.take`` (Issue #237).

- **App**: Fixed error "future for different event loop" when web views
           send messages to Kafka at startup.

- **Table**: Table views now return HTTP 503 status code during startup
  when table routing information not available.

- **App**: New ``App.BootStrategy`` class now decides what services
  are started when starting the app.

- Documentation fixes by:

    - Robert Krzyzanowski (:github_user:`robertzk`).

.. _version-1.4.1:

1.4.1
=====
:release-date: 2018-12-10 4:49 P.M PDT
:release-by: Ask Solem (:github_user:`ask`)

- **Web**: Disable :pypi:`aiohttp` access logs for performance.

.. _version-1.4.0:

1.4.0
=====
:release-date: 2018-12-07 4:29 P.M PDT
:release-by: Ask Solem (:github_user:`ask`)

- **Requirements**

    + Now depends on :ref:`Mode 3.0 <mode:version-3.0.0>`.

- **Worker**: The Kafka consumer is now running in a separate thread.

    The Kafka heartbeat background corutine sends heartbeats every 3.0 seconds,
    and if those are missed rebalancing occurs.

    This patch moves the :pypi:`aiokafka` library inside a separate thread,
    this way it can send responsive heartbeats and operate even when agents
    call blocking functions such as ``time.sleep(60)`` for every event.

- **Table**: Experimental support for tables where values are sets.

    The new ``app.SetTable`` constructor creates a table where values are sets.
    Example uses include keeping track of users at a location:
    ``table[location].add(user_id)``.

    Supports all set operations: ``add``, ``discard``, ``intersection``,
    ``union``, ``symmetric_difference``, ``difference``, etc.

    Sets are kept in memory for fast operation, and this way we also avoid
    the overhead of constantly serializing/deserializing the data to RocksDB.
    Instead we periodically flush changes to RocksDB, and populate the sets
    from disk at worker startup/table recovery.

- **App**: Adds support for crontab tasks.

    You can now define periodic tasks using cron-syntax:

    .. sourcecode:: python

        @app.crontab('*/1 * * * *', on_leader=True)
        async def publish_every_minute():
            print('-- We should send a message every minute --')
            print(f'Sending message at: {datetime.now()}')
            msg = Model(random=round(random(), 2))
            await tz_unaware_topic.send(value=msg).

    See :ref:`tasks-cron-jobs` for more information.

    Contributed by Omar Rayward (:github_user:`omarrayward`).

- **App**: Providing multiple URLs to the :setting:`broker` setting
  now works as expected.

    To facilitiate this change ``app.conf.broker`` is now
    ``List[URL]`` instead of a single :class:`~yarl.URL`.

- **App**: New :setting:`timezone` setting.

    This setting is currently used as the default timezone for crontab tasks.

- **App**: New :setting:`broker_request_timeout` setting.

    Contributed by Martin Maillard (:github_user:`martinmaillard`).

- **App**: New :setting:`broker_max_poll_records` setting.

    Contributed by Alexander Oberegger (:github_user:`aoberegg`).

- **App**: New :setting:`consumer_max_fetch_size` setting.

    Contributed by Matthew Stump (:github_user:`mstump`).

- **App**: New :setting:`producer_request_timeout` setting.

    Controls when producer batch requests expire, and when we give up
    sending batches as producer requests fail.

    This setting has been increased to 20 minutes by default.

- **Web**: :pypi:`aiohttp` driver now uses ``AppRunner`` to start the web
  server.

    Contributed by Mattias Karlsson (:github_user:`stevespark`).

- **Agent**: Fixed RPC example (Issue #155).

    Contributed by Mattias Karlsson (:github_user:`stevespark`).

- **Table**: Added support for iterating over windowed tables.

    See :ref:`windowed-table-iter`.

    This requires us to keep a second table for the key index, so support
    for windowed table iteration requires you to set a ``use_index=True``
    setting for the table:

    .. sourcecode:: python

        windowed_table = app.Table(
            'name',
            default=int,
        ).hopping(10, 5, expires=timedelta(minutes=10), key_index=True)

    After enabling the ``key_index=True`` setting you may iterate over
    keys/items/values in the table:

    .. sourcecode:: python

        for key in windowed_table.keys():
            print(key)

        for key, value in windowed_table.items():
            print(key, value)

        for value in windowed_table.values():
            print(key, value)

    The ``items`` and ``values`` views can also select time-relative
    iteration:

    .. sourcecode:: python

        for key, value in windowed_table.items().delta(30):
            print(key, value)
        for key, value in windowed_table.items().now():
            print(key, value)
        for key, value in windowed_table.items().current():
            print(key, value)

- **Table**: Now raises error if source topic has mismatching
   number of partitions with changelog topic. (Issue #137).

- **Table**: Allow using raw serializer in tables.

    You can now control the serialization format for changelog tables,
    using the ``key_serializer`` and ``value_serializer`` keyword
    arguments to ``app.Table(...)``.

    Contributed by Matthias Wutte (:github_user:`wuttem`).

- **Worker**: Fixed spinner output at shutdown.

- **Models**: ``isodates`` option now correctly parses
  timezones without separator such as `-0500`.

- **Testing**: Calling ``AgentTestWrapper.put`` now propagates exceptions
  raised in the agent.

- **App**: Default value for :setting:`stream_recovery_delay` is now 3.0
  seconds.

- **CLI**: New command "clean_versions" used to delete old version directories
  (Issue #68).

- **Web**: Added view decorators: ``takes_model`` and ``gives_model``.
