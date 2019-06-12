.. role:: json(code)
   :language: json
   
.. role:: sh(code)
    :language: sh

Registration, authentication and management of users
====================================================

How to work with coolsignup
^^^^^^^^^^^^^^^^^^^^^^^^^^^

*coolsignup* has a REST-like api. You communicate with the *coolsignup* server by sending http requests to it. The data you post to *coolsignup* is in *json* format and the responses you receive are also in this format. The api extensively uses POST requests even for actions without side effects. This departs from the traditional REST canons but helps avoiding problems with potential caching or logging of sensitive data.

In this documentation, we use ``curl`` to give examples of how to make requests to the API. ``curl`` is a UNIX utility used to send http requests. Obviously, when  using *coolsignup* in your application, you will use whatever is the preferred way to send http requests in the particular language or framework you're working with.

The requests you make are to be made server side: the web application running on your servers makes the requests to the *coolsignup* server. 
**The end user must not be able to access the coolsignup server directly**.

Registering users with an email address and a password
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

1. Check the email address
~~~~~~~~~~~~~~~~~~~~~~~~~~

The first step is to ask the user for their email address and to check the email address.
Say your user entered the ``user@example.com`` email address in a HTML form.
Make the following request to *coolsignup*:


.. code:: sh

    curl -X POST /api/email/send-registration-code
         -d '{"email": "user@example.com"}'
    
This will send an email containing an email confirmation link to the user. If the ``emailCodeLink`` parameter of your configuration file is ``"https://auth.mysite.org/code/"``, then this link will look like ``https://auth.mysite.org/code/QmQQaP0IKiDrGtdtjbVa5V8jjNYDsI8K``.

As stated in the Internationalization section, you can add a ``lang`` query string parameter to your url. For example:

.. code:: sh

    curl -X POST /api/email/send-registration-code?lang=fr
         -d '{"email": "user@example.com"}'
    
Then the user will receive an email in French.

If all is well, you get the following response:

.. code:: json

    {"success": true}

If something goes wrong, you receive:

.. code:: json

    {"success": false, "error": "message explaining what went wrong"}

For example:

.. code:: json

    {
        "success": false,
        "error": "This email already belongs to a registered account"
    }
            
2. Check the confirmation email link
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Assuming all went well in the previous step, your user has received a link looking like this:

.. code::
    
    https://auth.mysite.org/code/QmQQaP0IKiDrGtdtjbVa5V8jjNYDsI8K
    
They clicked on it or maybe they copy pasted it in the url bar of their browser.

You extract the code from the url, which, in this example, is ``QmQQaP0IKiDrGtdtjbVa5V8jjNYDsI8K``.

You need to check if the code is valid. The goal of this step is only to make sure your user didn't screw up when clicking or copy-pasting the link.

.. code:: sh

    curl -X POST /api/email/check-registration-code
         -d '{"code": "QmQQaP0IKiDrGtdtjbVa5V8jjNYDsI8K"}'

The response will be either:

.. code:: json

    {"success": true}

or

.. code:: json

    {"success": false, "error": "Invalid registration code."}
    
You can add a ``lang`` parameter to the query string to receive the error message in a different language.
    

If the code is not valid, you should inform your user about that so that they double check the link they followed.
If the code is valid, you can proceed to the next step: registering the user.

3. Register the user
~~~~~~~~~~~~~~~~~~~~~~

Use the registration code you extracted from the url in step 2. Then post to the following url:

.. code:: sh

    curl -X POST /api/email/register -d '{
        "registrationCode": "QmQQaP0IKiDrGtdtjbVa5V8jjNYDsI8K",
        "password": "my password is strong as hell"
    }'
    
This will register the user with the password supplied.

The result you get is either:

.. code:: json

    {"success": true, "_id": "5cfae1ae7be455eeb9580c9c"}
    
where ``_id`` is the user id of the new user.
    
or, an error response such as:

.. code:: json

   {"success": false, "error": "Password too easy to guess."}
   {"success": false, "error": "Invalid registration code."}
   
Once you have created an account, you must activate it if you want the user to be able to login.
This is the subject of the next section.
   
Activate the user's account
^^^^^^^^^^^^^^^^^^^^^^^^^^^

Even when the user's email was successfully checked, *coolsignup* does not automatically activate the account after registration.
Maybe you'll want to manually check registrations before accepting them, or maybe you'll want to verify users by sending a code by SMS.
So, once you've made all these verifications, you post the ``_id`` of your user to the following url to activate the account:

.. code:: sh

    POST /api/activate-user {"_id": "5cfae1ae7be455eeb9580c9c"}

You'll get

.. code:: json

    {"success": true}

or an error such as:

.. code:: json

    {"success": false, "error": "no such user"}
    
Login
^^^^^

To log your user in, post to the following url:

.. code:: sh

    curl -X POST /api/email/login -d '{
        "email": "user@example.com",
        "password": "my password is strong as hell"
    }'

