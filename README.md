# Documentation

## Facebook Request Query Mapper

* About
* Usage
* Mapping the fields
* Rules

### About

It can create Graph API or RESTFull request based on the certain rules and mapped fields. Fields mapper has the level where parameter has to be set and the api name that should be used for the query. Rules can be set for some fields so you can require some fields or set some fields dependencies on each other. FRQM returns Graph API query string or array of parameters that can be used for RESTFull request. Because of the incompatibility of some Instagram requests we have some rules for changing the nodes opened and closed tag.

### Usage

First query request class has to created by extending the **AbstractFacebookQueryRequestMapperService** abstract class with constants RULES and **FIELDS_MAPPER**.

```php
use Metrics\Data\Source\Facebook\Service\AbstractFacebookQueryRequestMapperService;

class InsightsQueryRequestMapperService extends AbstractFacebookQueryRequestMapperService
{
	const RULES = [];
	const FIELDS_MAPPER = [];
}
```
After FIELDS_MAPPER and RULES are set query request data will be returned by running: 

```php 
$requestParameters = new InsightsQueryRequestMapperService()->parseFields($qqlQuerySelectedColumns); 
```

### Mapping the fields

Important part is to set the right levels and names for the parameters. In fields mapper we have 4 parts:

* qqlSelectedColumnName
* Facebook Graph API name
* Level
* Value

Example(Instagram):
```php 
	const FIELDS_MAPPER = [ 
   		‘sinceDay' => ['name' => ‘since’, 'level' => '’], 
        'impressionsDay' => ['name' => 'impressions', 'level' => 'metric'], 
        'reachDay' => ['name' => 'reach', 'level' => 'metric’], 
   ];
```

Result: 
```javascript 
	metric{impressions,reach},since{1505113200} 
```

From the level that is set depends where parameter will be set in the query. If we use above example we can say that parameter **impressions** will be set in the metric node like its showed in the results.

If some filter has to be set we can set it by adding it in the level like this: *'cpp' => ['name' => 'cpp', 'level' => 'ads,insights.date_preset(lifetime)']*. So in this case lifetime filter will be used in the query so the query will look like this: *ads{insights.date_preset(lifetime){cpp}}*

### Rules

#### isRequired

If some field is required and we want it always set in the query regardless if its sent or not than we use this rule.

Example:
```php
	const RULES = [ 
   		time' => [ 'isRequired' ], 
   	];

	const FIELDS_MAPPER = [ 
    	'time' => ['name' => 'timestamp', 'level' => ''], 
    	'externalId' => ['name' => 'ig_id', 'level' => ''], 
    	'facebookPostId' => ['name' => 'id', 'level' => ''], 
    	'shortCode' => ['name' => 'shortcode', 'level' => ''], 
    	'message' => ['name' => 'caption', 'level' => ''], 
    ];
```

Results: 
```php 
timestamp,ig_id,id,shortcode,caption
```

#### childrenDecorator

Instagram is still not fully compatible with Facebook’s Graph API so as the default open-close tag for the mapper is ‘{’ and ‘}’ you can change it by using rule called **childrenDecorator**.

Example (Instagram):
```php

	const RULES = [ 
    	'insights.metric' => [ 'childrenDecorator' => [ '(', ')'] ], 
    ];

	const FIELDS_MAPPER = [ 
    	'impressions' => ['name' => 'impressions', 'level' => 'insights.metric'], 
        'reach' => ['name' => 'reach', 'level' => 'insights.metric'], 
    ];
```

Result: 
```php
insights.metric(impressions,reach)
```

#### metCriteria

If some fields have values (in case of RESTFull requests for the Instagram for example) we can set some criteria that has to be met for the value. For example **followerCount** field can be set only in the case where **period=day**. So if we want to prevent request in this case we can set rule **metCriteria** like in the example.

**Example:**
```php
	const RULES = [ 
    	'followerCount' => ['metCriteria' => ['fieldName' => 'period', 'expectedValue' => 'day', 'comparisonOperator' => ==']], 
    ];

	const FIELDS_MAPPER = [ 
    	'time' => ['name' => 'timestamp', 'level' => 'period’, value=‘day’], 
        'followerCountDay' => ['name' => 'follower_count', 'level' => 'metric'], 
    ];
```

Result: Exception will be thrown in case where field ‘period’ is not day.
