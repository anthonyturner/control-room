# Design principles

Read this before you design or change any code. The goal is less complexity,
not just working code. Based on John Ousterhout's *A Philosophy of Software
Design*.

## 1. Program strategically, never tactically

This rule outranks the rest of this file.

- **Tactical** means the smallest diff that makes it work: a special case, a
  shortcut, a reach into another module's internals, a "clean it up later".
  Each one looks harmless. Hundreds of them make spaghetti, and no single fix
  can undo it. **Do not program this way.**
- **Strategic** means working code is the minimum, not the goal. The goal is a
  design that keeps the next change cheap. Spend a little more time now
  (roughly 10–20%) to get it.

On every change:

1. Aim for the code you would have written if you had built the system from
   scratch, knowing what you know now. Get as close as the change allows.
2. Leave the code you touch better than you found it. A change usually makes
   something a little worse, so you have to improve something just to break
   even.
3. Pick the clean interface over the fewest changed lines. Fear of breaking
   something is a reason to write a test, not to hack around the code.
4. Allow zero small kludges. Each one is how the slide into tactical
   programming starts.
5. Take small steps, not heroics. Design the interface before you code a new
   module, then improve it as you learn.

When the clean fix is bigger than the issue, do the clean part that is in
scope. File the rest as a follow-up issue and name it in the pull request.
Never leave a hack in silently.

## 2. Make modules deep

- A deep module has a simple interface over a lot of hidden work. Its
  interface is everything a caller must know: signatures, side effects,
  ordering and dependencies.
- Avoid shallow modules: pass-through methods and thin wrappers that cost more
  to learn than they hide.
- Avoid *classitis*: many tiny classes or layers that each add a sliver.
  Prefer fewer, meatier ones.
- Measure depth, not length. Never split a function just because it is long.
  Split it when the pieces are each simpler to understand.
- Make the common case trivial for the caller. Rare cases take options.
- A slightly general-purpose interface is often simpler and deeper than one
  shaped around its single caller.

### The words for a design

Describing a design the same way every time makes reviews shorter. These are
the words the playbook uses, and the architecture review in particular.

A **module** is anything you can use without reading how it works: a function,
a class, a package, or a feature that cuts through several layers. Its
**interface** is everything a caller has to know to use it correctly, which is
more than the signatures: also the order calls must come in, the errors that
can surface, and the state it assumes.

**Depth** compares those two sizes. `readConfig(path)` that finds the file,
parses it, applies defaults and reports a bad file in one clear way is deep:
one thing to learn, a lot done. A `ConfigFileOpener` that renames the
arguments of `fs.open` is shallow: the caller learns a new interface and still
does the work.

Depth pays off in two directions:

- **Leverage** is the caller's side. Each thing they learn about the interface
  buys a lot of behaviour.
- **Locality** is the maintainer's side. When the rules change, say how a retry
  backs off, there is one place to edit and one place the bug lived, not a copy
  at every call site.

A **seam** is a spot where what runs behind an interface can be swapped without
editing the code on either side. A seam earns its place only when two things
really occupy it, usually the production implementation and a test stand-in.
With a single occupant it is just another layer to read through.

To check whether a module is worth having, run the **deletion test** as a
thought experiment: remove it and watch where its work lands. If the callers
each end up carrying that work, the module was paying for itself. If nothing
lands anywhere and the callers simply get simpler, it only forwarded calls.

Write tests as a caller would use the module. When that turns out to be
impossible, read it as a design finding: either the interface leaves out
something callers need, or the module is cut in the wrong place.

These words describe the shape of a design. Prefer "module" to "component" or
"service", "interface" to "API", and "seam" to "boundary", because the
alternatives mean different things to different readers. Names that belong to
the project's stack stay as they are: an Angular component is still a
component, and an HTTP service is still a service.

### Dependencies across a seam

How you test a deepened module depends on what it leans on. Sort each
dependency into one of four kinds:

- **Pure code**, such as a rounding rule or a parser. Call it for real. It
  needs no seam, and a test through the module's interface covers it.
- **Stand-ins that run on your machine**, such as an in-memory SQLite database
  or a temporary directory. Test against the stand-in, and keep the choice of
  stand-in inside the module.
- **A service your team operates**, such as an internal billing API reached over
  a network. The module is handed it, and tests hand in a fake.
- **A service someone else operates**, such as a payment gateway. Same
  treatment as the one above, with the fake written from the provider's
  documented behaviour.

A seam used only inside a module's implementation is an **internal seam**. It
is not part of the interface, no caller or test depends on it, and it stays
that way. Do not expose one just because a test would find it convenient.

*The ideas in these two parts derive from an upstream skill; see [third-party-notices.md](third-party-notices.md).*

## 3. Define errors out of existence

- Prefer semantics in which the error cannot happen. Deleting something that
  is already gone succeeds. An out-of-range slice returns the overlap.
- Throw only when you truly cannot keep the contract, such as a failed read.
  Throwing more is not "more defensive".
- Handle each error in as few places as possible. Let it travel to where it
  can actually be dealt with, not be caught and re-thrown at every layer.
- Defining an error away is not ignoring it. Never swallow an error that
  matters.

## Red flags

If you see one of these in your diff, stop and redesign:

- a special-case `if` added to make one scenario work;
- a reach into another module's internals or global state to avoid
  designing an interface;
- a method that only forwards to another method;
- a function split up for length alone;
- a `catch` that discards the error;
- a "TODO: clean up later" with no follow-up issue.

## How this fits the rules

- Improve only the code your change touches. Leave unrelated files alone
  ([rule 7](rules.md)).
- "Deep" is not "more abstract". Never add a layer or interface that hides
  nothing ([rule 9](rules.md)).
- In review, a tactical shortcut is a finding, with the same weight as a bug.
