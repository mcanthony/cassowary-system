# cassowary-system [UNMAINTAINED]

**cassowary-system** is an API wrapper over the awesome [Cassowary.js](https://github.com/slightlyoff/cassowary.js) library (itself a JavaScript port of the Cassowary constraint solver). The intent of this wrapper to make creating constraint systems with Cassowary.js a bit simpler to declare and manage. **This project is very much a proof of concept; YMMV!**

If you're unfamiliar with constraint-based programming, the Cassowary constraint solver, or Cassowary.js, first see [Adam Solove's presentation on constraint programming](http://www.youtube.com/watch?v=72sWgwaAoyk). If you do know what those things are, but don't yet see the purpose of this project, please keep reading:


## Background / example

Using good, old-fashioned [Cassowary.js](https://github.com/slightlyoff/cassowary.js), you would declare and solve a system of constraints like this:

    require('cassowary'); // Creates a global `c` object.

    // Create the simplex solver
    var solver = new c.SimplexSolver();

    // Declare variables
    var stark = new c.Variable({ value: 4 }),
        lannister = new c.Variable({ value: 6 }),
        tully = new c.Variable({ value: 2 }),
        frey = new c.Variable({ value: 3 });

    // Declare expressions
    var goodies = c.plus(stark, tully),
        baddies = c.plus(lannister, frey);

    // Declare constraints
    var wedding = new c.Inequality(baddies, c.GEQ, goodies);

    // Add constraints to the solver
    solver.add(wedding);

    // Suggest values and resolve
    c.addEditVar(stark);
    c.suggestValue(stark, 10);
    c.resolve();

    // Get a resolved value
    var starksRemaining = stark.value;

That's actually pretty dang simple, already. Still, I wanted to see if I could boil the declarations down even further into a single configuration object. This is where **cassowary-system** steps in. Here is an example of the way you would declare the same constraint system (as shown in the above example), plus some:

    // Create a constraint system
    var constraintSystem = new CassowarySystem({
      variables: {
        stark: 4,
        lannister: 6,
        frey: function(){ return towers; }, // Ever-changing
        tully: function(){ return fish; } // Ever-changing
      },
      expressions: {
        goodies: ['stark', '+', 'tully'],
        baddies: ['lannister', '-', 'frey']
      },
      constraints: [
        ['baddies', '>=', 'goodies', 'required']
      ]
    });

    // Get a resolved value
    var starksRemaining = system.variables.stark.value;

On purely aesthetic grounds, I myself happen to prefer this. (You might, too.) But beyond simply offering a cosmetic wrapper API, CassowarySystem also transparently does a few useful things for you:

**Management of Cassowary objects**: Each CassowarySystem instance keeps track of its own internal solver, experessions, variables, etc., so you don't have to. And if you _do_ want to, accessing them is easy, e.g.: `system.expressions`, etc.

**Ongoing recalculation:** By default, a CassowarySystem instance will resolve its constraints continuously. Meaning that, as variables change, you needn't call `resolve()` yourself. Just access the object property you want (e.g. `system.variables.starks.value`) to get the current computed value. _(Continuous recalculation is optional; see below.)_

**"Reactive" variables:** Variables whose values are declared as functions will update continuously as time passes. This is nifty when you want your constraint system to work with continuously updating input values and such.

**Flexibility when instantiating a constraint system:** When initializing a new CassowarySystem instance, you needn't use the shorthand API shown above. You can pass in a "mixed grill", so to speak, of number values and plain-old Cassowary.js objects. This could allow for, say, different solver systems to cross-reference each other. E.g.:


    new CassowarySystem({
      variables: {
        tyrell: 1,
        clegane: new c.Variable({ value: 2 }),
        baratheon: otherSystem.variables.baratheon
      }
      // etc.
    });


## Options

The CassowarySystem constructor takes two arguments: a `spec` object that defines the objects in the system, and an `options` object for, you guessed it, options. For example:

    new CassowarySystem({
      // Variables, expressions, constraints
    },{
      autoSetup: true,
      autoAddEditVars: true,
      autoAddConstraints: true,
      autoTriggerReactiveVariables: true
    });

* `autoSetup`: Default: `true`. Automatically and immediately process the spec object, building the internal system of Cassowary objects.

* `autoAddEditVars`: Default: `true`. Assume _all_ variables declared in the `variables` object are "edit variables" liable to be changed over the course of time.

* `autoAddConstraints`: Default: `true`. Add the declared constraints to the solver right away. Note: This may have the effect of changing the variables' values immediately.

* `autoTriggerReactiveVariables`: Default: `true`. Kick off a `requestAnimationFrame` loop that will continuously (a) call any functions that were supplied as members of the `variables` object, and (b) tell the solver to recompute the values on an ongoing basis.

## Development

You will need npm and Bower.

After cloning the repo, install the local dependencies:

    bower install

## Tests

None currently. This just is a proof of concept, at the moment.


## Contributing

Please fork and submit pull requests. I'm also happy to grant commit rights to anyone who has made a couple of substantial contributions.


## Author

[Matthew Trost](http://trost.co)


## License

The MIT License (MIT)

Copyright (c) 2014 Matthew Trost

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in
all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
THE SOFTWARE.
