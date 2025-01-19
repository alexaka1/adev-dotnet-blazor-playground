FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/aspnet:9.0-noble-chiseled@sha256:d24db805712b6bc67f4f18a50d3659fc9f8014dde1b36494c98f0159f6d1542f AS base
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

FROM --platform=$BUILDPLATFORM node:22-alpine@sha256:b09589de497084c656bb7f05078860793d1dffc019bfd1d733cdc1404bff3d1e AS tailwindcss
RUN corepack enable
WORKDIR /src
COPY ["package.json", "pnpm-lock.yaml", "./"]
RUN pnpm install --frozen-lockfile
ENV NODE_ENV=production
COPY . .
RUN pnpm run tailwind:build:minify

FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:9.0@sha256:26d070b5f4f32fbf3223f24cd27c5aef94f9074e54270eca2239b833563f6221 AS build
ARG BUILD_CONFIGURATION=Release
ARG TARGETARCH
RUN apt update; \
    apt install -y python3; \
    dotnet workload install wasm-tools;
WORKDIR /src
COPY ["Directory.Packages.props", "Directory.Build.props", "nuget.config", "./"]
COPY ["src/Web/Web.csproj", "src/Web/"]
COPY ["src/Web.Client/Web.Client.csproj", "src/Web.Client/"]
COPY ["src/Shared/Shared.csproj", "src/Shared/"]
RUN dotnet restore "src/Web/Web.csproj" -a $TARGETARCH
COPY . .
COPY --from=tailwindcss /src/src/Web/wwwroot/styles/app.css ./src/Web/wwwroot/styles/app.css
WORKDIR "/src/src/Web"
RUN dotnet build "Web.csproj" -c $BUILD_CONFIGURATION --no-restore -o /app/build -a $TARGETARCH

FROM build AS publish
ARG TARGETARCH
ARG BUILD_CONFIGURATION=Release
RUN dotnet publish "Web.csproj" -c $BUILD_CONFIGURATION --no-restore -o /app/publish -a $TARGETARCH

FROM base AS final
COPY --from=alexaka1/distroless-dotnet-healthchecks:latest / /healthchecks
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "Web.dll"]
