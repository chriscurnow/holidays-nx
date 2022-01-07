# Welcome to the Holidays-nx/workspace

## General

This workspace uses `@nrwl/nx` ([docs](https://nx.dev/l/a/getting-started/intro))

There are many additional inline docs in the starter source code, please search for `// [STARTER DOCS]` which is the prefix for such comments

## Scripts

### Run apps

Apps can be started with help of provided npm `serve:*` script, eg `npm run serve:contractor`, please add such script for every newly added application
In case more than one app has to be started simultaneously add the `--port` flag to the serve scripts

### Lint, Test, E2E & Build apps

Following commands work for `build`, `test`, `e2e` & `lint`...

- `npm run build:all` - force build all apps (or `npm run test:all`, ...)
- `npm run build:affected` - build apps that are affected by the most recent change (try to keep commits small)

#### E2E (end 2 end tests with cypress)

- `npm run e2e:<app-name>:watch` - build & serve application, then open Cypress UI to run and see tests in action against the runnign & rendered application to develop & debug them, it is a good practice to cover main business flows with e2e tests, [nx cypress integration](https://nx.dev/l/r/cypress/overview), [cypress docs]()

### Continuous Integration (CI)

- `npm run ci:pull-request` - run format:check, lint, test, e2e and build on **affected projects**
- `npm run ci:release` - run format:check, lint, test, e2e and build on **all projects**

### Other utils

- `npm run deps` - run nx analyzer to visualise workspace dependencies
- `npm run format:write` - apply prettier styles on all changed files
- `npm run format:check` - test if all files are prettier conform

## Schematics (generate apps, libs and others)

One of the best features of Angular (& @nrwl/nx) is the schematics which allow us to generate considerate chunks of code which alre always up to date and following desired best practices
Available schematics can be discovered using `nx list` and `nx list <plugin-name>`, eg `nx list @nrwl/workspace`, besides that `nx` supports all the standard Angular schematics which can be discovered [here](https://angular.io/cli/generate) (they will be accessed using `nx g` instead of `ng g` inside of the nx workspace)

**IMPORTANT** - always preview command output with `--dry-run` flag to make sure it is oging to generate what you expect it to, also don't hesitate to append `--help` to see additional options

Some `generate` flags are specified as defaults so they don't need to be provided explicitly with every invocation, please see the defaults in `nx.json` file

- **app** - `nx g app <app-name> --tags scope:app --routing`

- **lib** - `nx g lib <lib-group>/<lib-type>-<lib-name> --tags scope:<lib-type>` (generic command, see specific commands for particular lib types bellow)

### Lib types (for tags)

Lib types force clean one way dependency graph in the workspace, see the `.eslintrc.json` file to see (or adjust) import rules between various scopes, keep in mind that we always want to preserve clean one way dependency graph

The dependency tree can be validated by running `npm run lint:all` or (`npm run lint:affected`)

Read more [library groups](https://nx.dev/l/a/structure/grouping-libraries), [library types](https://nx.dev/l/a/structure/library-types), [tags](https://nx.dev/l/a/structure/monorepo-tags)

- **scope:feature** `nx g lib <lib-group>/feature-<lib-name> --routing --lazy --tags scope:feature --parentModule apps/<app-name>/src/app/app.module.ts` (if used in more than one app (eg when `<lib-group>` is `shared`) then copy the routing definition form the parent module of the original consumer app into all other consumers)
- **scope:data-access** `nx g lib <lib-group>/data-access-<lib-name> --tags scope:data-access`
- **scope:ui** `nx g lib <lib-group>/ui-<lib-name> --tags scope:ui`
- **scope:ui:shared** `nx g lib shared/ui-<lib-name> --tags scope:ui:shared`
- **scope:util** `nx g lib <lib-group>/util-<lib-name> --tags scope:util`

### Lib groups

Libraries are usually grouped into folders based on following structure

`/libs/<lib-group>/<lib-type>-<lib-name>`

Common `lib-group`s are:

- `<app-name>`, eg `contractor` or `admin`, which results in path like `/libs/contractor/feature-dashboard`
- `shared`, eg `/libs/shared/ui-main-layout`

Any lib which will be used in more than one app should belong to `shared` group so its src code will be in `/libs/shared/<lib-type>-<lib-name>`

### Generate components, services, ...

- **components** - `nx g c <optional-component-path>/<component-name> --project <project-name>`, eg `nx g c some-component --project shared-ui-main-navigation` will be generated in with `libs/shared/main-navigation/src/lib/some-component`, please note the `-` instead of `/` as that denotes the actual project name which can be double-checked in the `workspace.json`, for first (container) component generated in the `feature-<lib-name>` libraries with routing, the component has to be added to a route definition of that lib main module
- **services** - `nx g s <optional-service-path>/<service-name> --project <project-name>`, eg `nx g s user --project shared-data-access-user` will be generated in with `libs/shared/data-access-user/src/lib/user-service.ts`, please note the `-` instead of `/` as that denotes the actual project name which can be double-checked in the `workspace.json`, the services are by default `providedIn: 'root'` which makes them a global singletons, the value can be switched to `any` to get instance per lazy route, if service instance is needed per component then the `providedIn: *` can be removed and service has to be added to the `providers: []` of the desired component
- **other** - `nx g <schematics-type> <optional-path>/<name> --project <project-name>`, see list of supported [schematics](https://angular.io/cli/generate)

### Add 3rd party libraries with schematics

1. install the desired package manually, eg `npm i @angular/material`
2. (optional) check the list of available schematics `nx list @angular/material`
3. run `ng-add` schematics of the library using `nx g @angular/material:ng-add --project <app-name>`


### Move / Remove code (workspace maintenance)

- **move** - `nx g mv --project <project-name> <new-destination>`, eg `nx g mv --project shared-ui-main-layout shared/ui-layout`, please note the `-` instead of `/` in project name as that denotes the actual project name which can be double-checked in the `workspace.json`

- **remove** - `nx g rm <path>-<app-or-lib-name>`, optional `--dry-run` (to preview changes), eg `libs/admin/feature-dashboard` will be removed with `ng g rm admin-feature-dashboard`, please note the `-` instead of `/` as that denotes the actual project name which can be double-checked in the `workspace.json`

## General development organization

- **apps** - should be mostly containers which bring everything together but implement very little inline logic, apps should contain:

  - basic Angular vendor imports like `BrowserModule`, `BrowserAnimationModule` or `HttpClientModule` so any module which registers app global singletons (never import those in lib source code (tests are ok))
  - empty root NgRx configuration for store, effects, router store, dev tools...
  - eager `data-access-*` libs to register NgRx state slices available globally in the whole app
  - `ui-*` and eager `feature-*` (without lazy loading or routing) to create main layout
  - routing which lazy loads feature libs from `libs/<app-name>/feature-<feature-name>` or from `libs/shared/feature-<feature-name>`

- **libs** - should implement most of the logic
  - app specific libs `/libs/<app-name>/<lib-type>-<lib-name` - (mostly `feature-*` for lazy loaded app features & `data-access-*` for eager and lazy NgRx data slices & services)
  - shared libs `/libs/shared/<lib-type>-<lib-name` - shared across multiple apps, eg `ui-*` for reusable components (widgets), `data-access-*` or even whole reusable features, eg `feature-login` which is a lazy-loaded feature which may be included used in multiple apps

## State Management

### NgRx

Ngrx is the best (and most popular) state management library for Angular

Please see the official [docs](https://ngrx.io/docs)

The state management will be implemented in the `data-access` libraries.

1. generate `data-access` lib using `nx g lib <lib-group>/data-access-<lib-name>`
2. generate `ngrx state` in the lib using `nx g ngrx <state-name> --module libs/<lib-group>/data-access-<lib-name>/src/lib/<lib-group>-data-access-<lib-name>.module.ts --useDataPersistence true`

eg

1. `nx g lib contractor/data-access-contracts`
2. `nx g ngrx contract --module libs/contractor/data-access-contracts/src/lib/contractor-data-access-contracts.module.ts --useDataPersistence true`

#### NgRx Best practices

- create `select<ContainerComponentName>View` selectors which provide "perfect" state for a given component so that the component can render the state with ease without need to process data further
- combine selectors as needed
- feature selectors CAN use eager state and hence eager state slice selectors (one way dep graph)
- router state (path params, query params) should be always accessed with the help of the provided [router store](https://ngrx.io/guide/router-store/selectors)
- reducers are pure functions which just update state, never any service calls or side effects
- in general, testing of reducers & selectors is VERY simple as they are jsut pure functions unaware of Angular
- using NgRx selectors should dramatically simplify implementation of Angular components which should just receive state from store using `selectors` and dispatch `actions` based on user interaction
- NgRx is very popular so there is a lot of available community resources about the topic
- it really pays out to familiarize yourself with NgRx as the state handling is the cause of most bugs, NgRx allows us to FULLY decouple state management from the view (components)
- do NOT use `@ngrx/data` as it's too complex and inflexible if your use cases does NOT fit it 100% out of the box

### Redux Dev Tools

Add [Redux Dev Tools](https://chrome.google.com/webstore/detail/redux-devtools/lmhkpmbekcpmknklioeibfkpmmfibljd?hl=en) to your browser and access it through the developer tools (it's in its own Redux tab)

- see current state snapshot
- see actions as they happen over time
- see the state diff caused by a particular action

Apps activate dev tools with `!environment.production ? StoreDevtoolsModule.instrument() : [],` in their `app.module.ts` for DEV mode only (because of perf overhead), might need to be disabled (commented out) as the state grows and only enabled when debugging of state is desired

## Other

### Styles & themes

The starter uses `@angular/material` component library which needs a custom theme to apply to all the material components

This is implemented in `libs/shared/styles-theming` library (see local `// [STARTER DOCS]` comments for how to change the theme)
These styles then have to be included in every app in the `project.json` file in the `targets -> build -> options -> styles` array as the first item in the array, eg `"libs/shared/styles-theming/src/lib/styles.scss"`

Besides, it is possible to use css variables to theme own components, available variables are:

- `--primary`
- `--primary-lighter`
- `--primary-darker`
- `--accent`
- `--accent-lighter`
- `--accent-darker`
- `--text`
- `--text-contrast`
- `--contrast`

and can be used like `color: var(--text);` in any component stylesheet

### Icons

Included icon set can be explored [here](https://fonts.google.com/icons)

### Stay up to date

- **update nx workspace** - `nx migrate latest && npm i && nx migrate --run-migrations`, its better to do updates often (or in smaller steps), [read more](https://nx.dev/l/n/core-concepts/updating-nx#updating-nx)
