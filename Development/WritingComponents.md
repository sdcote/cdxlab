# Configuration

Configuration is called immediately after the component is instantiated.

It is recommended that `super.setConfiguration(cfg)` is called to configure all base classes. The following is a cut and paste starting point:

```java
  @Override
  public void setConfiguration(Config cfg) throws ConfigurationException {
    super.setConfiguration(cfg);
      
    final String timestampFieldName = getString(ConfigTag.TIMESTAMP);
    if (StringUtil.isBlank(timestampFieldName)) {
      throw new ConfigurationException("Null, empty or blank argument for " + ConfigTag.TIMESTAMP + " configuration parameter");
    }
  }	
```

When configuring you component is is a good practice to use the `getString(String)` method. THis will look up the string in you configuration then , if not found, look in the context. THis way, it is possible to use the values set in the context when the engine first starts.

## ConfigurationException

Throwing `ConfigurationException` during configuration will halt the engine and no other processing will occur. THis is the normal way to signal a configuration error and to terminate processing.

There are situations when a needed configuration option may net be available until after all the components are initialized. Some components may wait until being opened to throw an error, especially if the configuration value may not be populated in the context and the the time of configuration. In these cases the `open()` methos is where an exception would be thrown.

# Initialization

Components are initialized through the `open()` method call.

A last check of the configuration parameters should be performed here to make sure there are no conflicts with other settings. For example, data may be placed in the context that may invalidate the previous setting. Additionally, context data may be available during component initialization that was not present during configuration.

## Error in Context

To stop processing when initialization fails, place an error message in the transform context and return immediately:

```java
} catch (final Exception e) {
   context.setError("Problems loading field definition '" + field.getName() + "' - " + e.getClass().getSimpleName() + " : " + e.getMessage());
   return;
	}
```

The above will place the transform context in error preventing further processing. All components will be closed and the engine will terminate without entering the main read loop.





