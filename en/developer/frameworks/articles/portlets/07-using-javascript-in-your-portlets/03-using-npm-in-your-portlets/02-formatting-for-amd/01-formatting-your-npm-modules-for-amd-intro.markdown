---
header-id: formatting-your-npm-modules-for-amd
---

# Formatting Your npm Modules for AMD

[TOC levels=1-4]

For @product@ to recognize your npm modules, they must be formatted for the 
Liferay AMD Loader. Luckily, the liferay-npm-bundler handles this for you, you 
just have to provide the proper configuration and add it to your build script. 
This article shows how to use the liferay-npm-bundler to set up npm-based 
portlet projects. The example structure below is referenced throughout this 
article. You can download it [here](https://github.com/izaera/liferay-npm-bundler-2-example) 
if you want to follow along:

- `npm-angular5-portlet-say-hello/`
    - `META-INF/`
        - `resources/`
            - `package.json`
                - name: npm-angular5-portlet-say-hello
                - version: 1.0.0
                - main: js/angular.pre.loader.js
                - scripts:
                    - build: tsc && liferay-npm-bundler
            - `tsconfig.json`
                - target: es5
                - moduleResolution: node
            - `.npmbundlerrc`
                - exclude: 
                    - \*: true
                - config:
                    - imports:
                        - npm-angular5-provider:
                            - @angular/animations: ^5.0.0
                            - @angular/cdk: ^5.0.0
                            - @angular/common: ^5.0.0
                            - @angular/compiler: ^5.0.0
                        - "" :
                            - npm-angular5-provider: ^1.0.0
            - `js/`
                - `indigo-pink.css`
                - `angular.pre.loader.ts`
                // Bootstrap shims and providers
                - import npm-angular5-provider;
- `npm-angular5-provider`
    - `package.json`
        - name: npm-angular5-provider
        - version: 1.0.0
        - main: bootstrap.js
        - scripts:
            - build: liferay-npm-bundler
        - dependencies:
            - @angular/animations: ^5.0.0
            - @angular/cdk: ^5.0.0
            - @angular/common: ^5.0.0
            - @angular/compiler: ^5.0.0
            - ...
    - `src/main/resources/META-INF/resources/`
        - `bootstrap.js`
            // This file includes polyfills needed by Angular and must be loaded before the app
            - require('core-js/es6/reflect');
            - require('core-js/es7/reflect');
            - require('zone-js/dist/zone');

Follow these steps to configure your project to use the liferay-npm-bundler:

1.  Install NodeJS >= [v6.11.0](http://nodejs.org/dist/v6.11.0/) if you don't 
    have it installed.

2.  Navigate to your portlet's project folder and initialize a `package.json` 
    file if it's not present yet.

    If you don't have a portlet already, create an empty MVC portlet project. 
    For convenience, you can use 
    [Blade CLI](/docs/7-2/reference/-/knowledge_base/r/installing-blade-cli) 
    to create an empty portlet with the [mvc portlet blade template](/docs/7-2/reference/-/knowledge_base/r/using-the-mvc-portlet-template). 

    If you don't have a `package.json` file, you can run `npm init -y` to create 
    an empty one based on the project directory's name. 

3.  Run the following command to install the liferay-npm-bundler:

    ```bash
    npm install --save-dev liferay-npm-bundler
    ```

    | **Note:** Use npm from within your portlet project's root folder (where the
    | `package.json` file lives), as you normally do on a typical web project.

4.  Add the `liferay-npm-bundler` to your `package.json`'s build script to pack 
    the needed npm packages and transform them to AMD:

    ```json
    "scripts": {
          "build": "[... && ] liferay-npm-bundler"
    }
    ```

    The `[...&&]` refers to any previous steps you need to perform (for example, 
    transpiling your sources with Babel, compiling SOY templates, transpiling 
    Typescript, etc.). The example includes the Typescript compiler (`tsc`) in 
    its build script because Angular requires it for transpiling code to ES5:

    ```json
    "build": "tsc && liferay-npm-bundler" 
    ```

    | **Note:** You can use any languages you like as long as they can be
    | transpiled to ECMAscript 5 or higher. The only requirements are:
    | 
    | - That Babel can convert them to an AST to be able to process it
    | - That your browser can execute it.
    | - That modules are loaded using `require()` calls (this requirement can be
    |   relaxed by using customized plugins, but is mandatory for the default
    |   out-of-the-box configuration).
    | 
    |   When you deploy your portlet using Gradle, the build script is called as
    |   part of the process.
    
5.  Configure your project for the bundler, using the `.npmbundlerrc` file 
    (create this file in your project's root folder if it doesn't exist). See 
    the [liferay-npm-bundler's `.npmbundlerrc` structure reference](/docs/7-2/reference/-/knowledge_base/r/understanding-the-npmbundlerrcs-structure) 
    for more information on the available options. 
    
    The example excludes every dependency (using the wildcard (`*`) symbol) of 
    the `npm-angular5-portlet-say-hello` widget to prevent Angular from 
    appearing in its JAR, making the build process faster and optimizing 
    deployment. Note that `npm-angular5-provider` is also imported with no 
    namespace (`""`) because one of its modules is going to be invoked to 
    bootstrap Angular shims: see the `angular.pre.loader.ts` file, where 
    `npm-angular5-provider` is imported. That import, in turn, loads 
    `npm-angular5-provider`'s main file (`bootstrap.js`):

    ```json
    {
        ...
        "exclude": {
            "*": true
        },
        "config": {
            "imports": {
                "npm-angular5-provider": {
                    "@angular/animations": "^5.0.0",
                    "@angular/cdk": "^5.0.0",
                    "@angular/common": "^5.0.0",
                    "@angular/compiler": "^5.0.0",
                    ...
                },
                "": {
                    "npm-angular5-provider": "^1.0.0"
                }
            }
        }
    }
    ```

6.  Run `npm install` to install the required dependencies.

7.  Run the build script to bundle your dependencies with the 
    liferay-npm-bundler:

    ```bash
    npm run-script build
    ```

The resulting build for the example widget is shown below:

- `npm-angular5-portlet-say-hello/`
    - `build/`
        - `resources/main/META-INF/resources`
            - `package.json`
                - dependencies:
                    - @npm-angular5-provider$angular/animations: ^5.0.0
                    - @npm-angular5-provider$angular/cdk: ^5.0.0
                    - @npm-angular5-provider$angular/common: ^5.0.0
                    - @npm-angular5-provider$angular/compiler: ^5.0.0
            - `js/`
                - `angular.loader.js`
                    - Liferay.Loader.define("npm-angular5-portlet-say-hello@1.0.0/js/angular.loader"
                    - ['module', 'exports', 'require', 
                    '@npm-angular5-provider$angular/platform-browser-dynamic',
                    ...]
- `npm-angular5-provider`
    - `package.json`
        - name: npm-angular5-provider
        - version: 1.0.0
        - main: bootstrap.js
        - dependencies:
            - @npm-angular5-provider$angular/animations: ^5.0.0
            - @npm-angular5-provider$angular/cdk: ^5.0.0
            - @npm-angular5-provider$angular/common: ^5.0.0
            - @npm-angular5-provider$angular/compiler: ^5.0.0
            - ...
    - `bootstrap.js`
        - Liferay.Loader.define('npm-angular5-provider@1.0.0/bootstrap'
        - ['module', 'exports', 'require', 
        'npm-angular5-provider$core-js/es6/reflect', 
        'npm-angular5-provider$core-js/es7/reflect', 
        'npm-angular5-provider$zone.js/dist/zone']
    - `src/main/resources/META-INF/resources/`
        - `bootstrap.js`
            // This file includes polyfills needed by Angular and must be loaded before the app
            - require('core-js/es6/reflect');
            - require('core-js/es7/reflect');
            - require('zone-js/dist/zone');

| **Note:** By default, the AMD Loader times out in seven seconds. You can 
| configure this value through System Settings. Open the Control Panel and 
| navigate to *Configuration* &rarr; *System Settings* &rarr; *PLATFORM* &rarr; 
| *Infrastructure*, and select *JavaScript Loader*. Set the 
| *Module Definition Timeout* configuration to the time you want and click 
| *Save*.

Great! Now you know how to use the liferay-npm-bundler to bundle your npm-based 
portlets for the Liferay AMD Loader. 

## Related Topics

[Preparing Your JavaScript Files for ES2015+](/docs/7-2/frameworks/-/knowledge_base/f/using-javascript-in-your-portlets)
