---
id: adding-custom-environment-variables
title: Adding Custom Environment Variables
sidebar_label: Environment Variables
---

> Note: this feature is available with `react-scripts@0.2.3` and higher.

Your project can consume variables declared in your environment as if they were declared locally in your JS files. By default you will have `NODE_ENV` and `PUBLIC_URL` defined for you, and your environment variables that start with `REACT_APP_`.

## Key information for using environment variables

Environment variables are embedded into the code during build, which has a number of consequences
- All environment variables are visible in the client web page and **cannot** store secret information
- All `REACT_APP_` variables must be defined in **all** build versions to prevent javascript errors being created by webpage Javascript. Build versions would include dev/start, test, build and any configurations specific to your app
- By default **`.env` file is checked in** by Create React App. It is not included in the default `.gitignore`. This runs contrary to the common practice NOT to check in `.env`
- The build flow will optimize the build bundle size by removing code that would never be run, based on the values of `NODE_ENV` and any `REACT_APP_` variables. This is analogous to `#IFDEF` in the C preprocessor and is often called Tree shaking. The application Javascript is not executed, but it is parsed to find these places where code can be eliminated
- Custom variables can be referenced in code using `process.env.REACT_APP_`. `NODE_ENV` can also be referenced as `process.env.NODE_ENV`, and the value of `NODE_ENV` cannot be changed
- Custom variables can also be referenced in HTML, the syntax is different. `PUBLIC_URL` is used in the default Create React App setup


## Example

The following is included in the `.env` file located in the project root directory
``` bash
REACT_APP_USER_VERSION=10
```
This will ensure that all of the scripts defined in `package.json` will read the variable

The following code could be in a React component
``` jsx
let userMessage="";
if ( process.env.REACT_APP_USER_VERSION < 10) {
  if (process.env.NODE_ENV === "debug") {
    // message seen only by developer
    console.log("WARNING: check that correct user version has been declared")
  }
  userMessage = "Warning: old User version has been defined";
}
// userMessage is later displayed to the user in an information component
```

## Security and secrets

There is often a need to protect information, such as encryption keys or API keys. These are often called secrets. Unfortunately, there is **NO** way to
hide these in web page Javascript. The only way to keep the information secure is for it to be only referenced in the server, and for it not to be checked into any
publicly accessible repository.

### Example

It is likely that a web page might want to use an email service to send email to the maintainer (uncaught JS error event), the owner (contact message) or to an email list.

Most email services require some form of password or API key to access. The API key is issued specifically to the web page owner with intent of linking to a payment account or to limit usage to prevent Denial of Service attacks. If the API is a RESTful API, then the web page can issue a web request directly to the API URL. A URL might look like
```
www.amazingEmail.com/api/v5/user=ReactMan&operation=sendMail&API_KEY="ajsdh12jhg45b39"&message=hello
```
It is possible that a web crawler could extract the API_KEY and use it for its own mail sending, leaving the web site owner to pay the bill or be accused of a denial of service attack

Instead, a cloud function(lambda) service is created, that only adds the API_KEY and accesses the email server. When the function is deployed to the (cloud) service, a vendor specific mechanism is used to create API_KEY. This value cannot be read by programs other than the function and is not checked into source control 

A request from the web page would then use the following URL
```
www.cloudService.ReactManFunctionABC.com/operation=sendMail&message=hello
```
In response, the service would check that the request originated from the owner's web page and then issue the request to the email server with appropriate header
settings and adding the API_KEY.  
```
www.amazingEmail.com/api/v5/user=ReactMan&operation=sendMail&API_KEY="ajsdh12jhg45b39"&message=hello
```
When the email service returns its reply to the function, the reply is then forwarded back to the original web page

### Access secrets through a server

The previous example uses a cloud function created specifically to hide the secret. This makes sense for static web pages, but if the app already has its own server for database storage for example, then the mail request could be run through the same server

### Use of secrets in `.env` file

In server code, it is standard practice for secrets to be kept in a `.env` file that is not checked into source control (git). For a Node based server, the `dotenv` package is used to read the `.env` file, and variables are accessed as `process.env.VAR_NAME`. `react-scripts` use this `dotenv` mechanism to read in the `.env` file during the build flow

If the server code for your app is stored in the same repo as the React code, then there is a conflict between keeping secrets in a `.env` file and using `REACT_APP_` variables defined in `.env`. This must be resolved on a project basis. Possible solutions are
- set `REACT_APP_` variables as part of the script definition in `package.json`. The values must be set in every script
- separate code for server and React into different subdirectories, adding `server/.env` to the `.gitignore` file. Care must be taken to start the server from the correct directory
- separate server and client code into different Repos

## Adding Temporary Environment Variables In Your Shell

