---
title: Despaghettify your Javascript with Trait Mixins
kind: article
created_at: 2014-08-04
category: article
---

As an exercise in lerning to manage code complexity, 
one of my recent weekend projects has been to write
a simple node.js multiplayer game from the ground up. Shortly after banging out a prototype
it became obvious that my methods would not scale, and if I did not take a step back I 
would end up with either an unmanageable mess of OO code, or
dumping all of my game logic in 
some horrid giant [god object](http://gamedev.stackexchange.com/q/79261/49827).

## The Problem I Was Trying to Solve

If you've never tried to write a game, let me present a common scenario:

Suppose a simple game.
You have some dude that is being controlled by the user, a world with some coordinates,
and a bunch of objects with which the dude must interact. Naively one might start by instantiating
a `Player` object, with some methods on it (say, `player.moveLeft()`, `player.shoot()`),
and some other similar objects with their own methods (`Bullet`, `Cart`, you get the idea).

Now we want the dude to move left. But wait, it seems there already is an object in that position. 
A cart! (Why it's none other than Cekhov's cart from just a paragraph ago.) 
So what do we do? Well, maybe the cart moves when the player pushes on it.

Carts, dudes... dudes pushing carts. This game really is shaping up quite nicely. I
can already tell it's going to be a smash hit.

![](https://s3.amazonaws.com/clusterfoo.img/2014/cartsofthunder.png)

Great, so let's get coding.

Oops. Instantly we have a chicken and egg problem: we want the cart to somehow know that a 
dude wants to move into its position, and the cart must react accordingly.
And of course
the dude loses stamina when he pushes on the cart. So now the dude object too must somehow know
that he is interacting with a cart object -- but wait, what if the dude did not have enough
stamina to move the cart in the first place? Then we should have somehow checked for this
condition before we even started the whole chain of events ... extrapolate this to dozens of objects interacting on
any given game world at any given point in time,
and you see how quickly the situation can become quite messy. (At least 
for a newb like me who has no idea what he's doing.)

After banging my head for a bit on how to deal with this complexity in an elegant way,
I stumbled upon the wonderful world of [data driven design](http://gamedev.stackexchange.com/q/59638/49827)
and [component systems](http://www.raywenderlich.com/24878/introduction-to-component-based-architecture-in-games).

This got me thinking:
what would be an elegant way to implement more modular application logic, and to separate data from behavior? 

Wouldn't it be cool to implement simple, predictable data objects with common APIs? It would be nice to
know that all my "moveable" objects behave in some predictable way, as do all my "killable", or "draggable" objects.
Then game logic would simply be a matter of processing data structures in a linear, functional manner.

## The Solution: Traits, Dummy!

I really like Rust's system of [traits](http://doc.rust-lang.org/rust.html#traits) and [implementations](http://doc.rust-lang.org/rust.html#implementations)
and wanted something similar.

Features like these can be added to javascript with preprocessors like [typescript](http://www.typescriptlang.org/),
but I was looking for a solution that could be worked into vanilla javascript. Luckily, javascript's dynamic simplicity and
prototype system lends itself very well to such metaprogramming.

Granted, the following implementation of traits is a bit of a bastardization of the strict type/traits found in 
languages like Rust, but with some discipline I found it very helpful for managing larger javascript code bases.

The basic idea is to make constructors "traitable":

    /**
     * Traitable constructors all respond to a common API.
     * For example, `foo.registerType()`.
     */
    function makeTraitable(_this, _constructor) {
        _this.traits = [];
        
        _constructor.prototype.is = function (trait) {
            if (_this.traits.indexOf(trait) !== -1) return true;
            return false;
        };
        
        _constructor.prototype._registerTrait = function(trait) {
            _this.traits.push(trait);
        };
        
        _constructor.prototype._addValue = function (key, value) {
            _this[key] = value;
        };
    }
{: class="language-js" }

Now making a constructor traitable is just a matter of calling the mixin:

    function Dude() {
      makeTraitable(this, Dude);
    }
{: class="language-js" }

Now we can write our first trait:

    function Moveable(_constructor, initialX, initialY) {
        _constructor.prototype._registerTrait("Moveable");
        
        _constructor.prototype._addValue("posX", initialX);
        _constructor.prototype._addValue("posY", initialY);
        
        _constructor.prototype.moveLeft = function () {
            this.posX -= 1;
        };
    }; 
{: class="language-js" }

And that's all there is to it!

    /**
     * Our new dude will respond to the same predictable
     * API as any other Moveable objects.
     */
    function Dude(initialX, initialY) {
        makeTraitable(this, Dude);
        Moveable(Dude, initialX, initialY);
        Controllable();
        Killable();
    };

    /**
     * Inventing new game objects with different behavior is a
     * simple matter of trying mixin combinations.
     */
    function Cart(initialX, initialY) {
      makeTraitable(this, Cart);
      Moveable(Cart, initialX, initialY);
      Pushable();
    }
{: class="language-js" }

Usage:
     
    ted = new Dude(0,0);
     
    ted.is("Moveable");
    //-> true
     
    ted.is("Pushable");
    //-> false

    ted.posX
    //-> 0
     
    ted.moveLeft();
    ted.posX;
    //-> -1
{: class="language-js" }

I've found it really pleasant to try different things this way. For example,
one might imagine registering all moveable objects into some array, and a second array
of all directions in which they're expected to move:

    _.zipWith(move, moevables, directions);

    function move(objectId, direction) {
      World[objectId].move(direction);
    }
{: class="language-js" }

I can confidently register new objects, mix or remove traits, and not fear the
application will break.

Now I don't know how well this follows the principles
that data driven design proponents recommend, but at least for my current
needs it's been a step forward.
