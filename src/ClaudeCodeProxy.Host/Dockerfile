FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS base
USER root
ENV DOTNET_TieredCompilation=1
ENV DOTNET_TC_QuickJit=1
ENV DOTNET_TC_QuickJitForLoops=1
ENV DOTNET_TC_CallCountThreshold=30
ENV DOTNET_TC_OnStackReplacement=1
ENV DOTNET_JitStress=0
ENV DOTNET_JitMinOpts=0
ENV DOTNET_ReadyToRun=0
ENV TZ=Asia/Shanghai
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
RUN echo "Asia/Shanghai" > /etc/timezone
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
ARG BUILD_CONFIGURATION=Release
ENV DOTNET_TieredCompilation=1
ENV DOTNET_TC_QuickJit=1
ENV DOTNET_TC_QuickJitForLoops=1
ENV DOTNET_TC_CallCountThreshold=30
ENV DOTNET_TC_OnStackReplacement=1
ENV DOTNET_JitStress=0
ENV DOTNET_JitMinOpts=0
ENV DOTNET_ReadyToRun=0
WORKDIR /src
COPY ["src/ClaudeCodeProxy.Host/ClaudeCodeProxy.Host.csproj", "src/ClaudeCodeProxy.Host/"]
COPY ["src/ClaudeCodeProxy.Abstraction/ClaudeCodeProxy.Abstraction.csproj", "src/ClaudeCodeProxy.Abstraction/"]
COPY ["src/ClaudeCodeProxy.Domain/ClaudeCodeProxy.Domain.csproj", "src/ClaudeCodeProxy.Domain/"]
COPY ["src/Provide/ClaudeCodeProxy.EntityFrameworkCore.Sqlite/ClaudeCodeProxy.EntityFrameworkCore.Sqlite.csproj", "src/Provide/ClaudeCodeProxy.EntityFrameworkCore.Sqlite/"]
COPY ["src/ClaudeCodeProxy.Core/ClaudeCodeProxy.Core.csproj", "src/ClaudeCodeProxy.Core/"]
RUN dotnet restore "src/ClaudeCodeProxy.Host/ClaudeCodeProxy.Host.csproj"
COPY . .
WORKDIR "/src/src/ClaudeCodeProxy.Host"
RUN dotnet build "./ClaudeCodeProxy.Host.csproj" -c $BUILD_CONFIGURATION -o /app/build

FROM build AS publish
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "./ClaudeCodeProxy.Host.csproj" -c $BUILD_CONFIGURATION -o /app/publish /p:UseAppHost=false \
    -r linux-arm64 \
    --self-contained false \
    /p:PublishReadyToRun=false \
    /p:PublishSingleFile=false \
    /p:PublishReadyToRun=true \
    /p:TrimMode=link \
    /p:InvariantGlobalization=true \
    /p:DebugType=None \
    /p:DebugSymbols=false
    

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
COPY ./web/dist ./wwwroot
ENTRYPOINT ["dotnet", "ClaudeCodeProxy.Host.dll"]
