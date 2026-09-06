# GwentPPCompiler

GwentPPCompiler is a C# domain-specific language (DSL) compiler for building **Gwent-like card game content**.

It parses a G++ program, evaluates it, and turns the resulting card definitions into runtime card objects through a user-provided `ICardFactory`.

## What this project does

At a high level, the compiler:

1. Reads a G++ source string.
2. Tokenizes and parses it into an AST.
3. Executes the parsed program in an evaluation context.
4. Collects the cards created by the program.
5. Converts those cards into your game’s own card objects via `ICardFactory`.

This means the repository is not the game itself, but the language and compiler layer for describing game cards and their effects.

## How to use it

The main entry point is:

```csharp
DSL.Compiler.Compile(string programString, ICardFactory cardFactory, Action<string> printFunction)
```

### Required inputs

- `programString`: the G++ source code to compile.
- `cardFactory`: your implementation of `ICardFactory`, used to build your game’s card objects.
- `printFunction`: a callback used by the DSL `print(...)` statement.

### Expected flow

```csharp
var cards = DSL.Compiler.Compile(
    programString,
    myCardFactory,
    Console.WriteLine
);
```

### Implementing `ICardFactory`

The compiler delegates card creation to your factory:

```csharp
public interface ICardFactory
{
    ICard CreateCard(string name, string faction, string type,
        IList<string> range, double power, IEffect effect);
}
```

So your application decides how DSL-defined cards map to your own game model.

### Typical integration steps

1. Reference the project from your game/tooling application.
2. Implement `ICardFactory` and the related runtime interfaces (`ICard`, `IEffect`).
3. Pass a G++ script into `Compiler.Compile(...)`.
4. Use the returned `IEnumerable<ICard>` in your game.

## Language overview

The DSL appears to support:

- card and effect declarations
- variables and assignments
- arithmetic and boolean expressions
- `if`, `while`, and `for` statements
- function-like actions and delegates
- lists, selectors, and type restrictions
- `print(...)`
- ternary expressions

## Architecture

The project is organized as a pipeline:

```mermaid
flowchart LR
    A[G++ source string] --> B[LexerStream]
    B --> C[ProgramParser]
    C --> D[AST]
    D --> E[Execute program]
    E --> F[Evaluation Context]
    F --> G[Card definitions]
    G --> H[ICardFactory]
    H --> I[Runtime ICard objects]
```

### Main components

#### 1. Lexer
Converts raw text into tokens.

- `Lexer/Token.cs`
- `Lexer/TokenType.cs`

#### 2. Parser
Builds the program structure from tokens.

- `Parser/ProgramParser.cs`
- `Parser/ExpressionParser.cs`
- `Parser/InstructionParsercs.cs`

#### 3. AST / Evaluator
Represents and executes the program.

- `Evaluator/AST/...`
- `Evaluator/LenguajeTypes/...`

#### 4. Compiler facade
A single public API that ties everything together.

- `Compiler.cs`

## Execution flow inside `Compile(...)`

```mermaid
sequenceDiagram
    participant User as Your App
    participant Compiler as DSL.Compiler
    participant Parser as ProgramParser
    participant Program as GwentProgram
    participant Factory as ICardFactory

    User->>Compiler: Compile(programString, cardFactory, printFunction)
    Compiler->>Parser: Parse source
    Parser-->>Compiler: AST / GwentProgram
    Compiler->>Program: Execute()
    Program-->>Compiler: Populated context
    Compiler->>Factory: CreateCard(...) for each card
    Factory-->>Compiler: ICard instances
    Compiler-->>User: IEnumerable<ICard>
```

## Notes

- The repository description says: `DSL made it for GWENT-like games.`
- The compiler relies on the host application to provide the concrete card implementation.
- `printFunction` is injected, so output handling stays outside the DSL runtime.

## Example usage

```csharp
using DSL;
using DSL.Interfaces;
using System;
using System.Collections.Generic;

public class MyCardFactory : ICardFactory
{
    public ICard CreateCard(string name, string faction, string type, IList<string> range, double power, IEffect effect)
    {
        // Map DSL data to your game card here.
        throw new NotImplementedException();
    }
}

var program = @"
card { }
";

var cards = Compiler.Compile(program, new MyCardFactory(), Console.WriteLine);
```

## License

No license file was found in the repository at the time of writing.
