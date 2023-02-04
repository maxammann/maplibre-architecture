# Expression Architecture
This document is meant to be an introduction to working with “expressions” in the Mapbox GL codebase. It uses gl-js examples, but the gl-native implementation is usually closely analogous (with a few layers of templating on top). I wasn’t very involved in the original design discussions, so this is mostly a distillation of what I’ve learned in the process of extending the expression language with the “collator” and “format” types. In my experience, the expression language itself is not as hard to understand as its embedding within the rest of the map.

# Background

Key Mapbox GL concepts from pre-expression days:

- “**Layout**” properties in the style spec are properties that are calculated at tile generation time, in the background. e.g.
    'text-font': ["DIN Office Pro", "Arial Unicode MS"]
- “**Paint**” properties are calculated at render time, and can generally be updated much more quickly. Whether something is a paint or layout property often just depends on the implementation difficulty of calculating the property at run time. e.g.
    'circle-radius': 20
- There are monster hybrids, like `text-size`, which is a “layout” property but is also evaluated at render time.
- “**Data-Driven Styling (DDS)**” This is the ability to generate layout and paint properties separately for each individual feature in a layer, based on (1) zoom and (2) feature properties.
- “**Run-Time Styling**” The ability to change the style at run-time. Not directly relevant to expressions except that changes require re-evaluating expressions. Changes to paint properties can be made immediately, while changes to layout properties require re-parsing tiles.
- “**Function**” This is the pre-expression syntax for DDS. “functions” exposed a fixed set of interpolation functions that could map input parameters to output parameters. Functions are still supported, but under the hood we simply convert them into equivalent general purpose “expressions”.  Example:
    'circle-radius': {
     property: 'population',
     stops: [
     [{zoom: 8, value: 0}, 0],
     [{zoom: 8, value: 250}, 1],
     [{zoom: 11, value: 0}, 0],
     [{zoom: 11, value: 250}, 6],
     [{zoom: 16, value: 0}, 0],
     [{zoom: 16, value: 250}, 40]
     ]
    }
- “**Layer**” Know your “source layer” (this is a layer of data in a vector tile) from your “style layer” (this is a layer in the style sheet, you could also think of this as a “rendered” layer)
- “**Filter**” A filter expresses rules for filtering features from a source layer to be included or excluded from a style layer. Before expressions, filters used their own mini-language in the style spec JSON. Pre-expression filters are now converted into expressions before being evaluated.
- “**Interpolation**” We support linear, categorical, exponential, and cubic-bezier interpolation functions, both before and after expressions.
- “**Constant**" vs. “**Zoom-Dependent**” vs. “**Property-Dependent**” vs. “**Zoom-and-Property-Dependent**” Every property falls into one of these categories, both before and after expressions. Typically different code pathways are used for each category, with the most complex/expensive code pathway for zoom-and-property-dependent paint properties. In expression terms, you will see the equivalent **isFeatureConstant** or **isZoomConstant**, and also **CameraExpression** (zoom) **SourceExpression** (property), and **CompositeExpression** (zoom-and-property).
- “**Paint Property Binders**" These are a complex piece of ~~magic~~ logic that support rendering of property-dependent and zoom-and-property-dependent features within minimal render-time calculation. Hand-wavy, not-quite-accurate explanation: For each “bound” property (for example: `circle-radius`), and for each feature in the layer, there’s an entry recording the value for that feature evaluated at `tileZoom` and also evaluated at `tileZoom + 1`. At render time, an interpolation parameter is calculated based on the current zoom and the interpolation curve, and then passed to the shader.
  - This approach **doesn’t change** with expressions, which explains this limitation from the docs: “… in layout or paint properties, `["zoom"]` may appear only as the input to an outer `[interpolate](https://www.mapbox.com/mapbox-gl-js/style-spec/#expressions-interpolate)`or `[step](https://www.mapbox.com/mapbox-gl-js/style-spec/#expressions-step)` expression, or such an expression within a `[let](https://www.mapbox.com/mapbox-gl-js/style-spec/#expressions-let)` expression.”
  - For more detail, see `program_configuration.js`, specifically the `Binder` interface
- “**Tokens**” e.g. `{name_en}`. The pre-expression way to refer to per-feature data in vector tiles. Before expressions, these could only be used in specialized contexts (like the `text-field` property). After expressions, tokens are automatically converted into “get” expressions, e.g. `[``"``get``"``,` `"``name_en``"``]`.


