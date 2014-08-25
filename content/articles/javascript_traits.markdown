---
title: A Simple Implementation of Trait Mixins for Javascript
kind: article
created_at: 2014-08-04
category: article
---



The method I use
in development is a bit different from what I wrote here for brevity's sake, 
since it allows for inheritance among traits,
and includes various checks to avoid clashes; but the concept is the same.

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

And we can write our first trait:

    function Moveable(_constructor, initialX, initialY) {
        _constructor.prototype._registerTrait("Moveable");
        
        _constructor.prototype._addValue("posX", initialX);
        _constructor.prototype._addValue("posY", initialY);
        
        _constructor.prototype.moveLeft = function () {
            this.posX -= 1;
        };
    }; 
{: class="language-js" }

That's all there is to it!

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

This lends itself well to manipulating application objects with functional logic:

    _.zipWith(move, moevables, directions);

    function move(objectId, direction) {
      World[objectId].move(direction);
    }
{: class="language-js" }

It's also less prone to unexpected behavior.
I can confidently register new objects, mix or remove traits, and not fear the
application will break.
