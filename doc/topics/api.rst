HTTP API
========
For all endpoints you may provide HTTP Basic Auth credentials. Here is a quick
example that flushes and rebuilds the cache database::

    curl https://myadmin:myadminpass@pypi.myserver.com/admin/rebuild

``/simple/`` (or ``/pypi/``)
----------------------------
These endpoints are usually only used by ``pip``

``GET`` ``/simple/``
^^^^^^^^^^^^^^^^^^^^
Returns a webpage with links to all the pages for each unique package

**Example**::

    curl myserver.com/simple/

``POST`` ``/simple/``
^^^^^^^^^^^^^^^^^^^^^
Upload a package

**Parameters:**

* ``:action`` - The only valid value is ``'file_upload'``
* ``name`` - The name of the package being uploaded
* ``version`` - The version of the package being uploaded
* ``content`` (file) - The file object that contains the package data

**Example**::

    curl -F ':action=file_upload' -F 'name=flywheel' -F 'version=0.1.0' \
    -F 'content=@path/to/flywheel-0.1.0.tar.gz' myserver.com/simple/



``GET`` ``/simple/<package>/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Returns a webpage with all links to all versions of this package.

If :ref:`use fallback <use_fallback>` is enabled and the server does not
contain the package, this will return a ``302`` that points towards a fallback
server.

**Example**::

    curl myserver.com/flywheel/

``/api/``
---------
These endpoints are used by the web interface

``GET`` ``/api/package/[?verbose=true/false]``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
If ``verbose`` is False, return a list of all unique package names. If
``verbose`` is True, return a list of summarized data for each unique package
name.

**Parameters**:

* ``verbose`` (bool) - Determines the return format (default False)

**Example**::

    curl myserver.com/api/package/
    curl myserver.com/api/package/?verbose=true

**Sample Response**

for ``verbose=false``:

.. code-block:: javascript

    {
        "packages": [
            "flywheel",
            "pypicloud",
            "pyramid"
        ]
    }

for ``verbose=true``:

.. code-block:: javascript

    {
        "packages": [
            {
                "name": "flywheel",
                "stable": "0.1.0",
                "unstable": "0.1.0-2-g185e630",
                "last_modified": 1389945600
            },
            {
                "name": "pypicloud",
                "stable": "0.1.0",
                "unstable": "0.1.0-21-g4a739b0",
                "last_modified": 1390207478
            }
        ]
    }

``GET`` ``/api/package/<package>/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Get all versions of a package. Also returns if the user has write permissions
for that package.

**Example**::

    curl myserver.com/api/package/flywheel

**Sample Response**:

.. code-block:: javascript

    {
        "packages": [
            {
                "name": "flywheel",
                "last_modified": 1389945600
                "version": "0.1.0"
                "url": "https://pypi.s3.amazonaws.com/34c2/flywheel-0.1.0.tar.gz?Signature=%2FSJidAjDkXbDojzXy8P1rFwe1kw%3D&Expires=1390262542"
            },
            {
                "name": "flywheel",
                "last_modified": 1390207478
                "version": "0.1.0-21-g4a739b0",
                "url": "https://pypi.s3.amazonaws.com/81f2/flywheel-0.1.0-21-g4a739b0.tar.gz?Signature=%2FSJidAjDkXbDojzXy8P1rFwe1kw%3D&Expires=1390262542"
            },
        ],
        "write": true
    }

``POST`` ``/api/package/<package>/<version>/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Upload a package to the server. This is just a cleaner endpoint that does the
same thing as the ``POST`` ``/simple/`` endpoint.

**Parameters:**

* ``content`` (file) - The file object that contains the package data

**Example**::

    curl -F 'content=@path/to/flywheel-0.1.0.tar.gz' myserver.com/api/package/flywheel/0.1.0/


``DELETE`` ``/api/package/<package>/<version>/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Delete a package version from the server

**Example**::

    curl -X DELETE myserver.com/api/package/flywheel/0.1.0/

``PUT`` ``/api/user/<username>/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Register a new user account (if user registration is enabled). After
registration the user will have to be confirmed by an admin.

If the server doesn't have any admins then the first user registered becomes
the admin.

**Parameters:**

* ``password`` - The password for the new user account

**Example**::

    curl -X PUT -d 'password=foobar' myserver.com/api/user/LordFoobar

``POST`` ``/api/user/password``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Change your password

**Parameters:**

* ``old_password`` - Your current password
* ``new_password`` - The password you are changing to

**Example**::

    curl -d 'old_password=foobar&new_password=F0084RR' myserver.com/api/user/password

``/admin/``
-----------
These endpoints are used by the admin web interface. Most of them require your
:ref:`access backend <access_control>` to be mutable.

``GET`` ``/admin/rebuild/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^
Flush the cache database and rebuild it from the package list stored in S3

**Example**::

    curl myserver.com/admin/rebuild/

