# GraphQL for Delphi

[![License](https://img.shields.io/badge/License-Apache%202.0-yellowgreen.svg)](https://opensource.org/licenses/Apache-2.0)

Simple implementation for GraphQL, a query language for APIs created by Facebook.

GraphQL is a query language for your API and a server-side runtime for executing queries using a type system you define for your data. GraphQL isn't tied to any specific database or storage engine and is instead backed by your existing code and data.

See more complete documentation at https://graphql.org/.

## Features

* See the GraphQL structure
* Use simple API (RegisterFunction)
* Use more complex API (RegisterResolver) 

### GraphQL tree navigation

With this release of *GraphQL for Delphi* you can explore the GraphQL query and call your API. 

With a code like this you can build the GraphQL tree:

```pascal
  LScanner := TScanner.CreateFromString(SourceMemo.Text);
  try
    LBuilder := TGraphQLBuilder.Create(LScanner);
    try
      // This will create the tree
      LGraphQL := LBuilder.Build;
    finally
      LBuilder.Free;
    end;
  finally
    LScanner.Free;
  end;
```

Then you will have a struture like this:

```
IGraphQL
├── Name
└── Fields
    ├── IGraphQLField (first entity)
    │   ├── Name
    │   ├── Alias
    │   ├── Arguments / Parameters
    │   │   ├─ IGraphQLArgument 
    │   │   └─ IGraphQLArgument 
    │   │
    │   └── IGraphQLValue (IGraphQLNull | IGraphQLObject)
    │       └─ Fields
    │          ├─ ...
    │          ├─ ...
    │
    └── IGraphQLField (second entity)
        ├── Name
        ├── Alias
        ├── ...
```

You can see the demo to have an idea of the capabilities of this library.

![](https://raw.githubusercontent.com/wiki/lminuti/graphql/demo1.png)

### Query your API with GraphQL

First of all you need an `API` to query. At this moment *GraphQL for Delphi* supports `class` or simple `procedure and function`. In either case you have to tell the library how to call your API.

#### Basic API

If you have a simple API made of classic fuctions like this:

```pascal
function RollDice(NumDices, NumSides: Integer): Integer;

function ReverseString(const Value: string): string;

function StarWarsHero(const Id: string): TStarWarsHero;
```

Then you need to register your API in this way:

```pascal
  FRttiQuery := TGraphQLRttiQuery.Create;

  FRttiQuery.RegisterFunction('rollDice',
    function (AParams: TGraphQLParams) :TValue
    begin
      Result := RollDice(AParams.Get('numDice').AsInteger, AParams.Get('numSides').AsInteger);
    end
  );

  FRttiQuery.RegisterFunction('reverseString',
    function (AParams: TGraphQLParams) :TValue
    begin
      Result := ReverseString(AParams.Get('value').AsString);
    end
  );

  FRttiQuery.RegisterFunction('hero',
    function (AParams: TGraphQLParams) :TValue
    begin
      Result := StarWarsHero(AParams.Get('id').AsString);
    end
  );
```

Eventually you can query your API: 

```pascal

json := FRttiQuery.Run(MyQuery);

```

#### Run methods from a class using RTTI

If you have a class you need to tell the library:

* how to create che instance;
* if the class is a *singleton* (or if the library should create an new instance for every method call);
* which methods GraphQL should query.

For example if you have a class like this:

```pascal
  TTestApi = class(TObject)
  private
    FCounter: Integer;
  public
    [GraphQLEntity]
    function Sum(a, b: Integer): Integer;

    [GraphQLEntity('mainHero')]
    function MainHero: TStarWarsHero;

  end;
```

You need to add the `GraphQLEntity` to every method queryable by GraphQL and register the class:

```pascal
  FRttiQuery := TGraphQLRttiQuery.Create;
  FRttiQuery.RegisterResolver(TGraphQLRttiResolver.Create(TTestApi, True));
```

The `RegisterResolver` method can add a resolver (any class that implements `IGraphQLResolver`) to the GraphQL engine. A resolver is a simple object that explain to GraphQL how to get the data from the API. You can build your own resolvers or use the resolvers build-in with the library.

The `TGraphQLRttiResolver` is capable of run method from a class using the [RTTI](https://docwiki.embarcadero.com/RADStudio/Sydney/en/Working_with_RTTI).

Then you can query your API: 

```pascal

json := FRttiQuery.Run(MyQuery);

```

A simple query:

![](https://raw.githubusercontent.com/wiki/lminuti/graphql/GraphQL-Basic.gif)

How to use GraphQL aliases:

![](https://raw.githubusercontent.com/wiki/lminuti/graphql/GraphQL-Alias.gif)

How to call simple functions:

![](https://raw.githubusercontent.com/wiki/lminuti/graphql/GraphQL-RollDice.gif)

A more complex example:

![](https://raw.githubusercontent.com/wiki/lminuti/graphql/GraphQL-complex.gif)