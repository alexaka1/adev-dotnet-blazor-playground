FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/aspnet:9.0-noble-chiseled@sha256:d24db805712b6bc67f4f18a50d3659fc9f8014dde1b36494c98f0159f6d1542f AS base
WORKDIR /app
EXPOSE 8080
EXPOSE 8081

FROM --platform=$BUILDPLATFORM node:22-alpine@sha256:e2b39f7b64281324929257d0f8004fb6cb4bf0fdfb9aa8cedb235a766aec31da AS tailwindcss
RUN corepack enable
WORKDIR /src
COPY ["package.json", "pnpm-lock.yaml", "./"]
RUN pnpm install --frozen-lockfile
ENV NODE_ENV=production
COPY . .
RUN pnpm run tailwind:build:minify

FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:9.0@sha256:6894a71619e08b47ef9df7ff1f436b21d21db160e5d864e180c294a53d7a12f2 AS build
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
