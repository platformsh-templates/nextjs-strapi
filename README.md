# Next.js + Strapi demo for Platform.sh

<p align="center">
<a href="https://console.platform.sh/projects/create-project?template=https://raw.githubusercontent.com/platformsh/template-builder/master/templates/nextjs-strapi/.platform.template.yaml&utm_content=nextjs-strapi&utm_source=github&utm_medium=button&utm_campaign=deploy_on_platform">
    <img src="https://platform.sh/images/deploy/lg-blue.svg" alt="Deploy on Platform.sh" width="180px" />
</a>
</p>

Lorem ipsum

## Features

## Post-install

## Customizations 

## Local development

In all cases, it's important to develop on an isolated environment, and each of the options below assume the following starting point:

```bash
platform get PROJECT_ID
cd project-name
platform environment:branch updates
```

> **Note:**
>
> You will need to include the CLI flags `-p PROJECT_ID` and `-e ENVIRONMENT_ID` if you are not in the project directory or if the environment is associated with an existing pull request.

### Strapi

```bash
# Open a SSH tunnel to the environment's database.
platform tunnel:open -A strapi

# Mock environment variable that contains service credentials. 
export PLATFORM_RELATIONSHIPS="$(platform tunnel:info -A strapi --encode)"

# Pull public/uploads files from the environment.
cd api
platform mount:download -A strapi -m public/uploads --target public/uploads -y

# Build Strapi and start the server.
yarn --frozen-lockfile
yarn develop
```

Strapi will then serve on `localhost:1337` using a live service on the isolated Platform.sh environment.

### Next.js

1. Pulling data from Strapi on a Platform.sh environment:

    > **Requirements:**
    >
    > In order to retrieve the backend url within live environment from environment variables, this demo uses [jq](https://stedolan.github.io/jq/manual/v1.6/) - the JSON filtering command line tool. jq comes pre-installed on all Platform.sh runtime containers, and in order to replicate the behavior described below and build Next.js locally, you will need to also have it [installed on your system](https://stedolan.github.io/jq/download/). 

    ```bash
    cd client

    # Get the live backend Strapi url (note the 'id' attribute defined in .platform/routes.yaml).
    BACKEND_URL=$(platform ssh 'echo $PLATFORM_ROUTES | base64 --decode' -A nextjs -q | jq -r 'to_entries[] | select (.value.id == "api") | .key')

    # Get the preview secret.
    PREVIEW_SECRET=$(platform ssh 'echo $PLATFORM_PROJECT-$PLATFORM_BRANCH' -A nextjs -q)

    # Output to .env.development.
    printf "NEXT_PUBLIC_API_URL="${BACKEND_URL:8:${#BACKEND_URL}-9}"\nPREVIEW_SECRET=$PREVIEW_SECRET\n" > .env.development

    # Build and run the Next.js server.
    cd client
    yarn --frozen-lockfile
    yarn dev
    ```

    my notes:

    ```bash
    cd client

    # Get the live backend Strapi url (note the 'id' attribute defined in .platform/routes.yaml).
    BACKEND_URL=$(platform ssh 'echo $PLATFORM_ROUTES | base64 --decode' -p b3sqwzxrtdozm -e pr-3 -A nextjs -q | jq -r 'to_entries[] | select (.value.id == "api") | .key')

    # Get the preview secret.
    PREVIEW_SECRET=$(platform ssh 'echo $PLATFORM_PROJECT-$PLATFORM_BRANCH' -p b3sqwzxrtdozm -e pr-3 -A nextjs -q)

    # Output to .env.development.
    printf "NEXT_PUBLIC_API_URL="${BACKEND_URL:8:${#BACKEND_URL}-9}"\nPREVIEW_SECRET=$PREVIEW_SECRET\n" > .env.development

    # Build and run the Next.js server.
    cd client
    yarn --frozen-lockfile
    yarn dev
    ```

2. Pulling data from Strapi running locally:

    This demo assumes a locally running Strapi instance by default, so once you have followed the [steps above](#strapi) you will be able to start the Next.js development server normally.

    ```bash
    # Build and run the Next.js server.
    cd client
    yarn --frozen-lockfile
    yarn dev
    ```

    Next.js will be served from `localhost:3000` pulling data from a local Strapi instance running at `localhost:1337`.


