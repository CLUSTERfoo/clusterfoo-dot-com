---
title: A Simple Implementation of Trait Mixins for Javascript
kind: article
created_at: 2014-08-04
category: article
---

I wanted to learn more about hot to deal with code complexity, and figured writing a simple
multiplayer game engine would be the perfect exercise. Shortly after banging out a prototype
it became obvious that my methods would not scale, and if I did not take a step back I 
would end up with either an unmanageable mess of OO code, or
dumping all of my game logic in 
some horrid giant [god object](http://gamedev.stackexchange.com/q/79261/49827).

After banging my head for a bit,
I stumbled upon the wonderful world of [data driven design](http://gamedev.stackexchange.com/q/59638/49827)
and [component systems](http://www.raywenderlich.com/24878/introduction-to-component-based-architecture-in-games).

This insipired me to start thinking about how to apply more modular logic to my own
applications.

I really like Rust's system of [traits](http://doc.rust-lang.org/rust.html#traits) and [implementations](http://doc.rust-lang.org/rust.html#implementations)
and wanted something similar.

Granted, the following implementation of traits is a bit of a bastardization of the strict types/traits found in 
languages like Rust, but with some discipline I found that it provides
some of the same benefits.

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
