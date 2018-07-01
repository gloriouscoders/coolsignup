.. role:: json(code)
   :language: json
   
.. role:: sh(code)
    :language: sh

Getting started
===============

Installation
^^^^^^^^^^^^

You need `pip3`, the python module manager for Python 3.
Then just:

.. code:: sh

    pip3 install coolsignup --user
    
*coolsignup* stores users' accounts into a MongoDB database,  so, you need to have MongoDB installed and running.

Refer to your operating system's documentation or to MongoDB's documentation if you need help about installing and running MongoDB.

Configuration
^^^^^^^^^^^^^

A configuration file for `coolsignup` is just a json file that looks like the following:

.. code:: json
    
    {
        "smtp": {
            "from": "noreply@example.com",
            "smtpServer": "smtp.example.com",
            "username": "registration_account",
            "password": "try to hack this, just try",
            "senderName": "example.com"    
        },
        "serviceName": "nuclear power plant management board",
        "passwordStrength": 8.5,
        "secretKey": "BQRekALJCog0EM69y9Lb45/EYGAh17X82St5XKIbqyRRzBde7UQClN/sLJ",
        "emailCodeLink": "https://auth.mysite.org/code/",
        "mongo": {"host": "localhost", "port": "27017"},
        "sessionDurationSeconds": 2592000,
        "codeDurationSeconds": 3600
           
    }
    
``"smtp"``: contains your SMTP parameters. They are used to send emails to the users. For example for the purpose of checking that the email address of a registering user is real, or to send "reset password" links by email.

``"serviceName"``: the name of your service or website, for example "Acme Inc" (when receiving welcome emails, your users will then get emails such as "Welcome to Acme Inc!")

``"emailCodeLink"``: used when sending code links by email to your users. For example, if you set this to ``"https://auth.mysite.org/code/"``, then links such as ``https://auth.mysite.org/code/4U6ZXZ8BoGWfRoGkR9JAEKlrT5eQaynmf`` will be emailed to your users to confirm their email address.
    
``"passwordStrength"``: the minimum password strength allowed to your users (as defined in the *zxcvbn* library). The default is 9. Increase it to enforce stronger passwords, decrease it to allow weaker passwords. You can experiment and check what kind of passwords are allowed.

``"secretKey"``: a string serving as a secret key to sign the authentication tokens. It's important to choose a long, high entropy key.

``"mongo"``: the configuration of your connection to MongoDB (defaults to :json:`{"host": "localhost", "port": "27017"}` )

``"sessionDurationSeconds"``: the duration of validity, in seconds, of an authentication token.  Default to 2592000 seconds (that is, 30 days)

``"codeDurationSeconds"``: the duration of validity, in seconds, of a code sent by email (such as a code to reset a password). Defaults to 3600 seconds.


Starting the server
^^^^^^^^^^^^^^^^^^^

The coolsignup script used to launch the server is situated in the ``.local/bin/`` subdirectory of your home directory, unless you installed ``coolsignup`` globally in which case it will be available just by calling ``coolsignup`` on the command line. 

The command to start the server is:

.. code:: sh

    .local/bin/coolsignup serve -c <conf> -p <port>

For example:

.. code:: sh

    .local/bin/coolsignup serve -c /home/berndt/coolsignup.conf -p 4444
    
would start the server with configuration file ``/home/berndt/coolsignup.conf`` and serve requests over port 4444.

**It's very important that this port is not open to the public. Only your web application must have access to it. Not the clients directly!**
