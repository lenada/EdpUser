EdpUser
=======
Version 0.0.1 Created by Evan Coury

Introduction
------------

**NOTE:** This is a work-in-progress. The features listed may be incomplete or
entirely missing.

EdpUser is a user registration and authentication module for Zend Framework 2,
which can utilize either Doctrine2 or Zend\Db. It provides the foundations
for adding user authentication and registration to your ZF2 site. It is built to
be very simple and easily to extend.

Features / Goals
----------------

* Authenticate via username, email, or both (can opt out of the concept of 
  username and use strictly email) [COMPLETE]
* User registration [COMPLETE]
* Forms protected against CSRF [COMPLETE]
* Registration form protected with CAPTCHA [IN PROGRESS] (Needs more options)
* Out-of-the-box support for Doctrine2 _and_ Zend\Db [IN PROGRESS]
* Robust event system to allow for extending [IN PROGRESS]
* Support for additional authentication mechanisms via plugins (Google,
  Facebook, LDAP, etc) [INCOMPLETE]
* Optional E-mail address verification [INCOMPLETE]
* Forgot Password [INCOMPLETE]

Requirements
------------

* Zend Framework 2
* [EdpCommon](https://github.com/EvanDotPro/EdpCommon) (latest master).
* [SpiffyDoctrine](https://github.com/SpiffyJr/SpiffyDoctrine) (optional).

Installation (Doctrine2)
------------

1. Install the [EdpCommon](https://github.com/EvanDotPro/EdpCommon) ZF2 module.
    1. `cd path/to/project/modules`
    2. `git clone git://github.com/EvanDotPro/EdpCommon.git`
    3. Enable EdpCommon in your `application.config.php` modules array.
2. Install and configure the [SpiffyDoctrine](https://github.com/SpiffyJr/SpiffyDoctrine) ZF2 
   module per the [installation instructions](https://github.com/SpiffyJr/SpiffyDoctrine/blob/master/docs/INSTALL.md).
3. Install the [EdpUser](https://github.com/EvanDotPro/EdpUser) ZF2 module.
    1. `cd path/to/project/modules`
    2. `git clone git://github.com/EvanDotPro/EdpUser.git`
    3. Enable EdpUser in your `application.config.php` modules array.
4. Create the `user` table in your database via the Doctrine CLI:
    1. `cd path/to/SpiffyDoctrine/bin`
    2. `./doctrine orm:schema:update --dump-sql` (Import the schema into your DB)
    
Navigate to http://yourproject/user and you should land on a login page.

Installation (Zend\Db)
------------

1. Install the [EdpCommon](https://github.com/EvanDotPro/EdpCommon) ZF2 module.
    1. `cd path/to/project/modules`
    2. `git clone git://github.com/EvanDotPro/EdpCommon.git`
    3. Enable EdpCommon in your `application.config.php` modules array.
2. Install the [EdpUser](https://github.com/EvanDotPro/EdpUser) ZF2 module.
    1. `cd path/to/project/modules`
    2. `git clone git://github.com/EvanDotPro/EdpUser.git`
    3. Enable EdpUser in your `application.config.php` modules array.
3. Create the `user` table in your database. (The chema is located in
   `./EdpUser/data/schema.sql`)
4. Add the following to your `Application/configs/module.config.php`:

        // Application/configs/module.config.php
        array(
            'di' => array(
                'instance' => array(
                    'alias' => array(
                        'masterdb'            => 'PDO',
                        'edpuser_pdo'         => 'masterdb',
                        'edpuser_user_mapper' => 'EdpUser\Mapper\UserZendDb',
                    ),
                    'masterdb' => array(
                        'parameters' => array(
                            'dsn'            => 'mysql:dbname=CHANGEME;host=CHANGEME',
                            'username'       => 'CHANGEME',
                            'passwd'         => 'CHANGEME',
                            'driver_options' => array(PDO::MYSQL_ATTR_INIT_COMMAND => 'SET NAMES \'UTF8\''),
                        ),
                    ),
                ),
            ),
        );

Navigate to http://yourproject/user and you should land on a login page.

Options
-------

The EdpUser module has some options to allow you to quickly customize the basic
functionality. Options are defined in your Application module config like this:

    // Application/configs/module.config.php
    array(
        'edpuser' => array(
            'user_model_class'          => 'EdpUser\Model\User',
            'password_hash_algorithm'   => 'sha512',
            'enable_username'           => false,
            'enable_display_name'       => false,
            'require_activation'        => false,
            'login_after_registration'  => false,
            'registration_form_captcha' => true,
        ),
    )

The following options are available:

- **user_model_class** - Name of Entity class to use. Useful for using your own
  entity class instead of the default one provided. Default is `EdpUser\Model\User`.
- **password_hash_algorithm** - Name of the hashing algorithm to use for hashing.
  Checked against valid algorithms for PHP's `hash()` function. Examples include
  `sha1`, `sha224`, `sha256`, `sha384`, `sha512`, `md5`. Default is `sha512`.
  **Note:** This value can also be a valid PHP callback, or name of a static
  method such as `MyClass::hashPassword`.
- **enable_username** - Boolean value, enables username field on the registration
  form, and allows users to log in using their username _OR_ email address.
  Default is `false`.
- **enable_display_name** - Boolean value, enables a display name field on the
  registration form. Default value is `false`.
- **require_activation** - Boolean value, require that the user verify their email
  address to 'activate' their account. Default value is `false`. (Note, this doesn't
  actually work yet, but defaults an 'active' field in the DB to 0.)
- **login_after_registration** - Boolean value, automatically logs the user in
  after they successfully register. Default value is `false`.
- **registration_form_captcha** - Boolean value, determines if a captcha should
  be utilized on the user registration form. Default value is `true`. (Note,
  right now this only utilizes a weak Zend\Text\Figlet CAPTCHA, but I have
  plans to make all Zend\Captcha adapters work.)

Overriding / extending the User entity
--------------------------------------

Sometimes you may want to override the default user entity with your own. With
EdpUser, this is very easy.

First, create your extended User entity:

    // Application/src/Application/EdpUser/Model/User.php

    namespace Application\EdpUser\Model;

    use Doctrine\ORM\Mapping as ORM,
        EdpUser\ModelBase\UserBase;

    /**
     * @ORM\Entity
     * @ORM\Table(name="user")
     */
    class User extends UserBase
    {
        /**
         * You can add more stuff to the User entity here.
         */
    }

Next, tell EdpUser to utilize your new entity class:

    // Application/configs/module.config.php
    return array(
        'edpuser' => array(
            'user_model_class' => 'Application\EdpUser\Model\User',
        ),
    ),

If you're using Doctrine2, you'll also need to override the EdpUser entity path:

    // Application/configs/module.config.php
    return array(
        'di' => array(
            'instance' => array(
                'doctrine_driver_chain' => array(
                    'parameters' => array(
                        'drivers' => array(
                            'edpuser_annotationdriver' => array(
                                'class'           => 'Doctrine\ORM\Mapping\Driver\AnnotationDriver',
                                'namespace'       => 'Application\EdpUser\Model',
                                'paths'           => array(__DIR__ . '/../src/Application/EdpUser/Model'),
                            ),
                        ),
                    ),
                ),
            ),
        ),
    ),

Common Use-Cases
----------------

### Checking if a user is logged in from an ActionController
    
    if ($this->getLocator()->get('edpuser-user-service')->getAuthService()->hasIdentity()) {
        //...
    }

### Retrieving a user's identity from an ActionController
    
    $user = $this->getLocator()->get('edpuser-user-service')->getAuthService()->getIdentity();
    return array('user' => $user);

**Note:** `getIdentity()` returns an instance of `EdpUser\Entity\User`.

### Logging a user out from an ActionController

    $this->getLocator()->get('edpuser-user-service')->getAuthService()->clearIdentity();