``POST`` ``/admin/register/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Set whether registration is enabled or not

**Parameters:**

* ``allow`` (bool) - If True, allow new users to register

**Example**::

    curl -d 'allow=true' myserver.com/admin/register/

``GET`` ``/admin/pending_users/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Get a list of all users that are registered and need confirmation from an admin

**Example**::

    curl myserver.com/admin/pending_users/

**Sample Response**:

.. code-block:: javascript

    [
        "LordFoobar",
        "TotallyNotAHacker",
        "Wat"
    ]

``GET`` ``/admin/user/``
^^^^^^^^^^^^^^^^^^^^^^^^
Get a list of all users and their admin status

**Example**::

    curl myserver.com/admin/user/

**Sample Response**:

.. code-block:: javascript

    [
        {
            "username": "LordFoobar",
            "admin": true
        },
        {
            "username": "stevearc",
            "admin": false
        }
    ]

``GET`` ``/admin/user/<username>/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Get detailed data about a single user

**Example**::

    curl myserver.com/admin/user/LordFoobar/

**Sample Response**:

    .. code-block:: javascript

        {
            "username": "LordFoobar",
            "admin": true,
            "groups": [
                "cool_people",
                "group2"
            ]
        }

``GET`` ``/admin/user/<username>/permissions/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Get a list of packages that a user has permissions on

**Example**::

    curl myserver.com/admin/user/LordFoobar/permissions/

**Sample Response**:

.. code-block:: javascript

    [
        {
            "package": "flywheel",
            "permissions": ["read", "write"]
        },
        {
            "package": "pypicloud",
            "permissions": ["read"]
        }
    ]

``DELETE`` ``/admin/user/<username>/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Delete a user

**Example**::

    curl -X DELETE myserver.com/admin/user/chump/

``POST`` ``/admin/user/<username>/approve/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Mark a pending user as approved

**Example**::

    curl -X POST myserver.com/admin/user/LordFoobar/approve/

``POST`` ``/admin/user/<username>/admin/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Grant or revoke admin privileges for a user.

**Parameters**:

* ``admin`` (bool) - If True, promote to admin. If False, demote to regular user.

**Example**::

    curl -d 'admin=true' myserver.com/admin/user/LordFoobar/admin/

``PUT`` ``/admin/user/<username>/group/<group>/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Add a user to a group

**Example**::

    curl -X PUT myserver.com/admin/user/LordFoobar/group/cool_people/

``DELETE`` ``/admin/user/<username>/group/<group>/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Remove a user from a group

**Example**::

    curl -X DELETE myserver.com/admin/user/LordFoobar/group/cool_people/

``GET`` ``/admin/group/``
^^^^^^^^^^^^^^^^^^^^^^^^^
Get a list of all groups

**Example**::

    curl myserver.com/admin/group/

**Sample Response**:

.. code-block:: javascript

    [
        "cool_people",
        "uncool_people",
        "marginally_cool_people"
    ]

``GET`` ``/admin/group/<group>/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Get detailed information about a group

**Example**::

    curl myserver.com/admin/group/cool_people

**Sample Response**:

.. code-block:: javascript

    {
        "members": [
            "LordFoobar",
            "stevearc"
        ],
        "packages": [
            {
                "package": "flywheel",
                "permissions": ["read", "write"]
            },
            {
                "package": "pypicloud",
                "permissions": ["read"]
            }
        ]
    }

``PUT`` ``/admin/group/<group>/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Create a new group

**Example**::

    curl -X PUT myserver.com/admin/group/cool_people/

``DELETE`` ``/admin/group/<group>/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Delete a group

**Example**::

    curl -X DELETE myserver.com/admin/group/uncool_people/

``GET`` ``/admin/package/<package>/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Get the user and group permissions for a package

**Example**::

    curl myserver.com/admin/package/flywheel/

**Sample Response**:

.. code-block:: javascript

    {
        "user": [
            {
                "username": "LordFoobar",
                "permissions": ["read", "write"]
            },
            {
                "username": "stevearc",
                "permissions": ["read"]
            }
        ],
        "group": [
            {
                "group": "marginally_cool_people",
                "permissions": ["read"]
            },
            {
                "group": "cool_people",
                "permissions": ["read", "write"]
            }
        ]
    }

``PUT`` ``/admin/package/<package>/(user|group)/<name>/(read|write)/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Grant a permission to a user or a group on a package

**Example**::

    curl -X PUT myserver.com/admin/package/flywheel/user/LordFoobar/read
    curl -X PUT myserver.com/admin/package/flywheel/group/cool_people/write

``DELETE`` ``/admin/package/<package>/(user|group)/<name>/(read|write)/``
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Revoke a permission for a user or a group on a package

**Example**::

    curl -X DELETE myserver.com/admin/package/flywheel/user/LordFoobar/read
    curl -X DELETE myserver.com/admin/package/flywheel/group/cool_people/write
