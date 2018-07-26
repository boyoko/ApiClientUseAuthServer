FROM microsoft/dotnet:2.1-aspnetcore-runtime AS base
WORKDIR /app
EXPOSE 50627
EXPOSE 44398

FROM microsoft/dotnet:2.1-sdk AS build
WORKDIR /src
COPY ApiClientUseAuthServer/ApiClientUseAuthServer.csproj ApiClientUseAuthServer/
RUN dotnet restore ApiClientUseAuthServer/ApiClientUseAuthServer.csproj
COPY . .
WORKDIR /src/ApiClientUseAuthServer
RUN dotnet build ApiClientUseAuthServer.csproj -c Release -o /app

FROM build AS publish
RUN dotnet publish ApiClientUseAuthServer.csproj -c Release -o /app

FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "ApiClientUseAuthServer.dll"]
