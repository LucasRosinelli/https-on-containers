FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["src/HttpsOnContainers.WebApiFirst/HttpsOnContainers.WebApiFirst.csproj", "src/HttpsOnContainers.WebApiFirst/"]
RUN dotnet restore "src/HttpsOnContainers.WebApiFirst/HttpsOnContainers.WebApiFirst.csproj"
COPY . .
WORKDIR "/src/src/HttpsOnContainers.WebApiFirst"
RUN dotnet build "HttpsOnContainers.WebApiFirst.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "HttpsOnContainers.WebApiFirst.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "HttpsOnContainers.WebApiFirst.dll"]
