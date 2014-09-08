.. vim: set fileencoding=utf-8 :
.. @author: Manuel Guenther <Manuel.Guenther@idiap.ch>
.. @date:   Mon Sep  8 15:37:06 CEST 2014

.. testsetup:: *

  import os
  YOUR_DATABASE_DIRECTORY = '/idiap/resource/database/YouTubeFaces/frame_images_DB'

==============
 User's Guide
==============

This database interface implements the default evaluation protocols as they are given on the `YouTube Faces Database web page <http://www.cs.tau.ac.il/~wolf/ytfaces>`_.
It implements the :py:mod:`bob.db.verifcation.utils` interface, so that it can be used like any other of our databases.

.. note::
  This database interface does not include the original data.
  To be able to run experiments on the YouTube Faces database, you need to get a copy of the original data from the above mentioned web page.


The Protocols
-------------

To use the protocol interface, you have to create an instance of the :py:class:`bob.db.youtube.Database`:

.. doctest::

   >>> import bob.db.youtube
   >>> db = bob.db.youtube.Database(YOUR_DATABASE_DIRECTORY)

where ``YOUR_DATABASE_DIRECTORY`` is the base directory, where the original images from the database can be found, e.g., ``'/path/to/YouTubeFaces/frame_images_DB'``.

The database interface contains several functions to query the database.
For example, to get the list of supported protocols, you can query the list of supported protocols:

.. doctest::

   >>> db.protocol_names()
   ('fold1', 'fold2', 'fold3', 'fold4', 'fold5', 'fold6', 'fold7', 'fold8', 'fold9', 'fold10')

These protocol names define the 10 different splits of the YouTube Faces protocol, for which experiments can be run.
Some of the remaining query functions require a protocol to be selected.

For each protocol, the splits of the database are distributed into three different groups: ``('world', 'dev', 'eval')``.

* The ``eval`` group contains exactly the split that is requested for the protocol.
  For this group, the final evaluation should be run, e.g., by classifying the corresponding pairs to be same class or different class.

* The ``dev`` group contains two splits, which contain different identities than the ``eval`` group.
  This group can be used, e.g., to select a threshold that is used to classify the pairs.

* Finally, the ``world`` group contains up to seven splits of the database, with identities distinct from the ``dev`` and ``eval`` groups.
  This split can be used to train your classifier.

For the final evaluation it is required that 10 different experiments are executed, e.g., by training 10 different classifiers on the according ``world`` splits, selecting 10 different thresholds on the ``dev`` set and compute 10 different classification results.
Finally, the classification accuracy is reported as an average of the 10 classification results.



The Directory Objects
---------------------

The most important method of the interface is the :py:func:`bob.db.youtube.Database.objects` function.
You can use this function to query the `information` for the protocols.
For the YouTube database, the `information` consists of a list of :py:class:`bob.db.youtube.models.Directory`.
Each ``Directory`` contains information about a video, such as the identity of the client, the shot id and the (relative) path of the directory in the database:

.. doctest::

   >>> objects = db.objects(protocol='fold1')
   >>> type(objects)
   <type 'list'>
   >>> d = objects[0]
   >>> type(d)
   <class 'bob.db.youtube.models.Directory'>
   >>> d.client_id
   1
   >>> d.shot_id
   0
   >>> d.path
   u'AJ_Cook/0'

These ``Directory`` objects can be used to get the path for the image data.
Since the videos are stored as a list of frames, the ``Directory`` interface will return a list of image file names, sorted by frame number:

.. doctest::

   >>> file_names = db.original_image_list(d)
   >>> print (file_names[0])    #doctest:+SKIP
   [...]/AJ_Cook/0/0.123.jpg


Finally, bounding boxes are annotated in the images.
To get these bounding boxes for a specific (set of) images, you can use the :py:func:`bob.db.youtube.Database.annotations` function.
In the example below, the annotations for the first 20 images are read and returned:

.. doctest::

  >>> file_name_stems = [os.path.basename(f) for f in file_names[:20]]
  >>> annotations = db.annotations(d.id, file_name_stems)
  >>> sorted(annotations.keys()) == file_name_stems
  True
  >>> bounding_box = annotations[file_name_stems[0]]
  >>> print (bounding_box)
  {'topleft': (56.0, 205.0), 'bottomright': (112.0, 261.0)}

The annotations for one image can, for example, be used to cut out the face region from the image:

.. code-block:: python

  >>> import bob.io.base
  >>> import bob.io.image
  >>> import bob.ip.color
  >>> color_image = bob.io.load(file_names[0])
  >>> gray_image = bob.ip.color.rgb_to_gray(color_image)
  >>> face_region = gray_image[bounding_box['topleft'][0] : bounding_box['bottomright'][0],
                               bounding_box['topleft'][1] : bounding_box['bottomright'][1]]


