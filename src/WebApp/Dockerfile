FROM microsoft/dotnet:2.1-sdk AS build-env
WORKDIR /app

# Copy necessary files and restore as distinct layer
COPY *.csproj ./
RUN dotnet restore -s https://api.nuget.org/v3/index.json -s https://www.myget.org/F/pitstop/api/v3/index.json

# Install node
ENV NODE_VERSION 8.9.4
ENV NODE_DOWNLOAD_SHA 21fb4690e349f82d708ae766def01d7fec1b085ce1f5ab30d9bda8ee126ca8fc
RUN curl -SL "https://nodejs.org/dist/v${NODE_VERSION}/node-v${NODE_VERSION}-linux-x64.tar.gz" --output nodejs.tar.gz \
    && echo "$NODE_DOWNLOAD_SHA nodejs.tar.gz" | sha256sum -c - \
    && tar -xzf "nodejs.tar.gz" -C /usr/local --strip-components=1 \
    && rm nodejs.tar.gz \
    && ln -s /usr/local/bin/node /usr/local/bin/nodejs

# Copy everything else and build
COPY . ./
RUN npm i npm@latest -g
RUN npm install
RUN npm install gulp -g
RUN gulp default
RUN dotnet publish -c Release -o out

# Build runtime image
FROM microsoft/dotnet:2.1-aspnetcore-runtime
COPY --from=build-env /app/out .

# Expose ports
EXPOSE 7000/tcp
ENV ASPNETCORE_URLS http://*:7000

# Configure environment variables
ENV WEBSITE_SITE_NAME WebApp

# Start
ENTRYPOINT ["dotnet", "WebApp.dll"]
