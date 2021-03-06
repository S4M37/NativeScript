# NativeScript v3.0 Breaking Changes

## General notes

We are moving the modules closer to ES6 standard. This introduces few limitations. One of them is modules can no longer export variable, in such cases variables were replace with get/set functions.

## TypeScript
TypeScript projects need to reference the **ES6 and DOM libraries**. Add this to your tsconfig.json:

```
{
  "compilerOptions": {
    ...
    "lib": ["es6", "dom"]
    "baseUrl": ".",
    "paths": {
    "*": [
      "./node_modules/tns-core-modules/*",
      "./node_modules/*"]
  }
}
```

## Camera
The `camera` module is removed. Use `nativescript-camera` plugin instead.

## Location
The `location` module is removed. Use `nativescript-geolocation”` plugin instead.

## Changes in UI Modules

### Application
We are using the following import statement for the code samples in this section
```
import * as application from "tns-core-modules/application";
```

`application.mainEntry` was removed.

`application.mainModule` was removed.

Pass `mainModule` or `mainEntry` to `application.start` method.
If you need access to `mainEntry` use:
* `application.getMainEntry(): NavigationEntry`

The string `mainModule` is implicitly converted to `NavigationEntry` and set to `mainEntry`.

`application.resources` was removed, use get/set methods:
* `application.getResources(): any`
* `application.setResources(res: any)`

`application.cssFile` was removed, use get/set methods:
* `application.getCssFileName(): string`
* `application.setCssFileName(file: string)`

Removed application callback methods: 
* `application.onSuspend()`
* `application.onResume()`
* `application.onExit()`
* `application.onLowMemory()`. 

Use the corresponding events such: `application.on(“suspend”, (args: application.ApplicationEventData) => … );`

Removed all android specific callback methods. For example instead of `application.android.onActivityResult(…)` use `application.android.on("activityResult", (eventData) => …)`

### Console
`console.dump()` removed, use `console.dir()` instead

### KeyframeAnimation
Changed method `keyframeAnimationFromInfo(info: KeyframeAnimationInfo, valueSourceModifier: number): KeyframeAnimation` to `KeyframeAnimation.keyframeAnimationFromInfo(info: KeyframeAnimationInfo): KeyframeAnimation`. ValueSource no longer needed.

### Observable
Observable constructor that accepts an object is removed.
Use `observableModule.fromObject(obj)` method instead.

### SegmentedBar
Removed method `insertTab()`. You can use `items` property for setting additional tabs.
Removed method `getValidIndex()` - it was intended for internal use only.

### Span
Removed methods and properties. These were for internal use only and not needed any more:
* `parentFormattedString` property
* `updateSpanModifiers(parent: formattedString.FormattedString): void;`
* `beginEdit(): void;`
* `endEdit(): void;`

### TabView
Removed properties of `TabView` class (`ui/tab-view` module):
* `selectedColor` - use `selectedTabTextColor` instead.
* `tabsBackgroundColor` - use `tabBackgroundColor` instead.
* `textTransform` - `textTransform` is can be set on the individual `TabViewItem`s instead on the `TabView`

### Trace
The `enabled` exported variable is replaced with getter function: `isEnabled()`. You can still use the `enable()` and `disable()` methods to enable/disable tracing.

### Utils
We are using the following import statement for the code samples in this section
```
import * as utils from "tns-core-modules/utils/utils";
```
Removed `utils.parseJSON(source: string)` method – use `JSON.parse()` instead.

The following functions. These were for internal use only and not needed any more: 
* `utils.copyFrom(source: any, target: any)`
* `utils.ad.setTextTransform(view, value: string)`
* `utils.ad.setWhiteSpace(view, value: string)`
* `utils.ad.setTextDecoration(view, value: string)`
* `utils.ad.getTransformedString(textTransform: string, view, stringToTransform: string): string`
* `utils.ios.getTransformedText(view, source: string, transform: string): string`
* `utils.ios.setWhiteSpace(view, value: string, parentView?: any)`
* `utils.ios.setTextAlignment(view, value: string)`

