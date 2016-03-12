# RREST

**RREST** helps to build a REST API in PHP.

[![Build Status](https://api.travis-ci.org/RETFU/RREST.svg?branch=master)](https://travis-ci.org/RETFU/RREST)
[![Coverage Status](https://coveralls.io/repos/github/RETFU/RREST/badge.svg?branch=master)](https://coveralls.io/github/RETFU/RREST?branch=master)
![WIP](https://img.shields.io/badge/unstable-master-red.svg)
![PHP7 supported](http://php7ready.timesplinter.ch/RETFU/RREST/badge.svg)
> Important: **RREST** is in active development. The API is not frozen and BC break can happen at any time.

**RREST** binds an [API specification language](https://en.wikipedia.org/wiki/Overview_of_RESTful_API_Description_Languages) (APISpec) and a Router. It also provides convention to follow best practice.

## Installation

> **RREST** is in active development and not available in [packagist](https://packagist.org) for the moment.

Manually update your composer.json file:

```json
"repositories": [
    {
        "type": "git",
        "url": "https://github.com/RETFU/RREST"
    }
],
"require": {
    "retfu/rrest": "dev-master"
}
```

```bash
$ composer update
```

## Usage

```php
<?php
use RREST\Router;
use RREST\APISpec;

$ramlFile = 'api.raml';

$app = new Silex\Application();

//more application logic here if needed

$apiDefinition = (new Raml\Parser())->parse($ramlFile, true);
$apiSpec = new APISpec\RAML($apiDefinition);
$router = new Router\Silex($app);

//bind RAML + Silex
$rrest = new RREST\RREST($apiSpec, $router, 'Controllers');
$route = $rrest->addRoute();

//more application logic here if needed

$app->run();

?>
```

## Features

### Validation

Based on the APISpec description, **RREST** validate inputs.  
Depending of the specification you choose,
**RREST** will provide out of box validation for headers, query parameters, protocol & request payload.

#### RAML

##### Query parameters ([RAML spec](https://github.com/raml-org/raml-spec/blob/master/raml-0.8.md#named-parameters))
* validate type: `string`, `integer`, `number`, `boolean` & `date`
* validate if required
* validate with pattern (only `string`)
* validate minimum/maximum length (only `string`)
* validate enum (only `string`)
* validate minimum/maximum (only `integer` & `number`)

##### Headers

`Accept` is required for every request.  
`Content-Type` is required for `POST` & `PUT` method.

This is a best practice that **RREST** follow.

> **RREST** only support JSON & XML for input/output, so valid mime-types are: `application/json`, `application/x-json`, `text/xml`, `application/xml` & `application/x-xml`.

##### Protocol

If you define your API endpoint as https only, **RREST** will validate that.

##### Request payload

Depending of the `Content-Type` header, **RREST** will validate:
* if the format (`JSON`or `XML`) is valid
* if it follow the schema (`JSON Schema` or `XML Schema`)

The schema is required by **RREST**, you must define it in your APISPec.  
This will ensure that data input is valid.

##### Cast

If query parameters & request body payload are valid, **RREST** will convert to the type define in the APISPec.  
At the end in the controller, you will have true json, xml for payload & integer, boolean... for query parameters.

##### Controller

**RREST** bind the route to a controller by following this convention:
* `POST /item/` -> `Controllers\Item#postAction`
* `GET /item/{itemId}/` -> `Controllers\Item#getAction`
* `GET /item/{itemId}/comment` -> `Controllers\Item\Comment#getAction`
* `PUT /item/{itemId}/comment/{commentId}` -> `Controllers\Item\Comment#putAction`

> note you can define the controller namespace (here Controllers) in the RREST\RREST constructor

#####  Response

 A `RREST\Response` is injecting into the controller with pre-filled values based on the APISpec or errors/exceptions that occur during the handling of a request.

You only need to fill response data to this object because the rest is configured:
* status code: 200 , 201 for `POST`, 40x ...
* header `Content-Type` when the response have a payload body
* the data will be serialized by **RREST** based on the `Content-Type` (you can provided serialized data too and disable the auto-serialization)

If you are in a `POST` request, you can set the location of the newly  resource.

```php
<?php
//Use Silex for Router

namespaces Controllers

use Silex\Application;
use Symfony\Component\HttpFoundation\Request;
use RREST\Response;

class Resource
{
    //note the $response is a RREST\Response
    public function postAction(Application $app, Request $request, Response $response)
    {
        $data = ''; // coming from your business logic

        $response->setContent($data); // set the data
        $response->setLocation($url); // in a POST request, you can define the location of the newly resource

        return $response->getRouterResponse(); //return the Router response configured
    }

    public function getAction(Application $app, Request $request, Response $response, $resourceId)
    {
        $data = json_encode(''); // coming from your business logic

        $response->setContent($data); // set the data

        return $response->getRouterResponse(false); //disable the auto-serialization
    }
}

?>
```

## Support

##### Format input/output
* JSON
* XML

##### APISPec
* RAML
* More coming soon

##### Router
* Silex
* More coming soon

## Contributing

Launch unit test:
```bash
./vendor/bin/atoum
```

## Why RREST

All those projects make an **amazing work**:
* [Microrest](https://github.com/marmelab/microrest.php)
* [The API Platform framework](https://github.com/api-platform/api-platform)
* [PSX Framework](https://github.com/k42b3/psx)

But they are frameworks. And all the magics come with dependencies like ORM, other frameworks, response format...

I want:
* to write a RREST API with a specification language like RAML, Swagger...
    * to keep my documentation up to date
    * to apply functional testing against
    * to provide a strong spec to write a client
* to plug it to the router/framework of my choice
* to manage all the business logics by my own
* to manage all the persistence layers by my own

## Roadmap
* validate headers like query parameters
* validate response like request

## Author

Fabien Furet - http://fabienfuret.net

## License

**RREST** is licensed under the MIT License - see the LICENSE file for details
      
