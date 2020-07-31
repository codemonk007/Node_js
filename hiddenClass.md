In the last part of this series we discussed a bit about abstract syntax trees and how V8 compiles our code. Another cool thing V8 does when it's dealing with JavaScript is that it makes possible for a statically typed language, such as C++, to run dynamically typed code, like JS. One of the simplest examples we have of Dynamic Typing is an object declaration:
const myObj = {}
console.log(myObj) // {}

myObj.x = 1
console.log(myObj) // { x: 1 }

myObj.y = 2 // Dynamically changing the type
console.log(myObj) // { x: 1, y: 2 }
Since JavaScript is a dynamic language, properties from our objects can be added and removed on the fly - like we did. These operations require a dynamic lookup to resolve where this property's location is in memory so it can get back the value for you. Dynamic lookups are a high-cost operation for processors. So how does V8 handles this to make JS so fast? The answer is hidden classes. And it's one of the optimisation tricks V8 is so famous about.

We'll talk about other compiler optimisation techniques later

Generally when we have statically-typed languages, we can easily determine where a property is in memory, since all objects and variables are determined by a fixed object layout you'll define as its type, and new properties cannot be added during runtime, which makes pretty easy for the compiler to find this properties' values (or pointers) in memory since they can be stored as a continuous buffer with a fixed offset between each object. And this offset can be easily determined by the object type, since all types have a fixed memory value. V8 takes advantage of these fixed layout object concept to use the approach of a hidden class. Let's see how it works:

For each object type, V8 creates a hidden class, so our first declaration of const myObj = {} would create a class like this:



Now, as we add a new key to myObj, V8 creates a new hidden class based on C0 (copying it) called C1, and will update C0 to add a transition to C1:



Now as the last statement we add y, this does the exact same steps as before: creates a new class C2 based on C1, add a new transition to C1 pointing to C2:



This little trick makes possible for V8 to reuse hidden classes for new object. If we create a new object like {}, no new classes will be created, instead V8 will point the new object to C0. As we add the new properties x and y, the new object will point to the classes C1 and C2 writing the values on the offsets those classes specify. This concept makes possible for a compiler to bypass a dictionary lookup for when a proprety is accessed. Since it already knows to what class the object points to and where is the offset to that property, it can simply go straight there. This also makes V8 able to use class-based optimisations and inline caching - which we'll see later.

However, hidden classes are extremely volatile, they are one and only to that specific type of object. So, if we swap the order of our properties to be y and x instead of the opposite, V8 would have to create new hidden classes since C1 only has offsets for x in the position 0 and C2 only has offsets for y in the first position.

But keep in mind this is done in the C++ because JavaScript is a prototype-based language, therefore, it has no classes.