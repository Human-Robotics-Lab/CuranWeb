---
layout: "default"
permalink : "/user_interface/"
---

### User Interface

The user interface library is build upon [SKIA](https://skia.org/). This is a great library in cpp which renders geometries, images, paths, and many other geometries. Check the SKIA [API](https://skia.org/docs/user/api/) to see what is possible. The main thing you need to understand is that Curan only executes the connection between the GPU and your CPU, all other things are taken care of by SKIA. 

The first step in all our programming is to properly link our executable to the user interface library, we can achieve this through.

```cmake
add_executable(myexecutable main.cpp)


target_link_libraries(myexecutable PUBLIC
userinterface
)
```
Now the compiler can link safely to our library. 

* Signal(todo) : [Signal](#signal)
* SignalProcessor(todo) : [SignalProcessor](#signalprocessor)
* Empty Canvas(todo) : [Empty Canvas](#empty canvas)
* Containers and Buttons(todo) : [Containers and Buttons](#containers and buttons)
* ImageDisplay(todo) : [ImageDisplay](#imagedisplay)
* ImutableTextPanel(todo) : [ImutableTextPanel](#imutabletextpanel)
* ItemExplorer(todo) : [ItemExplorer](#itemexplorer)
* Loader(todo) : [Loader](#loader) 
* MiniPage(todo) : [MiniPage](#minipage)
* MutatingTextPanel(todo) : [MutatingTextPanel](#mutatingtextpanel)
* OpenIGTLinkViewer(todo) : [OpenIGTLinkViewer](#openigtlinkviewer)
* Overlay(todo) : [Overlay](#overlay) 
* Panel(todo) : [Panel](#panel)
* Plotter(todo) : [Plotter](#plotter)
* RadioButton(todo) : [RadioButton](#radiobutton)
* RuntimeEffect(todo) : [RuntimeEffect](#runtimeeffect)
* Slider(todo) : [Slider](#slider)
* SliderPanel(todo) : [SliderPanel](#sliderpanel)
* TextBlob(todo) : [TextBlob](#textblob)



## Signal

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp

```

## SignalProcessor

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp

```
 
## Empty Canvas

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp

```
 
## Containers  and Buttons

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp

```
 
## ImageDisplay

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp

```
 
## ImutableTextPanel

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp

```
 
## ItemExplorer

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp

```
 
## Loader

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp

```
 
## MiniPage

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp

```
 
## MutatingTextPanel

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp

```
 
## OpenIGTLinkViewer

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp

```
 
## Overlay

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp

```
 
## Panel

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp

```
 
## Plotter

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp

```
 
## RadioButton

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp

```
 
## RuntimeEffect

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp

```
 
## Slider

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp

```
 
## SliderPanel

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp

```
 
## TextBlob

The full source code of the following tutorial is shown next. We will explain line by line what each 
abstraction does. 
 
```cpp

```