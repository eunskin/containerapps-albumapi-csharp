FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build-env
WORKDIR /app

# Copy csproj and restore as distinct layers
COPY *.csproj ./
RUN dotnet restore

# curl in build image for later runtime image to copy from
# please see https://github.com/DataDog/dd-trace-dotnet/releases for the build that suits you
RUN curl -L -s https://github.com/DataDog/dd-trace-dotnet/releases/download/v2.24.1/datadog-dotnet-apm_2.24.1_amd64.deb --output datadog-dotnet-apm.deb

# Copy everything else and build
COPY . .
RUN dotnet publish -c Release -o out


### Build runtime image
FROM mcr.microsoft.com/dotnet/aspnet:6.0

# copy the Datadog `serverless-init` into your Docker image
COPY --from=datadog/serverless-init /datadog-init /app/datadog-init

# optionally add Datadog tags or later set them with `az containerapp create --env-vars "DD_API_KEY=$DD_API_KEY"`
ENV DD_SERVICE=datadog-demo-run-dotnet
ENV DD_ENV=datadog-demo
ENV DD_VERSION=1

ENV DD_TRACE_ENABLED=true
## choose the right site https://docs.datadoghq.com/getting_started/site/
ENV DD_SITE=datadoghq.com

# enable extension logs to show up in the Azure Log Stream's Console Log
ENV DD_LOGS_ENABLED=true
ENV DD_LOG_LEVEL=DEBUG

## install tracer for the whole machine
COPY --from=build-env /app/datadog-dotnet-apm.deb .
RUN dpkg -i ./datadog-dotnet-apm.deb

## Run the createLogPath script on Linux to ensure the automatic instrumentation logs are generated without permission issues
# Step 19/29 : RUN /opt/datadog/createLogPath.sh
#  ---> Running in dac534730108
# + mkdir -p /var/log/datadog/dotnet
# + chmod a+rwx /var/log/datadog/dotnet
RUN /opt/datadog/createLogPath.sh

# use env vars to enable tracer
ENV CORECLR_ENABLE_PROFILING=1
ENV CORECLR_PROFILER={846F5F1C-F9AE-4B07-969E-05C26BC060D8}
ENV CORECLR_PROFILER_PATH=/opt/datadog/Datadog.Trace.ClrProfiler.Native.so
ENV DD_DOTNET_TRACER_HOME=/opt/datadog

# change the entrypoint to wrap your application into the Datadog serverless-init process
ENTRYPOINT ["/app/datadog-init"]

WORKDIR /app
COPY --from=build-env /app/out .
EXPOSE 3500
ENV ASPNETCORE_URLS=http://+:3500

# execute your binary application wrapped in the entrypoint. Adapt this line to your needs
CMD ["dotnet", "albumapi_csharp.dll"]

#FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build-env
#WORKDIR /app

# Copy csproj and restore as distinct layers
#COPY *.csproj ./
#RUN dotnet restore

# Copy everything else and build
#COPY . .
#RUN dotnet publish -c Release -o out

# Build runtime image
#FROM mcr.microsoft.com/dotnet/aspnet:6.0
#WORKDIR /app
#COPY --from=build-env /app/out .
#EXPOSE 3500
#ENV ASPNETCORE_URLS=http://+:3500
#ENTRYPOINT ["dotnet", "albumapi_csharp.dll"]