# Design Goals
- Complete the work of DDS in exposing data in vector tiles as input for styling, without falling back to implementing one specialized “function” after another. The original name for this feature was “arbitrary expressions” — representing the alternative of implementing one expression at a time.
  - See https://github.com/mapbox/mapbox-gl-js/issues/4077 for some background
- Allow arbitrary logical and mathematical operations on per-feature data
- Do *not* allow looping, turing-completeness, etc. The idea is to make a declarative language in which it should be relatively difficult to shoot yourself in the foot. We don’t want style authors to be able to make a layer whose cost to evaluate scales super-linearly with the number of features in the layer.
- Can easily be extended with new “vocabulary” over time
- Can be embedded in existing style spec
- Can port all existing functionality to expressions
- Minimal evaluation overhead

The JSON/”lispy” syntax we went with satisfied the above goals, and had the attraction that the written form of expressions stayed close to the underlying AST. Our users are generally not excited about learning a new DSL to modify styling options — but one way to think of this is that the expression language itself can be a target of other, more appropriate DSLs. For example, on the iOS SDK, we wrap expressions in the platform-appropriate `[NSExpression](https://www.mapbox.com/ios-sdk/api/4.6.0/predicates-and-expressions.html)` [syntax](https://www.mapbox.com/ios-sdk/api/4.6.0/predicates-and-expressions.html). In Studio, we use [Jamsession](https://github.com/mapbox/expression-jamsession) to make a simplified expression builder.


# Implementation

For each filter or property that uses an expression, we parse the raw JSON using a “parsing context” for that property. The result is a parsed expression tree, which is then embedded in a `PropertyValue`. At evaluation time, we provide an “evaluation context” and then evaluate the expression tree starting from the root. The result will be a constant value matching the property’s type.

## Parsing

The root of the parsing logic is in `parsing_context.js`. You start parsing with a mostly empty context (it contains information such as the expected type of the result, which is used in some automatic coercion logic). The first item in an expression array is the name (or “operator”) of the expression (e.g. `"``concat``"`). The parser looks up the `Expression`  implementation for that operator, and then hands parsing off to the class. Each implementation has its own logic for parsing children: if it accepts arguments that are themselves expressions, it will recurse into the root parsing logic, but with added context (for instance, expected types, bound `"``let``"` variables, etc.).

**CompoundExpression**
Most expressions don’t need any special parsing rules beyond knowing their return type, their argument types, and any argument type overloads. All of these expressions are implemented using `CompoundExpression`. Example (from `definitions/index.js`):

        '^': [
            NumberType,
            [NumberType, NumberType],
            (ctx, [b, e]) => Math.pow(b.evaluate(ctx), e.evaluate(ctx))
        ],

This says `"``^``"`  is an expression that returns a number, and it expects as arguments two expressions that return numbers (they could be constant expressions). The evaluator evaluates both child expressions, and then applies `Math.pow` to the results. `ctx` is the “evaluation context” getting passed through — a child expression might use it to know the current zoom level, or to look up a feature property, etc.

**Types, Assertions, and Coercions**
The expression language has parse time and run-time type checking, based on this set of types:

    export type Type =
        NullTypeT |
        NumberTypeT |
        StringTypeT |
        BooleanTypeT |
        ColorTypeT |
        ObjectTypeT |
        ValueTypeT |
        ArrayType |
        ErrorTypeT |
        CollatorTypeT |
        FormattedTypeT

An **assertion** is a type of expression that allows you to give a return type to something that doesn’t have a type. So for instance `[``"``get``"``,` `"``feature_property``"``]` returns the generic `ValueType`, but if you want to pass it to an expression that requires a string argument, you can use an assertion: `[``"``string``"``, [``"``get``"``,` `"``feature_property``"``]]`. Assertions throw an evaluation-time error if the types don’t match during evaluation.

A **coercion** is a type of expression that allows you to convert return types. You can provide fallbacks in case coercion fails. e.g. `[``"``to-number``"``, [``"``get``"``,` `"``feature_property``"``], 0]`.

The initial implementation of the expression language erred on the side of requiring users to be explicit about types — for instance, `"``get``"` expressions very frequently had to be wrapped in assertions. In response to user feedbacks, we started building more “implicit” typing into the parsing engine. This is accomplished by automatically adding `Assertion` and `Coercion` expressions at parse time (they are called “annotations”). The basic rules are:

    // When we expect a number, string, boolean, or array but have a value, wrap it in an assertion.
    // When we expect a color or formatted string, but have a string or value, wrap it in a coercion.

**Constant Folding**
There’s not much compile-time optimization in expressions, but one thing we do at compile time whenever we parse a sub-expression, we check if it’s “constant” (i.e. it doesn’t depend on any evaluation context). If so, we evaluate the expression, and then replace it with a `Literal` expression containing the result of the evaluation.

## Evaluation

Evaluation is really pretty simple — you call “evaluate” on the root expression, it recurses, and eventually gives you back either a result or an error. The somewhat tricky part is the provided  `EvaluationContext`, which hooks the expression language up to actual data on the map. It contains:

- `GlobalProperties`: global map state, like the current zoom level
- `Feature`: if this expression is being evaluated per-feature, this is the actual data for the feature from the underlying vector-tile
- `FeatureState`: Global “feature state” index, if necessary for this expression
## Property, PropertyValue, and PossiblyEvaluatedValue, oh my…

This is technically outside the expression language, but understanding how style properties are hooked up to expressions is key to understanding how expressions are actually used. `properties.js` has lots of documentation! To start getting oriented:

- `Property` refers to a property in the style specification. e.g. `'``circle-radius``'`
- `PropertyValue` refers to the right hand-side of a property in the style *sheet*. e.g. `'``circle-radius``'``: 20`. It can be a constant value or an expression.
- `PossiblyEvaluatedValue` is all about not re-evaluating when you don’t need to. So if you have a “Property” that’s of type “DataDrivenProperty”, it will have a “PossiblyEvaluatedValue”. But for instance if that value is an expression and it’s “feature-constant”, then we can just store the value here and return it in future calls to `possiblyEvaluate` instead of continually re-evaluating the expression.
# Adding a New Expression

Adding a new expression is actually pretty easy, as long as you don’t have to modify the type system. If it fits the parsing pattern of `CompoundExpression`, then you can just add it to the CompoundExpression registry, with a custom evaluation function. If not, well let’s see how to implement a “Foo” expression!

Register the operator in `definitions/index.js`:

    const expressions: ExpressionRegistry = {
        ...,
        'foo', FooExpression
      };

Create an implementation file at `definitions/foo.js`:

    export default class FooExpression implements Expression

Implement static `parse` logic that’s used to create instances of the expression:

    static parse(args: Array<mixed>, context: ParsingContext) {
        // Here's where you enforce syntax -- if type checking fails, you pass
        // the error back up the chain with context.error
        if (args.length !== 2)
            return context.error(`'foo' expression requires exactly one argument.`);
    
        if (!isValue(args[1]))
            return context.error(`invalid value`);
    
        const child = (args[1]: any);
        return new FooExpression(child);
    }

Implement the `evaluate` method:

    evaluate(ctx: EvaluationContext) {
        // We don't use the context, but pass it through to our child
        return "bar" + this.child.evaluate(ctx);
    }

Implement the `eachChild` method — this is necessary for various algorithms that traverse the expression tree:

    eachChild(fn: (Expression) => void) {
        fn(this.child);
    }

Implement the `possibleOutputs` method — this is used to do a simple type of static analysis for expressions that have a finite number of possible outputs (for instance, if the top-level expression is a `"``match``"`  expression with three literal outputs, the only possible outputs are those three, no matter what goes on in the sub-expressions. Some properties are required to have a defined set of possible outputs (for instance `text-font`), because we need to be able to fetch them ahead of evaluation time. If your expression depends on external state, it could very easily have an infinite number of potential outputs, in which case simply return `[undefined]`.

    possibleOutputs() {
        // Cop-out!
        return [undefined];
    }

Finally, implement the `serialize` method — this is basically the inverse of the `parse` method. The serialized result may not look identical to the original input that created an expression (because of changes like constant folding), but when evaluated it should give the same result (and in fact our test harness asserts that):

    serialize() {
        return ["foo", this.child.serialize()];
    }

You’re done! Although you should head straight over to `test/integration/expression-tests`, find an expression that’s similar to yours, copy its tests, and modify them to fit yours. An expression test has an:

- “expression”: of course
- “inputs”: array of Feature/GlobalProperties pairs
- “expected”
  - “compiled”: type information
  - “outputs”: evaluation results or errors
  - “serialized”: result of round-tripping the expression through parse/serialize

