
# Welcome

I have been involved with building software applications, services, data pipelines and websites for more than a decade.

On introspection, it was clear that I need to learn something new, probably every few months.

I wanted to create this simple website, which serves as a cheat sheet, with a concise description and a simple example (wherever possible) of anything new that I learn.

## Table of Contents  
[Foundation DB cheatsheet](#foundation-db-cheatsheet)

[Dropwizard](#dropwizard-quickstart)

___

# Foundation DB Basic Operations

## How key/value is stored

The key can be created from a tuple. 

For example, if you want to store the number of times a page is visited in a day, you can have the following key value

__key - yyyy/mm/dd/pageid__

__value - count__

```

// assuming variables are initialized elsewhere
db.run((Transaction tr) -> {
  byte[] key = Tuple.from(yyyy, mm, dd, pageId).pack();
  byte[] value = Tuple.from(count).pack();
  tr.set(key, value);
  return null;
});
```

## Advantage of subspace

When writing records, there may be a scenario when you need to read ALL the keys.

Always writing a record with a pre-defined subspace helps reading all the records, with a readRange operation.

```
Subspace subspace = new Subspace(Tuple.from("tsdb"));

// assuming variables are initialized elsewhere
db.run((Transaction tr) -> {
  Tuple tuple = Tuple.from(yyyy, mm, dd, hour, minute);
  byte[] value = Tuple.from(count).pack();
  tr.set(subspace.pack(tuple), value);
  return null;
});
```

## Read a value

To read a particular value, you need to know the complete key.
The result is a Tuple. To get a particular field in the tuple, you need to know the index within the tuple.

```
db.run((Transaction tr) -> {
  
  byte[] key = Tuple.from("dailyPageCount", yyyy, mm, dd, pageId).pack();
  byte[] result = tr.get(key).join();
  return Tuple.fromBytes(result).getLong(0);
});

```

## Read range

To read ALL records, you can use the prefix.
To read range, you can use upto a certain precision.

```
db.run((Transaction tr) -> {
  byte[] start = Tuple.from("dailyPageCount", yyyy).pack();
  Range range = Range.startsWith(start);
  
  for(KeyValue kv : tr.getRange(range)) {
      // process the results
  }
  return null;
});

```

## References
- Foundation DB docs : https://apple.github.io/foundationdb/
- Foundation DB record layer : https://github.com/FoundationDB/fdb-record-layer

___

# Dropwizard Quickstart

Dropwizard pulls together stable, mature libraries from the Java ecosystem into a simple, light-weight package that lets you focus on getting things done.

Dropwizard has out-of-the-box support for sophisticated configuration, application metrics, logging, operational tools, and much more, allowing you and your team to ship a production-quality web service in the shortest time possible.

For a basic application, there are 5 main components -

- The Application class : The entry point for the application and responsible for bootstrapping configuration, resource and health checks
- The Configuration class : Dynamic inputs like environment variables can be defined here
- The Resource class : The RESTful services
- The Health Check : Checks whether the inputs to the application through the config, and behaving correctly
- Model classes : POJOs representing the models/entities

In this example, I will create a basic application which will expose a REST endpoint.
I will be using gradle for dependency management and packaging.

The application can be run by using the gradle task 'run'

```
public class SampleApp extends Application<SampleConfig> {

    private static String APP_NAME = "sample-app-name";

    public static void main(String[] args) throws Exception {
        new SampleApp().run(args);
    }

    @Override
    public String getName() {
        return APP_NAME;
    }

    @Override
    public void initialize(Bootstrap<SampleConfig> bootstrap) {
        super.initialize(bootstrap);
    }

    @Override
    public void run(SampleConfig configuration, Environment environment) throws Exception {
        final SampleResource resource = new SampleResource(
                configuration.getVersion());

        final SampleHealthCheck healthCheck =
                new SampleHealthCheck(configuration.getVersion());
        environment.healthChecks().register("healthCheck", healthCheck);
        environment.jersey().register(resource);
    }
}
```

The entire sample application is present at : https://github.com/shankarganesh1234/dropwizard-gradle-sample
___


