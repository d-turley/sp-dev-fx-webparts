# Services

<!--
    TODO - need to write the intro
    
    * overview of services framework
    * specifying services used in app (descriptors)
-->

## Consuming services
There are different methods for accessing services depending on the context.

### Function components - React hooks
To access the current user from the Directory Service within a React function component:

```
import React, { FC } from 'react';
import { useDirectoryService } from 'services';

interface IProps {
    ...
}

export default const Widget: FC<IProps> = (props) => {
    const { currentUser } = useDirectoryService();

    ...

    return <>{currentUser.title}</>;
};
```

### Class components
To access the current user from the Directory Service from a React class component:

```
import React, { Component } from 'react';
import { withServices, ServicesProp, DirectoryService, DirectoryServiceProp } from 'services';

interface IOwnProps {
    ...
}
type IProps = IOwnProps & ServicesProp<DirectoryServiceProp>;

interface IState {
    ...
}

class Widget extends Component<IProps, IState> {
    constructor(props: IProps) {
        // accessing current user from the constructor
        const { [DirectoryService]: { currentUser } } = props.services;

        this.state = {};
    }

    private readonly _somePrivateFunction = () => {
        // accessing current user from a function
        const { [DirectoryService]: { currentUser } } = this.props.services;
    }

    public render(): JSX.Element {
        // accessing current user in the render function
        const { 
            services: { [DirectoryService]: { currentUser } }
        } = this.props;

        return <>{currentUser.title}</>;
    }
}

export default withServices(Widget);
```

### From other services
To access the Directory Service from another service:

```
import { ServiceContext, DirectoryService, IDirectoryService, DirectoryServiceProp } from 'common/services';
import { ICoffeeService } from './CoffeeServiceDescriptor';

export class CoffeeService implements ICoffeeService {
    private readonly _directory: IDirectoryService;

    constructor({
        [DirectoryService]: directory
    }: ServiceContext<DirectoryServiceProp>) {
        this._directory = directory;
    }

    public async initialize(): Promise<void> {
        const { currentUser } = this._directory;

        ...
    }
}
```

Don't forget to add the Directory Service as a dependency on the Coffee Service descriptor:
```
export const CoffeeServiceDescriptor: IServiceDescriptor<typeof CoffeeService, ICoffeeService, CoffeeServiceProp> = {
    ...
    dependencies: [..., DirectoryService, ...],
    ...
};
```


## Common Library Services
The [Common Library (src/common)](../src/common/) provides several ready-to-use services for building apps.

### Developer Service
The Developer Service is a mechanism for exposing custom helper scripts for developers while building your application.  Any part of the application can access the Developer Service and register one or more functions that will only be available when the application is running in the SharePoint workbench or if 'isDev=true' is appended to the query string.

To use this service:
1. Access the service and register a script:
```
const developer = useDeveloperService();
developer.register({
    myHelpfulScripts: {
        doSomethingUseful: async () => {
            await ...
            console.log("I hope you find this useful.");
        }
    }
});
```
1. Run the application in the SharePoint workbench (or append 'isDev=true' to the query string)
1. Open the DevTools console
1. Execute the script:
```
dev.myHelpfulScripts.doSomethingUseful()
```

The `dev` object is globally accessible.

There are many possible uses for developer scripts:
* To quickly create useful sample data so that you can focus on building features rather than manually creating items in the app each time you need to recreate your lists or run the app on a new site.  Check out [OnlineEventsService.ts](../src/services/events/OnlineEventsService.ts) for an example of this.
* To create data to quickly set up scenarios for your testers, or for load testing the app
* Work on code for migrating data from a legacy list/app
* To toggle a feature on/off, or toggle a dev-only screen element
    * For example, say you are using React Router, you might build a small component that simply outputs the current route somewhere on the screen, and that component can register a dev script to easily toggle the display on and off to aid in debugging the routes.

### Directory Service
The Directory Service provides access to users and SharePoint groups.  It provides access to the current user and their permissions, plus functions to search/resolve users (which is utilized by the UserPicker component), find and manipulate SP groups, getting role definition IDs, and more.

Early on we recognized a need to unify the concept of a 'user' between SharePoint, PnPjs, MS Graph, etc., so we introduced the User class to the common library, which is our simple abstraction of a user, and is utilized everywhere from the UserPicker component, to entities, to the current user, SP group membership, and searching for users via the Directory Service, to loading and saving list items via the SharePoint service.

### Domain Isolation Service
The Domain Isolation Service provides functions for converting to and from the isolated domain URL and the tenant's SharePoint domain URL, which is especially useful for implementing deep-linking capability in a domain-isolated solution.

### Live Update Service
The Live Update Service provides high-level functionality for listening for changes to lists in order to support the Live Update feature.  It is mainly consumed by other code in the common library.

### SharePoint Service
The SharePoint service provides abstractions for loading and persisting entities as list items.  It is mainly consumed by other code in the common library.

### Timezone Service
The Timezone Service provides an abstraction SharePoint timezones and a mapping to their Moment.js equivalents.  This service is especially useful in applications that need to read and write date and time values to SharePoint lists.

## Defining a Custom Service

<!--
    TODO

    * describe how to create a custom service
        * descriptor file and contents
        * index file for exports
        * implementation
            * ctor pattern
            * init pattern
            * async pattern
            * track/persist pattern
        * loaders
            * refer to entities page for loading up relationships
            * how to load/persist various field types
-->