### View

Property `cssClass` removed, use `className` instead.

The `_createUI()` methods is renamed to `createNativeView()`. It should now return a native view instance instead of setting it locally. Read more [here](#view-life-cycle)

The `_onBindingContextChanged()` method is removed – if you are using this to set the `bindingContext` of an object if that object is extending `ViewBase` it will automatically have its `bindingContext` set. In case you need to know when `bindingContext` is changed you could add handler to `this.on("bindingContextChange", handlerMethod, this)`.

VerticalAlignment - `"center"` is removed from Typescript definition files but still supported through CSS/XML. In code – use `"middle"` instead of center.

### WebView  
The `url` property of `WebView` is removed, use `src` instead.

## Property System
The way View properties are defined, stored and applied to native view is completely re-written and this is the area where most of the breaking changes and performance improvements are.

### Properties Types in Modules 3.0
There are several type of Properties in modules 3.0:
* `Property` – property defined on `ViewBase` or another view class. These are properties like `id` on `ViewBase` or `text` on `Label`. 
* `CssProperty` – property defined on `Style` type. These are properties that could be set in CSS.
* `InheritedCssProperty `- property defined on `Style` type. These are inheritable CSS properties that could be set in CSS and propagates value on its children. These are properties like `FontSize`, `FontWeight`, `Color`, etc.

### Property Example
In here is how to define `text` (view property) and `text-align` (css property) for the text-View

`my-text-view-common.ts` with cross-platform code 

```
import { View, Property, CssProperty, InheritedCssProperty, Style, } from "tns-core-modules/ui/core/view";

export class MyTextViewCommon extends View {
  ...
  // Defined property to make typescript happy. Not needed in pure JS.
  text: string;

  // Css properties are defined on the Style when registered
  // You can optionally expose the property on the View class also
  get textAlignment(): TextAlignment {
    return this.style.textAlignment;
  }
  set textAlignment(value: TextAlignment) {
    this.style.textAlignment = value;
  }
}

// Define textProperty and register it
export const textProperty = new Property<MyTextViewCommon, string>({ name: "text", defaultValue: "" });
textProperty.register(MyTextViewCommon);

// Define and register the "text-align" CSS property
export type TextAlignment = "left" | "center" | "right";
export const textAlignmentProperty = new InheritedCssProperty<Style, TextAlignment>({
    name: "textAlignment",
    cssName: "text-align",
    valueConverter: TextAlignment.parse
});
textAlignmentProperty.register(Style);
```
Every property which type (U) is not string should define valueConverter. Even properties that are of type string but allow only some strings (like enums) should define valueConverter
and either convert from string or throw an exception in case value is not valid.
If equalityComparer is not defined we use === to compare currentValue and newValue so if type is not simple comparer will be needed.

Then in the platform specific implementation use `getDefault` and `setNative` symbols from the property object (ex. `textProperty`), to define how this property is applied to native views.

`my-text-view.android.ts` with android specific implementation:

```
import {
    MyTextViewCommon, textAlignmentProperty, textProperty, ...
} from "./my-text-view-common";

export class MyTextView extends MyTextViewCommon {
  ...

  // text property
  [textProperty.getDefault](): string {
    return '';
  }
  [textProperty.setNative](value: string) {
    this.nativeView.setText(value);
  }

  // text alignment property
  [textAlignmentProperty.getDefault](): TextAlignment {
    let nativeGravity = this.nativeView.getGravity();
    // Extract the align value based on the native gravity
    return this.extractAlignmentFromGravity(nativeGravity); 
  }

  [textAlignmentProperty.setNative](value: TextAlignment) {
    let verticalGravity = this.nativeView.getGravity() & android.view.Gravity.VERTICAL_GRAVITY_MASK;
    // set the native gravity for the view base on the property value
    switch (value) {
      case "left":
        this.nativeView.setGravity(android.view.Gravity.LEFT | verticalGravity);
        break;
        ...
    }
  }
}
``` 

### Removed Classes and Methods
The way of defining properties using `Styler` class and property handlers are no longer valid. 
* `"ui/styling/style"` modules:
  * `Styler` class
  * `StylePropertyChangedHandler` class
  * `IgnorePropertyHandler` class
  * `registerHandler(property: Property, handler: StylePropertyChangedHandler, className?: string)`
  * `registerNoStylingClass(className);`
  * `getHandler(property: Property, view: View): StylePropertyChangedHandler;`
* `"ui/core/dependency-observable"` module
  * `Property`
  * `PropertyEntry`
  * `DependencyObservable`
* `“ui/builder/special-properties”` module removed
* `"ui/styling/style-property"` module removed. 

### View Class Hierarchy
The class hierarchy prior 3.0 was:
```
Observable > DependencyObservable > Bindable > ProxyObject > View
```
In 3.0 the redesign of the property system allowed us to collapse it to:
```
Observable > ViewBase > View
```

`Bindable`(`“ui/core/bindable”` module), `ProxyObject` (`“ui/core/proxy”` module) and `DependencyObservable` (`"ui/core/dependency-observable"` module) are removed. 

Consider using `View`, `ViewBase` or `Observable` instead.

### Property Types and Enumerations
As a part of the we have changed the types of many properties. The reasons for the changes:
 * Make better use of the TypeScript typings. 
 * Support for units (`dip`, `px`, `%`) for properties like `width`, `height`, `margin`

 Here is a list of view nad style properties that have their types changes:

| class.property | old type | new type |
|---|---|---|
| Style.width | number | PercentLength |
| Style.height | number | PercentLength |
| Style.minWidth | number | Length |
| Style.minHeight | number | Length |
| Style.translateX | number | Length |
| Style.translateY | number | Length |
| Style.margin |  string | string  \| PercentLength |
| Style.margin[Left/Right/Top/Bottom] |  number | PercentLength |
| Style.padding |  string | string  \| Length |
| Style.padding[Left/Right/Top/Bottom] |  number | Length |
| Style.borderWidth | string \| number | string  \| PercentLength |
| Style.border[Left/Right/Top/Bottom]Width |  number | Length |
| Style.border[TopLeft/TopRight/BottomLeft/BottomRight]Radius | number  | Length |
| ListView.rowHeight  | number  | Length  |

Many of the Style properties are also defined on the `View` class - the changes in the types are the same.

Note:  The new types are back compatible when it comes to setters. For example:
```
let image = new Image();
// still works - sets the width in dips
image.width = 100; 

// also works - sets width in pixels
image.width = { value: 100, unit: "px" }; 
```

You will hae to be careful when getting the value - you might get an complex object instead of `number`

### View Life-cycle

With 3.0 we are introducing nativeView recycling. With nativeView recycling we aim to reduce instantiation of native view which is really expensive operation in android. In order to be able to recycle it we need all properties exposed from the View to be of our new property system.

In short we have method that gets the default value for a property which is get the first time a property value is changed. Once we know that our View is not needed anymore we will reset the native view to its original state and put it in a map where some future View of the same type could reuse it.
There are 3 new important methods:
* `createNativeView(): Object;` - method to create and return the native view instance.
* `initNativeView(): void;` - method to initialize the native view. Attach handlers, owner, etc.
* `disposeNativeView(): void;` - method to detach owner and eventually to reset manually the native view. 

### Iterating Over View Children
There are two methods that allow you to traverse view-hierarchy:

For getting `View` children use:
```
public eachChildView(callback: (child: View) => boolean): void
```

This method was previously known as `_eachChildView()`. It will return `View` descendants only. For example `TabView` returns the view of each `TabViewItem` because is `TabViewItem`s are of type `ViewBase`.

Getting `ViewBase` children use:
```
public eachChild(callback: (child: ViewBase) => boolean): void;
```
This method will return all views including `ViewBase`. It is used by the property system to apply native setters, propagate inherited properties, etc.
In the case of `TabView` – this method will return `TabViewItem`'s as well so that they could be styled through CSS.
