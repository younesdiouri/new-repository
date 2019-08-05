---
title: "GraphQL and SF4"
date: 2019-07-10T15:24:30-04:00
categories:
  - php
  - graphql
tags:
  - sf4
  - symfony
  - overblog
  - api
---

# Ever heard of GraphQL? 
GraphQL is a query language, fit for the website concept of data. It gives clients the power to ask for exactly what they need and nothing more.
You can check the [graphql](https://graphql.org) website.

Why GraphQL and not REST? I dunno really, I think both have their pros and cons, you can find on medium a lot of stories about benchmarking, how and why teams used graphql etc. 

Last thing about GraphQL is that it works perfectly with node. In the doc you can build a GraphQL relay / server and start creating your api.

Here's what you need to understand about GraphQL :

- Unlike a REST API, the GraphQL API has only one POST endpoint (ex : https://yoursite/graphql ). 
- In GraphQL you can **query** data, or **mutate** (=update) data. These 2 concepts are : Query and Mutation.
- You need to define your schema (as a GraphQL Schema) for all your entities (ex: Post, Comment, Like, User etc).

Let's not dig deep with the theory and let's move to the SF4 implementation.


# GraphQL and Symfony4 API : a piece of cake
SF4 is an amazing PHP framework, complying with the PSR-, using the Doctrine ORM and so much cool stuff. I will assume that you already know what are the basics of Symfony.
Now some of you never used a backend Symfony project as an **API**. By API, I mean that the backend is reachable by any of your frontends (mobile, PWA, React etc.) as a remote API.
Usually, we use TWIG as a front with SF or Encore (<3) for webpack, which is also very nice. Now that our backend is remote, you will have to secure it in a different manner than you did with the PHP SESSID, AKA sessions. We will use for example [Json Web Tokens](https://jwt.io) .

For REST, you may know the fabulous project Api-Platform (with a graphQL extension also). We will use [Overblog](https://github.com/overblog/GraphQLBundle) GraphQL Bundle. It is an amazing project that provides you a very cool graphQL server for Symfony. It uses also the [Webonyx/GraphqlPHP](https://github.com/webonyx/graphql-php), the GraphQL PHP reference today.

I used it for one big project of mine, and I must say **THANK YOU GUYS** for provinding this Open Source gem <3. 

> I came looking for copper and i found gold, Christopher Columbus.

Humhum, let's not heat the oven up, and put ourselves to work step by step. 

# Completely secured & working API with Symfony - JWT - GraphQL 

I am not fan of showing only the graphQL behavior with Symfony. If you know what are the concepts of an API (JWT, login, registration etc) you can jump like a beatiful kangouroo to the GraphQL implementation. 

## Devops :

Lets starts with the environnement. Should we use Docker? Wamp? Lamp? Xamp? Samp? Namp? Tamp?
I will show you two different ways of deployment (one with Docker, one without).

### With Docker and Docker-compose :

You'll obviously need Docker and Docker-compose on your local env. 
Here lays an example of docker-compose.yml :

```yaml

version: "3"

services:

# ------> adminer ------>
    adminer:
        container_name: adminer
        image: adminer:latest
        restart: always
        environment:
            ADMINER_PLUGINS: tables-filter tinymce
            ADMINER_DESIGN: pepa-linha
        ports:
            - 8080:8080
# <------ adminer <------

# ------> composer ------>
    composer:
        container_name: composer
        image: composer:latest
        volumes:
            - ./:/app
        command: ["composer", "update"]
# <------ composer <------

# ------> postgres ------>
    postgres:
        container_name: postgres
        image: postgres:latest
        restart: always
        environment:
            POSTGRES_USER: app
            POSTGRES_PASSWORD: root
            POSTGRES_DB: mydb
        volumes:
            - backer-db-data:/var/lib/postgresql/data
        ports:
            - 5432:5432
# <------ postgres <------

# ------> symfony ------>
    symfony:
        container_name: symfony
        build: ./docker/symfony
        image: skyflow/symfony
        restart: always
        working_dir: /webapp
        ports:
            - 80:80
        volumes:
            - ./:/webapp
            - ./docker/symfony/conf/apache2:/etc/apache2
            - ./docker/symfony/conf/php7/php.ini:/etc/php7/php.ini
# <------ symfony <------
```

Explanation :
The adminer part is optional as you can access your admin DB with HeidiSQL or some eq.
As for Symfony, I used a built version of apache2 php (from a friend of mine), but you can easy build something equivalent. 

Now you can run `docker-compose up` and use your composer container to generate a symfony 4 project : 

    docker-compose exec composer create-project symfony/website-skeleton my-api
    
The website skeleton will pull all the depedencies for a complete website (eg: twig, doctrine, etc). You can clean what you don't want in your API in your composer.json of course.

### Without Docker :

Now for another way of deploying your Symfony app you'll need to have PHP >=^7.1, PGSQL (> 9.x) and composer. It's very easy to install (even on Windows) and thanks to composer you can run:

    composer create-project symfony/website-skeleton my-api

### Common configurations for .env.local & postgres: 

We will need to perform a few changes before creating our entities etc.

In your SF4 project directory there should be a .env, .env.dist, .env.local. What will we use for our local development ? .env.local. 

```
#.env.local
    
APP_ENV=dev
APP_SECRET=thefamoussecretkeylol
DB_TYPE=postgres
DB_USER=app
DB_PASS=root
DB_ADDRESS=127.0.0.1
DB_PORT=5432
DB_NAME=mydb
DATABASE_URL=${DB_TYPE}://${DB_USER}:${DB_PASS}@${DB_ADDRESS}:${DB_PORT}/${DB_NAME}

CORS_ALLOW_ORIGIN=^https?://localhost(:[0-9]+)?$

MAILER_URL=null
```

Everything looks great. Of course you'll have to adapt the DB ids to yours.

As we are using postgres, you'll need to change the doctrine dbal driver :

```yaml
# config/packages/doctrine.yaml

doctrine:
  dbal:
    driver: 'pdo_postgresql'
    server_version: '10.5'
    charset: utf8
    default_table_options:
      charset: utf8
      collate: utf8_unicode_ci
    
    url: '%env(resolve:DATABASE_URL)%'
```

For the non-docker env users : 
You can now start your symfony app with `php bin/console server:run` !
No need for Apache, Active Directory, and all the pain with the php.ini.
Easy stuff isn't it?

## The JWT authentication :

Enough with the settings ladies & gentlemen. Let's secure our future graphQL API.
But before rushing deeply into the JWT implementation let's create a User entity and persist it. The maker bundle is mandatory for this command. 

    php bin/console make:user
    
Don't forget to d:s:u and now we have our User entity.


### The JWT token : Login and Registration

[JWT](https://jwt.io/) - Json Web Token - allows you to secure data between two parties. This is exactly what we need for our API.
For Symfony we have an amazing bundle made by [Lexik](https://github.com/lexik/LexikJWTAuthenticationBundle) very easy to deploy and well documented.
{: .notice}

First we'll add the dependency : 

    composer require "lexik/jwt-authentication-bundle"
    
Then generate the SSH keys :

    mkdir config/jwt
    openssl genrsa -out config/jwt/private.pem -aes256 4096
    openssl rsa -pubout -in config/jwt/private.pem -out config/jwt/public.pem
    
Remember the passphrase you used because you'll need it ;). 

To have proper dev and prod working environnement, I created a `lexik_jwt_authentication.yaml` file in **config/packages/dev** and **config/packages/prod**.

{% raw %}<img src="https://younesdiouri.github.io/assets/images/Capture.PNG" alt="jwt file structure">{% endraw %}

```yaml
# config/packages/dev/lexik_jwt_authentication.yaml
lexik_jwt_authentication:
    secret_key: '%env(resolve:JWT_SECRET_KEY)%'
    public_key: '%env(resolve:JWT_PUBLIC_KEY)%'
    pass_phrase: '%env(JWT_PASSPHRASE)%'
    user_identity_field: username
```

```yaml
# config/packages/prod/lexik_jwt_authentication.yaml
lexik_jwt_authentication:
    secret_key: '%env(JWT_SECRET_KEY)%'
    public_key: '%env(JWT_PUBLIC_KEY)%'
    pass_phrase: '%env(JWT_PASSPHRASE)%'
    user_identity_field: username
```

I used Heroku for production. You can easily have a CD with Heroku by hook with your master branch or a dedicated one. Don't forget to define the JWT environnement variables : `JWT_SECRET_KEY`, `JWT_PUBLIC_KEY`, `JWT_PASSPHRASE`.
{: .notice--info}


Now, let's add some firewalls (registration, login and API) :

```yaml
# config/packages/security.yaml
security:
    encoders:
       App\Entity\User:
            algorithm: argon2i

    # https://symfony.com/doc/current/security.html#where-do-users-come-from-user-providers
    providers:
        # used to reload user from session & other features (e.g. switch_user)
        app_user_provider:
            entity:
                class: App\Entity\User
    firewalls:
        dev:
            pattern: ^/(_(profiler|wdt)|css|images|js)/
            security: false
        login:
            pattern:  ^/login
            stateless: true
            anonymous: true
            json_login:
                check_path: /login
                username_path: username
                success_handler:          lexik_jwt_authentication.handler.authentication_success
                failure_handler:          lexik_jwt_authentication.handler.authentication_failure
        api:
            pattern:   ^/graphql/
            stateless: true
            anonymous: false
            provider: app_user_provider
            guard:
                authenticators:
                    - lexik_jwt_authentication.jwt_token_authenticator

        register:
            pattern:  ^/register
            stateless: true
            anonymous: true
            
    access_control:
        - { path: ^/login, roles: IS_AUTHENTICATED_ANONYMOUSLY }
        - { path: ^/register, roles: IS_AUTHENTICATED_ANONYMOUSLY }
```

The **register** firewall allows an anonymous access. 

The **login** firewall works the same way than the register but it will go through our JWT bundle handler. 

The **api** firewall is basically the graphql route : we need our user to be fully authenticated (we can also add it in the access_control area) and to secure it, we have the guard authenticator "JWT token authenticator".

This is not graphQL yet ! Indeed the register and login are regular Symfony routes. We can use graphQL with REST in the same project, depending on your needs. 

Now we append the routes :

```yaml
# config/routes.yaml

login:
  path:     /login
  methods:  [POST]

register:
  path: /register
  controller: App\Controller\ApiController::register
  methods: POST
  ```
  
  As you can see, the Lexik JWT Bundle is handling by itself the login method (no need to add it in the controller). For the registration, we will do it by ourselves :
  
  ```php
  //Controller/ApiController
  
 public function register(
        Request $request,
        UserPasswordEncoderInterface $encoder,
        JWTTokenManagerInterface $JWTManager,
        ValidatorInterface $validator,
        \Swift_Mailer $mailer
    ) {

        $em = $this->getDoctrine()->getManager();

        $data = json_decode($request->getContent());

        $user = $em->getRepository(User::class)->findOneBy(['username' => $data->username]);

        if($user || $em->getRepository(User::class)->findOneBy(['email' => $data->email]) ) {
            $response = new JsonResponse(["error" => 'Account already exist']);
            return $response->setStatusCode(409);
        }

        $user = new User();

        $user->setUsername($data->username);
        $user->setPassword($encoder->encodePassword($user, $data->password));
        $user->setEmail($data->email);

        $errors = $validator->validate($user);

        if (count($errors) > 0) {
            $errorsString = (string) $errors;
            $response = new Response($errorsString);
            $response->setStatusCode(400);
            return $response ;
        }

        $em->persist($user);
        $em->flush();

        $message = (new \Swift_Message('Welcome to the website'))
            ->setFrom('no-reply@website.com')
            ->setTo($user->getEmail())
            ->setBody(
                $this->renderView(
                    'api/registration.html.twig',
                    ['name' => $user->getUsername()]
                ),
                'text/html'
            )
        ;

        $mailer->send($message);

        return new JsonResponse(['token' => $JWTManager->create($user)]);
    }
```

Nothing weird in this Register method : we have the `$data = json_decode($request->getContent());` which get the Json data from the front-end machine, and if you want to work with php arrays you can use the `->toArray()` method. 
We check then if the email already exists (we can add some **password** strenght validators), then we persist and flush. 

I've added an emailing registration example, which can be easily implemented into your app and improves the UX / Security. 

Last but not least, we return a new JsonResponse with the Json Web Token freshly created, thanks to the `$JWTManager->create($user)`. 

Wait? That's it? 

Yup. We are set for our registration / login part. But what data could we send in this Token? Our client will decode the token and use it for example to display some data or query posts. 

### The JWT token : An Event Listener example

Let's assume we need the picture URL and the user ID.
We need to add these sets of key=>value into the JWT token.

> Thanks to Lexik, we can use the EventListner **JWTCreatedListener** .

```php
<?php
//src/EventListener
/**
 * Created by PhpStorm.
 * User: younesdiouri
 * Date: 01/03/2019
 * Time: 18:04
 */
namespace App\EventListener;
use App\Entity\User;
use Lexik\Bundle\JWTAuthenticationBundle\Event\JWTCreatedEvent;
use Symfony\Component\HttpFoundation\RequestStack;
use Symfony\Component\Security\Core\User\UserInterface;

class JWTCreatedListener
{
    /**
     * @var RequestStack
     */
    private $requestStack;

    /**
     * @param RequestStack $requestStack
     */
    public function __construct(RequestStack $requestStack)
    {
        $this->requestStack = $requestStack;
    }

    public function onJWTCreated(JWTCreatedEvent $event)
    {
        $request = $this->requestStack->getCurrentRequest();
        $payload = $event->getData();
        /** @var User $user */
        $user = $event->getUser();

        if (!$user instanceof UserInterface) {
            throw new \Exception('Does not match user interface', 500);
        }

        $payload['id'] = $user->getId()->toString();
        $payload['pictureUrl'] = $user->getPictureUrl();
        $event->setData($payload);
    }
}

```

This listener will listen (as expected) to the JWT created event. The $payload is the data stored by the JWT Token. 
The most important code block is `$user = $event->getUser()`. Why?

We need to minimize going back and forth between our server and our database. A good API should give a quick response to the client, thus using wisely the cache etc.
{: .notice--warning}

In this case, we are getting the user from the server cache, and not querying the DB (with the $em). 

Finally, we append the `$user->getId()` and `$user->getPicTureUrl()` to the payload.