You'll get either something like:

.. code:: json

    {
        "success": true,
        "authToken": "bmbfVLnt95S33IY6...MjaPTuZmjdWmpAQ-QA8OStz"
    }

or an error:

.. code:: json

    {"success": "false", "error": "Wrong identifiers"}

The `"authToken"` value is a string that was cryptographically signed using the private key in your ``coolsignup.conf``. This string can be used to identify your user when using the rest of the *coolsignup* api. Typically, in a web app, you would store that authToken string in a cookie in the client's browser. In a phone app, you would store it in the local storage.

On login, you might want to retrieve the _id of your user. This is a string that uniquely identifies your user. You might also want to retrieve some fields about your user, for example a screen name, or a favorite color.

To retrieve user fields on login, add a ``"fields"`` list of fields to your request. For example:

.. code:: sh

    curl -X POST /api/email/login -d '{
        "email": "user@example.com",
        "password": "my password is strong as hell",
        "fields": ["_id", "screenName", "favoriteColor"]
    }'
    
You get

.. code:: json
    
    {
        "success": true,
        "authToken": "bmbfVLnt95S33IY6...MjaPTuZmjdWmpAQ-QA8OStz",
        "user" : {
            "_id": "5cfae1ae7be455eeb9580c9c",
            "screenName": "Mauricio",
            "favoriteColor": "orange"
        }
    }

    
Log out
^^^^^^^

To logout, post to the following url.

.. code:: sh

    curl -X POST /api/logout
         -d '{"authToken": "bmbfVLnt95S33IY6...MjaPTuZmjdWmpAQ-QA8OStz"}'

This action succeeds even if the ``authToken`` is invalid or was deleted from a previous logout.

.. code:: json

    {"success": "true"}

This will delete the current session from the *coolsignup* server. It won't be possible to use this ``authToken`` any longer.
It's now useless and you should delete it from the client side (cookie or local storage).


Using the user account once logged in
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Once logged in, you can use the ``authToken``, to:

retrieve the user id
~~~~~~~~~~~~~~~~~~~~

.. code:: sh

    curl -X POST /api/get-user-id -d '{
        "authToken": "bmbfVLnt95S33IY6...MjaPTuZmjdWmpAQ-QA8OStz"
    }'

.. code:: json

    {
        "success": true,
        "_id": "5cfae1ae7be455eeb9580c9c"
    }
    
You can now use this `_id` to uniquely identify the user in your application.
You can, for example, query or modify the user account in MongoDB using this `_id`.
User accounts are stored in MongoDB in the `coolsignup` database, in the `accounts` collection.
But you're not forced to use MongoDB. You can use this `_id` to feed another database, using, say, MySQL, if you want.


retrieve fields about the user
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: sh

    curl -X POST /api/get-user-fields -d '{
        "authToken": "bmbfVLnt95S33IY6...MjaPTuZmjdWmpAQ-QA8OStz",
        "fields": ["screenName", "email", "_id"]
    }'

.. code:: json

    {
        "success": true,
        "user": {
            "screenName": "Mauricio",
            "email": "mauricio@example.com",
            "_id": "5cfae1ae7be455eeb9580c9c"
        }
    }
    
Then for example, use the user id to deal with resources owned by your user in your application.

Note: apart from the `"_id"`, `"email"` and `"emailLower"` fields, other fields you can access using `/api/get-user-fields` are stored in the `"fields"` subdocument of the user document in the MongoDB database. This is something you should know if you intend to use MongoDB requests directly.

set a user's fields
~~~~~~~~~~~~~~~~~~~

.. code:: sh

    curl -X POST /api/set-user-fields -d '{
        "authToken": "bmbfVLnt95S33IY6...MjaPTuZmjdWmpAQ-QA8OStz",
        "set": {"screenName": "Pascal", "favoriteColor": "yellow"}
    }'
    
You get

.. code:: json

    {"success": true}
    
or

.. code:: json

    {"success": false, "error": "invalid authToken"}


For more complex modification of a user account, you might want to retrieve the user's ``_id`` and then modify the user using direct requests to the MongoDB backend. Users are stored in the "accounts" collection of the "coolsignup" database.

Note: when using `/api/set-user-fields`, you're not allowed to modify the `"_id"`, `"email"` and `"emailLower"` fields. Also, fields other than `"_id"`, `"email"` and `"emailLower"` that you can access are actually stored in the `"fields"` subdocument of the user document in the MongoDB database. This is something you should know if you intend to use MongoDB requests directly.


Password reset
^^^^^^^^^^^^^^

Sometimes the user is stupid: they forget their password.
Of course, it's not their fault if they are stupid so we have to provide them with a way of setting a new password.
There are three steps to do this.

1. Send the "reset password" code
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: sh

    curl -X POST /api/email/send-reset-password-code?lang=fr
         -d '{"email": "user@example.com"}'

