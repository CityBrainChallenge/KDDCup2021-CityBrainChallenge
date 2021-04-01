.. _Visualization:

Visualization
==================

Engine could log replay file. You could follow these steps to easily use these files to get visualization of your algorithm. But `mapbox token` and `yarn` is required.


1. Put the ``lightinfo.json``, ``roadinfo.json``, ``time*.json`` from `/log` to `/ui/src/log`
2. modify `/ui/src/index.js`

.. code-block::

    mapboxgl.accessToken = Your_Token;
    this.maxTime = max_value_of_*_of_time*.json

3. cd to `/ui`

.. code-block::

    yarn start

4. open `localhost:3000` with your browser

tips:

1. Sky blue indicates left-turning cars, dark blue indicates straight ahead cars, and dark green indicates right-turning cars.
2. The color of signal is meaningless.
3. Lines indicate roads.the color of the line represents the average speed.
4. Lane is not painted, so the car may not be painted on the line.

