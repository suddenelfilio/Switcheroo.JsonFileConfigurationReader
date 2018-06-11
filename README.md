# Switcheroo.JsonFileConfigurationReader
A custom ConfigurationReader implementation for FeatureFlags library Switcheroo which is a lightweight framework for [feature toggling](http://martinfowler.com/bliki/FeatureToggle.html) to enable trunk based development.

Switcheroo was created by Riaan Hanekom: [github](https://github.com/rhanekom)

Switcheroo aims for simplicity, with a clean syntax and a minimal feature set while not compromising on extensibility and testability.

Getting Switcheroo.JsonFileConfigurationReader
------------------

Switcheroo.JsonFileConfigurationReader can be installed via [Nuget](https://www.nuget.org/packages/Switcheroo.JsonFileConfigurationReader/).

```powershell
> Install-Package Switcheroo.JsonFileConfigurationReader
```

License
--------

Switcheroo.JsonFileConfigurationReader is licensed under the [MIT license](http://opensource.org/licenses/MIT).


Quick Start
------------

**Installation**

Nuget packages can be found [here](https://www.nuget.org/packages/Switcheroo.JsonFileConfigurationReader/).

**Add configuration as a json file e.g. features.json**

```json
[
  {
    "name": "testDateRange",
    "enabled": true,
    "from": "1 November 2012",
    "until": "2 November 2012"
  },
  {
    "name": "testDateRangeFromOnly",
    "enabled": true,
    "from": "1 November 2012"
  },
  {
    "name": "testDateRangeUntilOnly",
    "enabled": true,
    "until": "2 November 2012"
  },
  {
    "name": "testEstablished",
    "established": true
  },
  {
    "name": "testDependencies",
    "enabled": true,
    "dependencies": [
      "testSimpleEnabled",
      "testSimpleDisabled"
    ]
  },
  {
    "name": "testSimpleEnabled",
    "enabled": true
  },
  {
    "name": "testSimpleDisabled",
    "enabled": false
  }
]
```

**Initializing the library**

```c#
Features.Initialize(x => x.FromSource(new JsonFileConfigurationReader("features.json")));
```

**Checking feature status**

```c#
if (Features.IsEnabled("testSimpleEnabled"))
{
    // Implement feature
}
```


Toggle types
--------------

**Boolean (true/false)**

Feature toggles based on a static binary value - either on or off.

```c#
features.Add(new BooleanToggle("Feature1", true));
```

```json
  [
  {
    "name": "Feature1",
    "enabled": true
  }
  ]
```

**Date Range (true/false, within date range)**

Date Range feature toggles are evaluated on both the binary enabled value and the current date.

```c#
features.Add(new DateRangeToggle("Feature2", true, DateTime.Now.AddDays(5), null));
```

```json
[
  {
    "name": "Feature2",
    "enabled": true,
    "from": "1 November 2012",
    "until": "2 November 2012"
  }
]
```
_From_ and _until_ dates can be any valid date format parseable by _DateTime.Parse_.


**Established features**

Marking a feature toggle as established makes the feature toggle throw a _FeatureEstablishedException_ exception to make sure that it is not queried any more.  

```c#
features.Add(new EstablishedFeatureToggle("establishedFeature"));
```

```json
[
  {
    "name": "establishedFeature",
    "established": true
  }
]
```

**Dependencies**

Features can depend on other features.  For instance, it is sometimes convenient to have a "main" feature, and then sub-features that depend on it.  Dependencies can be specified in configuration as a comma delimited list.

```c#
var mainFeature = new BooleanToggle("mainFeature", true);
var subFeature1 = new BooleanToggle("subFeature1", true);
var subFeature2 = new BooleanToggle("subFeature2", true);

var dependency1 = new DependencyToggle(subFeature1, mainFeature);
var dependency2 = new DependencyToggle(subFeature2, mainFeature);
features.Add(dependency1);
features.Add(dependency2);
```

```json
[
  {
    "name": "mainFeature",
    "enabled": true,
    "dependencies": [
      "subFeature1",
      "subFeature2"
    ]
  },
  {
    "name": "subFeature1",
    "enabled": true
  },
  {
    "name": "subFeature2",
    "enabled": true
  }
]
```