This is going to send an email to the stupid user. This email will contain a link for him to reset his password.

The response to this request will be:

.. code:: sh

     {"success": "true"}
     
unless the email doesn't belong to a registered user. Then you'll get

.. code:: json

    {"success": "false", "error": "no such email in the database"}

    
2. Check the "reset password" code
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

So your stupid user received their password reset code in their mail box, in the form of a link that looks like:

.. code::
    
    https://auth.mysite.org/code/Eqtp1kNZeNNmSan2MxOXhLhMuc

Now are they smart enough to follow that link without screwing up?

You can check this by doing a POST request to the following url:

.. code:: sh

    curl -X POST /api/email/check-reset-password-code
         -d '{"code": "Eqtp1kNZeNNmSan2MxOXhLhMuc"}'

This will check if the code is a valid "reset password" code.
Hopefully you'll get

.. code:: json

    {"success": true}
    
If the code is not a valid code however, you'll get:

.. code:: json

    {"success": false, "error": "Invalid reset password link"}
    
3. Actually reset the password
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Extract the "reset password" code from the url at step 1. It looks like this: ``Eqtp1kNZeNNmSan2MxOXhLhMuc``.
Then post:

.. code:: sh

    curl -X POST /api/email/reset-password '{
        "resetPasswordCode": "Eqtp1kNZeNNmSan2MxOXhLhMuc",
        "newPassword": "my new password"
    }'
    
The password of the corresponding account has been succesfully reset:

.. code:: json

    {"success": true}
    
Unless something went wrong:

.. code:: json

    {"success": false, "error": "Password too easy to guess."}

    
Users changing their email addresses
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Sometimes users want to change their email address.
Users need to be logged in to do that. That is, you need to have a valid authToken.
The email change is done in three steps:

1. First check the new email address
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: sh

    curl -X POST /api/email/send-change-email-code '{
        "authToken": "bmbfVLnt95S33IY6...MjaPTuZmjdWmpAQ-QA8OStz",
        "newEmail": "bobby@example.com"
    }'

This will send a "change email" link to your user at the new email address. This link will look like this:

.. code::

    https://auth.mysite.org/code/Eqtp1kNZeNNmSan2MxOXhLhMuc
    
2. Check the code for copy/paste errors
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When your user clicks the link, you check if the code is valid, to inform them in case they failed to copy paste the link correctly in their browser.

.. code:: sh

    curl -X POST /api/email/check-change-email-code
         -d '{"code": "Eqtp1kNZeNNmSan2MxOXhLhMuc"}'
    
If the code is valid, you get:

.. code:: json

    {"success": true}

Otherwise:

.. code:: json

    {"success": false}

3. Actually change the email of the user
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: sh

    curl -X POST /api/email/change-email '{
        "authToken": "bmbfVLnt95S33IY6...MjaPTuZmjdWmpAQ-QA8OStz",
        "changeEmailCode": "Eqtp1kNZeNNmSan2MxOXhLhMuc"
    '}

If the ``authToken`` or the ``changeEmailCode`` are invalid, you'll get an error:

.. code:: json

    {"success": false, "error": "Authentication failure."}
    {"success": false, "error": "The code was invalid."}
    
Otherwise it's a success:

.. code:: json

    {"success": true}


Users changing their password
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Maybe a user got suspicious after realizing their cat was observing them typing on the keyboard.
They want to change their password.

This action requires a valid `authToken` and the current password.

.. code:: sh

    curl -X POST /api/email/change-password '{
        "authToken": "bmbfVLnt95S33IY6...MjaPTuZmjdWmpAQ-QA8OStz",
        "currentPassword": "my current password",
        "newPassword": "my new password"
    }

You should get the following response from *coolsignup*:

.. code:: json

    {"success": true}

unless something went wrong. For example if the new password supplied wasn't strong enough, you'll get:

.. code:: json

    {"success": false, "error": "The new password is too easy to guess"}


Deactivate an account
^^^^^^^^^^^^^^^^^^^^^

If a user did silly things, you can punish them by deactivating their account.
They won't be able to login anymore.

.. code:: sh

    curl -X POST /api/deactivate-user '{"_id": "347C983638B376343"}'


Delete an account
^^^^^^^^^^^^^^^^^

To delete a user account user the user's ``_id``:

.. code:: sh

    curl -X POST /api/delete-user '{"_id": "347C983638B376343"}'


If you want to give the opportunity to a user to delete their own account, first retrieve the user's ``_id`` by calling

.. code:: sh

    curl -X POST /api/get-user-fields '{
        "authToken": "bmbfVLnt95S33IY6...MjaPTuZmjdWmpAQ-QA8OStz",
        "fields": ["_id"]
    }"

Then make a POST request to the ``/api/delete-user``  url sending the ``_id`` you just retrieved.

