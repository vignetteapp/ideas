# Model API RFC

*Author: [@sr229](https//git.io/sr229) and [@LeNitrous](https://github.com/LeNitrous)*

## Table of Contents

1. [Introduction](#Introduction)
2. [API](#API)
      - [API Paradigms](#API-Paradigms)
      - [Potential Limitations](#Potential-Limitations)

## Introduction

The Model API is one of the core APIs that Vignette implements, and is one of the main APIs we rely on to provide model format support both in-tree and externally as extensions. The main tenets of the API is as follows:

- The API must be robust and provide the necessary facilities for the developers.
- The API should be abstracted enough so developers will not write platform-specific rendering APIs (OpenGL, Vulkan, WebGPU, etc.), but powerful enough to support their model format's specialized rendering techniques.

- Registration of model providers should be as easy as one command invocation, and as much as possible **Model providers should not write their own UI to prevent fragmentation.**.


## API

At best, a model provider should be able to provide the following:

- A Model format support
- Skeleton and Rig
- Bodygroups
- Face Controls

Modeled after [Visual Studio Code](https://code.visualstudio.com/api/language-extensions/overview)'s Language Support, Every model provider should start their life by registering the necessary functions which are used by the Model UI, namely:

- The Model provider API: `vignette.model.registerProvider()`
- The Rig provider API: `vignette.model.registerRigProvider()`
- The Model Bodygroup API: `vignette.model.registerBodygroupProvider()`
- The Face Control API: `vignette.model.registerFaceProvider()`

of course, calling the Extension API can be done directly inside the `activate()` function, with the option to get the Context of the extension as `vignette.ExtensionContext`.

### API Paradigms

Following the principles stated in the introduction, our API follows the following:
  - [**DRY**](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself) - Do not Repeat Yourself Paradigm. That means we encourage code reuse as much as possible.
      - This also means that we already provide the necessary facilities such as the UI and the rendering primitives. All you need to do is implement your model format.
   - [**KISS**](https://en.wikipedia.org/wiki/KISS_principle) - Keep it Simple, Stupid. A paradigm established by the US. Navy in the 1960s, and is one of the core principles of the API design of Vignette. 
      - We get rid of any unnecessary complexities in the core API design and extensions are programmed in a way that they're very simple and straightforward with their purpose, this includes our Model providers.