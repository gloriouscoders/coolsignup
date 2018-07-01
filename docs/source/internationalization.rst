Internationalization
====================

When sending requests to *coolsignup*, you can add an optional ``lang`` argument (two letters language code, such as ``en``, ``fr`` or ``es``) to the urls' query strings.

If you do that and the response to your request contains an error message intended to be redisplayed to the end user (such as, for example, ``"this is not a valid email address"``, or ``"this password is too weak"``), or if an email needs to be sent to the user by `coolsignup`, this message will be sent in the language you requested (provided this language is supported by *coolsignup*, otherwise, you'll get the message in English).

For example, the url to check that an email confirmation code is valid (we describe how this works in a section below) is:

.. code::

    /api/email/check-registration-code

If the code is not valid you'll get the response:

.. code:: json

    {"success": false, "error": "The confirmation code is not valid."}

By adding ``?lang=fr`` to the url:

.. code::

    /api/email/check-registration-code?lang=fr

you'll get the error message in French:

.. code:: json

    {"success": false, "error": "Le code de confirmation n'est pas correct."}

