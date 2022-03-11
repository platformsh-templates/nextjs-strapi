<br />
<p align="left">
    <a href="https://platform.sh">
        <img src="https://platform.sh/logos/redesign/Platformsh_logo_black.svg" width="150px">
    </a>
</p>
<br /><br />
<p align="center">
    <a href="https://github.com/strapi/foodadvisor">
        <img src="foodadvisor.png">
    </a>
</p>
<h1 align="center">Deploying Foodadvisor (Next.js + Strapi) on Platform.sh</h1>

<p align="center">
    <strong>Contribute to the Platform.sh knowledge base, or check out our resources</strong>
    <br />
    <br />
    <a href="https://community.platform.sh"><strong>Join our community</strong></a>&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp
    <a href="https://docs.platform.sh"><strong>Documentation</strong></a>&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp
    <a href="https://platform.sh/blog"><strong>Blog</strong></a>&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp
    <a href="https://github.com/platformsh-templates/nextjs-strapi/issues"><strong>Report a bug</strong></a>&nbsp&nbsp&nbsp&nbsp&nbsp&nbsp
    <a href="https://github.com/platformsh-templates/nextjs-strapi/issues"><strong>Request a feature</strong></a>
    <br /><br />
</p>

<p align="center">
    <a href="https://github.com/platformsh-templates/metabase/issues">
        <img src="https://img.shields.io/github/issues/platformsh-templates/nextjs-strapi.svg?style=flat-square&labelColor=f4f2f3&color=ffd9d9&label=Issues" alt="Open issues" />
    </a>&nbsp&nbsp
    <a href="https://github.com/platformsh-templates/pulls">
        <img src="https://img.shields.io/github/issues-pr/platformsh-templates/nextjs-strapi.svg?style=flat-square&labelColor=f4f2f3&color=ffd9d9&label=Pull%20requests" alt="Open PRs" />
    </a>&nbsp&nbsp
    <a href="https://github.com/platformsh-templates/nextjs-strapi/blob/master/LICENSE">
        <img src="https://img.shields.io/static/v1?label=License&message=MIT&style=flat-square&labelColor=f4f2f3&color=ffd9d9" alt="License" />
    </a>&nbsp&nbsp
    <br /><br /><br />
    <a href="https://console.platform.sh/projects/create-project?template=https://raw.githubusercontent.com/platformsh/template-builder/master/templates/nextjs-strapi/.platform.template.yaml&utm_content=nextjs-strapi&utm_source=github&utm_medium=button&utm_campaign=deploy_on_platform">
        <img src="https://platform.sh/images/deploy/lg-blue.svg" alt="Deploy on Platform.sh" width="175px" />
    </a>
</p>
</p>

<hr>
<br />
<h2 align="center"><strong>Contents</strong></h2>


