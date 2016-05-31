---
layout: post
title: "asynchronous recursive pattern in node.js"
image: '/assets/img/'
description:
main-class: js
tags:
- "node.js"
- "javascript"
- "promise"
categories: js
twitter_text: "Asynchronous recursive pattern in Node.JS"
introduction: "Use Promise to invoke asynchronous function in recursive way"
---

# Challenge

Node.JS, its non-blocking asynchronous I/O calls made itself high performance. You shall call an I/O routine, and give it a function to call back when the operation is done. In the meanwhile you can go on continuing whatever business you still have without waiting for that operation to finish.

Well, it's elegent however it might be paintful when iterations is needed. We may find ourselves in need to iterate over many I/O calls before we can do some specific task.

In this article I will demostrate the challenge and solution pattern with a simple sample. The sample is going to generate a fibonacci sequence recursively until hitting the ```ceiling``` value and undoubtedly this works as expected.

```javascript
function genFibonacciSync(list, ceiling) {
  // sync recursive
  next = list[list.length - 1] + list[list.length - 2];
  if (next > ceiling)
    return;
  else {
    list.push(next);
    genFibonacciSync(list, ceiling);
  }
}

console.log('start sync...');
var list = [];
list.push(1);
list.push(1);
genFibonacciSync(list, 1000);
list.forEach(function(val) {
    console.log(val + ',');
});
console.log('end...');

```

To make thing complicated, we altered the sample with ```setTimeout()``` inside the function and make it calculate ```next``` value after 1 millisecond.

```javascript
function genFibonacciAsync(list, ceiling) {
  // async recursive
  setTimeout(function() {
    next = list[list.length - 1] + list[list.length - 2];
    if (next > ceiling)
      return;
    else {
      list.push(next);
      genFibonacciAsync(list, ceiling);
    }
  }, 1);
}

console.log('start async...');
var list = [];
list.push(1);
list.push(1);
genFibonacciAsync(list, 1000);
list.forEach(function(val) {
  console.log(val + ',');
});
console.log('end...');

```

Whenever you run above script you're going to get an [1,1] list instead.

```
start async...
1,
1,
end...
```

The problem is that the usual idea we have about recursion doesn’t work with asynchronous calls. The usual idea we have about recursion is that execution will not go past the call ```genFibonacciAsync(list, ceiling)``` untill all the recursive calls within are unwound and returned, which means that all operations on list are done and it’s safe to use its value when the execution goes past, but this does not  happen when the function involves an asynchronous call.

What happens here is that ```genFibonacciAsync``` doesn’t do any of the work itself, it just initiates an asynchronous ```setTimeout``` to calculate fibonacci sequence and then returns immediately. The actual work will be started by the callback of the ```setTimeout``` after 1 millisecond.

By the time all this waiting to happen, the execution would have already gone past the ```genFibonacciAsync(list, ceiling)``` line and printed the list before any work could be done on it.

# Solution Pattern

```Promise``` is a new feature introduced to javascript in ES6, they represent a way to write asynchronous code in a synchronous way and it’s available in Node.js since v4.0.

A Promise is essentially a representation of a value that is not available yet but will be resolved in the future. A promise is created using a function that takes two arguments: a resolve handler, and a reject handler. In this function you write your asynchronous call, and when the value you get from that call is available, you resolve the promise with that value using the resolve handler. And if any error occurs in the process, you reject the promise with that error using the reject handler.

```javascript
var promise = new Promise(function(resolve, reject) {
  try {
    asyncCall(function(err, data) {
      if (err)  // an error occur
        reject(new Error(err));
      else      // the result in now available in the callback
        resolve(data);
    });
  }
  catch(e) {
    reject(e);
  }
});
```

Now you have a promise of a value, with that you have the ability to wait for that value to be resolved and then do some processing on it. This behaviour (called the thenable behaviour) is accomplished using the Promise.then method that takes two functions: the first which is called if the promise is resolved and it gets passed the resolved value, the other gets called if the promise is rejected and gets passed the rejected value (which is usually an error).

```javascript
promise
.then(function(value) {
  console.log("This is the result of asyncCall: ", value);
}, function(error) {
  throw error;  // rethrow the error
});
```

We see that the promise allowed us to simplify our asynchronous code by limiting the callback to only the report of the value, and all the processing logic of that value is separated to another block (the then block) outside the callback in a way that appears to be synchronous. Going back to our initial challenge, our solution would be like code below.

```javascript
function genFibonacciAsync(list, ceiling) {
  return new Promise(function(resolve, reject) {
    // async recursive
    setTimeout(function() {
      next = list[list.length - 1] + list[list.length - 2];
      if (next > ceiling)
        resolve();
      else {
        list.push(next);
        genFibonacciAsync(list, ceiling)
          .then(function() {
            resolve();
          });
      }
    }, 1);
  });
}

console.log('start async...');
var list = [];
list.push(1);
list.push(1);
genFibonacciAsync(list, 1000)
  .then(function() {
    list.forEach(function(val) {
      console.log(val + ',');
    })
  });
console.log('end...');
```

Now we can be sure that the logs will not be printed until that promise is fulfilled. 
