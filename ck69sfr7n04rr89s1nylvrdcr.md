## TDD made simple with Mocha and Chai

## Intro

From the dark ol’ days of writing an entire application and only then starting to test it (often, manually) till nowadays, I have scoured a painful path of unending bug-fixing in production through the nights, many times not even knowing what was causing those bugs.

![disappearing-bugs.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1580924443476/utCTzwpjk.jpeg)
> Yeah, something crazy is going on!

Since I first heard of Test Driven Development, it changed the way I think about software development.

I will not be digressing about TDD philosophy and its implications here, because  [a lot of](http://www.agiledata.org/essays/tdd.html)  [more qualified people](https://www.martinfowler.com/bliki/TestDrivenDevelopment.html)  [have done it before me](https://medium.com/javascript-scene/tdd-changed-my-life-5af0ce099f80). So let’s get to the code!

***

## First, the problem and its solution

A long time ago in a galaxy far far away, I ended up in a problem: I had to monitor a “stream” (more like a polling) of events that were being created at a certain application in my Node.JS backend. This “stream” was not uniform and, most of the time, no event occurred.

I could not use websockets, so I would have to buffer these events in my backend. I thought using a database (even an in-memory one like Redis) just for that was too much. Then I decided that I would keep the events in memory and as my application did not care for all events that ever happened, I would keep only the last N of them.

Since Node.JS arrays are dynamic, they did not fit my needs. I did not want a fixed-size array implementation, what I needed was a fixed-sized first-in/first-out (FIFO) data structure, AKA a **queue**, which instead of overflowing when full, should pop its first element and then add the new one at the end.

## Expected behaviour

![keep-calm-behave-yourself.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580925435400/pKSQdIeel.png)
> You better do that!

The data structure described above is rather simple. Its expected behaviour could be summarized as follows:

Adding elements:

- When it is not full, it should add the new element to the end; its size should be increased by 1.
- When it is full, it should remove the first element and then add the new element to the end; its size must not change.
    - The removed element should be returned.

Removing elements:

- When it is not empty, it should remove the first element and return it; its size should be decreased by 1.
- When it is empty, it should throw an error.

## A Mocha to go, please!

![mocha-coffee.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1580925569725/ChNWZ_mGT.jpeg)
> Looks delicious! ;9

From the docs:

>  [Mocha](https://mochajs.org/)  is a feature-rich JavaScript test framework running on  [Node.js](https://nodejs.org/)  and in the browser, making asynchronous testing simple and fun. Mocha tests run serially, allowing for flexible and accurate reporting, while mapping uncaught exceptions to the correct test cases. Hosted on  [GitHub](https://github.com/mochajs/mocha).

### Installation

```bash
yarn add --dev mocha
# or with NPM:
# npm install --save-dev mocha
```

### Writing tests

To create a test suite, you use a globally defined function called `describe`. To add test cases to a suite, you should use another global function `it`:


%[https://gist.github.com/hbarcelos/d87cd45d24e3a0986ca7f0eb3221879c#file-example-test-suite-js]

Suites can be nested indefinitely when you want to group your test cases. Mocha will collect all your suites recursively and execute all test cases it find within them in the order they are declared.

And that’s probably about all you need to tedknow about Mocha to get star (at least for basic usage). It excels so much for simplicity and extensibility, that it allows you to use whatever assertion library and other plugins you want.

### Running tests

```bash
yarn mocha '<path-to-test-file>'
# or with NPM's npx:
# npx mocha '<path-to-test-file>'
```

## Enter Chai

![chai.jpeg](https://cdn.hashnode.com/res/hashnode/image/upload/v1580926329312/6Ji9lcmw2.jpeg)
> I used to think that Node.JS developers only like coffee… Guess I was not totally right (:

By default, Mocha can be used along with Node.js native [`assert`](https://nodejs.org/dist/latest-v12.x/docs/api/assert.html) module. It works just fine, however I don't find its developer experience to be exactly great. For that reason, we will use a 3rd-party assertion library called Chai.

From the docs:

>  [Chai](https://www.chaijs.com/)  is a BDD / TDD assertion library for node and the browser that can be delightfully paired with any JavaScript testing framework.

### Installation

```bash
yarn add --dev chai
# or with NPM:
# npm install --save-dev chai
```

### Usage

Chai offers 3 different styles for writing assertions:

![chai-assertion-styles.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580926746587/16qkOEPbr.png)
> Chai allows you to pick your own poison.

All of them have the same capabilities, so choosing one or another is more a matter of preference than of objective facts. I like to use the `expect` interface.

## Oh, tests! Oh, dreaded tests!

![bug.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580926931683/xm8mX-yOR.png)
> These bug’s bites hurt a lot; better kill’em before they kill you!

Going back to our original problem, let’s translate the expected behavior into mocha test suites. But first, let’s do some setup:

```js
const chai = require("chai");
const expect = chai.expect;

const RoundQueue = require("./round-linked-queue");

describe("Round-Queue", () => {
});
```
> [Source](https://github.com/hbarcelos/round-linked-queue/blob/initial/round-linked-queue.test.js)

### Testing queue creation

The main reason why we are creating this data structure is that it has to be a limited size, so let's make sure it has such property:

```js
const chai = require("chai");
const expect = chai.expect;

const RoundQueue = require("./round-linked-queue");

describe("Round-Queue", () => {
  describe("When creating an instance", () => {
    it("Should properly set the maxLength property", () => {
      const queueLength = 3;

      const queue = new RoundQueue(queueLength);

      expect(queue.maxLength).to.equal(queueLength);
    });
  });
});
```
> [Source](https://github.com/hbarcelos/round-linked-queue/blob/1st-test-case/round-linked-queue.test.js) and [diff](https://github.com/hbarcelos/round-linked-queue/commit/1st-test-case#diff-873978c023e14c5220c2c83000ee2c7b)

Next we implement just enough code to make the test above pass:

```js
class RoundLinkedQueue {
  constructor(maxLength) {
    this._maxLength = maxLength;
  }

  get maxLength() {
    return this._maxLength;
  }
}

module.exports = RoundLinkedQueue;
```
>  [Source](https://github.com/hbarcelos/round-linked-queue/blob/1st-test-case/round-linked-queue.js)

To run the suite, we do:

```bash
yarn mocha round-linked-queue.test.js
```

Keep moving and we must ensure that a queue is created empty:

```js
it("Should initially set the length to zero", () => {
  const queueLength = 3;

  const queue = new RoundQueue(queueLength);

  expect(queue.length).to.equal(0);
});
```
>  [Source](https://github.com/hbarcelos/round-linked-queue/blob/2nd-test-case/round-linked-queue.test.js)

In order to make the new test pass, we can do as follows:

```js
class RoundLinkedQueue {
  constructor(maxLength) {
    this._maxLength = maxLength;
    this._length = 0;
  }

  get maxLength() {
    return this._maxLength;
  }

  get length() {
    return this._length;
  }
}
```
> [Source](https://github.com/hbarcelos/round-linked-queue/blob/2nd-test-case/round-linked-queue.js) and  [diff](https://github.com/hbarcelos/round-linked-queue/commit/2nd-test-case#diff-68fb0aba2b1c0ad72bf0d44fa71fe5d1).

### Testing adding elements

Next we create another test suite inside the top-level suite to test the behavior of adding elements to a queue.

Our base use case happens when the queue is empty and we want to add an element to it:
 
```js
describe("When adding elements", () => {
  it("Should add an element to an empty queue", () => {
    const queue = new RoundQueue(3);
    const originalLength = queue.length;
    const elementToAdd = 1;

    queue.add(elementToAdd);

    // Element should've been added to the end of the queue
    expect(queue.last).to.equal(elementToAdd);
    // But since it is now the only element, it should also be the at beginning as well
    expect(queue.first).to.equal(elementToAdd);
    // Length should've been increased by 1
    expect(queue.length).to.equal(originalLength + 1);
  });
});
```
> [Source](https://github.com/hbarcelos/round-linked-queue/blob/3rd-test-case/round-linked-queue.test.js)

If you run the test suite right now, you will get the following error:

![01-failing-add.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580928879821/0hl-ZqR1j.png)
> D'oh!

The test failed because we didn't implement the `add` method yet. Now we add **just enough code to make this first test case pass**.

**Important:** the code bellow is not entirely correct, we will have to modify it further in order to make the `add` method work as expected. However, it does make our first test case "adding element to an empty queue" pass.

```js
class RoundLinkedQueue {
  // ...

  add(element) {
    this._root = element;
    this._first = element;
    this._last = element;

    this._length += 1;
  }
}
```
> [Source](https://github.com/hbarcelos/round-linked-queue/blob/3rd-test-case/round-linked-queue.js) and [diff](https://github.com/hbarcelos/round-linked-queue/commit/3rd-test-case#diff-68fb0aba2b1c0ad72bf0d44fa71fe5d1)

Now let's try adding a test for when the queue is not empty anymore and yet we still want to add an element to it:

```js
it("Should add an element to the end of a non-empty queue", () => {
  const queue = new RoundQueue(3);
  const previousElement = 1;
  const elementToAdd = 2;
  // Make the queue non-empty
  queue.add(previousElement);

  queue.add(elementToAdd);

  // Element should've been added to the end of the queue
  expect(queue.last).to.equal(elementToAdd, "last not properly set");
  // But the first pointer must remain the first element added
  expect(queue.first).to.equal(previousElement, "first not properly set");
  // Length should've been increased by 2
  expect(queue.length).to.equal(2, "length not properly set");
});
```

If we once again run the test suite without changing the implementation, we will get a failure:

![02-failing-add-multiple.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580929515125/tL0rKinye.png)

The more attentive readers should probably be expecting this error because the way we implemented the `add` method before would simply overwrite the elements in the queue. To fix this, we will need some more code:

```js
class RoundLinkedQueue {
  // ...

  add(element) {
    const node = {
      data: element,
      next: null,
    };

    if (!this._root) {
      this._root = node;
      this._first = node;
      this._last = node;
    } else {
      const previousLast = this._last;
      previousLast.next = node;

      this._last = node;
    }

    this._length += 1;
  }
}
```
> [Source](https://github.com/hbarcelos/round-linked-queue/blob/4th-test-case/round-linked-queue.js) and [diff](https://github.com/hbarcelos/round-linked-queue/commit/4th-test-case)

We had to convert our `_root`, `_first` and `_last` into a `node` object containing `data` &mdash; the actual value of the item &mdash; and `next` &mdash; a pointer to the next `node` in the linked list.

Moving on, now it's time to something a little bit more challenging. Whenever our queue is at capacity, adding a new element should should cause the removal of the element that was first added:

```js
it("Should remove the first element and add the new element to the end of a full queue", () => {
  const queue = new RoundQueue(3);
  queue.add(1);
  queue.add(2);
  queue.add(3);

  queue.add(4);

  // Element should've been added to the end of the queue
  expect(queue.last).to.equal(4, "last not properly set");
  // The second element should've been shifted to the first position
  expect(queue.first).to.equal(2, "first not properly set");
  // Length should still be the same
  expect(queue.length).to.equal(3, "length not properly set");
});
```
> [Source](https://github.com/hbarcelos/round-linked-queue/blob/5th-test-case/round-linked-queue.test.js)

Running tests once more we get:

![03-failing-add-full.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580933433526/6xu7iTV2A.png)

Looks like we will need some conditionals to make the new test case pass along with the previous ones:

```js
class RoundLinkedQueue {
  // ...

  add(element) {
    const node = {
      data: element,
      next: null,
    };

    if (this.length < this.maxLength) {
      if (!this._root) {
        this._root = node;
        this._first = node;
        this._last = node;
      } else {
        const previousLast = this._last;
        previousLast.next = node;

        this._last = node;
      }

      this._length += 1;
    } else {
      this._root = this._root.next;
      this._last.next = node;
      this._first = this._root;
      this._last = node;
    }
  }
}
```
> [Source](https://github.com/hbarcelos/round-linked-queue/blob/5th-test-case/round-linked-queue.js) and [diff](https://github.com/hbarcelos/round-linked-queue/commit/5th-test-case#diff-68fb0aba2b1c0ad72bf0d44fa71fe5d1)

#### Halt! Refactor time

![transformer.webp](https://cdn.hashnode.com/res/hashnode/image/upload/v1580930165372/nccviHTqB.webp)

So far we were writing code in a rather linear fashion: make a failing test, implement code to make it pass; make another failing test, write just enough code to make it pass, and so on.

In TDD jargon, creating a failing test is called the **red phase**, while implementing the code that will make it pass is the **green phase**.

In reality, things are not so pretty-neaty. You will not always get how to write the best code possible the first time. The truth is we've been cheating a little: we were skipping the **refactor** phase of the TDD cycle:

![gist-of-tdd.jpg](https://cdn.hashnode.com/res/hashnode/image/upload/v1580931502419/ACYP1JqUi.jpeg)
> The gist of TDD

Right now I see some possible improvements in our data structure:

1. Having both `_root` and `_first` properties seem redundant.
2. There is some duplication of code in the `add` method (remember [DRY](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)?)

Because we already know the expected behavior, which is coded in our test suite, we are comfortable to [refactor mercilessly](http://www.extremeprogramming.org/rules/refactor.html).

```js
class RoundLinkedQueue {
  // ...

  add(element) {
    const node = {
      data: element,
      next: null,
    };

    if (this.length < this.maxLength) {
      if (!this._first) {
        this._first = node;
        this._last = node;
      }

      this._length += 1;
    } else {
      this._first = this._first.next;
    }

    this._last.next = node;
    this._last = node;
  }
}
```
> [Source](https://github.com/hbarcelos/round-linked-queue/blob/5th-test-case-1st-refactor/round-linked-queue.js)  and [diff](https://github.com/hbarcelos/round-linked-queue/commit/5th-test-case-1st-refactor#diff-68fb0aba2b1c0ad72bf0d44fa71fe5d1)

Hopefully, our tests are still green:

![04-successfull-after-refactor.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580933540191/uLuSISdyy.png)

#### Taking some shortcuts

Now we are going to cheat a little bit. 

The last requirement is that the `add` method should return the removed element when the queue is full. What to return when the queue is not full is not in the specification though. In JavaScript, uninitialized values have a special value called `undefined`. It makes sense to return that when adding to the queue does not remove any element, so we can add the following two test cases.

![sheldon-triggered.webp](https://cdn.hashnode.com/res/hashnode/image/upload/v1580933298455/KGvC42Pai.webp)
> TDD purists gonna be trigged.

```js
it("Should return the removed element from a full queue", () => {
  const queue = new RoundQueue(3);
  queue.add(1);
  queue.add(2);
  queue.add(3);

  const result = queue.add(4);

  expect(result).to.equal(1, "removed wrong element");
});

it("Should return undefined when the queue is not full", () => {
  const queue = new RoundQueue(3);

  const result = queue.add(1);

  expect(result).to.equal(undefined, "should not return an element");
});
```
> [Source](https://github.com/hbarcelos/round-linked-queue/blob/6th-test-case/round-linked-queue.test.js)

Cool, so let's just return the element from the node we just removed:

```js
class RoundLinkedQueue {
  // ...

  add(element) {
    const node = {
      data: element,
      next: null,
    };

    let removedElement;

    if (this.length < this.maxLength) {
      if (!this._first) {
        this._first = node;
        this._last = node;
      }

      this._length += 1;
    } else {
      removedElement = this._first.data;
      this._first = this._first.next;
    }

    this._last.next = node;
    this._last = node;

    return removedElement;
  }
}
```
> [Source](https://github.com/hbarcelos/round-linked-queue/blob/6th-test-case/round-linked-queue.js) and [diff](https://github.com/hbarcelos/round-linked-queue/commit/6th-test-case#diff-68fb0aba2b1c0ad72bf0d44fa71fe5d1)

Look like we are are done with the `add method`!

![minions-yay.webp](https://cdn.hashnode.com/res/hashnode/image/upload/v1580932348080/Pd6-nnNDM.webp)

### Testing removing elements

Removing elements seems like a simpler operation. Our base use case is when the queue is not empty. We remove an element from it and decrease its length by one:

```js
describe("When removing elements", () => {
  it("Should remove the first element of a non-empty queue", () => {
    const queue = new RoundQueue(3);
    queue.add(1);
    queue.add(2);
    queue.add(3);
    const lengthBefore = queue.length;

    const result = queue.remove();

    const lengthAfter = queue.length;

    expect(lengthAfter).to.equal(lengthBefore - 1, "length should decrease by 1");
    expect(result).to.equal(1, "first element should the one being removed");
    expect(queue.first).to.equal(2, "should shift the second element to the head of the queue");
    expect(queue.last).to.equal(3, "should not change the last element");
  });
});
```
> [Source](https://github.com/hbarcelos/round-linked-queue/blob/7th-test-case/round-linked-queue.test.js) 

Running the tests will once again give us an error:

![05-failing-remove.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580934071054/5gRhh90ib.png)

Now we add some code just to make the test pass:

```js
class RoundLinkedQueue {
  // ...

  remove() {
    const removedElement = this.first;

    this._first = this._first.next;
    this._length -= 1;

    return removedElement;
  }
}
```
> [Source](https://github.com/hbarcelos/round-linked-queue/blob/7th-test-case/round-linked-queue.js) and [diff](https://github.com/hbarcelos/round-linked-queue/commit/7th-test-case#diff-68fb0aba2b1c0ad72bf0d44fa71fe5d1) 

The only other use case is when the queue is empty and we try to remove an element from it. When this happens, the queue should throw an exception:

```js
it("Should throw an error when the queue is empty", () => {
  const queue = new RoundQueue(3);

  expect(() => queue.remove()).to.throw("Cannot remove element from an empty queue");
});
```
> [Source](https://github.com/hbarcelos/round-linked-queue/blob/8th-test-case/round-linked-queue.test.js)

Running the test suite as is:

![06-failing-remove-throw.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580934743739/9f2zBy2v7.png)
> Ouch!

Adding some conditions to test for emptyness and throw the proper error:

```js
class RoundLinkedQueue {
  // ...

  remove() {
    const removedNode = this._first;
    if (!removedNode) {
      throw new Error("Cannot remove element from an empty queue");
    }

    this._first = this._first.next;
    this._length -= 1;

    return removedNode.data;
  }
}
```
> [Source](https://github.com/hbarcelos/round-linked-queue/blob/8th-test-case/round-linked-queue.js) and [diff](https://github.com/hbarcelos/round-linked-queue/commit/8th-test-case#diff-68fb0aba2b1c0ad72bf0d44fa71fe5d1)

And that's it!

### Testing edge cases

There are still some bugs in or code. When we wrote the `add` method, we included the `first` and `last` getters as well. But what happens if we try to access them when the queue is empty? Let's find out! `first` things first (ba dum tsss!):

```js
describe("When accessing elements", () => {
  it("Should throw a proper error when acessing the first element of an empty queue", () => {
    const queue = new RoundQueue(3);

    expect(() => queue.first).to.throw("Cannot access the first element of an empty queue");
  });
});
```
> [Source](https://github.com/hbarcelos/round-linked-queue/blob/9th-test-case/round-linked-queue.test.js)

Running the tests:

![07-failing-first-on-empty.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580935169896/NpERAVp-L.png)

Looks like the error message is not really helpful. In fact, it is a little too low level. Let's make it better:

```js
class RoundLinkedQueue {
  // ...

  get first() {
    if (!this._first) {
      throw new Error("Cannot access the first element of an empty queue");
    }

    return this._first.data;
  }

  // ...
}
```
> [Source](https://github.com/hbarcelos/round-linked-queue/blob/9th-test-case/round-linked-queue.js) and [diff](https://github.com/hbarcelos/round-linked-queue/commit/9th-test-case#diff-68fb0aba2b1c0ad72bf0d44fa71fe5d1)  

Lastly, for the `last` getter, we will do the same:

```js
it("Should throw a proper error when acessing the last element of an empty queue", () => {
  const queue = new RoundQueue(3);

  expect(() => queue.last).to.throw("Cannot access the last element of an empty queue");
});
```
> [Source](https://github.com/hbarcelos/round-linked-queue/blob/10th-test-case/round-linked-queue.test.js)

First the failing result:

![08-failing-last-on-empty.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1580935674581/s3ZGqWSY9.png)

Then fixing the code:

```js
class RoundLinkedQueue {
  // ...

  get last() {
    if (!this._last) {
      throw new Error("Cannot access the last element of an empty queue");
    }

    return this._last.data;
  }

  // ...
}
```
> [Source](https://github.com/hbarcelos/round-linked-queue/blob/10th-test-case/round-linked-queue.js) and [diff](https://github.com/hbarcelos/round-linked-queue/commit/10th-test-case#diff-68fb0aba2b1c0ad72bf0d44fa71fe5d1)

Aaaaaaand that's about it!

![were-are-done-here.webp](https://cdn.hashnode.com/res/hashnode/image/upload/v1580935929728/ebLYgaAu0.webp)

## Conclusion

I tried to make this a comprehensive introduction to TDD with the Node.js/JavaScript ecosystem. The data structure we had to implement here was intentionally simple so we could follow the methodology as much as possible.

When doing TDD in real world applications, things are usually not so linear. You will find yourself struggling from time to time with the design choices you make while writing your tests. It can be a little frustrating in the beginning, but once you get the gist of it, you will develop a "muscle memory" to avoid the most common pitfalls.

TDD is great, but as almost everything in life, it is not a silver bullet.

Be safe out there!

***

T-t-th-tha-that's i-is a-a-all f-f-fo-f-folks!


![thats-all-folks.webp](https://cdn.hashnode.com/res/hashnode/image/upload/v1580936166941/RIWu_DlPS.webp)

***

Did you like what you just read? Why don’t you buy me a beer (or a coffee if it is before 5pm 😅) with  [tippin.me](https://tippin.me/@hbarcelos909)?