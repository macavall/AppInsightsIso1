# Logging Everything on Dotnet-Isolated Azure Function App on Flex Consumption


## Program.cs
```csharp
using Microsoft.ApplicationInsights.Extensibility;
using Microsoft.ApplicationInsights.WindowsServer.TelemetryChannel;
using Microsoft.Azure.Functions.Worker;
using Microsoft.Azure.Functions.Worker.Builder;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using System.Linq;

internal class Program
{
    private static void Main(string[] args)
    {
        var builder = FunctionsApplication.CreateBuilder(args);

        builder.ConfigureFunctionsWebApplication();

        builder.Services
            .AddApplicationInsightsTelemetryWorkerService()
            .ConfigureFunctionsApplicationInsights();

        // Disable adaptive sampling in Application Insights
        builder.Services.Configure<TelemetryConfiguration>(config =>
        {
            // Find the AdaptiveSamplingTelemetryProcessor and disable sampling by setting percentage to 100%
            var adaptiveSamplingProcessor = config.DefaultTelemetrySink.TelemetryProcessors
                .OfType<AdaptiveSamplingTelemetryProcessor>()
                .FirstOrDefault();

            if (adaptiveSamplingProcessor != null)
            {
                adaptiveSamplingProcessor.MinSamplingPercentage = 100;
            }
        });

        builder.Build().Run();
    }
}
```

## Host.JSON file should look like this ensuring all logging outputs are sent to Application Insights

```Json
{
    "version": "2.0",
    "logging": {
        "fileLoggingMode": "always",
        "console": {
            "isEnabled": "true"
        },
        "logLevel": {
            "default": "Trace",
            "Host.Results": "Trace",
            "Microsoft.Hosting.Lifetime": "Trace",
            "Microsoft": "Trace",
            "Host.Aggregator": "Trace",
            "Function": "Trace",
            "Host.Executor": "Trace"
        },
        "applicationInsights": {
            "samplingSettings": {
                "isEnabled": true,
                "excludedTypes": "Request"
            },
            "enableLiveMetricsFilters": true
        }
    }
}
```