# JavaScript Class Private Fields and Methods

Many of us will remember what it was like to work with object oriented JavaScript before ES6 introduced support for Classes. Working with **prototype** wasn't an optimal experience, and could lead to confusion for programmers not familiar with it.

## Classes
Fortunately for us, **Classes** were introduced in **ECMAScript 2015 (ES6)**. Now we can work with classes without the need to deal with the prototype ourselves.

They support methods (what we will refer to later as public methods), and more recently, public fields.

```javascript
class Animal {

  name; // public field

  constructor (name) {
    this.name = name;
  }

  whatsMyName () { // public method
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

If you've worked with other Object-Oriented languages, you'll agree that this is a great addition to the language. But as you start working more with Classes in JavaScript you will find yourself missing something that’s common in other languages: **private fields**. Until now JavaScript hasn't supported private fields on classes, but there’s a **Stage 3 TC39 proposal** to change this. Before we talk about these, a quick refresher on public fields.

## Public fields

`name` in the `Animal` example above shows a public field in use. You can also initialise it in situ to give it a default value.

```javascript
class Animal {
  name = 'Fred';
}
```

Public fields are added either at construction time in the base class, or after super() returns in a subclass.

## Private fields: # is the new _

So, let’s take a look at the syntax of private fields. You may have seen code in the past that used `_` as a prefix to indicate that a property or method was private. Well **# is the new _**. To declare that we want a field in our class to be private, we use a # name (said "hash name"). Check out this example:

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

Here we’re creating a Marker class with two private fields: `lat` and `lng`. See what happens if we try and access them out of scope:

```javascript
const newMarker = new Marker(38.7652793, -9.0958296);
newMarker.#lat; // Syntax error
```

This code is **invalid** and will throw an error. You can only reference the private fields of a class within the class that defines them.

### Differences from public fields

While there are obvious similarities in syntax between public and private fields, it's important to note the differences. Unlike public fields, private fields are _not properties_; they make use of lexical scoping. This has important ramifications that we'll discuss below.

### Usage

Let's look at some examples.

```javascript
class Employee {

  constructor () {
    this.#nonExistantPrivateField = 42; // Syntax error
    this.nonExistantPublicField = 24;
  }
}
```

The above may seem unintuative at first, but if you remember that public fields are properties but private fields are not, it makes sense. Private fields make use of lexical scoping, so it is a syntax error to refer to # names not in scope.

For this same reason, it's a syntax error to have multiple same-named # names in the same scope:

```javascript
class Employee {
  #identifier;
  #identifier; // Syntax error
}
```

You can have public and private fields with the same name - because public fields cannot begin with `#` and private fields must - the two can never conflict.

```javascript
class Employee {
  #identifier; // private identifier
  identifier;  // public identifier
}
```

### Initialisation

Fields without initializers are initialized to undefined. As with public fields, initialisers are run in declaration order. Usefully, this is the object being constructed, so you can make use of it:

```javascript
class Employee {
  #identifier;
  #status = 'Employeed';
  #name = this.generateName();

  constructor () {
    assert(this.#identifier === undefined);
    assert(this.#status === 'Employeed');
    assert(this.#name === 'Su Chin');
  }

  generateName () {
    return 'Su Chin';
  }
}
```

### Why #?
If you have already worked with other languages that support private fields in classes, you may be used to using the `private` keyword. That works because access to public and private fields are the same. In JavaScript we can’t use `this.field` to access private fields as this would create unexpected behaviour in our code and lead to performance issues.

### Why do we need private fields at all?
If you're not yet convinced that JavaScript needs private fields at all, let's walk through an example where they're useful: creating a library. When you create a library, you want to provide your users with a clean and stable API. That means that you don’t want them to access things they aren’t supposed to. This allows you to change how things work under the hood, so long as the API remains functional.

Unfortunately, until now JavaScript has relied on convention to stop people accessing and thus relying on implementation details for their own code to work. Private fields change this.

Imagine you’re building a library that provides information on Pokémon. The code will look something like this (ignoring irrelevant details):

```javascript
class PokemonFetcher {

  pokemons = [];

  async getPokemons () {
    if (this.pokemons.length === 0) {
      this.pokemons = await this.fetchPokemons();
    }
    return this.formatResult();
  }

  async fetchPokemons () {
    const pokemons = await fetch(`https://myapi.com/api/v1/pokemons`);
    return pokemons.json();
  }

  formatResult () {
    return this.pokemons.map(pokemon => {
      // ...format the list
    });
  }
```

We publish our library and document that Pokémon info must be fetched using the `getPokemons` method. Unfortunately, despite this - we can't stop the user from getting Pokémon in other, unsupported ways. In fact there are four ways they could access them:

- Using the `getPokemons` method, the one that we recommend;
- Using the `fetchPokemons` method, (which will trigger a fetch every time, making unnecessary requests and returning the results unformatted);
- Using the `formatResult` method (which won't work if the `pokemons` property is empty, it won’t work);
- Accessing the `pokemons` property directly (which can return an empty array or an unformatted result);

As you can see - even leaving just a few private methods and fields accessible can leave you open to several avenues of abuse. To prevent this, we need to be able to lock down our API such that consumers can only access the methods we want them to use. We can achieve this with private fields, and private methods, which we'll talk about next.

## Private Methods

Private methods are similar to their public counterparts, but have their access restricted in the same manner as private fields, also using a `#` prefix.

Like private instance fields, they are added at construction time for the base classes and after super() returns for subclasses, and generator function, async function, and async generator function forms may also be private instance methods.

Let's update our code above to restrict our API:

```javascript
class PokemonFetcher {

  #pokemons = [];

  async getPokemons () {
    if (this.#pokemons.length === 0) {
      this.#pokemons = await this.#fetchPokemons();
    }
    return this.#formatResult();
  }

  async #fetchPokemons () {
    const pokemons = await fetch(`https://myapi.com/api/v1/pokemons`);
    return pokemons.json();
  }

  #formatResult () {
    return this.#pokemons.map(pokemon => {
      // Formats the list to the desired structure
    });
  }
```

Now, with the code above, the only source that the user can get the Pokémon info is by using the only public method: `getPokemons`. This way, we can totally change how our library works under the hood, as long as the `getPokemons` method keeps its behaviour.

## Conclusion

Private fields and methods are useful when working with Object-Oriented code, and bringing them to JavaScript enables engineers to write code and create apps or libraries in a much cleaner way. We hope this article has helped you to understand how private methods and fields work in JavaScript and the importance of having support for them in the language.

## Acknowledgements

For a detailed look at class fields and methods, read [this fantastic blog post](https://rfrn.org/~shu/2018/05/02/the-semantics-of-all-js-class-elements.html) by Shu-Yo Guo.
