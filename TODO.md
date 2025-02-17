# Fable.Plugin.Tracer

This is a helper module made to assist in Fable Plugin development experience.

It's a single file that you can include in any Plugin project that doesn't use any dependency not included in the Fable build process.

It's an opinionated tracer that can be as detailed or bare as you want.

## Motivation

I was trying to grok the Oxpecker.Solid.FablePlugin because it was causing me issues.
The AST was a headache to look at, and I couldn't trace where things were going wrong, and I wasn't the author so I was out of my depth.

I created this to clearly identify what the flow of transformation was, while outputting a more clear and easier to work with AST form.

## Demonstration

In the example of the Oxpecker.Solid.FablePlugin, there are some elements of the AST transformation that were collated into 'Tags' (JSX components). The pattern matchers were slightly adjusted to expect a container containing the expected value

```fsharp
let transform (expr: Expr) =
    match expr with
    | Let(name, value, expr) ->
        Let(name, value, transform expr)
    | _ -> ...
```

Simple adjustment to include the tracer

```fsharp
let transform (tracer: Expr Tracer) =
    match tracer.Value with
    | Let(name, value, expr) ->
        ...
    | _ -> ...
```

The tracer is a simple object that has a GUID to allow related transformations or expressions to be identified together.

In the previous case, the transform would no longer compile against the `expr` in the pattern `Let(name, value, expr)` since it is of the type `Expr` rather than `Expr Tracer` (`Tracer<Expr>`).

I can either create a new tracer of the expr with `Tracer.create expr`, or I can bind the expr to the current Tracers GUID with `Tracer.bind`

```fsharp
let transform (tracer: Expr Tracer) =
    match tracer.Value with
    | Let(name, value, expr) ->
        Let(name, value, expr |> Tracer.bind tracer |> transform)
    | _ -> ...
```

The idea is to try and identify the significant elements of the plugin/transformations that you'd also want to view the AST of, and make those parts of the code use types wrapped in Tracers.

Whenever you want to trace the 