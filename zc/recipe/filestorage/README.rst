===================================
Recipe for setting up a filestorage
===================================

This recipe can be used to define a file-storage.  It creates a ZConfig
file-storage database specification that can be used by other recipes to
generate ZConfig configuration files.

This recipe takes an optional path option.  If none is given, it creates and
uses a subdirectory of the buildout parts directory with the same name as the
part.

The recipe records a zconfig option for use by other recipes, e.g.
plone.recipe.zope2instance.

Supported options
=================

path
    The location of the ``Data.fs`` file to use for this filestorage.  If not
    specified, defaults to ``parts/part_name/Data.fs``.
blob-dir
    The location of the blobstorage directory associated with this filestorage.
    If this option is not specified or is set to the empty string, blobstorage will
    not be used for this fs.
mount-point
    The location where this filestorage will be mounted.  Must start with '/'.
    Defaults to '/' + part_name if not specified.


Examples
===============
Sample config::

    [buildout]
    extends = base.cfg
    parts =
        fs1
        instance
    
    [instance]
    recipe = plone.recipe.zope2instance
    user = me
    zope-conf-additional += ${fs1:zconfig}
    
    [fs1]
    recipe = zc.recipe.filestorage
    path = var/fs1/Data.fs
    blob-dir = var/fs1/blobstorage


We'll show a couple of interactive examples, using a dictionary as a simulated
buildout object:

    >>> import zc.recipe.filestorage
    >>> buildout = dict(
    ...   buildout = {
    ...      'directory': '/buildout',
    ...      },
    ...   db = {
    ...      'path': 'foo/Main.fs',
    ...      },
    ...   )
    >>> recipe = zc.recipe.filestorage.Recipe(
    ...                   buildout, 'db', buildout['db'])

    >>> print(buildout['db']['path'])
    /buildout/foo/Main.fs

    >>> from six import print_
    >>> print_(buildout['db']['zconfig'], end='')
    <zodb_db db>
      <filestorage>
        path /buildout/foo/Main.fs
      </filestorage>
      mount-point /db
    </zodb_db>

    >>> recipe.install()
    ()

    >>> import tempfile
    >>> d = tempfile.mkdtemp()
    >>> buildout = dict(
    ...   buildout = {
    ...      'parts-directory': d,
    ...      },
    ...   db = {},
    ...   )

    >>> recipe = zc.recipe.filestorage.Recipe(
    ...                   buildout, 'db', buildout['db'])

    >>> print(buildout['db']['path'])
    /tmp/tmpQo0DTB/db/Data.fs

    >>> print_(buildout['db']['zconfig'], end='')
    <zodb_db db>
      <filestorage>
        path /tmp/tmpQo0DTB/db/Data.fs
      </filestorage>
      mount-point /db
    </zodb_db>

    >>> recipe.install()
    ()

    >>> import os
    >>> os.listdir(d)
    ['db']

The update method doesn't do much, as the database part's directory
already exists, but it is present, so buildout doesn't complain and doesn't
accidentally run install() again:

    >>> recipe.update()

If the storage's directory is removed, is it re-added by the update method:

    >>> os.rmdir(os.path.join(d, 'db'))
    >>> os.listdir(d)
    []
    >>> recipe.update()
    >>> os.listdir(d)
    ['db']

This is useful in development when the directory containing the database is
removed in order to start the database from scratch.
