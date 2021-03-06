**********
Data Model
**********

Bluesky's event-based data model supports complex, asynchronous data collection
and enables sophisticated live, prompt, streaming, and *post-facto* data
analysis.

Overview
========

The data model is composed of six types of Documents, which in Python are
represented as dictionaries but could be represented as nested mappings (e.g.
JSON) in any language. Each document class has a defined, but flexible, schema.

* Run Start Document --- Everything we know about an experiment or simulation
  before any data acquisition begins: the who / why / what and metadata such as
  sample information.
* Event --- A "row" of measurements with associated timestamps.
* Event Descriptor --- Metadata about a series of Events. Envision
  richly-detail column headings in a table, encompassing physical units,
  hardware configuration information, etc.
* Run Stop Document ---  Everything that we can only know at the very end, such
  as the time it ended and the exit status (succeeded, aborted, failed due to
  error).

.. image:: _static/document-generation-timeline.svg
   :width: 100%
   :align: center

Event documents may contain the literal values or as *pointers* to values that
are stored in some external file or networked resource, yet to be loaded. Two
additional document types manage references to externally-stored data.

* Resource --- A pointer to an external file (or resource in general).
* Datum --- A pointer to a specific slice of data within a Resource.

Finally, Event and Datum can be represented in "paged" form, where multiple
rows are contained in one structure for efficient transport and vectorized
computation. The representations contain equivalent information: an EventPage
can always be losslessly transformed into an Event and vice versa.

The scope of a "run" is left to the user. It is quite generic: a set of
Documents generated by following a given sequence of instructions. This might
encompass a single short measurement (say, "a scan") or a multi-step procedure.
It can be whatever scope is convenient for accessing a unit of data for later
analysis.

Document Types in Detail
========================

For each type, we will show:

* the minimal nontrivial example that satisfies the schema
* one or more "typical" examples generated by bluesky's RunEngine during an
  experiment
* the formal schema, encoded using
  `JSON schema <https://json-schema.org/understanding-json-schema/>`_

.. _start:

Run Start Document
------------------

Minimal nontrivial valid example:

..

   Documentation note: It might seem more natural to use json code-blocks here
   than python ones, but using python allows us to include comments in line.

.. code-block:: python

   {'time': 1550069716.5092213,  # UNIX epoch (seconds since 1 Jan 1970)
    'uid': '10bf6945-4afd-43ca-af36-6ad8f3540bcd'}  # globally unique ID

Typical example:

.. code-block:: python

   {'detectors': ['random_walk:x'],
    'hints': {'dimensions': [(['random_walk:dt'], 'primary')]},
    'motors': ('random_walk:dt',),
    'num_intervals': 2,
    'num_points': 3,
    'plan_args': {'args': ["EpicsSignal(read_pv='random_walk:dt', " "name='random_walk:dt', " 'value=1.0, ' 'timestamp=1550070001.828528, ' 'auto_monitor=False, ' 'string=False, ' "write_pv='random_walk:dt', " 'limits=False, ' 'put_complete=False)', -1, 1],
                  'detectors': ["EpicsSignal(read_pv='random_walk:x', " "name='random_walk:x', " 'value=1.61472277847348, ' 'timestamp=1550070000.807677, ' 'auto_monitor=False, ' 'string=False, ' "write_pv='random_walk:x', " 'limits=False, ' 'put_complete=False)'],
                  'num': 3,
                  'per_step': 'None'},
    'plan_name': 'scan',
    'plan_pattern': 'inner_product',
    'plan_pattern_args': {'args': ["EpicsSignal(read_pv='random_walk:dt', " "name='random_walk:dt', " 'value=1.0, ' 'timestamp=1550070001.828528, ' 'auto_monitor=False, ' 'string=False, ' "write_pv='random_walk:dt', " 'limits=False, ' 'put_complete=False)', -1, 1],
                          'num': 3},
    'plan_pattern_module': 'bluesky.plan_patterns',
    'plan_type': 'generator',
    'scan_id': 2,
    'time': 1550070004.9850419,
    'uid': 'ba1f9076-7925-4af8-916e-0e1eaa1b3c47'}

Formal schema:

.. literalinclude:: ../../event_model/schemas/run_start.json

.. _descriptor:

Event Descriptor
----------------

Minimal nontrivial valid example:

.. code-block:: python

   {'configuration': {},
    'data_keys': {'camera_image': {'dtype': 'number',
                                   'shape': [512, 512],
                                   'source': 'PV:...'}},
    'hints': {},
    'name': 'primary',
    'object_names': {},
    'run_start': '10bf6945-4afd-43ca-af36-6ad8f3540bcd',  # foreign key
    'time': 1550070954.276659,
    'uid': 'd08d2ada-5f4e-495b-8e73-ff36186e7183'}

Typical example:

