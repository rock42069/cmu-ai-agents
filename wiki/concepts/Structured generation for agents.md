# Structured generation for agents

Agent outputs often cross a software boundary: a JSON function call is parsed,
SQL is executed, or code controls an environment. At that boundary, free-form
text is not enough. The output needs a machine-recognizable language.

## Three different correctness layers

1. **Syntactic validity:** Does the output conform to the grammar or schema?
2. **Semantic validity:** Do names, types, arguments, and values make sense in
   the current application state?
3. **Authority and safety:** Is this action allowed, appropriately scoped, and
   safe to execute now?

[[wiki/sources/XGrammar - Flexible and Efficient Structured Generation Engine for Large Language Models|XGrammar]]
solves the first layer with constrained decoding: before sampling each token, it
masks tokens that cannot continue a valid context-free grammar. Its adaptive
cache and parser optimizations make that guarantee inexpensive. It does not
solve the other two layers.

## Why prompting is insufficient

Prompts and examples can make valid output likely, and tool-use fine-tuning such
as [[wiki/sources/Toolformer - Language Models Can Teach Themselves to Use Tools|Toolformer]]
can teach the model when and how to call an API. But probabilistic generation
still permits malformed output. A grammar converts “usually valid” into a hard
syntactic invariant by setting invalid-token probability to zero.

## Relationship to code actions

[[wiki/sources/Executable Code Actions Elicit Better LLM Agents|CodeAct]] favors
Python because variables, loops, branches, and libraries make actions composable.
That flexibility increases the language and authority exposed to the model.
Grammar constraints can keep generated code within a chosen syntax or subset,
but safe execution still needs semantic validation, sandboxing, capability
restrictions, resource limits, and approval around consequential side effects.

## Durable takeaway

Structured generation is one layer in an agent runtime, not a substitute for a
runtime. Use constrained decoding to guarantee parseability; use validators to
check state-dependent meaning; use permissions and isolation to control effects.

## Sources

- [[wiki/sources/XGrammar - Flexible and Efficient Structured Generation Engine for Large Language Models|XGrammar: Flexible and Efficient Structured Generation Engine for Large Language Models]]
- [[wiki/sources/Executable Code Actions Elicit Better LLM Agents|Executable Code Actions Elicit Better LLM Agents]]
- [[wiki/sources/Toolformer - Language Models Can Teach Themselves to Use Tools|Toolformer: Language Models Can Teach Themselves to Use Tools]]
