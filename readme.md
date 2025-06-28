# Katasec.DStream.Plugin

**.NET library for building DStream-compatible plugins using gRPC and the HashiCorp go‑plugin protocol.**

Enable seamless integration of C# plugin components with the DStream CLI, leveraging the same Terraform-style plugin framework used by HashiCorp tools.

---

## 🚀 Getting Started

### Prerequisites

- .NET 9.0 SDK or later  
- DStream CLI (Go) installed and available on your PATH  
- `plugin.proto` definitions (included in this package)

### Installation

```bash
dotnet add package Katasec.DStream.Plugin
```

---

## 🛠️ Usage

### Implementing a plugin

```csharp
using static Katasec.DStream.Plugin.Plugin;
using Google.Protobuf.WellKnownTypes;
using Grpc.Core;

public class MyPlugin : PluginBase, IDStreamPlugin
{
    public override Task<GetSchemaResponse> GetSchema(Empty request, ServerCallContext context)
    {
        // Your implementation here
    }

    // Implement other RPC methods...
}
```

### Running the plugin

```csharp
var host = new DStreamPluginHost<MyPlugin>("my-plugin-id", new MyPlugin());
host.Run(); // Or host.RunAsync(CancellationToken)
```

Check the `Examples/` folder for a fully working example.

---

## 📘 Project Structure

```
src/
  Katasec.DStream.Plugin/     ← Main library
    Protos/                   ← Contains plugin.proto
tests/
  Katasec.DStream.Plugin.Tests/ ← Unit & integration tests
```

- **Generated gRPC code** lives in `Katasec.DStream.Plugin.Protos`  
- Core types: `IDStreamPlugin`, `DStreamPluginHost`, `HashiCorpPluginUtils`, along with gRPC service classes

---

## 📚 Documentation & Resources

- Protocol defined in `protos/plugin.proto`  
- Generated types available in `Katasec.DStream.Plugin.Protos`  
- Implements key RPCs: `GetSchema`, `WriteRows`, `Stop`, `HealthCheck`  
- Package includes XML documentation for IntelliSense support

---

## 🛡️ Best Practices

This README follows Microsoft’s NuGet recommendations, including:
- Clear purpose and summary  
- Quick‑start code examples  
- Project layout overview  

To embed this README in your NuGet package, add to your `.csproj`:

```xml
<PropertyGroup>
  <PackageReadmeFile>README.md</PackageReadmeFile>
</PropertyGroup>
<ItemGroup>
  <None Include="README.md" Pack="true" PackagePath="" />
</ItemGroup>
```

---

## ✅ Development

```bash
git clone https://github.com/katasec/Katasec.DStream.Plugin.git
cd Katasec.DStream.Plugin
dotnet build
dotnet test
```

---

## 🤝 Contributing

Contributions, issues, and feature requests are welcome!  
Follow the standard workflow:

1. Fork the repo  
2. Create a feature branch (`git checkout -b feature/foo`)  
3. Commit changes (`git commit -am 'Add foo feature'`)  
4. Push to your branch (`git push origin feature/foo`)  
5. Open a pull request

Refer to [CONTRIBUTING.md](CONTRIBUTING.md) for details.

---

## 📝 License & Support

- **License**: MIT  
- **Support**: Open issues or PRs via GitHub

---

## 🏆 Acknowledgements

Thanks to the creators of the HashiCorp go‑plugin protocol and the DStream CLI for inspiring this library.

---

*(This README is included in the NuGet package per Microsoft’s best practices.)*
