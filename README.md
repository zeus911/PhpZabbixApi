## PhpZabbixApi

### About

PhpZabbixApi is an open-source PHP class library to communicate with the Zabbix™ JSON-RPC API.

Because PhpZabbixApi is generated directly from the origin Zabbix™ 2.0 PHP front-end source code / files, each real Zabbix™ JSON-RPC API method is implemented (hard-coded) directly as an own PHP method. This means PhpZabbixApi is IDE-friendly, because you've a PHP method for each API method, and there are no PHP magic functions or alike.

### License

PhpZabbixApi is licensed under the MIT license.

## Building

If you want to build your own library, have a look at the configuration file `inc/config.inc.php`.
You might want to point `PATH_ZABBIX`  to your Zabbix™ installation directory.

There are also pre-built libraries available in the `build/` directory, if you don't want to build it yourself.

## Using

Build your own library or download the pre-built libraries from the `build/` directory.
Make sure the version of the build matches the Zabbix™ PHP front-end version.

### Download

[Download](https://github.com/domibarton/PhpZabbixApi/tree/master/build)  pre-built PHP class library files directly from gitub (`build/` directory).

Make sure you've downloaded the following files and stored them in the same directory:

* `ZabbixApi.class.php`
* `ZabbixApiAbstract.class.php`

For example:

```
my_application
├── index.php
└── lib
    ├── ZabbixApiAbstract.class.php
    └── ZabbixApi.class.php
```

### Naming concept

To translate a Zabbix™ API call into a PHP method call, you can simply

1. remove the dot
2. capitalize the first letter of the action

Example:

```
Zabbix™ API         PHP API
-----------         -------
graph.get           graphGet()
host.massUpdate     hostMassUpdate()
dcheck.isWritable   dcheckIsWritable()
```

### Customizing the API class

By default there are only 2 classes defined:

```
ZabbixApiAbstract
└── ZabbixApi
```

If you want to customize or extend the library, you might want to do that in the `ZabbixApi` class.
Out of the box, `ZabbixApi` is an empty class inherited from `ZabbixApiAbstract`.

By customizing only `ZabbixApi`, you're able to update `ZabbixApiAbstract` (the build) at any time, without merging your customizations manually.

### Basic usage

To use the PhpZabbixApi you just have to load `ZabbixApi.class.php`, create a new `ZabbixApi` instance, and you're ready to go:

```php
<?php
// load ZabbixApi
require_once 'lib/ZabbixApi.class.php';

try
{
    // connect to Zabbix API
    $api = new ZabbixApi('http://zabbix.confirm.ch/api_jsonrpc.php', 'zabbix', 'admin');

    /* ... do your stuff here ... */
}
catch(Exception $e)
{
    // Exception in ZabbixApi catched
    echo $e->getMessage();
}
?>
```

## Examples

Please see also [the old project page](http://zabbixapi.confirm.ch/) for more examples.

### Simple request

Here's a simple request to fetch all defined graphs via [graph.get API method](https://www.zabbix.com/documentation/2.4/manual/api/reference/graph/get):

```php
    // get all graphs
    $graphs = $api->graphGet();

    // print all graph IDs
    foreach($graphs as $graph)
        echo $graph->graphid."\n";
```

### Request with parameters

Most of the time you want to define some specific parameters.
Here's an example to fetch all CPU graphs via [graph.get API method](https://www.zabbix.com/documentation/2.4/manual/api/reference/graph/get):

```php
    // get all graphs named "CPU"
    $cpuGraphs = $api->graphGet(array(
        'output' => 'extend',
        'search' => array('name' => 'CPU')
    ));

    // print graph ID with graph name
    foreach($cpuGraphs as $graph)
        printf("id:%d name:%s\n", $graph->graphid, $graph->name);
```

### Define default parameters

Sometimes you want to define default parameters, which will be included in each API request.
You can do that by defining the parameters in an array via `setDefaultParams()`:

```php
    // use extended output for all further requests
    $api->setDefaultParams(array(
        'output' => 'extend'
    ));

    // get all graphs named "CPU"
    $cpuGraphs = $api->graphGet(array(
        'search' => array('name' => 'CPU')
    ));

    // print graph ID with graph name
    foreach($cpuGraphs as $graph)
        printf("id:%d name:%s\n", $graph->graphid, $graph->name);
```

### Get associative / un-indexed array

By default all API responses will be returned in an indexed array.

So if you then looking for a specific named graph, you've to loop through the indexed array and compare the `name` attribute of each element. This can be a bit of a pain, and because of that, there's a simple way to to get an associative instead of an indexed array. You just have to define the 2nd parameter of the API method, which is the name of attribute you'd like to use as an array key.

Here's an example to fetch all graphs in an associative array, with the graph's `name` as array key:

```php
    // get all graphs in an associative array (key=name)
    $graphs = $api->graphGet(array(), 'name');

    // print graph ID with graph name
    if(array_key_exists('CPU Load Zabbix Server', $graphs))
        echo 'CPU Load graph exists';
    else
        echo 'Could not find CPU Load graph';
```
