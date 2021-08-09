# Enumeration vs Iteration & Property Definitions in JavaScript
### `Created By: David Couch 8/7/2021`

## `About`
This file was created to explore the concept behind how object enumeration and iteration is handled. For the love of JavaScript!

### `About Property Definition`
  - It is important to understand what definitions are to better understand the enumerator.
  - An Object in JavaScript has two types of property descriptors - DATA or ACCESSOR.
```
  DATA
    -- value = the value of the object.
    -- writable = are you allowed to change the value.
    -- configurable = are you aloud to change the property descriptors that where set when the object was created.
    -- enumerable = are you allowed to see the value through enumeration.
```
```js
  const obj = {}
  Object.defineProperty(obj, 'propertyName', {
    // the initial value you will retrieve when calling on the property for its value.
    value: 'initial Value',
    // is the value aloud to be changed?
    writable: true,
    // can you change these initially set configuration?
    configurable: true,
    // should this be hidden from enumeration?
    enumerable: true
  })
```
```
  ACCESSOR
    -- get = retrieve the data.
    -- set = you can change the value of the property.
    -- configurable = are you aloud to change the property descriptors that where set when the object was created.
    -- enumerable = are you aloud to see the values through a for loop.
```
```js
  let obj = {}
  Object.defineProperty(obj, 'propertyName', {
    // can you change these initially set configurations?
    configurable: true,
    // should this be hidden from enumeration?
    enumerable: true,
    // what is the process of saving the data during value reassignment?
    // here because obj is the context we can use the this keyword to access itself.
    // when an object is constructed its value results to undefined. we can use the || operator to set a default value here.
    get: () => this.value || 'initial value',
    // here we are using set to reassign that value when making a statement like obj.propertyName = 'dogs'.
    set: (_val) => {
      this.value = _val
    }
  })
```
#### `for...in loops and the prototype`
- Property definitions are the reason why JavaScripts prototype methods do not show up.
- If someone decided to put something in the objects prototype with an enumeration set to true this could happen.

```js
const obj = {}
const prototypeFunction = {}

Object.defineProperty(
	prototypeFunction, 'helloWorld', {
  	configurable: false,
    enumerable: true,
    value: () => {
    	console.log('hello world')
    },
    writable: false
  })


Object.setPrototypeOf(obj, prototypeFunction)

// the prototype now contains the method hello world.

for (let i in obj) {
  // returns helloworld which is the key of our hello world function.
  // we can invoke it like this.
  obj[i]() // log > hello world
}

// Note: .hasOwnProperty() is also an extra caution that is commonly taken.
for (let i in obj) {
  if (Object.hasOwnProperty(i)/* returns true if the property is in the object */) {
    // execute code
  }
}

```
- To prevent this use
```js
const obj = {}

const prototypeFunction = {}

Object.defineProperty(
	prototypeFunction, 'helloWorld', {
  	configurable: false,
    enumerable: false,
    value: () => {
    	console.log('hello world')
    },
    writable: false
  })


Object.setPrototypeOf(obj, prototypeFunction)

// the prototype now contains the method hello world.

for (let i in obj) {
  // the for in function does not end up using i because the object appears to be empty.
}

```


### `What Is Enumeration All About?`
  - The Object data type in JavaScript do not use the iterator. This means for...in loops do not work on Arrays because they use the Symbol.iterator generator.
  - Javascript does have built in values to help with this and can be used as such
  ```js

    // say you have an object.
    const obj = {
      key1: 'value1',
      key2: 'value2'
    }

    // from here you have a few options.

    // returns an array of enumerable keys.
    Object.keys(obj).forEach(key => {
      // here the object key is given object and can be used to extract the values.
      obj[key]
    })

    // returns the object values that are and are not enumerable.
    Object.getOwnPropertyNames(obj).forEach(key => {
      // same results as before except now values are revealed
      // object keys that are symbols will not be shown
      obj[key]

    })

    for (let key in obj) {
      // this uses the same format as before except enumerable properties that are set to false will not be shown
      obj[key]
    }

    Object.entries(([key, value]) => {
      // this returns both the key and the value
      // this fallows the same enumeration rules as a for in loop
      // non enumerable properties will now show
      {[key]:value}
    })
  ```
  - An objects enumerable values are the properties of the object (keys)
  - An object has the option of configuring its enumeration capability to true or false.
  - An object where its enumerable property is set to false will result in the property being hidden from enumeration.
  - An object where its enumerable property is set to true will show on enumeration.

## How About Iteration?
  - In JavaScript the Array and string object have a built in method called the iterator.
  - From developer.mozilla: "Whenever an object needs to be iterated (such as at the beginning of a for...of loop), its @@iterator method is called with no arguments, and the returned iterator is used to obtain the values to be iterated."

- #### `Have You Heard of function*?`
    - Array and String have a method in them that return a type of generator function
    - calling the array as such.
      ```js
        const iterationGenerator = [1,1,1][Symbol.iterator]()
          // Array Iterator {}
          //   [[Prototype]]: Array Iterator
          //     next: ƒ next()
          //       Symbol(Symbol.toStringTag): "Array Iterator"
          //       [[Prototype]]: Object
      ```
    - This returns the iterator function object that can then be called like this.
      ```js
        iterationGenerator.next()
        // which returns
        {
          value: 1,
          done: false
        }
      ```
    - This means that you can even work this into an object if you wanted.
      ```js
      const symbolIterator = {
        [Symbol.iterator]: function() {
          let currentIndex = -1;
          let keys = Object.keys(this)
          return {
            next: () => {
              return {
                value: this[keys[++currentIndex]],
                done: currentIndex >= keys.length
              }
            }
          }
        }
      }

      const c =  {
        key1: 'Value 1',
        key2: 'Value 2',
        key3: 'value 3',
        ...symbolIterator
      }

      // destructure like its an array!
      const [a,b,x,v] = c

      console.log(a) // returns 'value 1'
      console.log(b) // returns 'value 2'
      console.log(x) // returns 'value 3'
      console.log(v) // returns undefined

      for (let u of c) {
        console.log(u)
        // loops through the object using for of
      }

      // loop returns
      // "Value 1"
      // "Value 2"
      // "value 3"

      ```
