# Model API RFC

*Author: [@sr229](https://git.io/sr229) and [@LeNitrous](https://github.com/LeNitrous)*

## Table of Contents

1. [Introduction](#Introduction)
2. [API](#API)
      - [Implementation](#Implementation)
      - [Rig Providers](#Rig-Providers)
         - [Setting Tracking Defaults](#setting-tracking-defaults)
      - [Face Control Providers](#Face-Control-Providers)
         - [Setting Facepoint Defaults](#setting-facepoint-defaults)
      - [Bodygroup Providers](#bodygroup-providers)
      - [API Paradigms](#API-Paradigms)


## Introduction

The Model API is one of the core APIs that Vignette implements, and is one of the main APIs we rely on to provide model format support both in-tree and externally as extensions. The main tenets of the API is as follows:

- The API must be robust and provide the necessary facilities for the developers.
- The API should be abstracted enough so developers will not write platform-specific rendering APIs (OpenGL, Vulkan, WebGPU, etc.), but powerful enough to support their model format's specialized rendering techniques.

- Registration of model providers should be as easy as one command invocation, and as much as possible **Model providers should not write their own UI to prevent fragmentation.**


## API

At best, a model provider should be able to provide the following:

- A Model format support
- Skeleton and Rig
- Bodygroups
- Face Controls

Modeled after [Visual Studio Code](https://code.visualstudio.com/api/language-extensions/overview)'s Language Support, Every model provider should start their life by registering the necessary functions which are used by the Model UI, namely:

- The Model Provider API: `vignette.model.registerProvider()`
- The Rig provider API: `vignette.model.registerRigProvider()`
- The Model Bodygroup API: `vignette.model.registerBodygroupProvider()`
- The Face Control API: `vignette.model.registerFaceProvider()`

of course, calling the Extension API can be done directly inside the `activate()` function, with the option to get the Context of the extension as `vignette.ExtensionContext`.

### Implementation

Any model provider should have the following implementation details, as detailed by `IModelProvider`.

```typescript
interface IModelProvider extends IDisposable {
    createModel: Promise<Model>;
    dispose: void;
}
```

A Model provider must provide a `createModel()` function that returns a `Model` which will provides the following:

- Rendering and Lifecycle
- Bone Data (Can be manipulated by `RigProvider`)
    - `RigProvider` can use this data to set `TrackingRegionDefaults` which allows Vignette to automatically designate which part of the tracking skeleton can be assigned to the bone.
- Bodygroup Data (Can be manipulated by `BodygroupProvider`)
- Face Controls (Can be manipulated by `FaceProvider`)

```typescript
interface IModel extends IDisposable {
    // Rendering and lifecycle
    initialize: Promise<void>;
    draw: Promise<void>;
    update: Promise<void>;
    dispose: void;
    // Bones data
    // (always available but hidden if skeleton is defined)
    bones: Bone[];
    // Skeleton data
    skeleton?: Skeleton;
    // bodygroup data (nullable, since not all models has bodygroups)
    bodygroups?: Bodygroup[];
    // face control groups
    faceFlexes: FaceControl[];
}
```

### Rig Providers

Every model must have a rig provider to allow Vignette to manipulate the skeleton data in your model. This is implemented via `IRigProvider`.

```typescript
interface IRigProvider extends IDisposable {
    createRigData: Promise<Rig>;
    dispose: void;
}
```

`RigProvider`s must implement `createRigData` which returns a `Rig`, which when initialized, returns a `Skeleton`.

```typescript
interface Rig extends IDisposable {
    // lifecycle
    init: Promise<ISkeleton>;
    dispose: void;
}
```

```typescript
interface ISkeleton {
    Head: IHead;
    Neck: IBone;
    Arms: IArm[];
    Hands: IHand[];
    Hip: IHip;
    Legs: ILeg[];
}
```

`Rig`s outputs a `Skeleton`, which defines a standard data structure to allow Vignette to properly visualize your model's skeleton. The following types inherits Moetion's data types, which allows a more standardized format between each model formats.

#### Setting Tracking Defaults

Model providers via `RigProvider` can set where to assign the tracking skeleton to their model's bones, given their model format has a standard naming scheme. 

```typescript

import { TrackingRegions, vignette } from './vignette'

function rigDefaultBones() {
    vignette.tracking.setDefaultTrackingRegion(Skeleton.Legs[1], TrackingRegions.Legs[1]);
}

```

### Face Control Providers

For the user to control the face parameters and/or allow the tracking interface to control them, a model provider must provide a `FaceProvider`, which facilitates the controls of the face control groups.

```typescript
interface IFaceControl extends IDisposable {
    createFaceController: Promise<Face>;
    dispose: void;
}
```

```typescript
interface IFace {
    Eyebrows: IEyebrow[];
    Eyes: IEye[];
    Nose: Nose;
    Mouth: Mouth;
}
```

A `FaceProvider` must have a `createFaceController()` method that returns a `Promise<Face>` which is used by the host to determine the face control groups.

#### Setting Facepoint Defaults

Just like `RigProvider`, model providers has the ability to provide provide tracking defaults for the face control groups.

```typescript
import { TrackingRegions, vignette } from 'vignette';

function rigDefaultBones() {
    vignette.tracking.setDefaultTrackingRegion(Face.Eyebrows[1], TrackingRegions.FaceRegions.Eyebrows[1]);
}
```

### Bodygroup Providers

Some model formats may have definitions for bodygroups, which are baked models that are part of the base model. To allow Vignette to use this, a model provider must define a `BodygroupProvider`.

```typescript
interface IBodygroupProvider extends Disposable {
    createBodygroups: Promise<Bodygroup[]>;
}
```
```typescript
interface IBodygroup {
    id: string;
    visible: boolean;
    replaceModel: Promise<any>;
}
```
`BodygroupProvider`s must provide a `createBodygroups` method which returns a `Promise<Bodygroup[]>`. A `Bodygroup` must contain the ID of the bodygroup, `isVisible`, which allows the host to modify it's visibility, and a `replaceModel` method which provides the function for the toggle to allow it to be visible within the viewport, which usually uses the lower-level rendering API provided in the Extension API.


### API Paradigms

Following the principles stated in the introduction, our API follows the following:
  - [**DRY**](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) - Do not Repeat Yourself Paradigm. That means we encourage code reuse as much as possible.
      - This also means that we already provide the necessary facilities such as the UI and the rendering primitives. All you need to do is implement your model format.
   - [**KISS**](https://en.wikipedia.org/wiki/KISS_principle) - Keep it Simple, Stupid. A paradigm established by the US. Navy in the 1960s, and is one of the core principles of the API design of Vignette. 
      - We get rid of any unnecessary complexities in the core API design and extensions are programmed in a way that they're very simple and straightforward with their purpose, this includes our Model providers.
