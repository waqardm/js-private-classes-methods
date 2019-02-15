# JavaScript Class Private Fields and Methods
Many of us will remember what it was like to work with object-oriented JavaScript before native support for classes. Working with `prototype` was often a less than optimal experience, and could confuse programmers not familiar with it.

## Classes
Fortunately classes were introduced in ECMAScript 2015 (ES6), allowing us to use them without the need to deal with the prototype ourselves. As you may know, classes can have constructor functions and methods:

```javascript
class Animal {

  constructor (name) {
    this.name = name;
  }

  whatsMyName () {
    return this.name;
  }
}

const cat = new Animal('Chaos');
console.log(cat.whatsMyName()); // "Chaos"
```

They also support inheritance:

```javascript
class Dog extends Animal {

  constructor (name) {
    super(name);
  }

  whatsMyName () {
    return `I am a dog and my name is ${this.name;}`;
  }
}

const dog = new Dog('Frederick');
console.log(dog.whatsMyName()); // "I am a dog and my name is Frederick"
```

If you've worked with other object-oriented languages, you'll agree classes are a great addition to the language - but as you start working with them you may find yourself missing something that's common in other languages: **private fields**. Until now JavaScript hasn't supported private fields on classes, but there’s a [Stage 3 TC39 proposal](https://tc39.github.io/proposal-class-fields/) to change this.

Before we dive in, let's have a quick refresher on public fields.

## Public fields
We can rewrite our `Animal` example above to use a public field. By declaring a public field we can ensure the field is always present, and the class definition is more self-documenting.

You can choose to provide a default value or leave the field undefined.

```javascript
class Animal {
  name = 'Fred';
  species; // undefined
}
```

Public fields are added either at construction time in the base class, or after super() returns in a subclass.

## Private fields: # is the new _
Now let's take a look at the syntax of private fields. You may have seen code in the past that used `_` as a prefix to indicate that a property or method was private.

Well, when you want to prevent others from accessing a field outside of your class, **# is the new _**. To declare a private field we use a # name (said "hash name"). Check out this example:

```javascript
class Marker {

  #lat;
  #lng;

  constructor (lat, lng) {
    this.#lat = lat;
    this.#lng = lng;
  }
}
```

Here we’re creating a `Marker` class with two private fields: `lat` and `lng`. See what happens if we try and access them out of scope:

```javascript
const newMarker = new Marker(38.7652793, -9.0958296);
newMarker.#lat; // Syntax error
```

This code is invalid and will throw an error. You can only reference the private fields of a class _within the class that defines them_.

### Differences from public fields
While there are obvious similarities in syntax between public and private fields, it's important to note the differences. Unlike public fields, private fields are _not properties_. Instead, they make use of scoping to restrict access.

The `#` in a # name forms part of the name itself, and must be used when both declaring and accessing the field. Because public fields can't begin with a `#` and private fields must, the two will never conflict.

### Usage
Let's look at some examples.

```javascript
class Employee {

  constructor () {
    this.nonExistentPublicField = 24;
    this.#nonExistentPrivateField = 42; // Syntax error
  }
}
```

While there are good reasons to declare public fields on the class, it's not strictly necessary. Private fields, on the other hand, make use of lexical scoping and so _must_ be declared. Setting `#nonExistentPrivateField` fails above because it is a syntax error to refer to # names not in scope.

For this same reason, it's a syntax error to have multiple same-named # names in the same scope:

```javascript
class Employee {
  #identifier;
  #identifier; // Syntax error
}
```

As mentioned above, because private field # names must begin with a `#`, you can use otherwise identical names for private and public variables with no issue:

```javascript
class Employee {
  #identifier; // private identifier
  identifier;  // public identifier
}
```

### Initialization

Fields without initializers are initialized to undefined. As with public fields, initializers are run in declaration order. Usefully, this is the object being constructed, so you can make use of it:

```javascript
class Employee {

  #identifier;
  #status = 'Employed';
  #name = this.generateName();

  constructor () {
    assert(this.#identifier === undefined);
    assert(this.#status === 'Employed');
    assert(this.#name === 'Su Chin');
  }

  generateName () {
    return 'Su Chin';
  }
}
```

### Why `#`?
If you have already worked with other languages that support private fields in classes, you may be used to using the `private` keyword. That works because access to public and private fields are the same. In JavaScript we can’t use `this.field` to access private fields as this would create unexpected behavior in our code and lead to performance issues. Read more about the decision making process [here](https://github.com/tc39/proposal-class-fields/blob/master/PRIVATE_SYNTAX_FAQ.md).

### Why do we need private fields at all?
If you're not yet convinced that JavaScript needs private fields, let's walk through an example where they're useful: creating a library. When you create a library, you want to provide your users with a clean and stable API. That means that you don’t want them to access things they aren’t supposed to. This allows you to change how things work under the hood, so long as the API remains functional.

Until now JavaScript has relied on convention to stop people accessing and thus relying on implementation details for their own code to work. Private fields change this.

Imagine you’re building a library that provides information on Pokémon. The code will look something like this (irrelevant details omitted):

```javascript
class PokemonFetcher {

  pokemon = [];

  async getPokemon () {
    if (this.pokemon.length === 0) {
      this.pokemon = await this.fetchPokemon();
    }
    return this.formatResult();
  }

  async fetchPokemon () {
    const pokemon = await fetch(`https://myapi.com/api/v1/pokemon`);
    return pokemon.json();
  }

  formatResult () {
    return this.pokemon.map(pokemon => {
      // ...format the list
    });
  }
```

We publish our library and document that Pokémon information must be fetched using the `getPokemon` method. Unfortunately, despite this - we can't actually stop the user from getting Pokémon in other, unsupported ways. There are in fact four ways they could access them:

- Using the `getPokemon` method, the one that we recommend;
- Using the `fetchPokemon` method, (which will trigger a fetch every time, making unnecessary requests and returning the results unformatted);
- Using the `formatResult` method (which won't work if the `pokemon` property is empty, it won’t work);
- Accessing the `pokemon` property directly (which can return an empty array or an unformatted result);

As you can see - even leaving just a few private methods and fields accessible can leave you open to several avenues of abuse. To prevent this, we need to be able to lock down our API such that consumers can only access the methods we want them to use. We can achieve this with private fields, and private methods, which we'll talk about next.

## Private Methods

Private methods are similar to their public counterparts, but have their access restricted in the same manner as private fields, also using a `#` prefix.

Private getters and setters are possible, as well as private generator, async and async generator instance methods.

Let's update our code above to restrict our API:

```javascript
class PokemonFetcher {

  #pokemon = [];

  async getPokemon () {
    if (this.#pokemon.length === 0) {
      this.#pokemon = await this.#fetchPokemon();
    }
    return this.#formatResult();
  }

  async #fetchPokemon () {
    const pokemon = await fetch(`https://myapi.com/api/v1/pokemon`);
    return pokemon.json();
  }

  #formatResult () {
    return this.#pokemon.map(pokemon => {
      // Formats the list to the desired structure
    });
  }
```

In the code above, the only way the user can get Pokémon information is by using the only public method: `getPokemon`. This way, we are free to change the implementation of our library, so long as the `getPokemon` method returns the same ultimate response.

## Conclusion

Private fields and methods are useful when working with object-oriented code, and bringing them to JavaScript enables engineers to write code and create apps or libraries in a much cleaner way. We hope this article has helped you to understand how private methods and fields work in JavaScript and the importance of having support for them in the language.

## Acknowledgements

For a detailed look at class fields and methods read [this fantastic blog post](https://rfrn.org/~shu/2018/05/02/the-semantics-of-all-js-class-elements.html) by Shu-yu Guo.