.. code-block:: python

   {'configuration': {'random_walk:dt': {'data': {'random_walk:dt': -1.0},
                                         'data_keys': {'random_walk:dt': {'dtype': 'number',
                                                                          'lower_ctrl_limit': 0.0,
                                                                          'precision': 0,
                                                                          'shape': [],
                                                                          'source': 'PV:random_walk:dt',
                                                                          'units': '',
                                                                          'upper_ctrl_limit': 0.0}},
                                         'timestamps': {'random_walk:dt': 1550070004.994477}},
                      'random_walk:x': {'data': {'random_walk:x': 1.9221013521832928},
                                        'data_keys': {'random_walk:x': {'dtype': 'number',
                                                                        'lower_ctrl_limit': 0.0,
                                                                        'precision': 0,
                                                                        'shape': [],
                                                                        'source': 'PV:random_walk:x',
                                                                        'units': '',
                                                                        'upper_ctrl_limit': 0.0}},
                                        'timestamps': {'random_walk:x': 1550070004.812525}}},
    'data_keys': {'random_walk:dt': {'dtype': 'number',
                                     'lower_ctrl_limit': 0.0,
                                     'object_name': 'random_walk:dt',
                                     'precision': 0,
                                     'shape': [],
                                     'source': 'PV:random_walk:dt',
                                     'units': '',
                                     'upper_ctrl_limit': 0.0},
                  'random_walk:x': {'dtype': 'number',
                                    'lower_ctrl_limit': 0.0,
                                    'object_name': 'random_walk:x',
                                    'precision': 0,
                                    'shape': [],
                                    'source': 'PV:random_walk:x',
                                    'units': '',
                                    'upper_ctrl_limit': 0.0}},
    'hints': {'random_walk:dt': {'fields': ['random_walk:dt']},
              'random_walk:x': {'fields': ['random_walk:x']}},
    'name': 'primary',
    'object_keys': {'random_walk:dt': ['random_walk:dt'],
                    'random_walk:x': ['random_walk:x']},
    'run_start': 'ba1f9076-7925-4af8-916e-0e1eaa1b3c47',
    'time': 1550070005.0109222,
    'uid': '0ad55d9e-1b31-4af2-865c-7ab7c8171303'}
   
Formal schema:

.. literalinclude:: ../../event_model/schemas/run_start.json

.. _event:

Event Document
--------------

The Event document may contain data directly:

.. code-block:: python

   {'data': {'camera_image': [[...512x512 array...]]},
    'descriptor': 'd08d2ada-5f4e-495b-8e73-ff36186e7183',  # foreign key
    'filled': {},
    'seq_num': 1,
    'time': 1550072091.2793343,
    'timestamps': {'camera_image': 1550072091.2793014},
    'uid': '8eac2f83-2b3e-4d67-ae2c-1d3aaff29ff5'}

or it may reference it via a ``datum_id`` from a Datum document.

.. code-block:: python

   {'data': {'camera_image': '272132cf-564f-428f-bf6b-149ee4287024/1'},  # foreign key
    'descriptor': 'd08d2ada-5f4e-495b-8e73-ff36186e7183',  # foreign key
    'filled': {'camera_image': False},
    'seq_num': 1,
    'time': 1550072091.2793343,
    'timestamps': {'camera_image': 1550072091.2793014},
    'uid': '8eac2f83-2b3e-4d67-ae2c-1d3aaff29ff5'}

Note the appearance of ``{'filled': {'camera_image': False}}``, indicating that
the value of camera image is a foreign key; the data is not "filled in".

Typical example:

.. code-block:: python

   {'data': {'random_walk:dt': -1.0,
             'random_walk:x': 1.9221013521832928},
    'descriptor': '0ad55d9e-1b31-4af2-865c-7ab7c8171303',
    'filled': {},
    'seq_num': 1,
    'time': 1550070005.0189056,
    'timestamps': {'random_walk:dt': 1550070004.994477,
                   'random_walk:x': 1550070004.812525},
    'uid': '7b5343fe-dfd7-4884-bc18-a0b571ff60b7'}


Formal schema:

.. literalinclude:: ../../event_model/schemas/event.json

.. _event_page:

Event contents can also be represented in "paged" form, where multiple
rows are contained in one structure for efficient transport and vectorized
computation. The representations contain equivalent information: an EventPage
can always be losslessly transformed into an Event and vice versa.

.. literalinclude:: ../../event_model/schemas/event_page.json

.. _stop:

Run Stop Document
-----------------

Minimal nontrivial valid example:

.. code-block:: python

   {'uid': '546cc556-5f69-46b5-bf36-587d8cfe67a9',
    'time': 1550072737.175858,
    'run_start': '61bb1db8-c95c-4144-845b-e248c06d80e1',
    'exit_status': 'success',
    'reason': '',
    'num_events': {}}

Formal schema:

.. literalinclude:: ../../event_model/schemas/run_stop.json

.. _resource:

Resource Document
-----------------

Minimal nontrivial valid example:

.. code-block:: python

   {'path_semantics': 'posix',
    'resource_kwargs': {},
    'resource_path': '/local/path/subdirectory/data_file',
    'root': '/local/path/',
    'run_start': '10bf6945-4afd-43ca-af36-6ad8f3540bcd',
    'spec': 'SOME_SPEC',
    'uid': '272132cf-564f-428f-bf6b-149ee4287024'}


Formal schema:

.. literalinclude:: ../../event_model/schemas/resource.json

.. _datum:

Datum Document
--------------

Minimal nontrivial valid example:

.. code-block:: python

   {'resource': '272132cf-564f-428f-bf6b-149ee4287024',  # foreign key
    'datum_kwargs': {'index': 0},  # format-specific parameters 
    'datum_id': '272132cf-564f-428f-bf6b-149ee4287024/1'}

Formal schema:

.. literalinclude:: ../../event_model/schemas/datum.json

.. _datum_page:

Like Events, Datum contents can also be represented in "paged" form, and the
representations contain equivalent information.

.. literalinclude:: ../../event_model/schemas/datum_page.json

.. _bulk_events:

"Bulk Events" Document (DEPRECATED)
-----------------------------------

This is another representation of Events. This representation is deprecated.
Use EventPage instead.

.. literalinclude:: ../../event_model/schemas/bulk_events.json

.. _bulk_datum:

"Bulk Datum" Document (DEPRECATED)
----------------------------------

This is another representation of Datum. This representation is deprecated. Use
DatumPage instead.

.. literalinclude:: ../../event_model/schemas/bulk_datum.json
