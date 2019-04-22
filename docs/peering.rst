=======
Peering
=======

All running operators communicate with each other via peering objects
(additional kind of custom resources), so they know about each other.


Priorities
==========

Each operator has a priority (the default is 0). Whenever the operator
notices that other operators start with a higher priority, it freezes
its operation until those operators stop working.

This is done to prevent collisions of multiple operators handling
the same objects.

To set the operator's priority, use :option:`--priority`:

.. code-block:: bash

    kopf run --priority=100 ...

As a shortcut, there is a :option:`--dev` option, which sets
the priority to ``666``, and is intended for the development mode.


Scopes
======

There are two types of custom resources used for peering:

* ``ClusterKopfPeering`` for the cluster-scoped operators.
* ``NamespacedKopfPeering`` for the namespace-scoped operators.

Kopf automatically chooses which one to use, depending on whether
the operator is restricted to a namespace with :option:`--namespace`,
or it is running unrestricted and cluster-wide.

.. deprecated:: 0.10
    Previously, ``KopfPeering`` was the only kind, and it was cluster-scoped.
    It is now abandoned and not supported.


Custom peering
==============

The operator can be instructed to use alternative peering objects::

    kopf run --peering=another ...

The operators from different peering objects do not see each other.

The default peering name (i.e. if no peering or standalone options are provided)
is ``default``.

This is especially useful for the cluster-scoped operators for different
resource kinds, which should not worry about other operators for other kinds.


Standalone mode
===============

To prevent an operator from peering and talking to other operators,
the standalone mode can be enabled::

    kopf run --standalone ...

In that case, the operator will not freeze if other operators with
the higher priority will start handling the objects, which may lead
to the conflicting changes and reactions from multiple operators
for the same events.


Multi-pod operators
===================

Usually, one and only one operator instance should be deployed for the resource.
If that operator's pod dies, the handling of the resource of this type
will stop until the operator's pod is restarted (and if restarted at all).

To start multiple operator pods, they must be distinctly prioritised.
In that case, only one operator will be active --- the one with the highest
priority. All other operators will freeze and wait until this operator dies.
Once it dies, the second highest priority operator will come into play.
And so on.

For this, assign a monotonically growing or random priority to each
operator in the deployment or replicaset:

.. code-block:: bash

    kopf run --priority=$RANDOM ...

``$RANDOM`` is a feature of bash (if you use another shell, find your own way).
It returns a random integer in the range 0..32767.
With high probability, 2-3 pods will get their unique priorities.

You can also use the pod's IP address in its numeric form as the priority,
or any other source of integers.