- [About this project](#about-this-project)
  - [Features](#features) 
- [Getting started](#-getting-started-)
  - [Deploying](#deploying)
  - [Post-install](#post-install)
- [Customizations](#customizations)
  - [Configuration](#configuration)
  - [Builds and deploys](#builds-and-deploys)
  - [Upstream modifications](#upstream-modifications)
- [About Platform.sh](#about-platformsh)
- [Usage](#usage)
  - [Logs](#logs)
  - [Local development](#local-development)
  - [Updating](#updating)
  <!-- - [Customization](#customization)
  - [Performance](#performance) -->
- [Migrating](#migrating)
- [License](#license)
- [Contact](#contact)
- [Resources](#resources)
- [Contributors](#contributors)

<hr>

## About this project

Lorem ipsum dolor sit amet, consectetur adipiscing elit. Nulla ut efficitur enim, ac congue ex. Interdum et malesuada fames ac ante ipsum primis in faucibus. Donec at nisl imperdiet, imperdiet orci nec, facilisis lacus. Nam euismod fringilla nisl, in pretium risus facilisis id. Proin ultricies consequat nunc at condimentum. Nullam rutrum, lectus quis hendrerit congue, nunc lectus tempus ligula, vel blandit diam metus et arcu. Curabitur nulla nisi, eleifend at justo ornare, euismod viverra dui. 

Nulla luctus elit volutpat, lacinia arcu quis, blandit sem. Proin malesuada risus quis quam scelerisque, at faucibus turpis maximus. Ut id leo odio. Pellentesque lobortis eget quam eget imperdiet.

## Features

- Strapi v4
- Node.js 16
- MySQL 8
- Automatic TLS certificates
- Multi-app configuration
- Delayed SSG build (post deploy hook)
- yarn-based build

## Post-install

This demo repository is set up to deploy both front and backend application containers to the production environment, and to initialize the database with files and data from the original Foodadvisor demo provided by Strapi. 

To login to the Strapi Admin UI, you can use the credentials `admin@example.com/Admin1234` to start, after which you are free to update them at `https://api.GENERATED_URL/admin/me`.

## Customizations 

## Local development

In all cases, it's important to develop on an isolated environment - do not open SSH tunnels to your production environment when developing locally. Each of the options below assume the following starting point:

```bash
platform get PROJECT_ID
cd project-name
platform environment:branch updates
```

> **Note:**
>
> For many of the steps below, you may need to include the CLI flags `-p PROJECT_ID` and `-e ENVIRONMENT_ID` if you are not in the project directory or if the environment is associated with an existing pull request.

### Running the Strapi backend

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

### Running the Next.js frontend

You have two options when running Next.js locally. You can connect to a Strapi instance on an active Platform.sh environment, or run Strapi locally in parallel and connect to that.

1. **Option 1:** Connecting to Strapi on a Platform.sh environment:

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
    printf "NEXT_PUBLIC_API_URL="${BACKEND_URL:8:${#BACKEND_URL}-9}"\nPREVIEW_SECRET=$PREVIEW_SECRET\n" > .env.local

    # Build and run the Next.js server.
    yarn --frozen-lockfile
    yarn dev
    ```

2. **Option 2:** Connecting to a locally running Strapi development server

    This demo assumes a locally running Strapi instance by default, so once you have followed the [steps above](#strapi) you will be able to start the Next.js development server normally.

    ```bash
    # Build and run the Next.js server.
    cd client
    yarn --frozen-lockfile
    yarn dev
    ```

    Next.js will be served from `localhost:3000` pulling data from a local Strapi instance running at `localhost:1337`.


## License

This template uses the [Foodadvisor demo repository]() provided by Strapi.io as its base, which is licensed under the [MIT License](https://github.com/strapi/foodadvisor/blob/master/LICENSE).

## Contact

This template is maintained primarily by the Platform.sh Developer Relations team, and they will be notified of all issues and pull requests you open here.

- **Community:** Share your question with the community, or see if it's already been asked on our [Community site](https://community.platform.sh).
- **Slack:** If you haven't done so already, you can join Platform.sh's [public Slack](https://chat.platform.sh/) channels and ping the `@devrel_team` with any questions.

## Resources

- [Strapi.io](https://strapi.io)
- [Strapi Documentation](https://docs.strapi.io/developer-docs/latest/getting-started/introduction.html)
- [Strapi Foodadvisor Repository](https://github.com/strapi/foodadvisor)
- [Node.js on Platform.sh](https://docs.platform.sh/languages/nodejs.html)

## Contributing

<h3 align="center">Help us keep top-notch templates!</h3>

Every one of our templates is open source, and they're important resources for users trying to deploy to Platform.sh for the first time or better understand the platform. They act as getting started guides, but also contain a number of helpful tips and best practices when working with certain languages and frameworks. 

See something that's wrong with this template that needs to be fixed? Something in the documentation unclear or missing? Let us know!

<h4 align="center"><strong>How to contribute</strong></h4>
<br />
<p align="center">
    <a href="#"><strong>Report a bug</strong></a><br />
    <a href="#"><strong>Submit a feature request</strong></a><br />
    <a href="#"><strong>Open a pull request</strong></a><br />
</p>
<br />
<h4 align="center"><strong>Need help?</strong></h4>
<br />
<p align="center">
    <a href="#"><strong>Ask the Platform.sh Community</strong></a><br />
    <a href="#"><strong>Join us on Slack</strong></a><br />
</p>
<br /><br />
<h3 align="center"><strong>Thanks to all of our amazing contributors!</strong></h3>

<br/>

![GitHub Contributors Image](https://contrib.rocks/image?repo=platformsh-templates/nextjs-strapi)

<br />