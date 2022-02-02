# Scripting System RFC
Scripting is a part of the Assets System and is integral in extending the client's capabilities. Scripts are written in Typescript and is compiled with the Typescript Compiler at runtime and run under the V8 Javascript engine.

## Table of Contents
1. Intents
2. Commands
    - Defining Commands
    - Invoking Commands
3. Configuration
    - Getting and Setting Values
    - Listening to Changes
4. Entities
5. Tracking
6. Notifications
    - Simple Notifications
    - Progress Notifications
    - Indeterminate Notifications
7. Localization

## Intents
Intents are integral in making the scripting system secure as the developer must explicitly state featuresets they may request access from the client. Although not fool-proof, transparency to the end-user is enough to know what the extension is capable of doing with the client.

The developer may define intents it may need from the asset's metadata file. The possible values are:

|Intent Name|Description|
|-----------|-----------|
|`data`|The script may read from or write to disk in the its given directory.|
|`network`|The script may download or upload resources online.|
|`entities`|The script may create and destroy its own entities.|
|`scene`|The script may access the scene's root and access to other entities.|
|`tracking`|The script may access tracking information.|
|`notifications`|The script may create notifications.|
|`system`|The script may access the client. This is only for internal use.|

## Commands
Commands are central to how interaction is possible between the script to the client and the script to other scripts.

### Defining Commands
In defining a command, it must be written in a Typescript module and must have a decorator defined to a function. By default, commands are accessible via the command palette.

*Example:*
```ts
@command
function sayHello(): void {
    // TODO: Write code that says hello to the user.
}
```

### Invoking Commands
Other scripts can also invoke commands. The string format of a command's ID is:

`<asset-name>:<module-name>.<function-name>`

|Part Name|Description|
|---------|-----------|
|`asset-name`|The unique ID of the asset defined in the metadata.|
|`module-name`|The module name of where the function is found. This is usually the file name sans the extension.
|`function-name`|The function name to invoke.

An example of a command invocation done in code is:

*Example:*
```ts
invoke("my-asset:myModule.myFunction", "value", 10);
```

## Configuration
Assets may provide configuration which can be modified in the client.

### Setting and Getting Values
Scripts may also get the values from configurations. To set or get a config value programatically, it is done so using:

*Example:*
```ts
// getter
let username: string = config.get("username");

// setter
config.set("username", "user");
```

### Listening to Changes
Scripts can set up functions that can listen for changes in values of the configuration item.

The function must return a boolean to determine whether the value has successfully been changed or not. If the function is asynchronous, then it must return a promise that resolves a boolean.

Any number of functions can be attached to the same configuration key.

*Example:*

`meta.json`
```json
{
    "config": [
        {
            "name": "username",
            "type": "string",
            "default": ""
        }
    ]
}
```

`scripts/configListener.ts`
```ts
@listen("username")
function onUsernameChange(username: string): bool {

}

@listen("username")
async function onUserNameChangeAsync(username: string): Promise<bool> {

}
```

## Notifications
Scripts are able to push notifications to the client to inform users of activities or notify them of events.

### Simple Notifications
To push a simple notification:

*Example:*
```ts
pushNotification("Hello World");

// or

pushNotification("Hello World", NotificationType.Error);

// or

pushNotification("Hello World", NotificationType.Info, onClick);
```

### Progress Notifications
To push notifications with progress:

*Example:*
```ts
let notif = pushNotification("Ongoing Task", NotificationType.Progress);

notif.update(0.4) // a value between 0 and 1

notif.complete();
```

### Indeterminate Notifications
To push notifications with an indeterminate end time:

*Example:*
```ts
let notif = pushNotification("Ongoing Task", NotificationType.Indeterminate);

// do tasks

notif.complete();
```

## Localization
Localization is optional but preferrable. Scripts may access localized strings defined in its own `lang` directory or the global registry via the localization API:

*Example:*
```ts
// lang/en-US.json
// "myAsset.sayHello": "Hello {0}"
getLocalizedString("myAsset.sayHello", "user");
```
