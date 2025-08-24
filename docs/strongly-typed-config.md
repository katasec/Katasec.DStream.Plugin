# Strongly Typed Plugin Configuration

This document explains the implementation of strongly typed configuration for dstream .NET plugins.

## Overview

The dstream plugin framework now supports strongly typed configuration through a generic plugin interface `IDStreamPlugin<TConfig>`. This allows plugin developers to define a strongly typed configuration class that matches their HCL configuration structure, providing compile-time safety, better IDE support, and cleaner code.

## Components

### 1. Generic Plugin Interface

The `IDStreamPlugin<TConfig>` interface extends the non-generic `IDStreamPlugin` interface:

```csharp
public interface IDStreamPlugin<TConfig> : IDStreamPlugin
{
    Task ProcessAsync(IInput input, IOutput output, TConfig config, CancellationToken cancellationToken);
}
```

This interface allows plugins to receive a strongly typed configuration object instead of a dictionary.

### 2. Configuration Deserialization

The framework includes utilities in `ConfigurationUtils.cs` to convert Protobuf `Struct` and `Value` objects into strongly typed C# configuration objects:

- JSON serialization/deserialization is used under the hood
- Support for nested objects, arrays, and primitive types
- Detailed logging for debugging deserialization issues

### 3. Generic Plugin Host

The `DStreamPluginHost<TPlugin, TConfig>` class supports plugins implementing the generic interface:

- Deserializes the incoming Protobuf configuration into the strongly typed config
- Maintains compatibility with the HashiCorp go-plugin protocol
- Handles logging and server lifecycle

## Usage

### 1. Define a Configuration Class

Create a class that matches your HCL configuration structure:

```csharp
public class MyPluginConfig
{
    public int Interval { get; set; } = 5000;
    public string ConnectionString { get; set; } = "";
    public bool EnableLogging { get; set; } = true;
    // Add other configuration properties as needed
}
```

### 2. Implement the Generic Interface

Make your plugin implement `IDStreamPlugin<TConfig>`:

```csharp
public class MyPlugin : IDStreamPlugin<MyPluginConfig>
{
    public string ModuleName => "my-plugin";
    
    public IEnumerable<FieldSchema> GetSchemaFields()
    {
        yield return new FieldSchema
        {
            Name = "interval",
            Type = "number",
            Description = "Interval between operations in milliseconds"
        };
        // Add other schema fields as needed
    }
    
    public async Task ProcessAsync(IInput input, IOutput output, MyPluginConfig config, CancellationToken cancellationToken)
    {
        // Use the strongly typed config directly
        logger.Info($"Interval: {config.Interval}ms");
        logger.Info($"Connection String: {config.ConnectionString}");
        logger.Info($"Enable Logging: {config.EnableLogging}");
        
        // Plugin implementation...
    }
}
```

### 3. Create a Program Entry Point

Use the generic plugin host in your program entry point:

```csharp
public static async Task Main(string[] args)
{
    // Create and run the generic plugin host with strongly-typed configuration
    var host = new DStreamPluginHost<MyPlugin, MyPluginConfig>();
    await host.RunAsync(args);
}
```

### 4. Configure in HCL

Define your plugin configuration in HCL:

```hcl
task "my-plugin" {
  type = "plugin"
  plugin_path = "path/to/plugin"
  
  // Global configuration for the plugin with strongly-typed properties
  config {
    interval = 3000
    connectionString = "Server=myServerAddress;Database=myDataBase;User Id=myUsername;Password=myPassword;"
    enableLogging = true
  }
  
  // Input and output configuration...
}
```

## Benefits

1. **Compile-time Safety**: Configuration errors are caught at compile time rather than runtime
2. **Better IDE Support**: IntelliSense and code completion for configuration properties
3. **Cleaner Code**: No need for manual dictionary-to-object conversion inside plugins
4. **Self-documenting**: Configuration classes serve as documentation for required settings
5. **Maintainability**: Easier to refactor and extend configuration

## Backward Compatibility

The framework maintains backward compatibility with the original non-generic interface and plugin host, allowing for gradual migration to strongly typed configurations.

## Testing

To test a plugin with strongly typed configuration:

1. Build the plugin: `dotnet publish -c Release`
2. Run with dstream: `dstream run -c your-config.hcl`