Defining environment variables can vary between OSes. It’s also important to know that this manner is temporary for the life of the shell session. Don't forget to
define the value for each possible run (all scripts in `package.json` for example). The following sections give different methods to create temporary environment variables

### Operating system independent

Setting environment variables is a common problem, and the `cross-env` npm package is a robust and effective method to solve the issue

Documentation can be found at [`cross-env`](https://github.com/kentcdodds/cross-env/blob/master/README.md) on github

Example use in `package.json` after adding `cross-env` as a development dependency using `npm` or `yarn`
```
"scripts": {
  "start": "npx cross-env REACT_APP_USER_VERSION=10 react-scripts start",
  "build: "npx cross-env REACT_APP_USER_VERSION=10 react-scripts build",
  "test": "eslint --quiet && npx cross-env REACT_APP_USER_VERSION=5 react-scripts test"
},
```

Don't forget to define the value for each `scripts` entry


### Windows (cmd.exe)

```cmd
set "REACT_APP_NOT_SECRET_CODE=abcdef" && npm start
```

(Note: Quotes around the variable assignment are required to avoid a trailing whitespace.)

### Windows (Powershell)

```Powershell
($env:REACT_APP_NOT_SECRET_CODE = "abcdef") -and (npm start)
```

### Linux, macOS (Bash)

```sh
REACT_APP_NOT_SECRET_CODE=abcdef npm start
```

## Adding Environment Variables In `.env` file

> Note: this feature is available with `react-scripts@0.5.0` and higher.

To define permanent environment variables, create a file called `.env` in the root of your project:

```
REACT_APP_NOT_SECRET_CODE=abcdef
```

> Note: You must create custom environment variables beginning with `REACT_APP_`. Any other variables except `NODE_ENV` will be ignored to avoid [accidentally exposing a private key on the machine that could have the same name](https://github.com/facebook/create-react-app/issues/865#issuecomment-252199527). Changing any environment variables will require you to restart the development server if it is running.

> Note: You need to restart the development server after changing `.env` files.

`.env` files **should be** checked into source control (with the exclusion of `.env*.local`).

### What other `.env` files can be used?

> Note: this feature is **available with `react-scripts@1.0.0` and higher**.

- `.env`: Default.
- `.env.local`: Local overrides. **This file is loaded for all environments except test.**
- `.env.development`, `.env.test`, `.env.production`: Environment-specific settings.
- `.env.development.local`, `.env.test.local`, `.env.production.local`: Local overrides of environment-specific settings.

Files on the left have more priority than files on the right:

- `npm start`: `.env.development.local`, `.env.local`, `.env.development`, `.env`
- `npm run build`: `.env.production.local`, `.env.local`, `.env.production`, `.env`
- `npm test`: `.env.test.local`, `.env.test`, `.env` (note `.env.local` is missing)

These variables will act as the defaults if the machine does not explicitly set them.

Please refer to the [dotenv documentation](https://github.com/motdotla/dotenv) for more details.

> Note: If you are defining environment variables for development, your CI and/or hosting platform will most likely need
> these defined as well. Consult their documentation how to do this. For example, see the documentation for [Travis CI](https://docs.travis-ci.com/user/environment-variables/) or [Heroku](https://devcenter.heroku.com/articles/config-vars).

### Expanding Environment Variables In `.env`

> Note: this feature is available with `react-scripts@1.1.0` and higher.

Expand variables already on your machine for use in your `.env` file (using [dotenv-expand](https://github.com/motdotla/dotenv-expand)).

For example, to get the environment variable `npm_package_version`:

```
REACT_APP_VERSION=$npm_package_version
# also works:
# REACT_APP_VERSION=${npm_package_version}
```

Or expand variables local to the current `.env` file:

```
DOMAIN=www.example.com
REACT_APP_FOO=$DOMAIN/foo
REACT_APP_BAR=$DOMAIN/bar
```

## Referencing Environment Variables in the HTML

> Note: this feature is available with `react-scripts@0.9.0` and higher.

You can also access the environment variables starting with `REACT_APP_` in the `public/index.html`. For example:

```html
<title>%REACT_APP_WEBSITE_NAME%</title>
```

Note that the same restrictions as using environment variables in Javascript apply:

- Apart from a few built-in variables (`NODE_ENV` and `PUBLIC_URL`), variable names must start with `REACT_APP_` to work.
- The environment variables are injected at build time. If you need to inject them at runtime, [follow this approach instead](title-and-meta-tags.md#generating-dynamic-meta-tags-on-the-server).
- variables must be defined for every `react-scripts` run 

> WARNING: Do not store any secrets (such as private API keys) in your React app!
>
> Environment variables are embedded into the build, meaning anyone can view them by inspecting your app's files.




