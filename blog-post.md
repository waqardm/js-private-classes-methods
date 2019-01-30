# JavaScript Class Private Fields and Methods

We all remember how we used to work on JavaScript before the support for classes. Working with **prototype** wasn’t a good experience and could lead to many issues, also if someone not familiarized with it had to work with the code it could be a nightmare.

### Classes
Fortunately for us, **Classes** were introduced on **ECMAScript 2015 (the worldwide famous ES6)**. Now we can work with classes without the messy code of dealing with the prototype.

```javascript
class Animal {
  constructor (name) {
    this.name = name;
  }
}
```
And we can even work with Class Inheritance:
```javascript
class Dog extends Animal {
  constructor (name) {
    super(name);
  }
}
```

That’s really great and if you already worked with other **Object-Oriented Languages**, this for sure brings JavaScript up to the game.

But when you start working more with Classes on JavaScript you will miss something that’s really common in other languages: Private Fields. Until now JavaScript doesn’t support **private fields** on classes, but there’s a **Stage 3 proposal on TC39** to add this support and that’s what I’ll talk about.

### A little about private fields on JavaScript
So let’s start checking the syntax on how private fields will work on JavaScript. To declare any property or method inside a class as private we only need to prepend the name of it with `#`. Check the example below:
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
Here we’re creating a Marker class with two private fields: `lat` and `lng` , that means that the code below is **invalid**:
```javascript
const newMarker = new Marker(38.7652793, -9.0958296);
newMarker.#lat; // INVALID
```
You can only reference the private fields of a class within the class that defines them. The private fields are undetectable outside of this scope:
```javascript
class Employee {
  #identifier; // PRIVATE identifier
  identifier; // PUBLIC identifier
}
```
You'll notice to fields, `#identifier` and `identifier` and though on first glance they may seem like they are a private and public field with the same name, it is suggested that the `#` prepended to the private field be read as one of the characters of the name i.e. `private identifier` and `identifier`. 

This also works for subclasses, that means that subclasses:
```javascript
class Employee {
  #identifier;
  ...
}
class Manager extends Employee {
  identifier;
  ...
}
```
Another thing to note is that this proposal allows access to private fields of other instances of the same class and that’s really useful for example when you need to create an `equals` function:
```javascript
class Employee {
  #identifier;
  ...
  equals (other) {
    return this.#identifier === other.#identifier;
  }
}
```
On the code above, we assume that `other` is an `instanceof` the class Employee.

### Why use # to declare private fields?
If you have already worked with other languages that support private fields on classes you must be used to the private keyword. That’s OK for these languages because the access to public and **private** fields are the same. On JavaScript, we can’t use `this.field` to access private fields as this would create unexpected behaviour on our code and lead to many performance issues.

### Benefits of having private fields
So, if you’re not convinced yet about why having private fields on JavaScript classes, let’s take a look when it comes to creating libraries. When you create a library, you want to provide your users with a clean and stable API. That means that you don’t want them to access things they aren’t supposed to and that you can change the work behind the curtain as long as the API remains functional. Private fields are very useful for this, let’s check a use case for that. 

Imagine that you’re building a library that provides Pokémon info and the code will be something like this (I’ll focus on a tiny part of what an actual library would look like, just to point what’s needed):
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
      // Formats the list to the desired structure
    });
  }
```
We publish our library and put on the docs that the Pokémon info needs to be fetched using the `getPokemons` method. Taking a look at it, all seems ok, but we do have some issues. Even though the docs instruct that the users need to use the `getPokemons` method, there’s no way that we can force the user to do so, the user can try to get the Pokémons info in four different ways from our library:

- Using the `getPokemons` method, the one that we recommend;
- Using the `fetchPokemons` method, but then it would fetch the API every time, making unnecessary requests and returning the results on unformatted;
- Using the `formatResult` method, but if the `pokemons` property is empty, it won’t work;
- Accessing the `pokemons` property directly, but it can return an empty array or an unformatted result;

So as you can see even a small part of a library can cause many issues. And things can get even worse, imagine that some users are using the library by getting the info using the `fetchPokemons` method and for some reason we had to change the API and the results now have a different format. It would break the apps. To fix that, we need it so the users can have only a single source of truth and we can achieve that by using private fields and private methods.

### Private Methods
Private methods essentially work the same as private fields and are prepended with the `#` symbol. Let's take a look at our updated code:
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
Now, with the code above, the only source that the user can get the Pokémons info is by using the only public method: `getPokemons` . That way, we can totally change how our library works as long as the `getPokemons` method keeps its behaviour.

### `#` is the New `_`
For those used to utilising the `_` to declare private fields and methods, the `#` is essentially identical with one key difference best shown in the example below.

#### Using the `_`
```javascript
class Marker {
  constructor (lat, lng) {
    this._lat = lat;
    this._lng = lng;
  }
}
const newMarker = new Marker(38.7652793, -9.0958296);
newMarker._lat; // accessible from outside of the class
```

#### Using the `#`
```javascript
class Marker {
  #lat;
  #lng;
  constructor (lat, lng) {
    this.#lat = lat;
    this.#lng = lng; // not accessible from outside of the class
  }
}
```
### Conclusion
Private fields and methods are a basic feature when working with Object-Oriented code, bringing this to the JavaScript language is taking a step forward on helping us engineers to write code, creating apps or libraries, in a much cleaner way. We hope that this article is helpful in understanding how the private fields and methods work in JavaScript and the importance of having this support on the language.
